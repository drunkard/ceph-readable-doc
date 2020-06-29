.. Supported Features of the Kernel Driver

内核驱动支持的功能
==================
The kernel driver is developed separately from the core ceph code, and as
such it sometimes differs from the FUSE driver in feature implementation.
The following details the implementation status of various CephFS features
in the kernel driver.

.. Inline data

内联数据
--------
Inline data was introduced by the Firefly release. This feature is being
deprecated in mainline CephFS, and may be removed from a future kernel
release.

Linux kernel clients >= 3.19 can read inline data and convert existing
inline data to RADOS objects when file data is modified. At present,
Linux kernel clients do not store file data as inline data.

See `Experimental Features`_ for more information.

.. Quotas

配额
----
Quota was first introduced by the hammer release. Quota disk format got renewed
by the Mimic release. Linux kernel clients >= 4.17 can support the new format
quota. At present, no Linux kernel client support the old format quota.

See `Quotas`_ for more information.

.. Multiple file systems within a Ceph cluster

单个 Ceph 集群内多个文件系统
----------------------------
The feature was introduced by the Jewel release. Linux kernel clients >= 4.7
can support it.

See `Experimental Features`_ for more information.

.. Multiple active metadata servers

多活元数据服务器
----------------
The feature has been supported since the Luminous release. It is recommended to
use Linux kernel clients >= 4.14 when there are multiple active MDS.

.. Snapshots

快照
----
The feature has been supported since the Mimic release. It is recommended to
use Linux kernel clients >= 4.17 if snapshot is used.

.. _Experimental Features: ../experimental-features
.. _Quotas: ../quota
