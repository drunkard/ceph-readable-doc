:orphan:

==================================================
 rbd-ggate -- 通过 FreeBSD GEOM 网关映射 rbd 映像
==================================================

.. program:: rbd-ggate

提纲
====

| **rbd-ggate** [--read-only] [--exclusive] [--device *ggate device*] map *image-spec* | *snap-spec*
| **rbd-ggate** unmap *ggate device*
| **rbd-ggate** list

描述
====

**rbd-ggate** 是一个 RADOS 块设备（ rbd ）映像的客户端。它会把\
一个 rbd 映像映射到 ggate （ FreeBSD GEOM 网关类）设备，这样就\
可以像访问普通的本地块设备一样访问它了。


命令
====

map
---

派生一个进程用于创建 ggate 设备，然后在 GEOM 网关内核子系统和
RADOS 之间转发 I/O 请求。

unmap
-----

销毁 ggate 设备，并结束负责它的进程。

list
----

罗列已映射的 ggate 设备。


选项
====

.. option:: --device *ggate device*

   指定 ggate 设备路径。

.. option:: --read-only

   以只读方式映射。

.. option:: --exclusive

   禁止其它客户端写入。


映像和快照格式
==============

| *image-spec* is [*pool-name*]/*image-name*
| *snap-spec*  is [*pool-name*]/*image-name*\ @\ *snap-name*

*pool-name* 默认为 rbd 。如果一个映像名内包含斜杠（ / ），就\
必须加 *pool-name* 。


使用范围
========

**rbd-ggate** 是 Ceph 的一部分，这是个伸缩力强、开源、分布式\
的存储系统，更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`rbd <rbd>`\(8)
:doc:`ceph <ceph>`\(8)
