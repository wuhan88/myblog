.. title: RPM打包时出现libtool: Version mismatch error问题解决方法
.. slug: the-solutions-to-libtool-Version-mismatch-error
.. date: 2012-05-27 20:17:59 UTC+08:00
.. tags: aclocal, autoconf, libtool, rpm
.. category: linux
.. link:
.. description:
.. type: text



.. code-block:: bash

  libtool: Version mismatch error.  This is libtool 2.2.6b Debian-2.2.6b-2ubuntu1,
  but the
  libtool: definition of this LT_INIT comes from an older release.
  libtool: You should recreate aclocal.m4 with macros from libtool 2.2.6b
  Debian-2.2.6b-2ubuntu1
  libtool: and run autoconf again.

这个是执行rpmbuild中的报错信息，解决方案很简单，删除包里的aclocal.m4，然后执行 aclocal 和 autoconf

.. code-block:: bash

  rm aclocal.m4 & aclocal & autoconf

最后再重新rpmbuild -bb一下就可以了

.. code-block:: bash

  rpmbuild -bb *.spec
