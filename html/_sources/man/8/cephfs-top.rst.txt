:orphan:

=====================================
 cephfs-top -- Ceph 文件系统热点工具
=====================================

.. program:: cephfs-top

提纲
====

| **cephfs-top** [flags]


描述
====

**cephfs-top** 为 Ceph 文件系统提供了 top(1) 风格的功能。
可以显示各种客户端指标并实时更新。

Ceph 元数据服务器周期性地向 Ceph 管理器发送客户端指标。
Ceph 管理器的 ``Stats`` 插件有接口获取这些指标。

选项
====

.. option:: --cluster

   集群：要连接的 Ceph 集群，默认是 ``ceph`` 。

.. option:: --id

   Id: 用于连接 Ceph 集群的客户端，默认是 ``fstop`` 。

.. option:: --selftest

   执行一次自检。这个模式对 ``stats`` 模块执行一次健全性检查。

字段描述
========

.. describe:: chit

   cap hit rate

.. describe:: dlease

   dentry lease rate

.. describe:: ofiles

   已打开文件的数量

.. describe:: oicaps

   number of pinned caps

.. describe:: oinodes

   打开的 inode 数量

.. describe:: rtio

   读 IO 的总量

.. describe:: wtio

   写 IO 的总量

.. describe:: raio

   读 IO 的平均尺寸

.. describe:: waio

   写 IO 的平均尺寸

.. describe:: rsp

   与上次刷新相比的读 IO 速度

.. describe:: wsp

   与上次刷新相比的写 IO 速度

.. describe:: rlatavg

   平均读延时

.. describe:: rlatsd

   读取延时的标准偏差（方差）

.. describe:: wlatavg

   平均写延时

.. describe:: wlatsd

   写入延时的标准偏差（方差）

.. describe:: mlatavg

   平均元数据延时

.. describe:: mlatsd

   元数据延时的标准偏差（方差）


使用范围
========

**cephfs-top** 是 Ceph 的一部分，这是个伸缩力强、开源、
分布式的存储系统，更多信息参见 https://ceph.com/ 。


参考
====

:doc:`ceph <ceph>`\(8),
:doc:`ceph-mds <ceph-mds>`\(8)
