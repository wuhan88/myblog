.. title: LVS的三种模式区别详解
.. slug: detailed_lvs_difference_between_the_three_models
.. date: 2012-09-29 20:17:59 UTC+08:00
.. tags: lvs
.. category:
.. link:
.. description:
.. type: text

.. contents:: 内容索引 Table of Contents

DR模式：(Direct Routing)直接路由模式
-----------------------------------------

DR模式的网络拓扑：
~~~~~~~~~~~~~~~~~~~~

.. image:: http://farm7.static.flickr.com/6236/6322179853_5899c2dfb6_z.jpg
   :width: 600

DR模式的工作过程： 
~~~~~~~~~~~~~~~~~~~~

当一个client发送一个WEB请求到VIP，LVS服务器根据VIP选择对应的real-server的Pool，根据算法，在Pool中选择一台Real-server，LVS在hash表中记录该次连接，然后将client的请求包发给选择的Real-server，最后选择的Real-server把应答包直接传给client；当client继续发包过来时，LVS根据更才记录的hash表的信息，将属于此次连接的请求直接发到刚才选择的Real-server上；当连接中止或者超时，hash表中的记录将被删除。


DR模式的几个细节：
~~~~~~~~~~~~~~~~~~~~~~

LVS和Real-server必须在相同的网段
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

DR模式在转发client的包时，只修改了包目的MAC地址为选定的Real-server的mac地址，所以如果LVS和Real-server在不通的广播域内，那么Real-server就没办法接收到转发的包。下面是mac地址的修改过程：

.. image:: http://farm7.static.flickr.com/6100/6322179927_eb8e928a02_z.jpg
   :width: 600

LVS不需要开启路由转发：
^^^^^^^^^^^^^^^^^^^^^^^^^

LVS的DR模式不需要开启路由转发功能，就可以正常的工作，出于安全考虑，如果不需要转发功能，最好关闭。

ARP问题：
^^^^^^^^^^^^^^^

通常，DR模式需要在Real-server上配置VIP，配置的方式为：

.. code-block:: bash

  /sbin/ifconfig lo:0 inet VIP netmask 255.255.255.255

原因在于，当LVS把client的包转发给Real-server时，因为包的目的IP地址是VIP，那么如果Real-server收到这个包后，发现包的目的IP不是自己的系统IP，那么就会认为这个包不是发给自己的，就会丢弃这个包，所以需要将这个IP地址绑到网卡上；当发送应答包给client时，Real-server就会把包的源和目的地址调换，直接回复给client。

**关于ARP广播：**

* 上面绑定VIP的掩码是”255.255.255.255″，说明广播地址是其本身，那么他就不会将ARP发送到实际的自己该属于的广播域了，这样防止与LVS上VIP冲突，而导致IP冲突。
* 另外在Linux的Real-server上，需要设置ARP的sysctl选项:（下面是举例说明设置项的）

假设服务器上ip地址如下所示::

  System Interface MAC Address IP Address
  HN eth0 00:0c:29:b3:a2:54 192.168.18.10
  HN eth3 00:0c:29:b3:a2:68 192.168.18.11
  HN eth4 00:0c:29:b3:a2:5e 192.168.18.12
  client eth0 00:0c:29:d2:c7:aa 192.168.18.129

