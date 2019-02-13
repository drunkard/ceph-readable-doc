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

**osdmaptool** 工具可用于创建、查看、修改 Ceph 分布式存储系统的
OSD 集群运行图。很显然，它可用于提取内嵌的 CRUSH 图，或者导入新
CRUSH 图。


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

   创建有 numosd 个设备的相对通用的 OSD 图。若指定了 --pgbits
   选项，每个 OSD 的归置组数量将是 bitsperosd 个位偏移。也就是
   pg_num 属性将被设置为 numosd 数值再右移 bitsperosd 位。

.. option:: --test-map-pgs [--pool poolid]

   打印出归置组到 OSD 的映射关系。

.. option:: --test-map-pgs-dump [--pool poolid]

   打印出所有归置组及其与 OSD 映射关系的汇总。


实例
====

要创建个有 16 个设备的简易图： ::

        osdmaptool --createsimple 16 osdmap --clobber

查看结果： ::

        osdmaptool --print osdmap

要查看存储池 0 的归置组映射情况： ::

        osdmaptool --test-map-pgs-dump rbd --pool 0

        pool 0 pg_num 8
        0.0     [0,2,1] 0
        0.1     [2,0,1] 2
        0.2     [0,1,2] 0
        0.3     [2,0,1] 2
        0.4     [0,2,1] 0
        0.5     [0,2,1] 0
        0.6     [0,1,2] 0
        0.7     [1,0,2] 1
        #osd    count   first   primary c wt    wt
        osd.0   8       5       5       1       1
        osd.1   8       1       1       1       1
        osd.2   8       2       2       1       1
         in 3
         avg 8 stddev 0 (0x) (expected 2.3094 0.288675x))
         min osd.0 8
         max osd.0 8
        size 0  0
        size 1  0
        size 2  0
        size 3  8

在上面的输出结果中，
 #. 存储池 0 有 8 个归置组，及后面的两张表：
 #. 一张表是归置组。每行表示一个归置组，列分别是：

    * 归置组 id ，
    * acting set ，和
    * 主 OSD 。
 #. 一张表是所有的 OSD 。每行表示一个 OSD ，列分别是：

    * 映射到此 OSD 的归置组数量，
    * 此 OSD 是它所属 acting set 的第一个，这样的归置组数量，
    * 此 OSD 是归置组的主 OSD ，这样的归置组数量，
    * 此 OSD 的 CRUSH 权重，还有
    * 此 OSD 的权重。
 #. 再看是托管着归置组的 OSD 数量，是 3 个。接下来是

    * avarge, stddev （标准偏差）, stddev/average, expected stddev, expected stddev / average
    * min and max
 #. 映射到 n 个 OSD 的归置组数量。在本例中，全部的 8 个归置组\
    都映射到了 3 个不同的 OSD 。

在一个均衡得不太好的集群中，我们也许会看到类似如下的归置组分布\
统计，其标准偏差是 1.41421 : ::

        #osd    count   first   primary c wt    wt
        osd.0   8       5       5       1       1
        osd.1   8       1       1       1       1
        osd.2   8       2       2       1       1

        #osd    count   first    primary c wt    wt
        osd.0   33      9        9       0.0145874     1
        osd.1   34      14       14      0.0145874     1
        osd.2   31      7        7       0.0145874     1
        osd.3   31      13       13      0.0145874     1
        osd.4   30      14       14      0.0145874     1
        osd.5   33      7        7       0.0145874     1
         in 6
         avg 32 stddev 1.41421 (0.0441942x) (expected 5.16398 0.161374x))
         min osd.4 30
         max osd.1 34
        size 00
        size 10
        size 20
        size 364


使用范围
========

**osdmaptool** 是 Ceph 的一部分，这是个伸缩力强、开源、分布式的存储系统，\
更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph <ceph>`\(8),
:doc:`crushtool <crushtool>`\(8),
