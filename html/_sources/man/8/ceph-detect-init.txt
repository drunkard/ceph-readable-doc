:orphan:

====================================================
 ceph-detect-init -- 显示 Ceph 应当使用的 init 系统
====================================================

.. program:: ceph-detect-init

提纲
====

| **ceph-detect-init** [--verbose] [--use-rhceph] [--default *init*]


描述
====

:program:`ceph-detect-init` is a utility that prints the init system
Ceph uses. It can be one of ``sysvinit``, ``upstart`` or ``systemd``.
The init system Ceph uses may not be the default init system of the
host operating system. For instance on Debian Jessie, Ceph may use
``sysvinit`` although ``systemd`` is the default.

If the init system of the host operating system is unknown, return on
error, unless :option:`--default` is specified.


选项
====

.. option:: --use-rhceph

   When an operating system identifies itself as Red Hat, it is
   treated as if it was CentOS. With :option:`--use-rhceph` it is
   treated as RHEL instead.

.. option:: --default INIT

   If the init system of the host operating system is unkown, return
   the value of *INIT* instead of failing with an error.

.. option:: --verbose

   Display additional information for debugging.


使用范围
========

:program:`ceph-detect-init` 是 Ceph 的一部分，这是个伸缩力强、开源、分布式\
的存储系统，更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph-disk <ceph-disk>`\(8),
:doc:`ceph-deploy <ceph-deploy>`\(8)