当我从192.168.18.129 ping 192.168.18.10时，tcpdump抓包发现::

  00:0c:29:d2:c7:aa > ff:ff:ff:ff:ff:ff, ARP, length 60: arp who-has 192.168.18.10 tell 192.168.18.129
  00:0c:29:b3:a2:5e > 00:0c:29:d2:c7:aa, ARP, length 60: arp reply 192.168.18.10 is-at 00:0c:29:b3:a2:5e
  00:0c:29:b3:a2:54 > 00:0c:29:d2:c7:aa, ARP, length 60: arp reply 192.168.18.10 is-at 00:0c:29:b3:a2:54
  00:0c:29:b3:a2:68 > 00:0c:29:d2:c7:aa, ARP, length 60: arp reply 192.168.18.10 is-at 00:0c:29:b3:a2:68
  00:0c:29:d2:c7:aa > 00:0c:29:b3:a2:5e, IPv4, length 98: 192.168.18.129 > 192.168.18.10: ICMP echo request, id 32313, seq 1, length 64
  00:0c:29:b3:a2:54 > 00:0c:29:d2:c7:aa, IPv4, length 98: 192.168.18.10 > 192.168.18.129: ICMP echo reply, id 32313, seq 1, length 64
  00:0c:29:d2:c7:aa > 00:0c:29:b3:a2:5e, IPv4, length 98: 192.168.18.129 > 192.168.18.10: ICMP echo request, id 32313, seq 2, length 64
  00:0c:29:b3:a2:54 > 00:0c:29:d2:c7:aa, IPv4, length 98: 192.168.18.10 > 192.168.18.129: ICMP echo reply, id 32313, seq 2, length 64
  00:0c:29:b3:a2:54 > 00:0c:29:d2:c7:aa, ARP, length 60: arp who-has 192.168.18.129 tell 192.168.18.10
  00:0c:29:d2:c7:aa > 00:0c:29:b3:a2:54, ARP, length 60: arp reply 192.168.18.129 is-at 00:0c:29:d2:c7:aa

三个端口都发送了arp的reply包，但是192.168.18.129使用的第一个回应的eth4的mac地址作为ping请求的端口，由于192.168.18.10是icmp包中的目的地址，那么ping的应答包，会从eth0端口发出。

如果Real-server有个多个网卡，每个网卡在不同的网段，那么可以过滤掉非本网卡ARP请求的回应；但是如果多个网卡的ip在一个网段，那么就不行了。

.. code-block:: bash

  sysctl -w net.ipv4.conf.all.arp_filter=1

对于多个接口在相同网段可以设置下面的来防止：

.. code-block:: bash

  sysctl -w net.ipv4.conf.all.arp_ignore=1
  sysctl -w net.ipv4.conf.all.arp_announce=2

还是从192.168.18.129 ping 192.168.18.10时，tcpdump抓包发现::

  00:0c:29:d2:c7:aa > ff:ff:ff:ff:ff:ff, ARP, length 60: arp who-has 192.168.18.10 tell 192.168.18.129
  00:0c:29:b3:a2:54 > 00:0c:29:d2:c7:aa, ARP, length 60: arp reply 192.168.18.10 is-at 00:0c:29:b3:a2:54
  00:0c:29:d2:c7:aa > 00:0c:29:b3:a2:54, IPv4, length 98: 192.168.18.129 > 192.168.18.10: ICMP echo request, id 32066, seq 1, length 64
  00:0c:29:b3:a2:54 > 00:0c:29:d2:c7:aa, IPv4, length 98: 192.168.18.10 > 192.168.18.129: ICMP echo reply, id 32066, seq 1, length 64
  00:0c:29:d2:c7:aa > 00:0c:29:b3:a2:54, IPv4, length 98: 192.168.18.129 > 192.168.18.10: ICMP echo request, id 32066, seq 2, length 64
  00:0c:29:b3:a2:54 > 00:0c:29:d2:c7:aa, IPv4, length 98: 192.168.18.10 > 192.168.18.129: ICMP echo reply, id 32066, seq 2, length 64
  00:0c:29:b3:a2:54 > 00:0c:29:d2:c7:aa, ARP, length 60: arp who-has 192.168.18.129 tell 192.168.18.10
  00:0c:29:d2:c7:aa > 00:0c:29:b3:a2:54, ARP, length 60: arp reply 192.168.18.129 is-at 00:0c:29:d2:c7:aa

看到了么，现在只有eth0会回应arp请求了。

**arp报文格式：**

.. image:: http://farm7.static.flickr.com/6219/6324726842_6f0aea5dab_z.jpg

