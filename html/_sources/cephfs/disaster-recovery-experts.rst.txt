.. _disaster-recovery-experts:

高级话题：元数据修复工具
========================

.. warning::
    如果你没有 CephFS 内部运行机制的专家级知识，
    你使用这些工具之前应该寻求帮助。

    这里提到的工具们能修好问题，同样也能造成破坏。

    尝试修复前，非常有必要理解清楚文件系统的问题到底是什么。

    如果你没有专业的技术支持，可以在 ceph-users 邮件列表\
    或 #ceph IRC 频道咨询。



导出日志
--------
.. Journal export

尝试危险的操作前，先备份个日志副本，像这样：

::

    cephfs-journal-tool journal export backup.bin

需要注意的是，此命令在日志损坏严重时也许会失效，在这种情况下，\
应该进行 RADOS 级的复制（ http://tracker.ceph.com/issues/9902 ）。



从日志中恢复 dentry
-------------------
.. Dentry recovery from journal

如果日志损坏、或因其它原因导致 MDS 不能重放它，
可以这样尝试恢复文件元数据： ::

    cephfs-journal-tool event recover_dentries summary

此命令默认会操作 rank 0 的 MDS ，用 --rank=<n> 指定其它 rank 。

在条件满足的情况下，此命令会把日志中可恢复的 inode/dentry 写入后端存储，
比如这些 inode/dentry 的版本号高于后端存储中的版本。
如果日志中的某一部分丢失或损坏，就会被跳过。

注意，除了写出 dentry 和 inode 之外，
此命令还会更新各 MDS rank “内”的 InoTables ，
以把写入的 inode 标识为正在使用。
在简单的案例中，此操作即可使后端存储回到完全正确的状态。

.. warning::

   此操作不能保证后端存储的状态达到自我一致，
   而且在此之后有必要执行 MDS 在线洗刷。
   此命令不会更改日志内容，所以把能恢复的给恢复之后，
   应该分别裁截日志。


舍弃日志
--------
.. Journal truncation

如果日志损坏或因故 MDS 不能重放它，你可以这样裁截它：

::

    cephfs-journal-tool [--rank=N] journal reset

如果这个文件系统有、或曾经有多个活跃的 MDS ，
可以用 ``--rank`` 选项指定 rank 。

.. warning::

    重置日志\ *会*\ 导致元数据丢失，除非你已经用其它方法
    （如 ``recover_dentries`` ）提取过了。
    此操作很可能会在数据存储池中留下一些孤儿对象，
    并导致已写过的索引节点被重分配，以致权限规则被破坏。


擦除 MDS 表
-----------
.. MDS table wipes

重置日志后，系统状态可能和 MDS 的几张表（ InoTables 、
SessionMap 、 SnapServer ）的内容变得不一致。

要重置 SessionMap （抹掉所有会话），用命令： ::

    cephfs-table-tool all reset session

此命令会在状态为 in 的所有 MDS rank 的表中执行。
如果只想在指定 rank 中执行，把 all 换成对应的 MDS rank 。

会话表是最有可能需要重置的表，但是如果你知道你还需要重置其它表，
那就把 session 换成 snap 或者 inode 。


重置 MDS 运行图
---------------
.. MDS map reset

一旦文件系统底层的 RADOS 状态（即元数据存储池的内容）恢复到一定程度，
也许有必要更新 MDS 图以反映元数据存储池的内容。
可以用下面的命令把 MDS 图重置到单个 MDS ：

::

    ceph fs reset <fs name> --yes-i-really-mean-it

运行此命令之后， MDS rank 保存在 RADOS 上的任何不为 0 的状态\
都会被忽略：因此这有可能导致数据丢失。

也许有人想知道 'fs reset' 和 'fs remove; fs new' 的不同。
主要区别在于，执行删除、新建操作会使 rank 0 处于 creating 状态，
那样会覆盖所有根索引节点、并使所有文件变成孤儿；
相反， reset 命令会使 rank 0 处于 active 状态，
这样下一个要认领此 rank 的 MDS 守护进程会继续、并使用已存在于 RADOS 中的元数据。


元数据对象丢失的恢复
--------------------
.. Recovery from missing metadata objects

取决于丢失或被篡改的是哪种对象，
你得运行几个命令生成这些对象的默认版本。

::

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

最后，根据数据存储池中的内容重新生成\
丢失文件和目录的元数据对象。
这要分两步完成，首先，扫描\ *所有*\ 对象以计算索引节点的尺寸和
mtime 元数据；其次，从每个文件的第一个对象扫描出\
元数据并注入元数据存储池。

::

    cephfs-data-scan scan_extents <data pool>
    cephfs-data-scan scan_inodes <data pool>
    cephfs-data-scan scan_links

