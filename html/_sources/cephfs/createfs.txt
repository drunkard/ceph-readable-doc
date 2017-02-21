====================
 创建 Ceph 文件系统
====================

.. tip::

    ``ceph fs new`` 命令是从 Ceph 0.84 起引入的，在此之前，无需手动创建文件\
    系统，名为 ``data`` 和 ``metadata`` 的存储池默认即存在。

    Ceph 命令行现在有了创建和删除文件系统的命令，但是当前一套集群只能有一个\
    文件系统存在。

一个 Ceph 文件系统需要至少两个 RADOS 存储池，一个用于数据、一个用于元数据。\
配置这些存储池时需考虑：

- 为元数据存储池设置较高的副本水平，因为此存储池丢失任何数据都会导致整个文件\
  系统失效。
- 为元数据存储池分配低延时存储器（像 SSD ），因为它会直接影响到客户端的操作\
  延时。

关于存储池的管理请参考 :doc:`/rados/operations/pools` 。例如，要用默认设置为\
文件系统创建两个存储池，你可以用下列命令：

.. code:: bash

    $ ceph osd pool create cephfs_data <pg_num>
    $ ceph osd pool create cephfs_metadata <pg_num>

创建好存储池后，你就可以用 ``fs new`` 命令创建文件系统了：

.. code:: bash

    $ ceph fs new <fs_name> <metadata> <data>

例如：

.. code:: bash

    $ ceph fs new cephfs cephfs_metadata cephfs_data
    $ ceph fs ls
    name: cephfs, metadata pool: cephfs_metadata, data pools: [cephfs_data ]

文件系统创建完毕后， MDS 服务器就能达到 *active* 状态了，比如在\
一个单 MDS 系统中：

.. code:: bash

    $ ceph mds stat
    e5: 1/1/1 up {0=a=up:active}

建好文件系统且 MDS 活跃后，你就可以挂载此文件系统了：

	- `挂载 CephFS 文件系统`_
	- `把 CephFS 挂载为用户空间文件系统`_

.. _挂载 CephFS 文件系统: ../../cephfs/kernel
.. _把 CephFS 挂载为用户空间文件系统: ../../cephfs/fuse