请求报文：MAC地址字段是空的。
应答报文：所有字段都又内容。::

  The arp_announce/arp_ignore reference：
  
  arp_announce – INTEGER
  Define different restriction levels for announcing the local
  source IP address from IP packets in ARP requests sent on
  interface:
  0 – (default) Use any local address, configured on any interface
  1 – Try to avoid local addresses that are not in the target’s
  subnet for this interface. This mode is useful when target
  hosts reachable via this interface require the source IP
  address in ARP requests to be part of their logical network
  configured on the receiving interface. When we generate the
  request we will check all our subnets that include the
  target IP and will preserve the source address if it is from
  such subnet. If there is no such subnet we select source
  address according to the rules for level 2.
  2 – Always use the best local address for this target.
  In this mode we ignore the source address in the IP packet
  and try to select local address that we prefer for talks with
  the target host. Such local address is selected by looking
  for primary IP addresses on all our subnets on the outgoing
  interface that include the target IP address. If no suitable
  local address is found we select the first local address
  we have on the outgoing interface or on all other interfaces,
  with the hope we will receive reply for our request and
  even sometimes no matter the source IP address we announce.
  
  The max value from conf/{all,interface}/arp_announce is used.
  
  Increasing the restriction level gives more chance for
  receiving answer from the resolved target while decreasing
  the level announces more valid sender’s information.

``arp_announce`` 用来限制，是否使用发送的端口的ip地址来设置ARP的源地址：

* “0″代表是用ip包的源地址来设置ARP请求的源地址。
*     “1″代表不使用ip包的源地址来设置ARP请求的源地址，如果ip包的源地址是和该端口的IP地址相同的子网，那么用ip包的源地址，来设置ARP请求的源地址，否则使用”2″的设置。
* “2″代表不使用ip包的源地址来设置ARP请求的源地址，而由系统来选择最好的接口来发送。

当内网的机器要发送一个到外部的ip包，那么它就会请求路由器的Mac地址，发送一个arp请求，这个arp请求里面包括了自己的ip地址和Mac地址，而linux默认是使用ip的源ip地址作为arp里面的源ip地址，而不是使用发送设备上面的 ，这样在lvs这样的架构下，所有发送包都是同一个VIP地址，那么arp请求就会包括VIP地址和设备 Mac，而路由器收到这个arp请求就会更新自己的arp缓存，这样就会造成ip欺骗了，VIP被抢夺，所以就会有问题。

现在假设一个场景来解释 ``arp_announce`` ：

.. code-block:: rst
   
   Real-server的ip地址：202.106.1.100(public local address)，
   172.16.1.100(private local address)，
   202.106.1.254(VIP)

如果发送到client的ip包产生的arp请求的源地址是202.106.1.254(VIP),那么LVS上的VIP就会被冲掉，因为交换机上现在的arp对应关系是Real-server上的VIP对应自己的一个MAC，那么LVS上的VIP就失效了。::

  arp_ignore – INTEGER
  Define different modes for sending replies in response to
  received ARP requests that resolve local target IP addresses:
  0 – (default): reply for any local target IP address, configured
  on any interface
  1 – reply only if the target IP address is local address
  configured on the incoming interface
  2 – reply only if the target IP address is local address
  configured on the incoming interface and both with the
  sender’s IP address are part from same subnet on this interface
  3 – do not reply for local addresses configured with scope host,
  only resolutions for global and link addresses are replied
  4-7 – reserved
  8 – do not reply for all local addresses
  
  The max value from conf/{all,interface}/arp_ignore is used
  when ARP request is received on the {interface}

“0″,代表对于arp请求，任何配置在本地的目的ip地址都会回应，不管该arp请求的目的地址是不是接口的ip；如果有多个网卡，并且网卡的ip都是一个子网，那么从一个端口进来的arp请求，别的端口也会发送回应。
“1″,代表如果arp请求的目的地址，不是该arp请求包进入的接口的ip地址，那么不回应。
“2″,要求的更苛刻，除了”1″的条件外，还必须要求arp发送者的ip地址和arp请求进入的接口的ip地址是一个网段的。
(后面略)

