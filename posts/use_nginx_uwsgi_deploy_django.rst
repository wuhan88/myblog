使用NGINX+UWSGI来部署Django
=============================
.. title: 使用NGINX+UWSGI来部署Django
.. slug: use_nginx_uwsgi_deploy_django
.. date: 2012-12-12 20:17:59 UTC+08:00
.. tags: nginx,uwsgi,django
.. category:
.. link:
.. description:
.. type: text


关于UWSGI的介绍不多说了，想了解的可以自己去搜，uwsgi性能还蛮不错，我们之前使用的fastcgi方式来跑django，前端用F5做负载均衡，用户数增加后服务器的load很高，但换uwsgi服务器的load下降不少，废话不多说，进入正题。
编译安装nginx，用的nginx-0.8.54，目前最新的stable版本

.. code-block:: bash

   wget http://nginx.org/download/nginx-0.8.54.tar.gz
   tar zxvf  nginx-0.8.54.tar.gz
   ./configure --user=nobody
     --group=nobody
     --prefix=/usr/local/nginx
     --with-http_ssl_module
     --http-log-path=/var/log/nginx/access.log
     --with-http_gzip_static_module
   make&&make install

下面来安装uwsgi

.. code-block:: bash

   wget http://projects.unbit.it/downloads/uwsgi-0.9.6.5.tar.gz
   tar zvxf uwsgi-0.9.6.5.tar.gz
   cd uwsgi-0.9.6.5
   make -f Makefile.Py26
   cp uwsgi /usr/sbin/uwsgi    #将uwsgi放到PATH下

下面来说说怎么配置：
新建一个uwsgi.xml的文件放到django的目录下

.. code-block:: xml

  #cat uwsgi.xml
  <uwsgi>

  <socket>0.0.0.0:8000</socket>

  <listen>204800</listen>

  <!-- 开启32个线程 -->
  <processes>32</processes> 

  <max-requests>2048000</max-requests>

  <buffer-size>8192</buffer-size>

  <!-- 你的配置文件 -->
  <module>django_wsgi</module> 

  <profiler>true</profiler>

  <enable-threads>true</enable-threads>
   
  <!-- 限制内存空间256M -->
  <limit-as>256</limit-as> 
 
  <!-- 使用async模式来运行，这里要注意一下，如果你的app的是no-async-friendly 那就不要用这个模式 -->
  <async>10</async>    

  <disable-logging/>

  <daemonize>/home/app01/uwsgi.log</daemonize>

  </uwsgi>

.. code-block:: python

  #cat django_wsgi.py
  import os
  import django.core.handlers.wsgi
  os.environ['DJANGO_SETTINGS_MODULE'] = 'your settings'
  application = django.core.handlers.wsgi.WSGIHandler()

下面是nginx的配置：

.. code-block:: bash

  server {
          listen   80;
          server_name jasonwu.me;
          access_log /var/log/jasownu.me/access_log;
          location / {
              root   /home/app01/;
              uwsgi_pass 127.0.0.1:8000;
              include        uwsgi_params;
        }
    }

启动服务：

.. code-block:: bash

   /usr/bin/uwsgi -x /home/app01/uwsgi.xml
   /usr/local/nginx/sbin/nginx

这样部署完成了

下面来说说遇到的一个问题，不知道大家有没有遇到，
在我们启动uwsgi后在uwsgi的日志中会出现如下的信息::

    – unavailable modifier requested: 1 –
    – unavailable modifier requested: 1 –

表现的现象就是启动一段时间没法访问app，在查看uwsgi的源代码中我们找到打印这部份日志的段落，正常情况下应该返回的-1，目前还在查找这个出现这个错误的原因。

**参考文档:**

* uwsgi的 `官方wiki`_ 

* `Programming in Python with Medusa and the Async Sockets Library`_

.. _`官方wiki`:  http://projects.unbit.it/uwsgi/wiki/TitleIndex  
.. _`Programming in Python with Medusa and the Async Sockets Library`: http://www.      nightmare.com/medusa/programming.html
