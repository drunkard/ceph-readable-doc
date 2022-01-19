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
.. confval:: rbd_default_order

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

librbd 缓存默认就是开启的，支持三种缓存策略：
write-around 、 write-back 、 and write-through 。
采用 write-around 和 write-back 策略时，写入会立即返回，
除非没写入存储集群的字节数大于 ``rbd_cache_max_dirty`` 。
write-around 策略不同于 write-back 策略的地方在于，
它不会用缓存提供读请求服务，这点和 write-back 策略不同，
并因此获得了更高的写入载荷性能。采用 write-through 策略时，
只有在数据的所有副本都存入磁盘后它才返回，
但是读取的可以出自缓存。

在收到回刷请求前，缓存的工作方式类似 write-through 缓存，
是为了确保能让较老的操作系统安全地运作，因为它们不会回刷，
也就不能保证崩溃一致性。

如果禁用了 librbd 缓存，写入和读取操作都是直接发生在存储集群上，
而且写入操作只在数据进入所有副本的磁盘上时才返回。

.. note::
   缓存位于客户端的内存中，而且每个 RBD 映像都有自己的缓存。
   正因为缓存位于客户端本地，所以，
   如果有其他人访问这个映像就不能保证一致性。
   在 RBD 之上运行的 GFS 或 OCFS 与这个缓存机制不兼容。

RBD 选项应该位于 ``ceph.conf`` 配置文件或\
中央配置库的 ``[client]`` 段下，\
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

RBD 支持一些高级功能，可以在创建映像时通过命令行指定，
或者用 ``rbd_default_features = <sum of feature numeric values>`` 或者
``rbd_default_features = <comma-delimited list of CLI values>`` 来配置默认功能。


``Layering``

:描述: 分层可以启用克隆功能。
:内置值: 1
:CLI 值: layering
:哪版加入: v0.52 (Bobtail)
:KRBD 支持情况: since v3.10
:默认值: yes


``Striping v2``

:描述: 条带化功能使得数据可以横跨多个对象。
       条带化有助于顺序读写载荷的并行化。
:内置值: 2
:CLI 值: striping
:哪版加入: v0.55 (Bobtail)
:KRBD 支持情况: 从 v3.10 起(只有默认的条带化， "fancy" 条带化在 v4.17 新加)
:默认值: yes


``Exclusive locking``

:描述: 启用后，客户端要写入一个对象前要先获取一个锁。
       独占锁只适用于任何时候只能有一个客户端访问一个映像的情形。
:内置值: 4
:CLI 值: exclusive-lock
:哪版加入: v0.92 (Hammer)
:KRBD 支持情况: since v4.9
:默认值: yes


``Object map``

:描述: 对象表的支持情况依赖于独占锁。块设备是简配的，
       意味着它只存储真的写入过的数据，也就是说，
       它们是 *稀疏的（ sparse ）* 。支持对象表才能追踪哪些对象真的存在
       （真的在设备上存储了数据）。对象表功能启用后可以加速的 I/O 操作有
       克隆、导入和导出稀疏分布的映像、和删除。
:内置值: 8
:CLI 值: object-map
:哪版加入: v0.93 (Hammer)
:KRBD 支持情况: since v5.3
:默认值: yes


``Fast-diff``

:描述: Fast-diff 功能依赖于对象表和独占锁。
       它向对象表增加了另外一个属性，
       使得它能够更快地生成一个映像的两个快照之间的差别。
       也能更快地计算出一个快照或者卷宗的实际数据量（ ``rbd du`` ）。
:内置值: 16
:CLI 值: fast-diff
:哪版加入: v9.0.1 (Infernalis)
:KRBD 支持情况: since v5.3
:默认值: yes


``Deep-flatten``

:描述: Deep-flatten 功能使得 ``rbd flatten`` 不但适用于映像自身，
       还能适用于一个映像的所有快照。没有它，
       映像的快照就必须依赖其父映像，
       这样如果不先删除这些映像就无法删除它们的父映像。
       Deep-flatten 功能能让一个父映像独立于它的克隆映像，
       即使它们有快照也不影响，只是额外牺牲一些 OSD 设备空间而已。
:内置值: 32
:CLI 值: deep-flatten
:哪版加入: v9.0.2 (Infernalis)
:KRBD 支持情况: since v5.1
:默认值: yes


``Journaling``

:描述: 日志记录功能依赖于独占锁功能。此功能\
       会按照发生顺序记录对一个映像做出的所有更改。
       RBD 镜像功能能利用日志功能把一个崩溃一致映像复制到一个远程集群。
       最好让 ``rbd-mirror`` 按需管理这个功能，
       因为长期启用它会导致大量额外的 OSD 空间被消耗。
:内置值: 64
:CLI 值: journaling
:哪版加入: v10.0.1 (Jewel)
:KRBD 支持情况: no
:默认值: no


``Data pool``

:描述: 在纠删码存储池里，映像的数据块对象需要单独存储到\
       映像元数据之外的存储池里。
:内置值: 128
:哪版加入: v11.1.0 (Kraken)
:KRBD 支持情况: since v4.11
:默认值: no


``Operations``

:描述: 用于限制较老的客户端对一个映像执行特定的维护性操作（比如克隆、创建快照）。
:内置值: 256
:哪版加入: v13.0.2 (Mimic)
:KRBD 支持情况: since v4.16


``Migrating``

:描述: 用于限制较老的客户端在迁移状态下打开一个映像。
:内置值: 512
:哪版加入: v14.0.1 (Nautilus)
:KRBD 支持情况: no


``Non-primary``

:描述: 用于限制利用基于快照的镜像功能对非主映像进行更改。
:内置值: 1024
:哪版加入: v15.2.0 (Octopus)
:KRBD 支持情况: no


QOS 选项
========
.. QOS Settings

librbd 支持限制单个映像的 IO ，有多种途径。
都会在一个指定进程内应用到一个指定映像 - 同一映像用在多个地方，
例如两个独立的 VM ，会有相互独立的限制。

* **IOPS:** 每秒的 I/O 数量（任意类型的 I/O ）
* **read IOPS:** 每秒的读  I/O 数
* **write IOPS:** 每秒的写 I/O 数
* **bps:** 每秒的字节数（任意类型的 I/O ）
* **read bps:** 每秒读取的字节数
* **write bps:** 每秒写入的字节数

这些限制中的每一个都与其他的无关。默认都是关闭的。
每一种限制都是用一个令牌桶算法来减速 I/O 的，
还得能配置限额（一段时间内的平均速度）、
和短时间内（爆发秒数）潜在的高速率（一次爆发）。
当这些限制中的任意一个达到时，而且爆发量也用完了，
librbd 就会把那种类型的 I/O 速率压低到限额之下。

例如，假设读取 bps 配置成了 100MB ，而写入的没限制，
写入就可以尽快处理，而读取就会被减速到平均 100MB/s 。
如果设置了读取 bps 的爆发量是 150MB ，
而且读取爆发秒数设置成了 5 秒，
读操作可以在 150MB/s 下维持 5 秒，
然后再降低到 100MB/s 的限额。

下列选项可以配置这些减速阀：

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
.. confval:: rbd_qos_exclude_ops
