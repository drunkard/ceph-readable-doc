.. _ceph-volume:

ceph-volume
===========

通过不同的设备技术（如 lvm 或物理硬盘）部署 OSD ，用可替换的\
工具（ :doc:`lvm/index` 它自己就被当成一个插件了），并按照可\
预见、健壮的方法来准备、激活、并启动 OSD 。

:ref:`概览 <ceph-volume-overview>` |
:ref:`插件手册 <ceph-volume-plugins>` |


**命令行子命令**

当前已经支持 ``lvm`` ，和普通硬盘（分区为 GPT ），它可能是用
``ceph-disk`` 部署的。

* :ref:`ceph-volume-lvm`
* :ref:`ceph-volume-simple`
* :ref:`ceph-volume-zfs`

**节点清单**

:ref:`ceph-volume-inventory` 子命令可查看一个节点上物理磁盘库\
的信息和元数据。


.. Migrating

迁移
----
Ceph 从 13.0.0 版起， ``ceph-disk`` 已弃用，弃用警告消息将链接\
到本页。我们强烈建议用户转向 ``ceph-volume`` ，迁移有两种途径：

#. 保留用 ``ceph-disk`` 部署的各 OSD: :ref:`ceph-volume-simple`
   命令提供了一条途径，可以在 ``ceph-disk`` 被禁用时接手管理；
#. 用 ``ceph-volume`` 重新部署已有的各 OSD: 这在 
   :ref:`rados-replacing-an-osd` 里有详细说明。

``ceph-disk`` 被去除的原因请看
:ref:`ceph-disk 为何被取代了？ <ceph-disk-replaced>` 一节。


.. New deployments

全新部署
^^^^^^^^
对于全新部署，我们推荐用 :ref:`ceph-volume-lvm` ，它可以用逻辑\
卷作数据 OSD 、或者把设备配置成一个最小化/全新的逻辑卷。


.. Existing OSDs

已有 OSD 怎么办
^^^^^^^^^^^^^^^
如果集群内已经有了用 ``ceph-disk`` 开通的 OSD ，
``ceph-volume`` 可以用 :ref:`ceph-volume-simple` 接管它们。扫\
描完数据设备或 OSD 目录后，就会完全禁用 ``ceph-disk`` 。加密已\
完全支持。


.. toctree::
   :hidden:
   :maxdepth: 3
   :caption: Contents:

   intro
   systemd
   inventory
   lvm/index
   lvm/activate
   lvm/batch
   lvm/encryption
   lvm/prepare
   lvm/create
   lvm/scan
   lvm/systemd
   lvm/list
   lvm/zap
   simple/index
   simple/activate
   simple/scan
   simple/systemd
   zfs/index
   zfs/inventory
