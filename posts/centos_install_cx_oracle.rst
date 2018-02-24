.. title: CENT OS安装cx_Oracle遇到的问题及解决方法
.. slug: centos_install_cx_oracle
.. date: 2012-10-09 20:17:59 UTC+08:00
.. tags: cx_oracle, python
.. category: python
.. link:
.. description:
.. type: text


这个是 ``cx_Oracle`` 的说明链接http://cx-oracle.sourceforge.net/README.txt，我是采用源码安装
安装过程：

.. code-block:: bash

  wget  http://prdownloads.sourceforge.net/cx-oracle/cx_Oracle-5.0.3.tar.gz?download
  tar zxvf   cx_Oracle-5.0.3.tar.gz
  cd cx_Oracle-5.0.3
  python setup.py build
  python setup.py install

下面是检查它是否可以用

.. code-block:: python

  >>> import cx_Oracle
  接着出现了两个错误，错误信息如下：
  /usr/local/lib/python2.6/site-packages/cx_Oracle-5.0.3-py2.6-linux-x86_64.egg/cx_Oracle.py:3: 
  UserWarning: Module cx_Oracle was already imported from /usr/local/lib/python2.6/site-packages
  /cx_Oracle-5.0.3-py2.6-linux-x86_64.egg/cx_Oracle.pyc,
  but /usr/local/cx_Oracle-5.0.3 is being added to sys.path
  ImportError: libclntsh.so.10.1: cannot open shared object file: No such file or directory

下面来说说这两个错误的解决方法，
第一个问题，提示说/usr/local/cx_Oracle-5.0.3已经加入到python的 ``sys.path`` 里面了
那么我们就在 ``sys.path`` 中去掉这个路径：
具体方法

.. code-block:: python

   >>> import sys
   >>> sys.path
   ['', '/usr/local/lib/python2.6/site-packages/setuptools-0.6c11-py2.6.egg',
   '/usr/local/lib/python2.6/site-packages/fudge-0.9.4-py2.6.egg',
    '/usr/local/lib/python2.6/site-packages/python_memcached-1.45-py2.6.egg',
    '/usr/local/lib/python2.6/site-packages/MySQL_python-1.2.3c1-py2.6-linux-x86_64.egg',
   '/usr/local/lib/python2.6/site-packages/flup-1.0.3.dev_20100525-py2.6.egg',
   '/usr/local/lib/python2.6/site-packages/regex-0.1.20100706.1-py2.6-linux-x86_64.egg',
    '/usr/local/lib/python2.6/site-packages/cx_Oracle-5.0.3-py2.6-linux-x86_64.egg',
    '/usr/local/lib/python26.zip', '/usr/local/lib/python2.6', '/usr/local/lib/python2.6/plat-linux2',
    '/usr/local/lib/python2.6/lib-tk', '/usr/local/lib/python2.6/lib-old',
   '/usr/local/lib/python2.6/lib-dynload', '/usr/local/lib/python2.6/site-packages',
    '/usr/local/lib/python2.6/site-packages/PIL', '/usr/local/cx_Oracle-5.0.3']
   
   >>> sys.path.remove(r'/usr/local/cx_Oracle-5.0.3')

第一个问题解决，接着第二个问题，这个网上有解决方案，下面说说解决方法：
这个的原因是 ``Oracle`` 的路径没有设定

.. code-block:: bash

   locate libclntsh.so.10.1
   /opt/oracle/product/10.2/db_1/lib/libclntsh.so.10.1
   /opt/oracle/product/10.2/db_1/lib32/libclntsh.so.10.1
   echo /opt/oracle/product/10.2/db_1/lib/ >> /etc/ld.so.conf
   ldconfig

就这样第二个问题解决
再试一下：

.. code-block:: python

   >>> import cx_Oracle
   >>>
