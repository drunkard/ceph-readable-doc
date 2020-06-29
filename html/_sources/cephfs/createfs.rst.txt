====================
 创建 Ceph 文件系统
====================

.. Creating pools

创建存储池
==========

一个 Ceph 文件系统需要至少两个 RADOS 存储池，一个用于数据、一\
个用于元数据。配置这些存储池时需考虑：

- 为元数据存储池设置更高的副本级别，因为此存储池丢失任何数据都\
  会导致整个文件系统失效。
- 为元数据存储池分配延时较低的存储器（像 SSD ），因为它会直接\
  影响到客户端的操作延时。
- 用于创建文件系统的数据存储池是“默认”存储池，它还存储着所有\
  的索引节点回溯信息，可用于管理硬链接和灾难恢复。正因为如此，
  CephFS 内创建的所有索引节点都至少有一个对象存储于默认的\
  数据存储池。如果文件系统打算采用纠删码存储池，一般来说，\
  最好用副本存储池作为默认的数据存储池，以提升更新回溯信息时\
  涉及的小对象读写性能。另外，可以单独加一个纠删码存储池（参见
  :ref:`ecpool` ）用于存储整个子树下的目录和文件（参见
  :ref:`file-layouts` ）。

关于存储池的管理请参考 :doc:`/rados/operations/pools` 。例如，\
要用默认设置为文件系统创建两个存储池，你可以用下列命令：

.. code:: bash

    $ ceph osd pool create cephfs_data <pg_num>
    $ ceph osd pool create cephfs_metadata <pg_num>

通常，元数据存储池最多也就有几个 GB 的数据，因此我们建议用较小\
的 PG 数，在用的大型集群通常用 64 或 128 个。


.. Creating a file system

创建文件系统
============

创建好存储池后，你就可以用 ``fs new`` 命令启用文件系统了：

.. code:: bash

    $ ceph fs new <fs_name> <metadata> <data>

例如：

.. code:: bash

    $ ceph fs new cephfs cephfs_metadata cephfs_data
    $ ceph fs ls
    name: cephfs, metadata pool: cephfs_metadata, data pools: [cephfs_data ]

文件系统创建完毕后， MDS 服务器就能达到 *active* 状态了，比如\
在一个单 MDS 系统中：

.. code:: bash

    $ ceph mds stat
    cephfs-1/1/1 up {0=a=up:active}

建好文件系统且 MDS 活跃后，你就可以挂载此文件系统了。如果你创\
建的文件系统不止一个，挂载的时候还需指定挂载哪个。

	- `挂载 CephFS 文件系统`_
	- `把 CephFS 挂载为用户空间文件系统`_

.. _挂载 CephFS 文件系统: ../../cephfs/kernel
.. _把 CephFS 挂载为用户空间文件系统: ../../cephfs/fuse

如果你创建了不止一个文件系统，而且客户端在挂载时还不指定哪个\
文件系统，这时你可以用 `ceph fs set-default` 命令来决定他们\
能看到的文件系统。


.. Using Erasure Coded pools with CephFS

在 CephFS 中使用纠删码存储池
============================

纠删码存储池在启用覆写功能后可以用作 CephFS 的数据存储池，\
这样启用：

.. code:: bash

    ceph osd pool set my_ec_pool allow_ec_overwrites true

注意， EC 的覆写功能只有在 OSD 们都使用 BlueStore 后端时才支持。

纠删码存储池不能用作 CephFS 的元数据存储池，因为 CephFS 元数据\
是用 RADOS *OMAP* 数据结构存储的，而 EC 存储池不能存储。