IP Tunneling模式：
------------------


IP Tunneling的拓扑图
~~~~~~~~~~~~~~~~~~~~~~~~

.. image:: http://farm7.static.flickr.com/6094/6323980713_082c909dd5_b.jpg
   :width: 600

IP Tunneling的工作过程
~~~~~~~~~~~~~~~~~~~~~~~~~~

1> client 发送request包到LVS服务器的VIP上。

2> VIP按照算法选择后端的一个Real-server，并将记录一条消息到hash表中，然后将client的request包封装到一个新的IP包里，新IP包的目的IP是Real-server的IP，然后转发给Real-server。

3> Real-server收到包后，解封装，取出client的request包，发现他的目的地址是VIP，而Real-server发现在自己的lo:0口上有这个IP地址，于是处理client的请求，然后将relpy这个request包直接发给client。

4> 该client的后面的request包，LVS直接按照hash表中的记录直接转发给Real-server，当传输完毕或者连接超时，那么将删除hash表中的记录。

IP Tunneling的几个细节问题
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

IP封包的过程：(如图)
^^^^^^^^^^^^^^^^^^^^^

.. image:: http://farm7.static.flickr.com/6237/6324734370_e543f436fb_z.jpg
   :width: 600

LVS和Real-server不需要在一个网段：
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

由于通过IP Tunneling 封装后，封装后的IP包的目的地址为Real-server的IP地址，那么只要Real-server的地址能路由可达，Real-server在什么网络里都可以，这样可以减少对于公网IP地址的消耗，但是因为要处理IP Tunneling封装和解封装的开销，那么效率不如DR模式。

Real-server的系统设置：
^^^^^^^^^^^^^^^^^^^^^^^^^

由于需要Real-server支持IP Tunneling，所以设置与DR模式不太一样，LVS不需要设置tunl设备，LVS本身可以进行封装
i) 需要配置VIP在tunl设备上：(VIP：172.16.1.254)

.. code-block:: bash

  shell> ifconfig tunl0 172.16.1.254 netmask 255.255.255.255
  shell> ifconfig tunl0
  tunl0 Link encap:IPIP Tunnel HWaddr
  inet addr:172.16.1.254 Mask:255.255.255.255
  UP RUNNING NOARP MTU:1480 Metric:1
  RX packets:0 errors:0 dropped:0 overruns:0 frame:0
  TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
  collisions:0 txqueuelen:0
  RX bytes:0 (0.0 b) TX bytes:0 (0.0 b)


当添加tunl0设备时，自动载入需要的模块：

.. code-block:: bash

  shell> lsmod |grep ipip
  ipip 7516 0
  tunnel4 2700 1 ipip

其中，ipip依赖于tunnel4，假如现在删除tunnel4的话：

.. code-block:: bash

  shell> rmmod tunnel4
  ERROR: Module tunnel4 is in use by ipip

如果添加tunl0失败，那么可能是内核没有开启tunneling功能，默认是以模块形式，加载到内核里的：

.. image:: http://farm7.static.flickr.com/6115/6323980779_b0369d8de9.jpg



ARP问题：
^^^^^^^^^^^

如果LVS和Real-server不在一个网络内，不需要处理ARP问题，如果在相同网络，那么处理方法和DR模式一样，但是如果一样，我就不知道选择tun模式有什么好理由了，DR似乎效率更高些吧。

内核的包转发：
^^^^^^^^^^^^^^^

IP Tunneling模式不需要开启ip_forward功能。

NAT模式：
-------------

NAT模式的拓扑图
~~~~~~~~~~~~~~~~~~~

.. image:: http://farm7.static.flickr.com/6099/6323980789_b90f2546bf_b.jpg
   :width: 600

NAT模式的工作过程:
~~~~~~~~~~~~~~~~~~~~~

.. image:: http://farm7.static.flickr.com/6117/6324734426_fc573643bf.jpg

