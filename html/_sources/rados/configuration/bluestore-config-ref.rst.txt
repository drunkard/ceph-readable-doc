====================
 BlueStore 配置参考
====================
.. BlueStore Configuration Reference 

设备
====
.. Devices

BlueStore 可管理一个、两个、某些情况下还能管理三个存储设备。
这些\ *设备*\ 是 Linux/Unix 意义上的 “设备（ devices ）”。
这意味着它们是在 ``/dev`` 或 ``/devices`` 下可以看得到的资产。
每个设备可以是整个存储驱动器，或是存储驱动器的一个分区，或是一个逻辑卷。
BlueStore 不会在它使用的设备上创建或挂载传统文件系统；
BlueStore 以“原始”方式直接读写设备。

在最简单的情况下， BlueStore 会使用整个存储设备，
这个设备被称为\ *主设备*\ 。
主设备由数据目录中的 ``block`` 符号链接标识。

数据目录挂载的是 ``tmpfs`` 文件系统。当该数据目录被启动或\
被 ``ceph-volume`` 激活时，它将被装入元数据文件和链接，
这些文件和链接包含着关于 OSD 的信息：例如， OSD 的标识符、
OSD 所属集群的名称以及 OSD 的私钥。

在更复杂的情况下， BlueStore
会跨越一个或两个设备部署：

* 可以使用\ *预写日志（ write-ahead log, WAL ）设备*\
  （数据目录中的 ``block.wal`` ）来分离 BlueStore 的内部日志或预写日志。
  只有在 WAL 设备的速度比主设备快的时候，使用 WAL 设备才有优势
  （例如，如果 WAL 设备是 SSD ，而主设备是 HDD ）。
* *DB 设备*\ （数据目录中的 ``block.db`` ）可用于存储 BlueStore 的内部元数据。
  为了提高性能， BlueStore （更准确地说，是嵌入式 RocksDB ）
  会尽可能多地把元数据存入 DB 设备。如果 DB 设备满了，
  元数据就会溢出，回到主设备上（没有 DB 设备时，元数据会放在主设备上）。
  同样，只有当 DB 设备的速度比主设备快的时候，
  配置 DB 设备才是有利的。

如果可用的快速存储空间比较少（例如，少于 1 GB ），
我们建议将这些空间用作 WAL 设备。
但如果可用的快速存储空间较多，则配置 DB 设备更为合理。
由于 BlueStore 日志总是放在可用的最快设备上，
因此使用 DB 设备与使用 WAL 设备具有同等优势，
*同时还允许*\ 在主设备以外存储额外的元数据
（前提是适合）。 DB 设备之所以能做到这一点，
是因为只要指定了 DB 设备，而没有明确指定 WAL 设备时，
WAL 就会隐式地与 DB 共存于这个速度更快的设备上。

要配置单设备（共存） BlueStore OSD ，
运行以下命令：

.. prompt:: bash $

   ceph-volume lvm prepare --bluestore --data <device>

要指定 WAL 设备或 DB 设备，请运行以下命令：

.. prompt:: bash $

   ceph-volume lvm prepare --bluestore --data <device> --block.wal <wal-device> --block.db <db-device>

.. note:: ``--data`` 选项可以用以下设备作为参数：
   用 *vg/lv* 方式指定的逻辑卷、
   现有的逻辑卷、 GPT 分区。


存储配置策略
------------
.. Provisioning strategies

BlueStore 与 Filestore 不同，它有多种部署 BlueStore OSD 的方法。
不过，只需研究这两种常见的做法，就可以明白 BlueStore 的整体部署策略：


.. _bluestore-single-type-device-config:

**block (data) only**
^^^^^^^^^^^^^^^^^^^^^

如果所有设备都是同一类型（例如，它们都是 HDD ），
而且没有可用来存储元数据的快速设备，
那么只需要指定块设备即可，不用分离出 ``block.db`` 和 ``block.wal`` 。
针对单个设备 ``/dev/sda`` 的 :ref:`ceph-volume-lvm` 命令如下：

.. prompt:: bash $

   ceph-volume lvm create --bluestore --data /dev/sda

如果要用于 BlueStore OSD 的设备已经预先创建了逻辑卷，那么在名为
``ceph-vg/block-lv`` 的逻辑卷上调用 :ref:`ceph-volume-lvm` 的命令如下：

.. prompt:: bash $

   ceph-volume lvm create --bluestore --data ceph-vg/block-lv


.. _bluestore-mixed-device-config:

