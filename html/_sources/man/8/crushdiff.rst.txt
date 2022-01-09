:orphan:

.. _crushdiff:

====================================
 crushdiff -- ceph crush 图测试工具
====================================

.. program:: crushdiff

提纲
====

| **crushdiff** [ --osdmap *osdmap* ] [ --pg-dump *pg-dump* ]
  [ --compiled ] [ --verbose ] *command* *crushmap*


描述
====

**crushdiff** 是让我们测试 crushmap 更改
（ PG 数、挪动的对象和字节数）效果的工具。
这是 :doc:`osdmaptool <osdmaptool>`\(8) 的一个包装，
用它的 **--test-map-pgs-dump** 选项获取更改过的 PG 列表。
另外，它用 pg 统计功能计算要挪动的对象和字节数。

默认情况下， **crushdiff** 会使用集群的当前 osdmap 和 pg 统计信息，
因此需要集群的访问权限。虽然你也可以\
用 **--osdmap** 和 **--pg-dump** 选项来测试前面获取到的数据。


选项
====

.. option:: --compiled

   用来指示输入、输出的 crushmap 是编译过的。
   如果没指定此选项，想要的、返回的 crushmap 就是
   txt （反编译过的）格式的。

.. option:: --pg-dump <pg-dump>

   **ceph pg dump** 的 JSON 格式输出。如果你没指定，
   **crushdiff** 会尝试自己调用命令获取数据。

.. option:: --osdmap <osdmap>

   就是集群的 osdmap ，通过 **ceph osd getmap** 命令获取到的。
   如果没指定此选项， **crushdiff** 会尝试自己调用命令\
   获取数据。

.. option:: --verbose

   输出诊断信息。

命令
====

:command:`compare` *crushmap*
  比较 *crushmap* 文件里的和集群 osdmap 里的 crushmap 。
  其输出将会展示期望的 pg 数、安装新 crushmap 后\
  需要挪动的对象数和字节数。

:command:`export` *crushmap*
  把集群 osdmap 里的 crushmap 导出到 *crushmap* 文件。

:command:`import` *crushmap*
  把 *crushmap* 文件里的 crushmap 导入到集群的 osdmap 里。


实例
====

获取当前的 crushmap::

        crushdiff export cm.txt

编辑此图::

        $EDITOR cm.txt

检查结果::

        crushdiff compare cm.txt

        79/416 (18.99%) pgs affected
        281/1392 (20.19%) objects affected
        80/1248 (6.41%) pg shards to move
        281/4176 (6.73%) pg object shards to move
        730.52Mi/10.55Gi (6.76%) bytes to move

运行时如果加了 **--verbose** 选项，
输出内容里也会包含受影响 PG 的详细信息，如下： ::

        4.3	[0, 2, 1] -> [1, 4, 2]
        4.b	[0, 1, 3] -> [2, 1, 3]
        4.c	[4, 0, 1] -> [4, 1, 2]

即，一个 PG 号、及其 osd 的新旧 active set 对比。

如果对结果满意就安装更新过的图： ::

        crushdiff import cm.txt


使用范围
========

**crushdiff** 是 Ceph 的一部分，这是个伸缩力强、开源、
分布式的存储系统，更多信息参见 https://docs.ceph.com 。


参考
====

:doc:`ceph <ceph>`\(8),
:doc:`crushtool <crushtool>`\(8),
:doc:`osdmaptool <osdmaptool>`\(8),
