.. title: LVS网卡软中断配置
.. slug: adjust_nic_irq_for_lvs
.. date: 2012-09-08 20:17:59 UTC+08:00
.. tags: lvs,irq
.. category: network
.. link:
.. description:
.. type: text


这是之前做LVS的网卡软中断配置时整理的一个文档，网上的资料不是很全，将配置方法share给大家。

为什么要配置网卡软中断，主要是因为在网络非常 heavy 的情况下，对于文件服务器、高流量 Web 服务器这样的应用来说，把不同的网卡 IRQ 均衡绑定到不同的 CPU 上将会减轻某个 CPU 的负担，提高多个 CPU 整体处理中断的能力。合理的根据自己的生产环境和应用的特点来平衡 IRQ 中断有助于提高系统的整体吞吐能力和性能。
先看下未升级之前的效果：可以看到网卡软中断被分配到了两个指定的CPU核心上:

.. image:: https://farm8.staticflickr.com/7067/6928555291_87c0dfcc68_z.jpg

经过升级内核调整参数后的效果：

.. image:: https://farm8.staticflickr.com/7061/6928555437_450d1a548b_z.jpg
   :width: 600

软中断被均匀的分配到8个核心上，下面来说下具体过程

第一步
~~~~~~~~

将内核升级到2.6.32以上，升级过程略去

.. code-block:: bash

    为什么要将2.6.18内核升级到2.6.32？这个主要是因为2.6.18还不支持RPS这个特性那什么是rps呢？具体可以参看：
    http://lwn.net/Articles/328339/
    http://lwn.net/Articles/378617/
    为什么要将2.6.18内核升级到2.6.32？
    这个主要是因为2.6.18不支持RPS这个特性
    那什么是rps呢？具体可以参看：
    http://lwn.net/Articles/328339/
    http://lwn.net/Articles/378617/ 

第二步：
~~~~~~~~~~~

如果你的服务器网卡和我一样是Broadcom的，那么你就得做这一步，不是请跳到第三步

.. image:: https://farm8.staticflickr.com/7052/6928573033_5dd56c20dc_z.jpg
   :height: 44
   :width: 600

在/etc/modprobe.conf加上下面这行： options bnx2 disable_msi=1
改完这个重新加载下网卡模块modprobe -r bnx2;modprobe bnx2或者重新启动服务器。

.. code-block:: bash

    为什么要加这个？ 
    这个主要是因为broadcom网卡开启msi后，会造成后面的修改smp_affinity丌生效，intel的网卡没这个问题。 

    msi是什么？下面的链接有解析：
    http://lwn.net/Articles/44139/

第三步：
~~~~~~~~~~~~~~

停用irqbalance
/etc/init.d/irqbalance stop
这个是一个自动调整中断的工具，有兴趣的可以看下irqbalance的官方网站：
http://irqbalance.org/ 

第四步：
~~~~~~~~~~~

设置eth0、eth1对应中断号的 ``smp_affinity`` 为 “ff”
先看一下网卡的中断号： 

.. image:: https://farm8.staticflickr.com/7046/6928573141_959e7d24ff_z.jpg
   :width: 600

从图中可以看到网卡eth1的中断号为16,eth0的中断号为18
将/proc/irq/中断号/smp_affinity修改为ff，修改完成后就可以开启lvs了，现在中断应该均分到各个核心上了。

.. code-block:: bash

    smp_affinity这个参数是怎么得来的？ 可参考下面链接：
    http://www.cs.uwaterloo.ca/~brecht/servers/apic/SMP-affinity.txt
