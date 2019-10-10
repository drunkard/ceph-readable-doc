:orphan:

===========================================
 ceph-bluestore-tool -- bluestore 管理工具
===========================================

.. program:: ceph-bluestore-tool

提纲
====

| **ceph-bluestore-tool** *command*
  [ --dev *device* ... ]
  [ --path *osd path* ]
  [ --out-dir *dir* ]
  [ --log-file | -l *filename* ]
  [ --deep ]
| **ceph-bluestore-tool** fsck|repair --path *osd path* [ --deep ]
| **ceph-bluestore-tool** show-label --dev *device* ...
| **ceph-bluestore-tool** prime-osd-dir --dev *device* --path *osd path*
| **ceph-bluestore-tool** bluefs-export --path *osd path* --out-dir *dir*
| **ceph-bluestore-tool** bluefs-bdev-new-wal --path *osd path* --dev-target *new-device*
| **ceph-bluestore-tool** bluefs-bdev-new-db --path *osd path* --dev-target *new-device*
| **ceph-bluestore-tool** bluefs-bdev-migrate --path *osd path* --dev-target *new-device* --devs-source *device1* [--devs-source *device2*]
| **ceph-bluestore-tool** free-dump|free-score --path *osd path* [ --allocator block/bluefs-wal/bluefs-db/bluefs-slow ]


描述
====

**ceph-bluestore-tool** 工具可在 BlueStore 例程上执行底层\
管理操作。


命令
====

:command:`help`

   显示帮助

:command:`fsck` [ --deep ]

   对 BlueStore 元数据进行一致性检查。如果加了 *--deep* ，\
   还会读取所有对象数据并核对校验和。

:command:`repair`

   运行一致性检查，\ *并且*\ 修复所有可修复的错误。

:command:`bluefs-export`

   把 BlueFS 内容（即 rocksdb 文件）导出到一个输出目录。

:command:`bluefs-bdev-sizes` --path *osd path*

   打印出设备尺寸，即 BlueFS 所认为的尺寸。

:command:`bluefs-bdev-expand` --path *osd path*

   让 BlueFS 检查它的块设备尺寸，而且，如果发现它们扩大了，把\
   那些额外空间也用起来。

:command:`bluefs-bdev-new-wal` --path *osd path* --dev-target *new-device*

   给 BlueFS 增加 WAL 设备，如果已有 WAL 设备此命令就会失败。

:command:`bluefs-bdev-new-db` --path *osd path* --dev-target *new-device*

   给 BlueFS 增加 DB 设备，如果已有 DB 设备此命令就会失败。
   
:command:`bluefs-bdev-migrate` --dev-target *new-device* --devs-source *device1* [--devs-source *device2*]

   把一个或多个源设备上的 BlueFS 数据移动到目标设备，成功后\
   源设备（除了主要的那个）将被删除。目标设备可以是已加入集群\
   的、或新设备。稍后，它将被加进 OSD ，以替换某一个源设备。\
   遵循下面的替换规则（按优先级，匹配到即停止）：

      - 如果源列表中有 DB 卷——目标设备替换它；
      - 如果源列表中有 WAL 卷——目标设备替换它；
      - 如果源列表中只有慢速卷——操作不允许，要显式地用
        new-db 、 new-wal 命令分配。

:command:`show-label` --dev *device* [...]

   出示设备标签。

:command:`free-dump` --path *osd path* [ --allocator block/bluefs-wal/bluefs-db/bluefs-slow ]

   展示分配器中的所有空闲区域。

:command:`free-score` --path *osd path* [ --allocator block/bluefs-wal/bluefs-db/bluefs-slow ]

   会收到一个 0-1 之间的数字，用于表示分配器中碎片的质量。\
   0 表示所有空闲空间都在一个块中的情形； 1 表示最糟糕的\
   碎片散布情形。


选项
====

.. option:: --dev *device*

   把设备 *device* 加进涉及到的设备列表中。

.. option:: --devs-source *device*

   把设备 *device* 加进迁移操作涉及到的源设备列表中。

.. option:: --dev-target *device*

   指定用于迁移操作或新增设备的目标设备 *device* ，以便新增
   DB/WAL 。

.. option:: --path *osd path*

   指定一个 osd 路径。大多数情况下，设备列表都是从 *osd path*
   里的符号链接推断出来的。通常比显式地用 --dev 指定几个设备\
   要简单些。

.. option:: --out-dir *dir*

   bluefs-export 的输出目录。

.. option:: -l, --log-file *log file*

   记录日志的文件

.. option:: --log-level *num*

   调试日志级别。默认是 30 （极其详细）， 20 是非常详细，
   10 是详细， 而 1 是不怎么详细。

.. option:: --deep

   深度洗刷、修复（读取并校验对象数据，而不只是元数据）

.. option:: --allocator *name*

   适用于 *free-dump* 和 *free-score* 操作。选择分配器。


设备标签
========

每个 BlueStore 块设备都有一个单独的块标签，位于设备起始处。你\
可以用此命令查看标签内容： ::

  ceph-bluestore-tool show-label --dev *device*

主设备会有很多元数据，包括以前在 OSD 数据目录下存储的小文件内\
的信息。辅助设备（ db 和 wal ）只含有必需的最少字段（
OSD UUID 、尺寸、设备类型、创建时间）。


.. OSD directory priming

OSD 目录启动
============

你可以给一个 OSD 数据目录生成些数据，才能用 *prime-osd-dir*
启动 BlueStore OSD ： ::

  ceph-bluestore-tool prime-osd-dir --dev *main device* --path /var/lib/ceph/osd/ceph-*id*


使用范围
========

**ceph-bluestore-tool** 是 Ceph 的一部分，这是个伸缩力强、\
开源、分布式的存储系统，更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph-osd <ceph-osd>`\(8)