**block 和 block.db**
^^^^^^^^^^^^^^^^^^^^^
.. **block and block.db**

如果混合使用快慢设备（例如 SSD 和 HDD），
我们建议将 ``block.db`` 放在速度较快的设备上，
而 ``block`` （即数据）则存储在速度较慢的设备上（也就是机械硬盘）。

您必须手动创建这些卷组和逻辑卷，因为 ``ceph-volume`` 工具\
目前还无法自动创建这些卷组和逻辑卷。

下面的步骤展示了如何手动创建卷组和逻辑卷。在本例中，
我们假设有四个机械硬盘（ ``sda`` 、 ``sdb`` 、 ``sdc`` 和 ``sdd`` ）
和一个（高速的） SSD （ ``sdx`` ）。首先，运行以下命令创建卷组：

.. prompt:: bash $

   vgcreate ceph-block-0 /dev/sda
   vgcreate ceph-block-1 /dev/sdb
   vgcreate ceph-block-2 /dev/sdc
   vgcreate ceph-block-3 /dev/sdd

接下来，为 ``block`` 创建逻辑卷，运行以下命令：

.. prompt:: bash $

   lvcreate -l 100%FREE -n block-0 ceph-block-0
   lvcreate -l 100%FREE -n block-1 ceph-block-1
   lvcreate -l 100%FREE -n block-2 ceph-block-2
   lvcreate -l 100%FREE -n block-3 ceph-block-3

因为有四个 HDD ，所以就会有四个 OSD 。假设 ``/dev/sdx`` 中有一个 200GB 的 SSD ，
我们可以创建四个 50GB 的逻辑卷，执行下列命令：

.. prompt:: bash $

   vgcreate ceph-db-0 /dev/sdx
   lvcreate -L 50GB -n db-0 ceph-db-0
   lvcreate -L 50GB -n db-1 ceph-db-0
   lvcreate -L 50GB -n db-2 ceph-db-0
   lvcreate -L 50GB -n db-3 ceph-db-0

最后，创建四个 OSD ，执行下列命令：

.. prompt:: bash $

   ceph-volume lvm create --bluestore --data ceph-block-0/block-0 --block.db ceph-db-0/db-0
   ceph-volume lvm create --bluestore --data ceph-block-1/block-1 --block.db ceph-db-0/db-1
   ceph-volume lvm create --bluestore --data ceph-block-2/block-2 --block.db ceph-db-0/db-2
   ceph-volume lvm create --bluestore --data ceph-block-3/block-3 --block.db ceph-db-0/db-3

完成这些步骤后，应该有四个 OSD ， ``block`` 应该在四个 HDD 上，
每个 HDD 在 SSD 上共享着一个 50GB 的逻辑卷（往细了说，是 DB 设备）。


调整尺寸
========
.. Sizing

:ref:`混合使用机械硬盘和固态硬盘 <bluestore-mixed-device-config>`\ 时，
给 BlueStore 创建足够大的 ``block.db`` 逻辑卷很重要。
与 ``block.db`` 相关联的逻辑卷应该\ *尽可能大*\ 。

一般建议， ``block.db`` 的大小应该是 ``block``
大小的 1% 至 4% 。对于 RGW 工作载荷，
建议 ``block.db`` 至少是 ``block`` 大小的 4% ，
因为 RGW 会大量使用 ``block.db`` 来存储元数据（尤其是 omap 键）。例如，
如果 ``block`` 大小为 1TB ，那么 ``block.db`` 的大小最低应该是 40GB 。
不过，对于 RBD 工作载荷， ``block.db`` 所需的大小\
通常不会超过 ``block`` 大小的 1% 到 2% 。

在旧版本中，内部会这样对准尺寸：数据库只能完全利用那些特定的尺寸，
就是与 L0 、 L0+L1 、 L1+L2 等等总和相对应的分区/逻辑卷的尺寸，
也就是说，在默认设置下，尺寸大致为 3GB 、 30GB 、 300GB 等等。
大多数部署不会从 L3 以及更大的容量中获得实质性好处，
尽管 DB 压缩技术可以将这些数字翻倍至 6GB 、 60GB 和 600GB 。

Nautilus 14.2.12 、 Octopus 15.2.6 以及后续版本做了改进，
可以更好地使用任意大小的 DB 设备。此外，
Pacific 版还试验性地支持动态对准。由于这些改进，
旧版本的用户可能希望未雨绸缪，
现在就配置更大的 DB 设备，
以便将来升级的时候实现规模效益。

