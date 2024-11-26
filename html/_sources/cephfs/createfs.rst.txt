====================
 创建 Ceph 文件系统
====================
.. Create a Ceph file system

创建存储池
==========
.. Creating pools

一个 Ceph 文件系统需要至少两个 RADOS 存储池，一个用于数据、\
一个用于元数据。规划这些存储池时有几个重要的注意事项：

- 我们建议给元数据存储池配置\ *至少三个*\ 副本，
  因为此存储池丢失数据可能会导致整个文件系统不可访问。
  配置 4 份都不算多，因为元数据存储池占用不了多大容量。
- 我们建议给元数据存储池分配最快的、延时低的存储器
  （ NVMe 、 Optane 、或者至少是 SAS/SATA SSD ），
  因为它会直接影响到客户端的操作延时。
- 我们强烈建议在专用 SSD / NVMe OSD 上配置 CephFS 元数据存储池。
  这样可确保客户端工作处于高负载时不会对元数据操作造成不利影响。
  参阅 :ref:`device_classes` ，以这种方式配置存储池。
- 用于创建文件系统的数据存储池默认是名为 default 的存储池，
  它还存储着所有的索引节点回溯信息，可用于管理硬链接和灾难恢复。
  正因为如此， CephFS 内创建的所有索引节点\
  都至少有一个对象存储在 default 数据存储池内。
  如果文件系统的数据打算放在纠删码存储池内，
  最好还是用副本存储池作为默认的数据存储池，
  以提升更新回溯信息时涉及的小对象读写性能。
  另外，可以单独加一个纠删码存储池（参见 :ref:`ecpool` ）
  用于存储整个子树下的目录和文件（参见 :ref:`file-layouts` ）。

关于存储池的管理请参考 :doc:`/rados/operations/pools` 。
例如，要用默认配置信息为文件系统创建两个存储池，你可以用下列命令：

.. code:: bash

    $ ceph osd pool create cephfs_data
    $ ceph osd pool create cephfs_metadata

通常，元数据存储池最多也就有几个 GB 的数据，因此我们建议用较小的 PG 数，
在用的大型集群通常用 64 或 128 个。

.. note:: 文件系统、元数据存储池、和数据存储池的名字只能用
   [a-zA-Z0-9\_-.] 里的字符。


创建文件系统
============
.. Creating a file system

创建好存储池后，你就可以用 ``fs new`` 命令启用文件系统了：

.. code:: bash

    $ ceph fs new <fs_name> <metadata> <data> [--force] [--allow-dangerous-metadata-overlay] [<fscid:int>] [--recover] [--yes-i-really-really-mean-it] [<set>...]

该命令用指定的元数据和数据存储池创建新文件系统。
指定的数据存储池是默认数据存储池，一旦设置就不能更改。
每个文件系统都有自己的一套 MDS 守护进程，分别负责各个 rank ，
因此要确保有足够多的备用守护进程来容纳新文件系统。

.. note::
   有些 ``fs set`` 命令也许需要加 ``--yes-i-really-really-mean-it`` 选项。

``--force`` 选项用于实现以下功能：

- 用来把纠删码存储池设置为默认的数据存储池。
  用纠删码存储池作为默认数据存储池是不受鼓励的。详情见\ `创建存储池`_\ 。
- 用来把非空存储池（存储池内已经有一些对象）设置为元数据存储池。
- 用来创建具有指定文件系统 ID （ fscid ）的文件系统。
  指定了 --fscid 选项时，必须加 --force 选项。

``--allow-dangerous-metadata-overlay`` 选项允许\
重用已在使用中的元数据和数据存储池。
只有在紧急情况下，并仔细阅读了相关文档后，才可以这样做。

如果加了 ``--fscid`` 选项，就会创建一个具有指定 fscid 的文件系统。
当应用程序希望文件系统的 ID 在恢复后仍然保持稳定时，
就可以用该选项，比如监视器数据库丢失并重建后。
因此，文件系统 ID 不一定随着新文件系统的创建而增加。

``--recover`` 选项会将文件系统的 rank 0 的状态设置为存在但失败。
这样的话，当一个 MDS 守护进程最终接管 rank 0 时，
这个守护进程就会读取现有 RADOS 内的元数据，而不会覆盖它。
这个标志还能防止备用 MDS 守护进程加入文件系统。

``set`` 选项允许在创建文件系统的同时，
原子地设置 ``fs set`` 支持的多个选项。

例如：

.. code:: bash

    $ ceph fs new cephfs cephfs_metadata cephfs_data set max_mds 2 allow_standby_replay true
    $ ceph fs ls
    name: cephfs, metadata pool: cephfs_metadata, data pools: [cephfs_data ]

文件系统创建完毕后， MDS 服务器就能达到 *active* 状态了，
比如在一个单 MDS 系统中：

.. code:: bash

    $ ceph mds stat
    cephfs-1/1/1 up {0=a=up:active}

建好文件系统且 MDS 活跃后，你就可以挂载此文件系统了。
如果你创建的文件系统不止一个，挂载的时候还需指定挂载哪个。

  - `挂载 CephFS 文件系统`_
  - `把 CephFS 挂载为用户空间文件系统`_
  - `在 Windows 上挂载 CephFS`_

.. _挂载 CephFS 文件系统: ../../cephfs/mount-using-kernel-driver
.. _把 CephFS 挂载为用户空间文件系统: ../../cephfs/mount-using-fuse
.. _在 Windows 上挂载 CephFS: ../../cephfs/ceph-dokan

如果你创建了不止一个文件系统，而且客户端在挂载时还没有指定哪个文件系统，
这时你可以用 ``ceph fs set-default`` 命令来决定他们能看到的文件系统。


向文件系统增加数据存储池
------------------------
.. Adding a Data Pool to the File System 

见 :ref:`adding-data-pool-to-file-system` 。


在 CephFS 中使用纠删码存储池
============================
.. Using Erasure Coded pools with CephFS

纠删码存储池在启用覆写功能（ overwrites ）后可以用作 CephFS 的数据存储池，
这样启用：

.. code:: bash

    ceph osd pool set my_ec_pool allow_ec_overwrites true

注意， EC 的覆写功能只有在 OSD 们都使用 BlueStore 后端时才支持。

纠删码存储池不能用作 CephFS 的元数据存储池，因为 CephFS 元数据\
是用 RADOS *OMAP* 数据结构存储的，而 EC 存储池不能存储这个。
