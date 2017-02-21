:orphan:

=======================================
 osdmaptool -- ceph osd 运行图操作工具
=======================================

.. program:: osdmaptool

提纲
====

| **osdmaptool** *mapfilename* [--print] [--createsimple *numosd*
  [--pgbits *bitsperosd* ] ] [--clobber]


描述
====

**osdmaptool** 工具可用于创建、查看、修改 Ceph 分布式存储系统的 OSD 集群运行\
图。很显然，它可用于提取内嵌的 CRUSH 图，或者导入新 CRUSH 图。


选项
====

.. option:: --print

   修改完成后，打印此图的一份纯文本转储。

.. option:: --clobber

   修改时允许 osdmaptool 覆盖 mapfilename 。

.. option:: --import-crush mapfile

   从 mapfile 载入 CRUSH 图并把它嵌入 OSD 图。

.. option:: --export-crush mapfile

   从 OSD 图提取出 CRUSH 图并写入 mapfile 。

.. option:: --createsimple numosd [--pgbits bitsperosd]

   创建有 numosd 个设备的相对通用的 OSD 图。若指定了 --pgbits 选项，每个 OSD \
   的归置组数量将是 bitsperosd 个位偏移。也就是 pg_num 属性将被设置为 numosd \
   数值再右移 bitsperosd 位。


实例
====

要创建个有 16 个设备的简易图： ::

        osdmaptool --createsimple 16 osdmap --clobber

查看结果： ::

        osdmaptool --print osdmap


使用范围
========

**osdmaptool** 是 Ceph 的一部分，这是个伸缩力强、开源、分布式的存储系统，\
更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph <ceph>`\(8),
:doc:`crushtool <crushtool>`\(8),