如果\ *不*\ 混合使用快速和慢速设备，
则无需为 ``block.db`` 或 ``block.wal`` 创建单独的逻辑卷。
BlueStore 会自动把这些设备放置在 ``block`` 的空间内。


自动调整缓存尺寸
================
.. Automatic Cache Sizing

只要满足特定条件， BlueStore 就能配置为自动调整缓存大小：
必须用 TCMalloc 作为内存分配器、还必须启用 ``bluestore_cache_autotune`` 配置选项
（注意，目前是默认启用的）。当自动调整缓存大小生效时，
BlueStore 会尝试将 OSD 堆内存使用量保持在一定的目标尺寸
（由 ``osd_memory_target`` 确定）之内。
这种方法采用尽力而为的算法，
缓存不会缩减到小于 ``osd_memory_cache_min`` 所定义的尺寸。
缓存比率是根据优先级层次来选择的。
但如果没有优先级信息，就会用 ``bluestore_cache_meta_ratio`` 和
``bluestore_cache_kv_ratio`` 选项指定的值作为后备缓存比率。

.. confval:: bluestore_cache_autotune
.. confval:: osd_memory_target
.. confval:: bluestore_cache_autotune_interval
.. confval:: osd_memory_base
.. confval:: osd_memory_expected_fragmentation
.. confval:: osd_memory_cache_min
.. confval:: osd_memory_cache_resize_interval


手动调整缓存尺寸
================
.. Manual Cache Sizing

每个 OSD 用于 BlueStore 缓存的内存消耗量由 ``bluestore_cache_size`` 配置选项决定。
如果未指定该选项（即该选项保持为 0 ），
那么 Ceph 会使用别的配置选项来确定默认的内存预算：
如果主设备是 HDD ，则使用 ``bluestore_cache_size_hdd`` ；
如果主设备是 SSD ，则使用 ``bluestore_cache_size_ssd`` 。

BlueStore 和 Ceph OSD 守护进程的其他部分会尽力在此内存预算范围内运行。
注意，除了配置的缓存大小外，
OSD 本身也会消耗内存。
内存碎片和其他分配器开销会导致额外的内存使用。

配置的缓存内存预算可用于存储以下类型的\
内容：

* Key/Value 元数据（也就是 RocksDB 的内部缓存）
* BlueStore 元数据
* BlueStore 数据（就是最近读出或最近写入的对象数据）

缓存内存用量受以下选项控制：
``bluestore_cache_meta_ratio`` 和 ``bluestore_cache_kv_ratio`` 。
缓存中用于数据的部分受有效 BlueStore 缓存大小
（取决于 ``bluestore_cache_size[_ssd|_hdd]`` 选项\
和主设备的设备类别）以及 meta 比率和 kv 比率的影响。
数据部分的计算公式为： ``<effective_cache_size> * (1 -
bluestore_cache_meta_ratio - bluestore_cache_kv_ratio)`` 。

.. confval:: bluestore_cache_size
.. confval:: bluestore_cache_size_hdd
.. confval:: bluestore_cache_size_ssd
.. confval:: bluestore_cache_meta_ratio
.. confval:: bluestore_cache_kv_ratio


校验和
======
.. Checksums

BlueStore 会对写入磁盘的所有元数据、所有数据做校验和。
元数据校验和由 RocksDB 处理，使用 `crc32c` 算法；
而数据校验由 BlueStore 处理，可以使用 `crc32c` 、 `xxhash32` 或 `xxhash64` 。
不过， `crc32c` 是默认的校验和算法，适合大多数用途。

完整的数据校验和会增加元数据量， BlueStore 必须存储和管理它们。
只要有可能（例如，当客户端提示数据是按顺序写入和读出时），
BlueStore 就会对较大的数据块进行校验。但在许多情况下，
它必须为每 4 KB 数据块存储一个校验和结果
（通常为 4 字节）。

把校验和截断成 1 或 2 个字节可以得到一个较小的校验和数值，
这样可以削减元数据开销。这样做的缺点\
是增加了随机错误未被发现的概率：
32 位（ 4 字节）校验和的概率约为 40 亿分之一，
16 位（ 2 字节）校验和的概率约为 65536 分之一，
8 位（ 1 字节）校验和约为 256 分之一。要使用较小的校验和值，
选用 `crc32c_16` 或 `crc32c_8` 校验和算法。

可以通过每个存储池的 ``csum_type`` 配置选项或\
全局配置选项指定\ *校验和算法*\ 。例如：