.. code-block:: rst

   client：202.100.1.2
   VIP：202.103.106.5
   Real-server：172.16.0.2 和 172.16.0.3（提供http和ftp服务）


1> client发送request到LVS的VIP上，VIP选择一个Real-server，并记录连接信息到hash表中，然后修改client的request的目的IP地址为Real-server的地址，将请求发给Real-server;

2> Real-server收到request包后，发现目的IP是自己的IP，于是处理请求，然后发送reply给LVS;

3> LVS收到reply包后，修改reply包的的源地址为VIP，发送给client;

4> 从client来的属于本次连接的包，查hash表，然后发给对应的Real-server。

5> 当client发送完毕，此次连接结束或者连接超时，那么LVS自动从hash表中删除此条记录。

下面是地址转换的过程：

.. image:: http://farm7.static.flickr.com/6239/6323980835_6e32e61090_b.jpg
   :width: 600

NAT模式的几个细节问题
~~~~~~~~~~~~~~~~~~~~~~~~~


NAT模式的Bugs
^^^^^^^^^^^^^^^^^^^^^^^^

* 在Linux的2.6版本，LVS-NAT不能做防火墙，在只有一个网关的情况下，没有任何问题。

* 防火墙不兼容：LVS的架构中，LVS的前端不能设置防火墙，修复的补丁”NFCT” patch。

* 源路由问题

ICMP重定向问题
^^^^^^^^^^^^^^^^^^^^^


一. 对于路由器来说，只有当如下条件同时满足的时候，才进行重定向


* 数据包的入接口和路由后的指定的出接口是同一个接口。

* 数据包的源IP地址和该包应走的下一跳IP地址属于同一个网段。

* 数据报非源路由的（这种情况应该比较少见了，源路由多见于Token Ring）。

* 系统开启重定向功能。

**例如：**
  两个路由器都开启了IP重定向功能。HostA 的默认网关为1.1.1.1。当HostA要和不在同一网段中的HostB通信的时候，会把数据报递交给默认网关RT1。然而RT1经过查找发现到达3.3.3.3的路径下一跳恰恰是经由自己的E0/1口的RT2接口1.1.1.2。满足上述条件，将会发生重定向。

二. LVS为什么会产生ICMP重定向问题：
* 在LVS-NAT模式下，如果LVS的各个成员，client，LVS，Real-server在同一个网段(比如：192.168.1.*/24)；

* 当Real-server将Reply发送回LVS时，Reply包是 RIP -> CIP的，LVS看到RIP-> CIP实际上根本没必要经过LVS，直接到网关就行了，因为大家在一个网段，所以产生ICMP重定向发送给Real-server；

* Real-server收到ICMP重定向包后，如果Real-server的ICMP重定向开启了，Real-server就会处理ICMP重定向包，直接将Reply包发给网关，这时Reply包头并没有被LVS重写，所以LVS负载出现了问题。

注意：这种情况只会出现在所有的LVS的成员都在一个网段的情况下。

重定向的处理办法（Real-server的配置）：

1> 关闭Real-server的重定向，忽略LVS发来的重定向包

2> 删除到网段的路由：

.. image:: http://farm7.static.flickr.com/6105/6324754200_9780818531_z.jpg
   :width: 600

执行：

.. code-block:: bash

  realserver:/etc/lvs#route del -net 192.168.1.0 netmask 255.255.255.0 dev eth0

路由已经被删除了：

.. image:: http://farm7.static.flickr.com/6214/6324754218_a6258829e5_z.jpg
   :width: 600

3> LVS-NAT模式支持四层的端口重写：
LVS-DR，LVS-TUN不能修改client发来的请求的目的端口，但是LVS-NAT可以，参考命令：

.. code-block:: bash

  shell> ipvsadm -a -t VIP:PORT -r RIP:NEWPORT -m -w 1

LVS的三种转发模式就先说到这吧，具体的细节可以参考LVS的HOWTO文档。
