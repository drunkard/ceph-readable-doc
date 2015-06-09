:orphan:

===============================
 crushtool -- CRUSH 图操作工具
===============================

.. program:: crushtool

提纲
====

| **crushtool** ( -d *map* | -c *map.txt* | --build --num_osds *numosds*
  *layer1* *...* | --test ) [ -o *outfile* ]


描述
====

**crushtool** 是 CRUSH 图处理工具，它允许你创建、编译、反编译和测试 CRUSH 图。

CRUSH 是个伪随机数据分布算法，它能高效地把输入值（通常是数据对象）映射到异\
构、结构化的分级设备图中。此算法最初是在下面的论文（已改进过了）中描述的：

       http://www.ssrc.ucsc.edu/Papers/weil-sc06.pdf

此工具有四种操作模式。

.. option:: --compile|-c map.txt

   把纯文本 map.txt 编译为二进制图文件。

.. option:: --decompile|-d map

   接受已编译的图，并把它反编译为适合编辑的纯文本源文件。

.. option:: --build --num_osds {num-osds} layer1 ...

   将用指定分级结构创建图。详细解释见下文。

.. option:: --test

   在一系列输入对象名上试运行 CRUSH 映射。详细解释见下文。

不像其它 Ceph 工具， **crushtool** 不接受命令行输入的像 **--debug-crush** 这\
样的通用选项；却可以通过 CEPH_ARGS 环境变量提供。例如，可以这样压制 CRUSH 子\
系统的所有输出： ::

	CEPH_ARGS="--debug-crush 0" crushtool ...


用 --test 进行测试
==================

测试模式会采用指定的 crush 图（用 **-i map** 指定的），并试运行 CRUSH 映射\
或随机放置（若设置了 **--simulate** ）。完成后可创建两种报告类型：
1) **--show-...** 选项会把人类可读的信息输出到标准错误；
2) **--output-csv** 选项会创建 CSV 文件，具体文档可在 **--help-output** 选\
项中见到。

.. option:: --show-statistics

   对每条规则都显示各对象的映射。例如： ::

       CRUSH rule 1 x 24 [11,6]

   表明对象 **24** 被规则 **1** 映射到了设备 **[11,6]** 。在映射详情的末尾，\
   显示了一个分布情况汇总，例如： ::

       rule 1 (metadata) num_rep 5 result size == 5:	1024/1024

   以上输出表明，规则 **1** 名字为 **metadata** ，在试图把 **1024** 个对象映\
   射为 **num_rep 5** 个副本时，最终映射到了 **result size == 5** 个设备。当\
   没能达到要求的映射数量时（假设已\ **尝试**\ 了规定次数），就会显示失败情\
   况。比如： ::

       rule 1 (metadata) num_rep 10 result size == 8:	4/1024
       rule 1 (metadata) num_rep 10 result size == 9:	93/1024
       rule 1 (metadata) num_rep 10 result size == 10:	927/1024

   以上表明，虽然要求的副本数是 **num_rep 10** ，但是 **1024** 个对象中有 \
   **4** 个（ **4/1024** ）只映射到了 **result size == 8** 个设备。

.. option:: --show-bad-mappings

   查看哪些对象的映射数量没达到要求，例如： ::

       bad mapping rule 1 x 781 num_rep 7 result [8,10,2,11,6,9]

   表明规则 **1** 要求映射到 **7** 个设备，实际上只映射了六个： \
   **[8,10,2,11,6,9]** 。

.. option:: --show-utilization

   显示各设备的期望和实际利用率，各种数量的副本也计算在内。例如： ::

     device 0: stored : 951      expected : 853.333
     device 1: stored : 963      expected : 853.333
     ...

   表明设备 **0** 实际存储了 **951** 个对象，本来期望存储 **853** 个。隐含了 \
   **--show-statistics** 。

.. option:: --show-utilization-all

   显示结果与 **--show-utilization** 相同，只是不剔除权重为 0 的设备。隐含了 \
   **--show-statistics** 。

.. option:: --show-choose-tries

   显示要尝试多少次才能映射到设备。例如： ::

      0:     95224
      1:      3745
      2:      2225
      ..

   表明有 **95224** 次映射没重试就成功了， **3745** 次映射尝试一次后成功，等\
   等。显示的最大行数与 **--set-choose-total-tries** 选项相同。

.. option:: --output-csv

   在当前目录内创建 CSV 文件用于保存输出信息，具体请参考 **--help-output** 。\
   文件被命名为收集统计信息时涉及的规则，比如使用了 metadata 规则时， CSV 文\
   件将会是： ::

      metadata-absolute_weights.csv
      metadata-device_utilization.csv
      ...

   文件的首行是本列的简单描述，例如： ::

      metadata-absolute_weights.csv
      Device ID, Absolute Weight
      0,1
      ...

.. option:: --output-name NAME

   用了 **--output-csv** 选项时生成的文件名要加 **NAME** 前缀，例如 \
   **--output-name FOO** 将创建这些文件： ::

      FOO-metadata-absolute_weights.csv
      FOO-metadata-device_utilization.csv
      ...

用 **--set-...** 选项可修改指定 crush 图内的可调值，在内存中修改。例如： ::

      $ crushtool -i mymap --test --show-bad-mappings
      bad mapping rule 1 x 781 num_rep 7 result [8,10,2,11,6,9]

上面的问题可通过增加 **choose-total-tries** 来修正，如： ::

      $ crushtool -i mymap --test \
          --show-bad-mappings \
          --set-choose-total-tries 500


用 ``--build`` 构建新图
=======================

构建模式可生成一个分级图。第一个参数指定了 CRUSH 分级结构中的设备（叶子）数\
量。每一层都要描述如何分组前一层（或设备）。

各层都由如下要素组成： ::

       bucket ( uniform | list | tree | straw ) size

这里的 **bucket** 是本层桶的类型（如 "rack" ）。构建时各桶名 **bucket** 后将\
追加一个惟一的数字（如 "rack0" 、 "rack1" ……）。

第二个组件是桶类型：大多用 **straw** 。

第三个组件是此桶的最大尺寸，为零时表示容量无限。


实例
====

假设我们有 2 行、每行有 2 个机架、每机架有 20 个节点、每个节点有 4 个存储设\
备用于 OSD 守护进程，这样的配置允许部署 320 个 OSD 守护进程。这里按照机架高 \
42U ，节点都是 2U 高的，另外空余 2U 装机架交换机。

要如实反映我们的设备、节点、机架、行构成的分级结构，要用此命令： ::

    $ crushtool -o crushmap --build --num_osds 320 \
           node straw 4 \
           rack straw 20 \
           row straw 2 \
           root straw 0
    # id	weight	type name	reweight
    -87	320	root root
    -85	160		row row0
    -81	80			rack rack0
    -1	4				node node0
    0	1					osd.0	1
    1	1					osd.1	1
    2	1					osd.2	1
    3	1					osd.3	1
    -2	4				node node1
    4	1					osd.4	1
    5	1					osd.5	1
    ...

这样就创建了 CRUSH 规则集，以便测试。此规则集与创建集群时默认创建的规则集相\
同，可用下面的方法编辑它们： ::

       # 反编译
       crushtool -d crushmap -o map.txt

       # 编辑
       emacs map.txt

       # 重新编译
       crushtool -c map.txt -o crushmap


使用范围
========

**crushtool** 是 Ceph 的一部分，这是个伸缩力强、开源、分布式的存储系统，\
更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph <ceph>`\(8),
:doc:`osdmaptool <osdmaptool>`\(8),


作者
====

John Wilkins, Sage Weil, Loic Dachary