.. prompt:: bash $

   ceph osd pool set <pool-name> csum_type <algorithm>

.. confval:: bluestore_csum_type


内联压缩
========
.. Inline Compression

BlueStore 支持内联压缩，可用 `snappy` 、 `zlib` 、 `lz4` 、 `zstd` 压缩算法。

BlueStore 中的数据是否压缩由两个因素决定： (1) *压缩模式*\ 和
(2) 与写入操作相关的客户端提示。压缩模式如下：

* **none**: 永不压缩数据；
* **passive**: 不压缩数据，
  除非写入操作设置了 *compressible* （可压缩）提示。
* **aggressive**: 总是压缩数据，
  除非写操作设置了 *incompressible* （不可压缩）提示。
* **force**: 压缩一切数据（忽视提示）。

有关 *compressible* 和 *incompressible* I/O 提示的更多信息，
参阅 :c:func:`rados_set_alloc_hint()` 。

注意，只有当数据块的尺寸减少得足够多时，才会压缩 BlueStore 中的数据
（由 ``bluestore compression required ratio`` 选项决定）。
无论使用的是哪种压缩模式，
如果数据块过大，就会被忽略，而是存储原始（未压缩的）数据。
例如，如果 ``bluestore compression required ratio`` 设置为 ``.7`` ，
那么只有当压缩数据的大小不超过原始数据大小的 70% 时，
才会存储压缩的数据。

*compression mode* 、 *compression algorithm* 、 *compression required ratio* 、
*min blob size* 和 *max blob size* 这些选项，
可以通过每个存储池的属性设置，也可以通过全局配置选项设置。
要指定存储池属性，执行以下命令：

.. prompt:: bash $

   ceph osd pool set <pool-name> compression_algorithm <algorithm>
   ceph osd pool set <pool-name> compression_mode <mode>
   ceph osd pool set <pool-name> compression_required_ratio <ratio>
   ceph osd pool set <pool-name> compression_min_blob_size <size>
   ceph osd pool set <pool-name> compression_max_blob_size <size>

.. confval:: bluestore_compression_algorithm
.. confval:: bluestore_compression_mode
.. confval:: bluestore_compression_required_ratio
.. confval:: bluestore_compression_min_blob_size
.. confval:: bluestore_compression_min_blob_size_hdd
.. confval:: bluestore_compression_min_blob_size_ssd
.. confval:: bluestore_compression_max_blob_size
.. confval:: bluestore_compression_max_blob_size_hdd
.. confval:: bluestore_compression_max_blob_size_ssd


.. _bluestore-rocksdb-sharding:

RocksDB 分片
============
.. RocksDB Sharding

BlueStore 内部使用了多种类型的键值数据，这些数据都存储在 RocksDB 中。
BlueStore 中的每种数据类型都分配了一个唯一前缀。
在 Pacific 版之前，所有键值数据都存储在单个 RocksDB 列族： default 。
从 Pacific 版开始， BlueStore 可以将这些数据拆分到多个 RocksDB 列族中。
键名相似时， BlueStore 就能实现更好的缓存和更精确的压缩，
特别是一些键的访问频率、修改频率和生命周期相似时更加如此。
这不仅提升了性能，而且压缩后需要的磁盘空间也更少，
因为每个列族都更小，可以独立地分别压缩。

用 Pacific 或更高版本部署的 OSD 默认使用 RocksDB 分片。
但是，如果 Ceph 是从以前的版本升级到 Pacific 或之后的版本，
那些 OSD 是用 Pacific 之前的版本创建的，
它们的分片功能都是关闭的。

要在指定 OSD 上启用分片并应用 Pacific 的默认值，先停止 OSD ，
并执行下列命令：

    .. prompt:: bash #

       ceph-bluestore-tool \
        --path <data path> \
        --sharding="m(3) p(3,0-12) O(3,0-13)=block_cache={type=binned_lru} L P" \
        reshard

.. confval:: bluestore_rocksdb_cf
.. confval:: bluestore_rocksdb_cfs


节流
====
.. Throttling

.. confval:: bluestore_throttle_bytes
.. confval:: bluestore_throttle_deferred_bytes
.. confval:: bluestore_throttle_cost_per_io
.. confval:: bluestore_throttle_cost_per_io_hdd
.. confval:: bluestore_throttle_cost_per_io_ssd


SPDK 用法
=========
.. SPDK Usage

如果你想让 NVMe 设备使用 SPDK 驱动，你得先配置好系统。详情见 `SPDK 文档`_\ 。

