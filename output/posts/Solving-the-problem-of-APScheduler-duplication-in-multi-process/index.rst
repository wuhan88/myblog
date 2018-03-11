.. title: 解决django使用多进程部署时apscheduler重复运行的问题
.. slug: Solving-the-problem-of-APScheduler-duplication-in-multi-process
.. date: 2018-03-11 10:45:42 UTC+08:00
.. tags: django, apscheduler
.. category: 
.. link: 
.. description: 
.. type: text

问题
-------------------------------

在一个django应用中需要定时执行一些任务，所以用了APScheduler这个库。在开发中直接测试运行是没有问题的，但是用gunicorn部署以后发生了重复运行的问题：每个任务在时间到的时刻会同时执行好几遍。注意了一下重复的数量，恰恰是gunicorn里配置的worker进程数量，显然是每个worker进程都启动了一份scheduler造成。

解决
-------------------------------

可以想到的方案有几个：

  * 用--preload启动gunicorn，确保scheduler只在loader的时候创建一次
  * 另外创建一个单独的定时任务项目，单独以一个进程运行
  * 用全局锁确保scheduler只运行一次

经过实践，只有第三个方案比较好。

1. preload的问题：

  虽然这样可以使用scheduler创建代码只执行一次，但是问题也在于它只执行一次，重新部署以后如果用kill -HUP重启gunicorn，它并不会重启，甚至整个项目都不会更新。这是preload的副作用，除非重写部署脚本，完全重启应用。

2. 单独进程的问题：

  也是因为部署麻烦，需要多一套部署方案，虽然用Docker会比较方便，但仍然不喜欢，而且同时维护两个项目也多出很多不必要的事情。

3. 全局锁是一个较好的方案，但问题在于找一个合适的锁。

  python自带的多进程多线程锁方案都需要一个共享变量来维护，但是因为worker进程是被gunicorn的主进程启动的，并不方便自己维护，所以需要一个系统级的锁。在Stackoverflow上看到有人是用了一个socket端口来做锁实现这个方案，但是我也不喜欢这样浪费一个宝贵的端口资源。不过这倒给了我一个启发，可以用文件锁，于是有了这个解决方案：

.. code:: python

  import atexit
  import fcntl
  from apscheduler.scheduler import Scheduler

  f = open("scheduler.lock", "wb")
  try:
      fcntl.flock(f, fcntl.LOCK_EX | fcntl.LOCK_NB)
  except:
      pass
  else:
      sched = Scheduler()

      @sched.interval_schedule(seconds=60)
      def mytask():
      '''
      定义你的定时任务执行的内容
      '''
          pass

      sched.start()

  def unlock():
      fcntl.flock(f, fcntl.LOCK_UN)
      f.close()

  atexit.register(unlock)




原理
--------------------------------

 首先打开（或创建）一个scheduler.lock文件，并加上非阻塞互斥锁。成功后创建scheduler并启动。

 如果加文件锁失败，说明scheduler已经创建，就略过创建scheduler的部分。

 最后注册一个退出事件，如果这个django退出，则解锁并关闭scheduler.lock文件的锁。
