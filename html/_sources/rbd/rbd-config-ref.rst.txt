.. Config Settings

==========
 配置选项
==========

详情见\ `块设备`_\ 。

.. Generic IO Settings

通用 IO 选项
============

``rbd compression hint``

:描述: Hint to send to the OSDs on write operations. If set to `compressible` and the OSD `bluestore compression mode` setting is `passive`, the OSD will attempt to compress the data. If set to `incompressible` and the OSD compression setting is `aggressive`, the OSD will not attempt to compress the data.
:类型: Enum
:是否必需: No
:默认值: ``none``
:Values: ``none``, ``compressible``, ``incompressible``

``rbd read from replica policy``

:描述: policy for determining which OSD will receive read operations. If set to `default`, the primary OSD will always be used for read operations. If set to `balance`, read operations will be sent to a randomly selected OSD within the replica set. If set to `localize`, read operations will be sent to the closest OSD as determined by the CRUSH map. Note: this feature requires the cluster to be configured with a minimum compatible OSD release of Octopus.
:类型: Enum
:是否必需: No
:默认值: ``default``
:Values: ``default``, ``balance``, ``localize``


.. Cache Settings

缓存选项
========

.. sidebar:: 内核缓存

	Ceph 块设备的内核驱动可利用 Linux 页缓存来提升性能。

Ceph 块设备的用户空间实现（即 ``librbd`` ）不能利用 Linux
页缓存，所以它自己实现了内存缓存，名为“ RBD 缓存”。 RBD 缓存\
就像硬盘缓存一样工作，当 OS 发送了 barrier 或 flush 请求时，\
所有脏数据都会写入 OSD ，这意味着只要 VM 会正确地发送回写命令\
（即内核版本大于 2.6.32 ），使用回写缓存就像用正常的物理硬盘\
一样安全。此缓存用最近最少使用（ Least Recently Used, LRU ）\
算法，而且在回写模式下它能合并相邻请求以提高吞吐量。

The librbd cache is enabled by default and supports three different cache
policies: write-around, write-back, and write-through. Writes return
immediately under both the write-around and write-back policies, unless there
are more than ``rbd cache max dirty`` unwritten bytes to the storage cluster.
The write-around policy differs from the write-back policy in that it does
not attempt to service read requests from the cache, unlike the write-back
policy, and is therefore faster for high performance write workloads. Under the
write-through policy, writes return only when the data is on disk on all
replicas, but reads may come from the cache.

Prior to receiving a flush request, the cache behaves like a write-through cache
to ensure safe operation for older operating systems that do not send flushes to
ensure crash consistent behavior.

If the librbd cache is disabled, writes and
reads go directly to the storage cluster, and writes return only when the data
is on disk on all replicas.

.. note::
   The cache is in memory on the client, and each RBD image has
   its own.  Since the cache is local to the client, there's no coherency
   if there are others accessing the image. Running GFS or OCFS on top of
   RBD will not work with caching enabled.

RBD 选项应该位于 ``ceph.conf`` 配置文件的 ``[client]`` 段下，\
可用选项有：


``rbd cache``

:描述: 允许为 RADOS 块设备提供缓存。
:类型: Boolean
:是否必需: No
:默认值: ``true``


``rbd cache policy``

:描述: Select the caching policy for librbd.
:类型: Enum
:是否必需: No
:默认值: ``writearound``
:Values: ``writearound``, ``writeback``, ``writethrough``


``rbd cache writethrough until flush``

:描述: 开始进入 write-through 模式，并且在首个 flush 请求收到\
       后切回 write-back 模式。启用它保守但安全，以防 rbd 之上\
       的虚拟机内核太老、不能发送 flush ，像 2.6.32 之前的
       virtio 驱动。
:类型: Boolean
:是否必需: No
:默认值: ``true``


``rbd cache size``

:描述: RBD 缓存尺寸，字节。
:类型: 64-bit Integer
:是否必需: No
:默认值: ``32 MiB``
:适用策略: write-back and write-through


``rbd cache max dirty``

:描述: 使缓存触发写回的 ``dirty`` 临界点，若为 ``0`` ，直接\
       使用写透缓存。
:类型: 64-bit Integer
:是否必需: No
:约束条件: 必须小于 ``rbd cache size`` 。
:默认值: ``24 MiB``
:适用策略: write-around and write-back


