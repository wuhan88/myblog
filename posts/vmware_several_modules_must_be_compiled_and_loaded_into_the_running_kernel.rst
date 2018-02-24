VMware-several-modules-must-be-compiled-and-loaded-into-the-running-kernel
==========================================================================
.. title: VMware-several-modules-must-be-compiled-and-loaded-into-the-running-kernel
.. slug: VMware-several-modules-must-be-compiled-and-loaded-into-the-running-kernel
.. date: 2011-04-29 20:17:59 UTC+08:00
.. tags: vmware
.. category:
.. link:
.. description:
.. type: text


在ubuntu 12.04上安装vmware workstation 8.02,

在vmware kernel module updater这个窗口上，编译第二步Virtual Network Device 时出现错误

查看Log，有如下报错：

.. code-block:: rst 

    2012-06-24T14:56:24.833+08:00| vthread-3| I120: Your GCC version: 4.6
    2012-06-24T14:56:24.839+08:00| vthread-3| I120: Your GCC version: 4.6
    2012-06-24T14:56:24.855+08:00| vthread-3| I120: Header path /lib/modules/3.2.0-25-generic/build/include for kernel release 3.2.0-25-generic is valid.
    2012-06-24T14:56:24.867+08:00| vthread-3| I120: Trying to find a suitable PBM set for kernel 3.2.0-25-generic.

解决方案：

.. code-block:: bash 

   # cd /usr/lib/vmware/modules; 
   # wget http://pavlinux.ru/vmware/8.0.2/source.tar.lzma; 
   # tar -xf source.tar.lzma; 
   # vmware-modconfig --console --install-all

.. image:: https://pavlinux.ru/vmware/8.0.2/make.png
