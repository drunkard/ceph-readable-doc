==========
 存储设备
==========

在一套存储集群中有几种 Ceph 守护进程：

.. _rados_configuration_storage-devices_ceph_osd:

* **Ceph OSDs** （对象存储守护进程）存储着 Ceph 里的大多数数据。
  通常，每个 OSD 后面都有一个独立的存储设备，
  可以是传统硬盘（ HDD ）或者是固态硬盘（ SSD ）。
  OSD 后端也可以是设备的组合：例如，
  一个硬盘存储大多数数据、一个 SSD （或者一个 SSD 分区）存储一些元数据。
  一套集群里的 OSD 数量通常是一个函数，
  参数是要存储的数据量、各个存储设备的容量、
  还有冗余的级别和类型（多副本或是纠删码）。
* **Ceph Monitor （监视器）**
  守护进程管理着集群的关键状态。
  这其中有集群成员和认证信息。
  小集群仅需要几个 GB 的存储空间来保存监视器数据库；
  然而，在大型集群上，监视器数据库的尺寸\
  能达到数十至数百 GB 。
* **Ceph Manager （管理器）** 守护进程随监视器守护进程一起运行，
  向外部监控和管理系统们\
  提供了另外一套监控手段及其接口。

.. _rados_config_storage_devices_osd_backends:

OSD 后端
========
.. OSD Backends

OSD 有两种管理它们存储着的数据的方法。
从 Luminous 12.2.z 版起，新的默认（也是推荐的）后端是 *BlueStore* ；
在 Luminous 之前，默认的（且是唯一选项）是 *FileStore* 。

.. _rados_config_storage_devices_bluestore:

BlueStore
---------

BlueStore 是个专用存储后端，是为 Ceph OSD 工作载荷专门设计的，
用于管理磁盘上的数据。 BlueStore 是在支持和管理 FileStore OSD
十年之久的基础上设计的。

BlueStore 的关键功能包括：

* 直接管理存储设备。 BlueStore 使用的是原始块设备或分区。
  这样可以避免抽象的中间层（如类似 XFS 的本地文件系统），
  它们会降低性能、或增加复杂性。
* 用 RocksDB 管理元数据。 RocksDB 的键值数据库是嵌入式的，
  可以用于管理内部元数据，包括对象名到磁盘上数据块位置\
  的映射情况。
* 完整的数据及其元数据校验和。默认情况下，
  写入 BlueStore 的所有数据和元数据都会被一个或多个校验和保护起来。
  数据或元数据从磁盘读出来，
  没有校验不会返回给用户。
* 内联压缩。数据写入磁盘前可以\
  选择性地压缩。
* 元数据在多个设备上分级存储。
  BlueStore 允许它的内部日志（预写日志）写到一个单独的、
  高速设备（像 SSD 、 NVMe 、或 NVDIMM ）上，以提升性能。
  如果高速存储的数量足够，
  内部元数据可以存储在高速设备上。
* 高效的写时复制（ copy-on-write ）。 RBD 和 CephFS 快照功能全靠
  BlueStore 实现的一种高效的写时复制 *克隆（ clone ）* 机制。
  这就产生了高效的 I/O ，对于常规快照和纠删码存储池
  （它仰仗克隆功能实现高效的二阶段提交）都是如此。

更多信息请看 :doc:`bluestore-config-ref` 和
:doc:`/rados/operations/bluestore-migration` 。


FileStore
---------
.. warning:: Filestore has been deprecated in the Reef release and is no longer supported.

FileStore 是 Ceph 存储对象的老方法。
它依赖于标准文件系统（通常是 XFS ）和\
用于元数据的键值数据库（以前是 LevelDB ，现在是 RocksDB ）
组合。

FileStore 久经考验而且在生产环境下广泛应用。
然而，由于其总体设计问题，遇到了很多性能不足，
以及用传统文件系统存储对象数据时的可靠性。

虽说 FileStore 可以利用大多数兼容 POSIX 的文件系统
（包括 btrfs 和 ext4 ），我们还是只推荐 Ceph 搭配 XFS 。
btrfs 和 ext4 都有已知的缺陷和不足，
使用它们可能会导致数据丢失。默认情况下，
Ceph 的所有部署工具都用 XFS 。

更多信息请看 :doc:`filestore-config-ref` 。
