:orphan:

=======================================
 rbd-nbd -- 把 rbd 镜像映射为 nbd 设备
=======================================

.. program:: rbd-nbd

提纲
====

| **rbd-nbd** [-c conf] [--read-only] [--device *nbd device*] [--nbds_max *limit*] [--max_part *limit*] [--exclusive] [--notrim] [--encryption-format *format*] [--encryption-passphrase-file *passphrase-file*] [--io-timeout *seconds*] [--reattach-timeout *seconds*] map *image-spec* | *snap-spec*
| **rbd-nbd** unmap *nbd device* | *image-spec* | *snap-spec*
| **rbd-nbd** list-mapped
| **rbd-nbd** attach --device *nbd device* *image-spec* | *snap-spec*
| **rbd-nbd** detach *nbd device* | *image-spec* | *snap-spec*


描述
====

**rbd-nbd** 是个 RADOS 块设备（ rbd ）映像的客户端，与 rbd
内核模块类似。它可以把一个 rbd 映像映射为 nbd （网络块设备）\
设备，这样就可以当常规的本地块设备使用了。


选项
====

.. option:: -c ceph.conf

   指定 ceph.conf 配置文件，而不是用默认的 /etc/ceph/ceph.conf
   来确定启动时需要的监视器。

.. option:: --read-only

   以只读方式映射。

.. option:: --nbds_max *limit*

   modprobe 载入 NBD 内核模块时覆盖其 nbds_max 参数，\
   用于限制 nbd 设备数量。

.. option:: --max_part *limit*

   覆盖（内核的）模块参数 max_part 。

.. option:: --exclusive

   禁止其它客户端写入。

.. option:: --notrim

   Turn off trim/discard.

.. option:: --encryption-format

   Image encryption format.
   Possible values: *luks1*, *luks2*

.. option:: --encryption-passphrase-file

   Path of file containing a passphrase for unlocking image encryption.

.. option:: --io-timeout *seconds*

   会覆盖设备超时值。 Linux 内核请求的默认超时时间是 30 秒。\
   这个可选参数允许你另外指定超时时长。

.. option:: --reattach-timeout *seconds*

   Specify timeout for the kernel to wait for a new rbd-nbd process is
   attached after the old process is detached. The default is 30
   second.


映像名和快照名规则
==================
.. Image and snap specs

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
