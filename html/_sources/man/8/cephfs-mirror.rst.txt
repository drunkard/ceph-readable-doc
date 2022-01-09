:orphan:

=======================================================
 cephfs-mirror -- 用于镜像 CephFS 快照的 Ceph 守护进程
=======================================================

.. program:: cephfs-mirror

提纲
====

| **cephfs-mirror**


描述
====

:program:`cephfs-mirror` 是个守护进程，用于\
在 Ceph 集群间异步地镜像 Ceph 文件系统快照。

它通过 libcephfs 连接远程集群，通过默认搜索路径找到
ceph.conf 文件，即 ``/etc/ceph/$cluster.conf`` ，其中，
``$cluster`` 是人类可读的集群名字。


选项
====

.. option:: -c ceph.conf, --conf=ceph.conf

   启动时用 ``ceph.conf`` 配置文件，而非默认的
   ``/etc/ceph/ceph.conf`` 来确定监视器地址。

.. option:: -i ID, --id ID

   给 cephfs-mirror 设置名字的 ID 部分。

.. option:: -n TYPE.ID, --name TYPE.ID

   设置 rados 用户名（如 client.mirror ）

.. option:: --cluster NAME

   指定集群名（默认 ceph ）

.. option:: -d

   在前台运行，日志记录到标准错误（ stderr ）。

.. option:: -f

   在前台运行，日志记录到常规位置。



使用范围
========

**cephfs-mirror** 是 Ceph 的一部分，这是个伸缩力强、开源、
分布式的存储系统，更多信息参见 https://docs.ceph.com 。


参考
====

:doc:`ceph <ceph>`\(8)
