.. Advanced: Metadata repair tools
.. _disaster-recovery-experts:

高级话题：元数据修复工具
========================

.. warning::

    If you do not have expert knowledge of CephFS internals, you will
    need to seek assistance before using any of these tools.

    The tools mentioned here can easily cause damage as well as fixing it.

    It is essential to understand exactly what has gone wrong with your
    filesystem before attempting to repair it.

    If you do not have access to professional support for your cluster,
    consult the ceph-users mailing list or the #ceph IRC channel.


.. Journal export

导出日志
--------

尝试危险的操作前，先备份个日志副本，像这样： ::

    cephfs-journal-tool journal export backup.bin

需要注意的是，此命令在日志损坏严重时也许会失效，在这种情况下，\
应该进行 RADOS 级的复制（ http://tracker.ceph.com/issues/9902 ）。


.. Dentry recovery from journal

从日志中恢复 dentry
-------------------

如果日志损坏、或因其它原因导致 MDS 不能重放它，可以这样尝试\
恢复文件元数据： ::

    cephfs-journal-tool event recover_dentries summary

此命令默认会操作 rank 0 的 MDS ，用 --rank=<n> 指定其它 rank 。

在条件满足的情况下，此命令会把日志中可恢复的 inode/dentry 写入\
后端存储，比如这些 inode/dentry 的版本号高于后端存储中的版本。\
如果日志中的某一部分丢失或损坏，就会被跳过。

注意，除了写出 dentry 和 inode 之外，此命令还会更新各 MDS rank
“内”的 InoTables ，以把写入的 inode 标识为正在使用。在简单的\
案例中，此操作即可使后端存储回到完全正确的状态。

.. warning::

   此操作不能保证后端存储的状态达到自我一致，而且在此之后\
   有必要执行 MDS 在线洗刷。此命令不会更改日志内容，所以把能\
   恢复的给恢复之后，应该分别裁截日志。


.. Journal truncation

舍弃日志
--------

如果日志损坏或因故 MDS 不能重放它，你可以这样裁截它： ::

    cephfs-journal-tool journal reset

.. warning::

    重置日志\ *会*\ 导致元数据丢失，除非你已经用其它方法（如 \
    ``recover_dentries`` ）提取过了。此操作很可能会在\
    数据存储池中留下一些孤儿对象，并导致已写过的索引节点被\
    重分配，以致权限规则被破坏。


.. MDS table wipes

擦除 MDS 表
-----------

重置日志后，系统状态可能和 MDS 的几张表（ InoTables 、
SessionMap 、 SnapServer ）的内容变得不一致。

要重置 SessionMap （抹掉所有会话），用命令： ::

    cephfs-table-tool all reset session

此命令会在所有 MDS rank “内”的表中执行。如果只想在指定 rank 中\
执行，把 all 换成对应的 MDS rank 。

会话表是最有可能需要重置的表，但是如果你知道你还需要重置其它\
表，那就把 session 换成 snap 或者 inode 。


.. MDS map reset

重置 MDS 运行图
---------------

一旦文件系统底层的 RADOS 状态（即元数据存储池的内容）恢复到\
一定程度，也许有必要更新 MDS 图以反映元数据存储池的内容。可以\
用下面的命令把 MDS 图重置到单个 MDS ：

::

    ceph fs reset <fs name> --yes-i-really-mean-it

运行此命令之后， MDS rank 保存在 RADOS 上的任何不为 0 的状态\
都会被忽略：因此这有可能导致数据丢失。

也许有人想知道 'fs reset' 和 'fs remove; fs new' 的不同。主要\
区别在于，执行删除、新建操作会使 rank 0 处于 creating 状态，\
那样会覆盖所有根索引节点、并使所有文件变成孤儿；相反， reset
命令会使 rank 0 处于 active 状态，这样下一个要认领此 rank 的
MDS 守护进程会继续、并使用已存在于 RADOS 中的数据。


.. Recovery from missing metadata objects

元数据对象丢失的恢复
--------------------

取决于丢失或被篡改的是哪种对象，你得运行几个命令生成这些对象的\
默认版本。 ::

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

最后，根据数据存储池中的内容重新生成丢失文件和目录的元数据\
对象。这要分两步完成，首先，扫描\ *所有*\ 对象以计算索引节点的\
尺寸和 mtime 元数据；其次，从每个文件的第一个对象扫描出元数据\
并注入元数据存储池。 ::

    cephfs-data-scan scan_extents <data pool>
    cephfs-data-scan scan_inodes <data pool>
    cephfs-data-scan scan_links

如果数据存储池内的文件很多、或者有很大的文件， scan_extents 和
scan_inodes 命令就要花费\ *很长时间*\ 。

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

**切记！！！**\ 所有运行 scan_extents 阶段的例程都结束后才能\
开始 scan_inodes 。

元数据恢复完后，你可以清理掉恢复期间产生的辅助数据。 ::

    cephfs-data-scan cleanup <data pool>


.. Using an alternate metadata pool for recovery

用另一个元数据存储池进行恢复
----------------------------

.. warning::

   这个方法尚未全面地测试过，下手时要格外小心。

如果一个在用的文件系统损坏了、且无法使用，可以试着创建一个新的\
元数据存储池、并尝试把文件系统元数据重构进这个新存储池，旧的\
元数据仍原地保留。这是一种比较安全的恢复方法，因为不会覆盖已有\
的元数据存储池。

.. caution::

   在此过程中，多个元数据存储池包含着指向同一数据存储池的\
   元数据。在这种情况下，必须格外小心，以免更改数据存储池\
   内容。一旦恢复结束，就应该删除损坏的元数据存储池。

开始前，先创建好新的元数据存储池，并用空文件系统数据结构初始化\
它。 ::

    ceph fs flag set enable_multiple true --yes-i-really-mean-it
    ceph osd pool create recovery <pg-num> replicated <crush-rule-name>
    ceph fs new recovery-fs recovery <data pool> --allow-dangerous-metadata-overlay
    cephfs-data-scan init --force-init --filesystem recovery-fs --alternate-pool recovery
    ceph fs reset recovery-fs --yes-i-really-mean-it
    cephfs-table-tool recovery-fs:all reset session
    cephfs-table-tool recovery-fs:all reset snap
    cephfs-table-tool recovery-fs:all reset inode

接下来，运行恢复工具集，加上 ``--alternate-pool`` 参数即可把结\
果输出到别的存储池： ::

    cephfs-data-scan scan_extents --alternate-pool recovery --filesystem <original filesystem name> <original data pool name>
    cephfs-data-scan scan_inodes --alternate-pool recovery --filesystem <original filesystem name> --force-corrupt --force-init <original data pool name>
    cephfs-data-scan scan_links --filesystem recovery-fs

如果损坏的文件系统包含脏日志数据，随后可以用如下命令恢复： ::

    cephfs-journal-tool --rank=<original filesystem name>:0 event recover_dentries list --alternate-pool recovery
    cephfs-journal-tool --rank recovery-fs:0 journal reset --force

恢复完之后，有些恢复过来的目录其链接计数不对。首先确保
``mds_debug_scatterstat`` 参数为 ``false`` （默认值），以防 MDS
检查链接计数，再运行正向洗刷以修复它们。确保有一个 MDS 在运行，\
然后执行命令： ::

    ceph tell mds.a scrub start / recursive repair
