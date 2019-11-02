:orphan:

=======================================
 ceph-fuse -- 基于 FUSE 的 Ceph 客户端
=======================================

.. program:: ceph-fuse

提纲
====

| **ceph-fuse** [-n *client.username*] [ -m *monaddr*:*port* ] *mountpoint* [ *fuse options* ]


描述
====

**ceph-fuse** 是 Ceph 分布式文件系统的 FUSE
（用户空间文件系统）客户端，它会把 Ceph 文件系统（用 -m 选项或
ceph.conf 指定）挂载到指定挂载点。详情见\
`用 FUSE 挂载 CephFS`_ 。

文件系统可这样卸载： ::

        fusermount -u mountpoint

或向 ``ceph-fuse`` 进程发送 ``SIGINT`` 信号。


选项
====

ceph-fuse 识别不了的选项将传递给 libfuse 。

.. option:: -o opt,[opt...]

   挂载选项。

.. option:: -d

   在前台运行，把所有日志输出发送到标准错误 stderr 、并打开
   FUSE 调试（ -o debug ）。

.. option:: -c ceph.conf, --conf=ceph.conf

   用指定的 *ceph.conf* 而非默认的 ``/etc/ceph/ceph.conf`` 来\
   查找启动时需要的监视器地址。

.. option:: -m monaddress[:port]

   连接到指定监视器，而不是从 ceph.conf 里找。

.. option:: -k <path-to-keyring>

   提供密钥环路径，在标准位置没有时就用的上了。

.. option:: --client_mountpoint/-r root_directory

   把文件系统内的 root_directory 作为根挂载，而不是整个
   Ceph 文件系统树。

.. option:: -f

   前台：启动后不要退居后台（在前台运行）。不要生成 PID 文件。

.. option:: -s

   禁止多线程运行。


使用范围
========

**ceph-fuse** 是 Ceph 的一部分，这是个伸缩力强、开源、分布式的\
存储系统，更多信息参见 http://ceph.com/docs 。


参考
====

fusermount(8),
:doc:`ceph <ceph>`\(8)

.. _用 FUSE 挂载 CephFS: ../../../cephfs/fuse/