如果数据存储池内的文件很多、或者有很大的文件， scan_extents 和
scan_inodes 命令就要花费\ *很长时间*\ 。

要加快处理，可以让这个工具多跑几个例程。

先确定例程数量、再传递给每个例程一个数字 N ，此数字应大于 0 且\
小于 (worker_m - 1) 。

下面的实例演示了如何同时运行 4 个例程：

::

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

元数据恢复完后，你可以清理掉恢复期间产生的辅助数据。

::

    cephfs-data-scan cleanup <data pool>



用另一个元数据存储池进行恢复
----------------------------
.. Using an alternate metadata pool for recovery

.. warning::

   这个方法尚未全面地测试过，
   下手时要格外小心。

如果一个在用的文件系统损坏了、且无法使用，
可以创建一个新的元数据存储池、
并尝试把此文件系统的元数据重构进这个新存储池，
旧的元数据仍原地保留。这是一种比较安全的恢复方法，
因为不会更改现有的元数据存储池。

.. caution::

   在此过程中，多个元数据存储池包含着指向同一数据存储池的元数据。
   在这种情况下，必须格外小心，
   以免更改数据存储池内容。一旦恢复结束，
   就应该归档或删除损坏的元数据存储池。

开始前，应该关闭现有文件系统，如果还没关闭，
为防止数据存储池被更改更多，先卸载所有客户端、
然后把这个文件系统标记为已失效：

::

    ceph fs fail <fs_name>

接下来，创建一个恢复文件系统，我们将给它迁移新的元数据存储池、
其后端还挂着原来的数据存储池。

::

    ceph fs flag set enable_multiple true --yes-i-really-mean-it
    ceph osd pool create cephfs_recovery_meta
    ceph fs new cephfs_recovery recovery <data_pool> --allow-dangerous-metadata-overlay


这个恢复文件系统的启动需要一个 MDS rank ，
它会写入一些元数据来初始化这个新的元数据存储池。
这对于启动恢复是必要的。然而，我们现在要关闭这个 MDS ，
因为我们不想让它和元数据存储池有更多互动。

::

    ceph fs fail cephfs_recovery

接下来，我们会重置 MDS 刚创建的初始元数据：

::

    cephfs-table-tool cephfs_recovery:all reset session
    cephfs-table-tool cephfs_recovery:all reset snap
    cephfs-table-tool cephfs_recovery:all reset inode

现在进行从数据存储池到元数据存储池的恢复：

::

    cephfs-data-scan init --force-init --filesystem cephfs_recovery --alternate-pool cephfs_recovery_meta
    cephfs-data-scan scan_extents --alternate-pool cephfs_recovery_meta --filesystem <fs_name> <data_pool>
    cephfs-data-scan scan_inodes --alternate-pool cephfs_recovery_meta --filesystem <fs_name> --force-corrupt <data_pool>
    cephfs-data-scan scan_links --filesystem cephfs_recovery

.. note::

   上面的每一个扫描都要覆盖整个数据存储池。
   需要相当多的时间才能完成。
   看看前面的段落，把这个任务分配到多个作业进程上。

如果损坏的文件系统包含脏日志数据，
随后可以用如下命令恢复：

::

    cephfs-journal-tool --rank=<fs_name>:0 event recover_dentries list --alternate-pool cephfs_recovery_meta
    cephfs-journal-tool --rank cephfs_recovery:0 journal reset --force

恢复完之后，有些恢复过来的目录其链接计数不对。
首先确保 ``mds_debug_scatterstat`` 参数为 ``false`` （默认值），
以防 MDS 检查链接计数：

::

    ceph config rm mds mds_verify_scatter
    ceph config rm mds mds_debug_scatterstat

（注意，这个配置可能已经全局地设置过了、或者可以通过 ceph.conf 文件。）
现在，允许 MDS 接管恢复的文件系统：

::

    ceph fs set cephfs_recovery joinable true

最后，运行正向\ `洗刷 scrub </cephfs/scrub>` 以修复统计信息。\
确保有一个 MDS 在运行，然后执行命令：

::

    ceph fs status # get active MDS
    ceph tell mds.<id> scrub start / recursive repair

.. note::

   符号链接被恢复成了空的普通文件。
   `符号链接恢复 <https://tracker.ceph.com/issues/46166>`_
   按计划会在 Pacific 版支持。

强烈建议尽快迁移已恢复文件系统上的数据。
已恢复的文件系统可以运作后，不要再恢复旧文件系统。

.. note::

    如果数据存储池也损坏了，有些文件可能没法恢复，
    因为回溯信息丢失了。如果有数据对象丢失了
    （由于数据存储池上的归置组丢失之类的问题），
    恢复的文件里在丢失数据的位置会有空洞。



.. _符号链接恢复: https://tracker.ceph.com/issues/46166
