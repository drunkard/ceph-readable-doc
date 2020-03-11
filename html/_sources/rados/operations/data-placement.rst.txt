.. Data Placement Overview

==============
 数据归置概览
==============

Ceph 通过 RADOS 集群动态地存储、复制和重新均衡数据对象。它可以\
让很多不同用户把对象存储到无数 OSD 之上的不同的存储池里，以实\
现各自的目标，所以 Ceph 的运营需要些数据归置计划。 Ceph 的数据\
归置计划概念主要有：

- **存储池（ Pool ）：** Ceph 在存储池内存储数据，它是对象存储\
  的逻辑组；存储池管理着归置组数量、副本数量、和存储池的
  CRUSH 规则。要往存储池里存数据，用户必须通过认证、且权限\
  合适，存储池可做快照。详情参见\ `存储池`_\ 。

- **归置组（ Placement Group ）：** Ceph 把对象映射到归置组（
  PG ），归置组是一逻辑对象池的片段，这些对象组团后再存入到
  OSD 。归置组减少了各对象存入对应 OSD 时的元数据数量，更多的\
  归置组（如每 OSD 100 个）使得均衡更好。详情见\ `归置组`_\ 。

- **CRUSH 图（ CRUSH Map ）：** CRUSH 是重要组件，它使 Ceph 能\
  伸缩自如而没有性能瓶颈、没有扩展限制、没有单点故障，它为
  CRUSH 算法提供集群的物理拓扑，以此确定一个对象的数据及它的副\
  本应该在哪里、怎样跨故障域存储，以提升数据安全。详情见 \
  `CRUSH 图`_\ 。

- **均化器：** The balancer is a feature that will automatically optimize the
  distribution of PGs across devices to achieve a balanced data distribution,
  maximizing the amount of data that can be stored in the cluster and evenly
  distributing the workload across OSDs.

起初安装测试集群的时候，可以使用默认值。但开始规划一个大型 Ceph
集群，做数据归置操作的时候会涉及存储池、归置组、和 CRUSH 。


.. _存储池: ../pools
.. _归置组: ../placement-groups
.. _CRUSH 图: ../crush-map
.. _均衡器: ../balancer
