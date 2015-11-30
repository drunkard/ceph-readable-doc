灾难恢复
========

.. danger::

    本章节是为专家准备的，尽可能地恢复损坏的文件系统。这些操作有\
    可能改善你的处境，也可能更糟糕。如果你不太确定，最好别下手。


导出日志
--------

尝试危险的操作前，先备份个日志副本，像这样：

::

	cephfs-journal-tool journal export backup.bin

需要注意的是，此命令在日志损坏严重时也许会失效，在这种情况下，应该\
进行 RADOS 级的复制（ http://tracker.ceph.com/issues/9902 ）。


从日志恢复 dentry
-----------------

如果日志损坏、或因其它原因导致 MDS 不能重放它，可以这样尝试恢复文\
件元数据： ::

	cephfs-journal-tool event recover_dentries summary

此命令默认会操作 rank 0 的 MDS ，用 --rank=<n> 指定其它 rank 。

在条件满足的情况下，此命令会把日志中可恢复的 inode/dentry 写入后端\
存储，比如这些 inode/dentry 的版本号高于后端存储中的版本。如果日志\
中的某一部分丢失或损坏，就会被跳过。

注意，除了写出 dentry 和 inode 之外，此命令还会更新各 MDS rank \
“内”的 InoTables ，以把写入的 inode 标识为正在使用。在简单的案例\
中，此操作即可使后端存储回到完全正确的状态。

.. warning::

    此操作不能保证后端存储的状态达到自我一致，而且在此之后有必要\
    执行 MDS 在线洗刷。此命令不会更改日志内容，所以把能恢复的给恢\
    复之后，应该分别裁截日志。


日志裁截
--------

如果日志损坏或因故 MDS 不能重放它，你可以这样裁截它：

::

	cephfs-journal-tool journal reset

.. warning::

    重置日志\ *会*\ 导致元数据丢失，除非你已经用其它方法（如 \
    ``recover_dentries`` ）提取过了。此操作很可能会在数据存储池中\
    留下一些孤儿对象，并导致已写过的索引节点被重分配，以致权限规\
    则被破坏。


擦除 MDS 表
-----------

重置日志后，可能 MDS 表（ InoTable 、 SessionMap 、 SnapServer ）\
的内容就不再一致了。

要重置 SessionMap （擦掉所有会话），用此命令：

::

	cephfs-table-tool all reset session

此命令会在所有 MDS rank “内”的表中执行。如果只想在指定 rank 中\
执行，把 all 换成对应的 MDS rank 。

会话表是最有可能需要重置的表，但是如果你知道你还需要重置其它表，\
那就把 session 换成 snap 或者 inode 。


MDS 图重置
----------

一旦文件系统底层的 RADOS 状态（即元数据存储池的内容）恢复到一定程\
度，也许有必要更新 MDS 图以反映元数据存储池的内容。可以用下面的命\
令把 MDS 图重置到单个 MDS ：

::

	ceph fs reset <fs name> --yes-i-really-mean-it

运行此命令之后， MDS rank 保存在 RADOS 上的任何不为 0 的状态都会被\
忽略：因此这有可能导致数据丢失。

也许有人想知道 'fs reset' 和 'fs remove; fs new' 的不同。主要区别在\
于，执行删除、新建操作会使 rank 0 处于 creating 状态，那样会覆盖所\
有根索引节点、并使所有文件变成孤儿；相反， reset 命令会使 rank 0 处\
于 active 状态，这样下一个要认领此 rank 的 MDS 守护进程会继续、并使\
用已存在于 RADOS 中的数据。


元数据对象丢失的恢复
--------------------

取决于丢失或被篡改的是哪种对象，你得运行几个命令生成这些对象的默认\
版本。 ::

	# 会话表
	cephfs-table-tool 0 reset session
	# SnapServer 快照服务器
	cephfs-table-tool 0 reset snap
	# InoTable 索引节点表
	cephfs-table-tool 0 reset inode
	# Journal 日志
	cephfs-journal-tool --rank=0 journal reset
	# 根索引节点（ / 和所有 MDS 目录）
	cephfs-data-scan init

最后，根据数据存储池中的内容重新生成丢失文件和目录的元数据对象。\
这要分两步完成，首先，扫描\ *所有*\ 对象以计算索引节点的尺寸和 \
mtime 元数据；其次，从每个文件的第一个对象扫描出元数据并注入元数\
据存储池。 ::

	cephfs-data-scan scan_extents <data pool>
	cephfs-data-scan scan_inodes <data pool>

如果数据存储池内的文件很多、或者有很大的文件，这个命令就要花费很\
长时间。要加快处理，可以让这个工具多跑几个例程。先确定例程数量、\
再传递给每个例程一个数字 N ，此数字应大于 0 且小于 (N - 1) ，像\
这样： ::

    # Worker 0
    cephfs-data-scan scan_extents <data pool> 0 1
    # Worker 1
    cephfs-data-scan scan_extents <data pool> 1 1

    # Worker 0
    cephfs-data-scan scan_inodes <data pool> 0 1
    # Worker 1
    cephfs-data-scan scan_inodes <data pool> 1 1

切记！！！所有运行 scan_extents 阶段的例程都结束后才能开始 \
scan_inodes 。
