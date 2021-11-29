==========
 存储设备
==========
.. Storage Devices

有两种 Ceph 守护进程会在硬盘上存储数据：

* **Ceph OSDs** (Object Storage Daemons) store most of the data
  in Ceph. Usually each OSD is backed by a single storage device.
  This can be a traditional hard disk (HDD) or a solid state disk
  (SSD). OSDs can also be backed by a combination of devices: for
  example, a HDD for most data and an SSD (or partition of an
  SSD) for some metadata. The number of OSDs in a cluster is
  usually a function of the amount of data to be stored, the size
  of each storage device, and the level and type of redundancy
  specified (replication or erasure coding).
* **Ceph Monitor** daemons manage critical cluster state. This
  includes cluster membership and authentication information.
  Small clusters require only a few gigabytes of storage to hold
  the monitor database. In large clusters, however, the monitor
  database can reach sizes of tens of gigabytes to hundreds of
  gigabytes.  
* **Ceph Manager** daemons run alongside monitor daemons, providing
  additional monitoring and providing interfaces to external
  monitoring and management systems.


OSD 后端
========
.. OSD Backends

OSD 有两种管理它们存储着的数据的方法。从 Luminous 12.2.z 版\
起，新的默认（也是推荐的）后端是 *BlueStore* ；在 Luminous 之\
前，默认的（且是唯一选项）是 *FileStore* 。

BlueStore
---------

BlueStore is a special-purpose storage backend designed specifically for
managing data on disk for Ceph OSD workloads.  BlueStore's design is based on
a decade of experience of supporting and managing Filestore OSDs. 

Key BlueStore features include:

* Direct management of storage devices. BlueStore consumes raw block devices or
  partitions. This avoids intervening layers of abstraction (such as local file
  systems like XFS) that can limit performance or add complexity.
* Metadata management with RocksDB. RocksDB's key/value database is embedded
  in order to manage internal metadata, including the mapping of object
  names to block locations on disk.
* Full data and metadata checksumming. By default, all data and
  metadata written to BlueStore is protected by one or more
  checksums. No data or metadata is read from disk or returned
  to the user without being verified.
* Inline compression.  Data can be optionally compressed before being written
  to disk.
* Multi-device metadata tiering. BlueStore allows its internal
  journal (write-ahead log) to be written to a separate, high-speed
  device (like an SSD, NVMe, or NVDIMM) for increased performance.  If
  a significant amount of faster storage is available, internal
  metadata can be stored on the faster device.
* Efficient copy-on-write. RBD and CephFS snapshots rely on a
  copy-on-write *clone* mechanism that is implemented efficiently in
  BlueStore. This results in efficient I/O both for regular snapshots
  and for erasure-coded pools (which rely on cloning to implement
  efficient two-phase commits).

更多信息请看 :doc:`bluestore-config-ref` 和
:doc:`/rados/operations/bluestore-migration` 。


FileStore
---------

FileStore is the legacy approach to storing objects in Ceph. It
relies on a standard file system (normally XFS) in combination with a
key/value database (traditionally LevelDB, now RocksDB) for some
metadata.

FileStore is well-tested and widely used in production. However, it
suffers from many performance deficiencies due to its overall design
and its reliance on a traditional file system for object data storage.

Although FileStore is capable of functioning on most POSIX-compatible
file systems (including btrfs and ext4), we recommend that only the
XFS file system be used with Ceph. Both btrfs and ext4 have known bugs and
deficiencies and their use may lead to data loss. By default, all Ceph
provisioning tools use XFS.

更多信息请看 :doc:`filestore-config-ref` 。
