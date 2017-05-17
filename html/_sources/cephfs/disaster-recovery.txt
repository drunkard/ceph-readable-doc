.. _Disaster recovery:

灾难恢复
========

.. danger::

   本章节是为专家准备的，尽可能地恢复损坏的文件系统。这些操作有\
   可能改善你的处境，也可能更糟糕。如果你不太确定，最好别下手。


.. _Journal export:

导出日志
--------

尝试危险的操作前，先备份个日志副本，像这样： ::

	cephfs-journal-tool journal export backup.bin

需要注意的是，此命令在日志损坏严重时也许会失效，在这种情况下，\
应该进行 RADOS 级的复制（ http://tracker.ceph.com/issues/9902 ）。


.. _Dentry recovery from journal:

从日志恢复 dentry
-----------------

如果日志损坏、或因其它原因导致 MDS 不能重放它，可以这样尝试恢\
复文件元数据： ::

	cephfs-journal-tool event recover_dentries summary

此命令默认会操作 rank 0 的 MDS ，用 --rank=<n> 指定其它 rank 。

在条件满足的情况下，此命令会把日志中可恢复的 inode/dentry 写入后端\
存储，比如这些 inode/dentry 的版本号高于后端存储中的版本。如果日志\
中的某一部分丢失或损坏，就会被跳过。

注意，除了写出 dentry 和 inode 之外，此命令还会更新各 MDS rank \
“内”的 InoTables ，以把写入的 inode 标识为正在使用。在简单的案例\
中，此操作即可使后端存储回到完全正确的状态。

.. warning::

   此操作不能保证后端存储的状态达到自我一致，而且在此之后有必\
   要执行 MDS 在线洗刷。此命令不会更改日志内容，所以把能恢复的\
   给恢复之后，应该分别裁截日志。


.. _Journal truncation:

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


.. _MDS table wipes:

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


.. _Recovery from missing metadata objects:

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

如果数据存储池内的文件很多、或者有很大的文件，这个命令就要花费\
*很长时间*\ 。

要加快处理，可以让这个工具多跑几个例程。

先确定例程数量、再传递给每个例程一个数字 N ，此数字应大于 0 且\
小于 (worker_m - 1) 。

下面的实例演示了如何同时运行 4 个例程： ::

    # Worker 0
    cephfs-data-scan scan_extents --worker_n 0 --worker_m 4 <data pool>
    # Worker 1
    cephfs-data-scan scan_extents --worker_n 1 --worker_m 4 <data pool>
    # Worker 2
    cephfs-data-scan scan_extents --worker_n 2 --worker_m 4 <data pool>
    # Worker 3
    cephfs-data-scan scan_extents --worker_n 3 --worker_m 4 <data pool>

    # Worker 0
    cephfs-data-scan scan_inodes --worker_n 0 --worker_m 4 <data pool>
    # Worker 1
    cephfs-data-scan scan_inodes --worker_n 1 --worker_m 4 <data pool>
    # Worker 2
    cephfs-data-scan scan_inodes --worker_n 2 --worker_m 4 <data pool>
    # Worker 3
    cephfs-data-scan scan_inodes --worker_n 3 --worker_m 4 <data pool>

**切记！！！**\ 所有运行 scan_extents 阶段的例程都结束后才能开\
始 scan_inodes 。

元数据恢复完后，你可以清理掉恢复期间产生的辅助数据。 ::

    cephfs-data-scan cleanup <data pool>


.. _Finding files affected by lost data PGs:

找出受数据 PG 丢失影响的文件
----------------------------

丢失一个数据 PG 会影响很多文件。文件被拆分成了很多对象，所以要\
找出哪些文件受这些 PG 丢失的影响，需要扫描所有的对象 ID 以确认\
文件的副本数合格。这种扫描有助于找出哪些文件需要从备份恢复。

.. danger::

   这个命令不会修复任何元数据，所以在恢复文件时，你必须\ *删除*\
   损坏的文件，然后再替换它，这样才会获得完好的 inode 。不要原\
   地替换损坏的文件。

如果你已经知道了哪些 PG 丢失了对象，可以用 ``pg_files`` 子命令\
扫描可能损坏的文件，命令为： ::

    cephfs-data-scan pg_files <path> <pg id> [<pg id>...]

例如，假设你已经知道 PG 1.4 和 4.5 有数据丢失，然后你想知道
/home/bob 下面的哪些文件可能损坏了： ::

    cephfs-data-scan pg_files /home/bob 1.4 4.5

输出是可能损坏的文件路径列表，每行一个。

请注意，此命令是作为一个普通的 CephFS 客户端来搜寻文件系统内的\
所有文件、并读取它们的布局的，所以 MDS 必须是正常运行的。


.. _Using an alternate metadata pool for recovery:

用另一个元数据存储池进行恢复
----------------------------

.. warning::

   这个方法尚未全面地测试过，下手时要格外小心。

如果一个在用的文件系统损坏了、且无法使用，可以试着创建一个新的\
元数据存储池、并尝试把文件系统元数据重构进这个新存储池，旧的元\
数据仍原地保留。这是一种比较安全的恢复方法，因为不会覆盖已有的\
元数据存储池。

.. caution::

   在此过程中，多个元数据存储池包含着指向同一数据存储池的元数\
   据。在这种情况下，必须格外小心，以免更改数据存储池内容。一\
   旦恢复结束，就应该删除损坏的元数据存储池。

开始前，先创建好新的元数据存储池，并用空文件系统数据结构初始化\
它。 ::

    ceph fs flag set enable_multiple true --yes-i-really-mean-it
    ceph osd pool create recovery <pg-num> replicated <crush-ruleset-name>
    ceph fs new recovery-fs recovery <data pool> --allow-dangerous-metadata-overlay
    cephfs-data-scan init --force-init --filesystem recovery-fs --alternate-pool recovery
    ceph fs reset recovery-fs --yes-i-realy-mean-it
    cephfs-table-tool recovery-fs:all reset session
    cephfs-table-tool recovery-fs:all reset snap
    cephfs-table-tool recovery-fs:all reset inode

接下来，运行恢复工具集，加上 ``--alternate-pool`` 参数即可把结\
果输出到别的存储池： ::

    cephfs-data-scan scan_extents --alternate-pool recovery --filesystem <original filesystem name>
    cephfs-data-scan scan_inodes --alternate-pool recovery --filesystem <original filesystem name> --force-corrupt --force-init <original data pool name>

如果损坏的文件系统包含脏日志数据，随后可以用如下命令恢复： ::

    cephfs-journal-tool --rank=<original filesystem name>:0 event recover_dentries list --alternate-pool recovery
    cephfs-journal-tool --rank recovery-fs:0 journal reset --force

恢复完之后，有些恢复过来的目录其链接计数不对。首先确保
``mds_debug_scatterstat`` 参数为 ``false`` （默认值），以防 MDS
检查链接计数，再运行正向洗刷以修复它们。确保有一个 MDS 在运行，\
然后执行命令： ::

    ceph daemon mds.a scrub_path / recursive repair
