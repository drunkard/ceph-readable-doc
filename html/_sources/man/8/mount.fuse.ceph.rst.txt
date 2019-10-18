:orphan:

===================================================
 mount.fuse.ceph -- 在 /etc/fstab 里挂载 ceph-fuse
===================================================

.. program:: mount.fuse.ceph

提纲
====

| **mount.fuse.ceph** [-h] [-o OPTIONS [*OPTIONS* ...]]
                      device [*device* ...]
                      mountpoint [*mountpoint* ...]

描述
====

**mount.fuse.ceph** 是在 ``/etc/fstab`` 里挂载 ceph-fuse 的\
一个辅助程序。

要用 mount.fuse.ceph ，在 ``/etc/fstab`` 里增加一条类似如下的： ::

  DEVICE    PATH        TYPE        OPTIONS
  none      /mnt/ceph   fuse.ceph   ceph.id=admin,_netdev,defaults  0 0
  none      /mnt/ceph   fuse.ceph   ceph.name=client.admin,_netdev,defaults  0 0
  none      /mnt/ceph   fuse.ceph   ceph.id=myuser,ceph.conf=/etc/ceph/foo.conf,_netdev,defaults  0 0

ceph-fuse 的选项写在 ``OPTIONS`` 一列，且必须以 ``ceph.``
打头。这样 ceph 相关的文件系统选项就会传递给 ceph-fuse ，而\
其余的 ceph-fuse 会忽略。


选项
====

.. option:: ceph.id=<username>

   让 ceph-fuse 以给定的用户认证。

.. option:: ceph.name=client.admin

   让 ceph-fuse 以 client.admin 认证。

.. option:: ceph.conf=/etc/ceph/foo.conf

   把 ceph-fuse 命令行的 conf 选项设置为 /etc/ceph/foo.conf 。


所有 ceph-fuse 的有效选项都可以这样传入。


附加信息
========

/etc/fstab 的旧格式条目也可以： ::

  DEVICE                              PATH        TYPE        OPTIONS
  id=admin                            /mnt/ceph   fuse.ceph   defaults   0 0
  id=myuser,conf=/etc/ceph/foo.conf   /mnt/ceph   fuse.ceph   defaults   0 0


使用范围
========

**mount.fuse.ceph** 是 Ceph 的一部分，这是个伸缩力强、开源、\
分布式的存储系统，更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph-fuse <ceph-fuse>`\(8),
:doc:`ceph <ceph>`\(8)
