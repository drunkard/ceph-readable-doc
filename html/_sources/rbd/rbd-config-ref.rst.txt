==========
 配置选项
==========
.. Config Settings

详情见\ `块设备`_\ 。

通用 IO 选项
============
.. Generic IO Settings

.. confval:: rbd_compression_hint
.. confval:: rbd_read_from_replica_policy


缓存选项
========
.. Cache Settings

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

.. confval:: rbd_cache
.. confval:: rbd_cache_policy
.. confval:: rbd_cache_writethrough_until_flush
.. confval:: rbd_cache_size
.. confval:: rbd_cache_max_dirty
.. confval:: rbd_cache_target_dirty
.. confval:: rbd_cache_max_dirty_age

.. _块设备: ../../rbd


预读选项
========
.. Read-ahead Settings

librbd 支持预读或预取功能，以此优化小块的顺序读。此功能通常\
应该由访客操作系统（是虚拟机）处理，但是引导加载程序还不能进行\
高效的读。如果缓存功能停用、或策略为 write-around ，预读就会\
自动被禁用。

.. confval:: rbd_readahead_trigger_requests
.. confval:: rbd_readahead_max_bytes
.. confval:: rbd_readahead_disable_after_bytes


映像功能
========
.. Image Features

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


QOS 选项
========
.. QOS Settings

librbd supports limiting per image IO, controlled by the following
settings.

.. confval:: rbd_qos_iops_limit
.. confval:: rbd_qos_bps_limit
.. confval:: rbd_qos_read_iops_limit
.. confval:: rbd_qos_write_iops_limit
.. confval:: rbd_qos_read_bps_limit
.. confval:: rbd_qos_write_bps_limit
.. confval:: rbd_qos_iops_burst
.. confval:: rbd_qos_bps_burst
.. confval:: rbd_qos_read_iops_burst
.. confval:: rbd_qos_write_iops_burst
.. confval:: rbd_qos_read_bps_burst
.. confval:: rbd_qos_write_bps_burst
.. confval:: rbd_qos_iops_burst_seconds
.. confval:: rbd_qos_bps_burst_seconds
.. confval:: rbd_qos_read_iops_burst_seconds
.. confval:: rbd_qos_write_iops_burst_seconds
.. confval:: rbd_qos_read_bps_burst_seconds
.. confval:: rbd_qos_write_bps_burst_seconds
.. confval:: rbd_qos_schedule_tick_min