.. _SPDK 文档: http://www.spdk.io/doc/getting_started.html#getting_started_examples

SPDK 有个脚本可以自动配置设备，以 root 身份执行：

.. prompt:: bash $

   sudo src/spdk/scripts/setup.sh

你需要给 ``bluestore_block_path`` 指定 NVMe 设备的设备选择器，
它以 spdk: 为前缀。

例如，执行下列命令，先找出一个 Intel NVMe SSD 的设备选择器：

.. prompt:: bash $

   lspci -mm -n -d -d 8086:0953

设备选择器的格式是 ``DDDD:BB:DD.FF`` 或
``DDDD.BB.DD.FF`` 。

接下来，假设 ``lspci`` 命令输出中的设备选择器是 ``0000:01:00.0`` ，
可以用以下命令来指定设备选择器： ::

  bluestore_block_path = "spdk:trtype:pcie traddr:0000:01:00.0"

您也可以通过 TCP 传输指定一个远程 NVMeoF 目标，
如下例所示： ::

  bluestore_block_path = "spdk:trtype:tcp traddr:10.67.110.197 trsvcid:4420 subnqn:nqn.2019-02.io.spdk:cnode1"

要在每个节点运行多个 SPDK 实例，
必须让每个实例使用自己的 DPDK 内存，
方法是为每个实例指定它将使用的 DPDK 内存量（单位： MB ）。

在大多数情况下，单个设备可以同时用作数据、 DB 和 WAL 。
我们把这种策略描述为将这些组件扎堆放置。
务必做出以下设置，以确保所有 I/O 都通过 SPDK 发出： ::

  bluestore_block_db_path = ""
  bluestore_block_db_size = 0
  bluestore_block_wal_path = ""
  bluestore_block_wal_size = 0

如果没有做这些设置，那么当前的软件将会\
用内核文件系统符号填充 SPDK 映射文件，
并使用内核驱动程序发出 DB/WAL I/O。


最低分配尺寸
============
.. Minimum Allocation Size

BlueStore 在底层存储设备上分配存储空间时，有一个最低配置量。
实践中，这是一个对象会占用的最低容量，即使是一个很小的 RADOS 对象，
在每个 OSD 的主设备上也会占用这么大的容量。相关的配置选项 --
:confval:`bluestore_min_alloc_size` -- 的值来自 :confval:`bluestore_min_alloc_size_hdd`
或 :confval:`bluestore_min_alloc_size_ssd` 的值，
具体取决于 OSD 的 ``rotational`` 属性。因此，如果在 HDD 上创建 OSD ，
BlueStore 将使用 :confval:`bluestore_min_alloc_size_hdd` 当前的值进行初始化；
而对于 SSD OSD （包括 NVMe 设备）， Bluestore 将使用
:confval:`bluestore_min_alloc_size_ssd` 当前的值进行初始化。

在 Mimic 和早期版本中，旋转介质（ HDD ）的默认值为 64KB 、
非旋转介质（ SSD ）的默认值为 16KB 。
Octopus 版本将非旋转介质（SSD）的默认值改成了 4KB ，
Pacific 版将旋转介质（ HDD ）的默认值改成了 4KB 。

之所以做出这些更改，是因为 Ceph RADOS 网关 (RGW) 会托管海量小文件
（ S3/Swift 对象），而这些小文件会导致空间放大。

例如，当 RGW 客户端存入一个 1 KB 的 S3 对象时，
该对象会被写入一个单独的 RADOS 对象。
根据默认的 :confval:`min_alloc_size` 值，将给它分配 4 KB 的底层驱动器空间。
这意味着大约有 3 KB （即 4 KB 减去 1 KB）的空间分配了却没使用：
这相当于 300% 的额外开销或 25% 的效率。同样，
一个 5 KB 的用户对象将被存储为两个 RADOS 对象，即一个 4 KB 的 RADOS 对象\
和一个 1 KB 的 RADOS 对象，结果是它占用了 4KB 的设备容量。
不过，在这个案例中，额外开销百分比小多了。
可以把它当作是取模运算的余数。因此，
额外开销的\ *百分比*\ 会随着对象尺寸的增加而迅速降低。

还有一个容易被忽略的微妙之处：
刚才描述过的放大现象发生在\ *每一个*\ 副本中。例如，
使用默认的三副本 (3R) 配置时，一个 1 KB 的 S3 对象\
实际上会浪费大约 9 KB 的设备存储容量。
如果用的是纠删码（ EC ）而不是多副本，放大的幅度可能会更高：
对于一个 ``k=4, m=2`` 的存储池，会给 1 KB S3 对象会分配 24 KB
（即 4 KB 乘以 6）的设备容量。