``rbd cache target dirty``

:描述: 缓存开始写回数据的目的地 ``dirty target`` ，不会阻塞到\
       缓存的写动作。
:类型: 64-bit Integer
:是否必需: No
:约束条件: 必须小于 ``rbd cache max dirty``.
:默认值: ``16 MiB``
:适用策略: write-back


``rbd cache max dirty age``

:描述: 写回开始前，脏数据在缓存中的暂存时间。
:类型: Float
:是否必需: No
:默认值: ``1.0``
:适用策略: write-back

.. _块设备: ../../rbd


.. Read-ahead Settings

预读选项
========

librbd 支持预读或预取功能，以此优化小块的顺序读。此功能通常\
应该由访客操作系统（是虚拟机）处理，但是引导加载程序还不能进行\
高效的读。如果缓存功能停用、或策略为 write-around ，预读就会\
自动被禁用。


``rbd readahead trigger requests``

:描述: 触发预读的顺序读请求数量。
:类型: Integer
:是否必需: No
:默认值: ``10``


``rbd readahead max bytes``

:描述: 预读请求最大尺寸，零为禁用预读。
:类型: 64-bit Integer
:是否必需: No
:默认值: ``512 KiB``


``rbd readahead disable after bytes``

:描述: 从 RBD 映像读取这么多字节后，预读功能将被禁用，直到关闭。这样访客操作\
       系统启动后就可以接管预读了，设为 0 时则仍开启预读。
:类型: 64-bit Integer
:是否必需: No
:默认值: ``50 MiB``


.. Image Features

映像功能
========

RBD supports advanced features which can be specified via the command line when creating images or the default features can be specified via Ceph config file via 'rbd_default_features = <sum of feature numeric values>' or 'rbd_default_features = <comma-delimited list of CLI values>'


``Layering``

:描述: Layering enables you to use cloning.
:内置值: 1
:CLI 值: layering
:哪版加入: v0.52 (Bobtail)
:KRBD 支持情况: since v3.10
:默认值: yes


``Striping v2``

:描述: Striping spreads data across multiple objects. Striping helps with parallelism for sequential read/write workloads.
:内置值: 2
:CLI 值: striping
:哪版加入: v0.55 (Bobtail)
:KRBD 支持情况: since v3.10 (default striping only, "fancy" striping added in v4.17)
:默认值: yes


``Exclusive locking``

:描述: When enabled, it requires a client to get a lock on an object before making a write. Exclusive lock should only be enabled when a single client is accessing an image at the same time. 
:内置值: 4
:CLI 值: exclusive-lock
:哪版加入: v0.92 (Hammer)
:KRBD 支持情况: since v4.9
:默认值: yes


``Object map``

:描述: Object map support depends on exclusive lock support. Block devices are thin provisioned—meaning, they only store data that actually exists. Object map support helps track which objects actually exist (have data stored on a drive). Enabling object map support speeds up I/O operations for cloning; importing and exporting a sparsely populated image; and deleting.
:内置值: 8
:CLI 值: object-map
:哪版加入: v0.93 (Hammer)
:KRBD 支持情况: since v5.3
:默认值: yes


``Fast-diff``

:描述: Fast-diff support depends on object map support and exclusive lock support. It adds another property to the object map, which makes it much faster to generate diffs between snapshots of an image, and the actual data usage of a snapshot much faster.
:内置值: 16
:CLI 值: fast-diff
:哪版加入: v9.0.1 (Infernalis)
:KRBD 支持情况: since v5.3
:默认值: yes


``Deep-flatten``

:描述: Deep-flatten makes rbd flatten work on all the snapshots of an image, in addition to the image itself. Without it, snapshots of an image will still rely on the parent, so the parent will not be delete-able until the snapshots are deleted. Deep-flatten makes a parent independent of its clones, even if they have snapshots.
:内置值: 32
:CLI 值: deep-flatten
:哪版加入: v9.0.2 (Infernalis)
:KRBD 支持情况: since v5.1
:默认值: yes


``Journaling``

