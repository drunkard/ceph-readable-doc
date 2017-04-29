:orphan:

=======================================
 rbd-nbd -- 把 rbd 镜像映射为 nbd 设备
=======================================

.. program:: rbd-nbd

提纲
====

| **rbd-nbd** [-c conf] [--nbds_max *limit*] [--read-only] [--device *nbd device*] map *image-spec* | *snap-spec*
| **rbd-nbd** unmap *nbd device*
| **rbd-nbd** list-mapped


描述
====

**rbd-nbd** 是个 RADOS 块设备（ rbd ）映像的客户端，与 rbd 内核\
模块类似。它可以把一个 rbd 映像映射为 nbd （网络块设备）设备，这\
样就可以当常规的本地块设备使用了。


选项
====

.. option:: -c ceph.conf

   指定 ceph.conf 配置文件，而不是用默认的 /etc/ceph/ceph.conf 来确\
   定启动时需要的监视器。

.. option:: --nbds_max *limit*

   载入 NBD 内核模块时覆盖其参数，用于限制 nbd 设备数量。


映像名和快照名规则
==================

| *image-spec* is [*pool-name*]/*image-name*
| *snap-spec*  is [*pool-name*]/*image-name*\ @\ *snap-name*

*pool-name* 的默认值是 "rbd" 。如果映像名里包含字符串斜杠（ / ），\
那就必须指定 *pool-name* 。


使用范围
========

**rbd-nbd** 是 Ceph 的一部分，这是个伸缩力强、开源、分布式的\
存储系统，更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`rbd <rbd>`\(8)
