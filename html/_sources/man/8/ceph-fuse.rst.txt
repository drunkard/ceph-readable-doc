:orphan:

=======================================
 ceph-fuse -- 基于 FUSE 的 Ceph 客户端
=======================================

.. program:: ceph-fuse

提纲
====

| **ceph-fuse** [ -m *monaddr*:*port* ] *mountpoint* [ *fuse options* ]


描述
====

**ceph-fuse** 是 Ceph 分布式文件系统的 FUSE （用户空间文件系统）客户端，它会把 \
Ceph 文件系统（用 -m 选项或 ceph.conf 指定）挂载到指定挂载点。

文件系统可这样卸载： ::

        fusermount -u mountpoint

或向 ``ceph-fuse`` 进程发送 ``SIGINT`` 信号。


选项
====

ceph-fuse 识别不了的选项将传递给 libfuse 。

.. option:: -d

   启动后将脱离终端、进入守护状态。

.. option:: -c ceph.conf, --conf=ceph.conf

   用指定的 *ceph.conf* 而非默认的 ``/etc/ceph/ceph.conf`` 来查找启动时需要的监视\
   器地址。

.. option:: -m monaddress[:port]

   连接到指定监视器，而不是从 ceph.conf 里找。

.. option:: -r root_directory

   把文件系统内的 root_directory 作为根挂载，而不是整个 Ceph 文件系统树。


使用范围
========

**ceph-fuse** 是 Ceph 的一部分，这是个伸缩力强、开源、分布式的\
存储系统，更多信息参见 http://ceph.com/docs 。


参考
====

fusermount(8),
:doc:`ceph <ceph>`\(8)