当 RGW 桶存储池中包含许多相对较大的用户对象时，
这种现象的影响往往可以忽略不计。不过，
如果系统可能会存储很大一部分相对较小的用户对象时，
那就应该考虑这种影响。

默认值为 4KB 时，与传统的 HDD 和 SSD 设备非常吻合。
不过，如果在创建 OSD 时指定 :confval:`bluestore_min_alloc_size_ssd` ，
使之与设备的 IU （可能是 8KB 、 16KB 甚至 64KB ）匹配，
某些新型粗 IU （方向单元， Indirection Unit ） QLC SSD 的性能和磨损会达到最佳。
这些新型存储驱动器的读取性能可与传统的 TLC SSD 相媲美，
写入性能比 HDD 更快，
而且密度更高、成本比 TLC SSD 更低。

注意，在这些新型设备上创建 OSD 时，
必须小心，只给这些设备配置非默认值，
不能影响传统的 HDD 和 SSD 设备。通过仔细安排 OSD 创建顺序、
自定义 OSD 设备类别，特别是使用中央配置\ *掩码*\ ，
可以避免出现错误。

在 Quincy 及更高版本中，可以使用
:confval:`bluestore_use_optimal_io_size_for_min_alloc_size` 选项，
以便在创建每个 OSD 时自动发现正确的值。注意，
使用 ``bcache`` 、 ``OpenCAS`` 、 ``dmcrypt`` 、 ``ATA over Ethernet`` 、 `iSCSI`
或其他设备分层技术和抽象技术可能会影响正确值的确定。
此外，在 VMware 存储之上部署的 OSD
有时会报告与底层硬件不一致的 ``rotational`` 属性。

我们建议在启动时通过日志和管理套接字检查此类 OSD ，
以确保其行为正确。需要注意的是，
这种检查在旧内核上可能无法正常工作。要确认此问题，
应该检查 ``/sys/block/<drive>/queue/optimal_io_size`` 是否存在以及它的值。

.. note:: 在运行 Reef 或更高版本的 Ceph 时，
   可以用 ``ceph osd metadata`` 方便地查询每个 OSD 的 ``min_alloc_size`` 。

要查询某个 OSD ，执行下列命令：

.. prompt:: bash #

   ceph osd metadata osd.1701 | egrep rotational\|alloc

这种空间放大可能表现为 ``ceph df`` 报告的原始空间与所存储数据的比率异常高。
``ceph osd df`` 报告的 ``%USE`` / ``VAR`` 值与\
其他表面上看起来相同的 OSD 相比也可能异常地高。
最后， OSD 的 ``min_alloc_size`` 值不匹配时，
平衡器在使用它们的存储池中也可能出现意料之外的行为。

BlueStore 的这一属性\ *仅仅*\ 在创建 OSD 时有效；
如果后来更改了这个属性，这个特定 OSD 的行为也不会改变，还是保持原样，
除非销毁这个 OSD 并用适当的选项值重新部署。
Ceph 升级到更高版本后，也\ *不会*\ 更改 OSD 一直沿用的值，
它们是在旧版本或其他配置下部署的。

.. confval:: bluestore_min_alloc_size
.. confval:: bluestore_min_alloc_size_hdd
.. confval:: bluestore_min_alloc_size_ssd
.. confval:: bluestore_use_optimal_io_size_for_min_alloc_size


DSA （数据流加速器）的用法
==========================
.. DSA (Data Streaming Accelerator) Usage

如果要使用 DML 库驱动 DSA 设备，以卸载 BlueStore 中持久内存（ PMEM ）
上的读/写操作，那就需要安装 `DML`_ 和 `idxd-config`_ 库。
此做法仅适用于配备了 SPR (Sapphire Rapids) CPU 的计算机。

.. _dml: https://github.com/intel/dml
.. _idxd-config: https://github.com/intel/idxd-config

安装 DML 软件后，参照下面的 WQ 配置示例，
把共享工作队列 (WQs) 配置好：

.. prompt:: bash $

   accel-config config-wq --group-id=1 --mode=shared --wq-size=16 --threshold=15 --type=user --name="myapp1" --priority=10 --block-on-fault=1 dsa0/wq0.1
   accel-config config-engine dsa0/engine0.1 --group-id=1
   accel-config enable-device dsa0
   accel-config enable-wq dsa0/wq0.1