:描述: Journaling support depends on exclusive lock support. Journaling records all modifications to an image in the order they occur. RBD mirroring utilizes the journal to replicate a crash consistent image to a remote cluster.
:内置值: 64
:CLI 值: journaling
:哪版加入: v10.0.1 (Jewel)
:KRBD 支持情况: no
:默认值: no


``Data pool``

:描述: On erasure-coded pools, the image data block objects need to be stored on a separate pool from the image metadata.
:内置值: 128
:哪版加入: v11.1.0 (Kraken)
:KRBD 支持情况: since v4.11
:默认值: no


``Operations``

:描述: Used to restrict older clients from performing certain maintenance operations against an image (e.g. clone, snap create).
:内置值: 256
:哪版加入: v13.0.2 (Mimic)
:KRBD 支持情况: since v4.16


``Migrating``

:描述: Used to restrict older clients from opening an image when it is in migration state.
:内置值: 512
:哪版加入: v14.0.1 (Nautilus)
:KRBD 支持情况: no


``Non-primary``

:描述: Used to restrict changes to non-primary images using snapshot-based mirroring.
:内置值: 1024
:哪版加入: v15.2.0 (Octopus)
:KRBD 支持情况: no


.. QOS Settings

QOS 选项
========

librbd supports limiting per image IO, controlled by the following
settings.


``rbd qos iops limit``

:描述: The desired limit of IO operations per second.
:类型: Unsigned Integer
:是否必需: No
:默认值: ``0``


``rbd qos bps limit``

:描述: The desired limit of IO bytes per second.
:类型: Unsigned Integer
:是否必需: No
:默认值: ``0``


``rbd qos read iops limit``

:描述: The desired limit of read operations per second.
:类型: Unsigned Integer
:是否必需: No
:默认值: ``0``


``rbd qos write iops limit``

:描述: The desired limit of write operations per second.
:类型: Unsigned Integer
:是否必需: No
:默认值: ``0``


``rbd qos read bps limit``

:描述: The desired limit of read bytes per second.
:类型: Unsigned Integer
:是否必需: No
:默认值: ``0``


``rbd qos write bps limit``

:描述: The desired limit of write bytes per second.
:类型: Unsigned Integer
:是否必需: No
:默认值: ``0``


``rbd qos iops burst``

:描述: The desired burst limit of IO operations.
:类型: Unsigned Integer
:是否必需: No
:默认值: ``0``


``rbd qos bps burst``

:描述: The desired burst limit of IO bytes.
:类型: Unsigned Integer
:是否必需: No
:默认值: ``0``


``rbd qos read iops burst``

:描述: The desired burst limit of read operations.
:类型: Unsigned Integer
:是否必需: No
:默认值: ``0``


``rbd qos write iops burst``

:描述: The desired burst limit of write operations.
:类型: Unsigned Integer
:是否必需: No
:默认值: ``0``


``rbd qos read bps burst``

:描述: The desired burst limit of read bytes.
:类型: Unsigned Integer
:是否必需: No
:默认值: ``0``


``rbd qos write bps burst``

:描述: The desired burst limit of write bytes.
:类型: Unsigned Integer
:是否必需: No
:默认值: ``0``


``rbd qos iops burst seconds``

:描述: The desired burst duration in seconds of IO operations.
:类型: Unsigned Integer
:是否必需: No
:默认值: ``1``


``rbd qos bps burst seconds``

:描述: The desired burst duration in seconds of IO bytes.
:类型: Unsigned Integer
:是否必需: No
:默认值: ``1``


``rbd qos read iops burst seconds``

:描述: The desired burst duration in seconds of read operations.
:类型: Unsigned Integer
:是否必需: No
:默认值: ``1``


``rbd qos write iops burst seconds``

:描述: The desired burst duration in seconds of write operations.
:类型: Unsigned Integer
:是否必需: No
:默认值: ``1``


``rbd qos read bps burst seconds``

:描述: The desired burst duration in seconds of read bytes.
:类型: Unsigned Integer
:是否必需: No
:默认值: ``1``


``rbd qos write bps burst seconds``

:描述: The desired burst duration in seconds of write bytes.
:类型: Unsigned Integer
:是否必需: No
:默认值: ``1``


``rbd qos schedule tick min``

:描述: The minimum schedule tick (in milliseconds) for QoS.
:类型: Unsigned Integer
:是否必需: No
:默认值: ``50``
