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

.. note:: 在 Ceph 文件系统上用元数据修复工具操作之前，
   该文件系统必须处于离线状态。如果在文件系统处于在线状态时调用这些工具，
   它们将发出警告。如果有任何一个恢复步骤未能成功完成，
   **决不要**\ 继续运行任何其他恢复步骤。如果任何恢复步骤失败，
   请通过邮件列表、 IRC 频道和 Slack 频道向专家寻求帮助。


导出日志
--------
.. Journal export

尝试危险的操作前，先备份个日志副本，执行下列命令：

.. prompt:: bash #

   cephfs-journal-tool journal export backup.bin

如果恢复过程未按预期进行，备份的日志将派上用场。
通过从备份的日志导入，
可以把 MDS 日志恢复到它最初的状态：

.. prompt:: bash #

   cephfs-journal-tool journal import backup.bin


从日志中恢复 dentry
-------------------
.. Dentry recovery from journal

如果日志损坏、或因其它原因导致 MDS 不能重放它，
可以这样尝试恢复文件元数据，执行下列命令：

.. prompt:: bash #

   cephfs-journal-tool event recover_dentries summary

此命令默认会操作 rank ``0`` 的 MDS ，给 ``cephfs-journal-tool`` 命令\
加选项 ``--rank=<n>`` 来操作其它 rank ，
或者加 ``--rank=all`` 来操作所有 MDS rank 。

此命令会把日志中所有可恢复的 inode/dentry 写入后端存储，
但是，仅限于这些 inode/dentry 的版本号高于后端存储中已有内容的版本时才行。
日志中丢失或损坏的部分会被跳过。

除了写出 dentry 和 inode 之外，
此命令还会更新状态为 ``in`` 的每个 MDS rank 的 InoTables ，
为的是把已写入的 inode 标识为正在使用。
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

.. prompt:: bash #

   cephfs-journal-tool [--rank=<fs_name>:{mds-rank|all}] journal reset --yes-i-really-really-mean-it

如果这个文件系统有、或曾经有多个活跃的 MDS ，
可以用 ``--rank`` 选项指定 rank 。

.. warning:: 重置日志\ *会*\ 导致元数据丢失，除非你已经用其它方法
   （如 ``recover_dentries`` ）提取过了。
   重置日志很可能会在数据存储池中留下一些孤儿对象，
   并导致已写过的索引节点被重分配，进而导致文件系统产生故障。


擦除 MDS 表
-----------
.. MDS table wipes

在恢复过程中，并非每张 MDS 表都必须重置。
如果对应的 RADOS 对象缺失（例如，由于某些 PG 丢失）
或者表与 RADOS 后端存储的元数据不一致，
就需要重置 MDS 表。要检查表对象是否缺失，执行下列命令：

Session 表：

.. prompt:: bash #

   rados -p <metadata-pool> stat mds0_sessionmap

Inode 表:

.. prompt:: bash #

   rados -p <metadata-pool> stat mds0_inotable

Snap 表：

.. prompt:: bash #

   rados -p <metadata-pool> stat mds_snaptable

.. note:: ``sessionmap`` 和 ``inotable`` 对象是和 MDS rank 对应的
   （对象名里包含了 rank 号 - mds0_inotable 、 mds1_inotable 等等）。

即使表对象存在，它也可能与 RADOS 后端存储的元数据不一致。
然而，如果元数据事实上已经不一致或已损坏，
在不将文件系统上线的情况下，很难检测到不一致。
在类似情况下， MDS 会将自己标记为 `down:damaged` 。
要重置单个的表，执行下列命令：

Session 表：

.. prompt:: bash #

   cephfs-table-tool 0 reset session
table:
SnapServer ：

.. prompt:: bash #

   cephfs-table-tool 0 reset snap

InoTable ：

.. prompt:: bash #

   cephfs-table-tool 0 reset inode

上述命令会针对一个特定 MDS rank 上的表执行。
要在状态为 ``in`` 的所有 MDS rank 的表中执行，
把上述命令里的 MDS rank 号换成 ``all`` ，如下列命令所示：

Session 表：

.. prompt:: bash #

   cephfs-table-tool all reset session

SnapServer ：

.. prompt:: bash #

   cephfs-table-tool all reset snap

InoTable ：

.. prompt:: bash #

   cephfs-table-tool all reset inode

.. note:: session 表重置之后，需要重挂载或重启所有 CephFS 客户端。


重置 MDS 运行图
---------------
.. MDS map reset

一旦文件系统底层的 RADOS 状态（即元数据存储池的内容）恢复到一定程度，
也许有必要更新 MDS 图以反映元数据存储池的内容。
可以用下面的命令把 MDS 图重置到单个 MDS ：

.. prompt:: bash #

    ceph fs reset <fs name> --yes-i-really-mean-it

运行此命令之后， MDS rank 保存在 RADOS 上的任何不为 ``0`` 的状态\
都会被忽略：因此这有可能导致数据丢失。

``fs reset`` 命令和 ``fs remove`` 命令有个不同点。
``fs reset`` 命令会把 rank ``0`` 留在 ``active`` 状态，
这样下一个要认领此 rank 的 MDS 守护进程会继续使用已存在于 RADOS 中的元数据；
``fs remove`` 命令会把 rank ``0`` 留在 ``creating`` 状态，
意味着磁盘上的现有根索引节点（ root inodes ）会被覆盖。
执行 ``fs remove`` 命令会把现有文件变成孤儿文件。


元数据对象丢失的恢复
--------------------
.. Recovery from missing metadata objects

取决于丢失或被篡改的是哪种对象，
你也许得额外运行几个命令生成这些对象的默认版本。

注意，数据扫描工具可能耗时巨长。要想看到操作进度，启用 cli_api 工具。

.. prompt:: bash #

    ceph mgr module enable cli_api

用 ceph status 跟踪进度，它会显示长时间任务的预计完成时间。

.. prompt:: bash #

    ceph -s

如果启用了 ``cli_api`` 模块，数据扫描工具
（ ``scan_extents``, ``scan_inodes``, and ``scan_links`` ）
就会自动把它们的进度报告给 Ceph 管理器。进度更新包括：

- 处理过的对象数和总对象数
- 已完成的百分比
- 预计完成时间（ ETA ）
- 平均处理速率

大概每 5 秒更新一次进度，并出现在 ``ceph -s`` 输出的 Progress 段内。
每次扫描操作都会创建一个唯一的进度事件，
用操作名和进程 ID 标识。

.. note:: ``ceph`` 命令必须位于系统 PATH 之内，
   进度更新才能正常运转。
   如果进度更新没出现在 ``ceph -s`` 里，检查一下：

   - ``cli_api`` 是否启用
   - ``ceph`` 命令位于 PATH 变量之内
   - ``CEPH_CONF`` 环境变量（如果设置过）
     指向一个正确的配置文件

如果系统无法与 Ceph 管理器通信或必需模块不可用，
进度更新将自动禁用。
即使管理器更新被禁用，
控制台输出仍能继续显示本地进度信息。 

如果根节点或 MDS 目录（ ``~mdsdir`` ）丢失或损坏，执行下列命令：

根节点（ "/" 和 MDS 目录）：

.. prompt:: bash #

   cephfs-data-scan init

运行此命令通常是安全的，因为如果根索引节点和
mdsdir 索引节点已存在，此命令会跳过生成这些节点。
但如果这些索引节点已损坏，则必须重新生成。
可以通过尝试使文件系统重新上线来鉴定损坏情况
（此时， MDS 将转变为 ``down:damaged`` 状态，并在 [ MDS 日志中]
显示相应的日志消息，指出在加载根索引节点或 mdsdir 索引节点时可能存在的问题）。
另一种方法是用 ``ceph-dencoder`` 工具来解码索引节点。
此步骤稍微复杂一些。 

最后，根据数据存储池中的内容重新生成丢失文件和目录的元数据对象。
这是个三阶段过程：

#. 扫描\ *所有*\ 对象以计算索引节点的尺寸和 mtime 元数据；

   .. prompt:: bash #

      cephfs-data-scan scan_extents [<data pool> [<extra data pool> ...]]

#. 从每个文件的第一个对象扫描出元数据并注入元数据存储池。

   .. prompt:: bash #

      cephfs-data-scan scan_inodes [<data pool>]

#. 检查 inode 的链接情况并修复发现的错误。

   .. prompt:: bash #

      cephfs-data-scan scan_links

如果数据存储池内的文件很多、或者有很大的文件， ``scan_extents`` 和
``scan_inodes`` 命令可能得花费\ *很长时间*\ 。

要加快 ``scan_extents`` 或 ``scan_inodes`` 的处理进程，
可以让这个工具多跑几个例程。

确定例程数量、再传递给每个例程一个数字，在 ``(worker_m - 1)`` 范围内
（也就是 '0 到 worker_m 减 1'）。

下面的实例演示了如何同时运行 4 个例程：

::

    # Worker 0
    cephfs-data-scan scan_extents --worker_n 0 --worker_m 4
    # Worker 1
    cephfs-data-scan scan_extents --worker_n 1 --worker_m 4
    # Worker 2
    cephfs-data-scan scan_extents --worker_n 2 --worker_m 4
    # Worker 3
    cephfs-data-scan scan_extents --worker_n 3 --worker_m 4

    # Worker 0
    cephfs-data-scan scan_inodes --worker_n 0 --worker_m 4
    # Worker 1
    cephfs-data-scan scan_inodes --worker_n 1 --worker_m 4
    # Worker 2
    cephfs-data-scan scan_inodes --worker_n 2 --worker_m 4
    # Worker 3
    cephfs-data-scan scan_inodes --worker_n 3 --worker_m 4

**切记！！！**\ 所有运行 ``scan_extents`` 阶段的例程都结束后\
才能开始进入 ``scan_inodes`` 阶段。

元数据恢复完后，你可以清理掉恢复期间产生的辅助数据。
执行下列命令运行清理操作：

.. prompt:: bash #

   cephfs-data-scan cleanup <data pool>

清理阶段可以运行多个例程来加速执行： ::

    # Worker 0
    cephfs-data-scan cleanup --worker_n 0 --worker_m 4
    # Worker 1
    cephfs-data-scan cleanup --worker_n 1 --worker_m 4
    # Worker 2
    cephfs-data-scan cleanup --worker_n 2 --worker_m 4
    # Worker 3
    cephfs-data-scan cleanup --worker_n 3 --worker_m 4

.. note::

   ``scan_extents`` 、 ``scan_inodes`` 和 ``cleanup`` 命令\
   的数据存储池参数是可选的，通常工具能够自动探测存储池。
   不过，你也可以覆盖它。
   ``scan_extents`` 命令需要指定所有数据存储池，
   而 ``scan_inodes`` 和 ``cleanup`` 命令\
   只需要指定主数据存储池。


已知的局限性和陷阱
------------------
.. Known Limitations And Pitfalls

灾难恢复过程可能既耗时又艰巨。
必须严格按照上述详细步骤谨慎执行恢复操作，
以确保任何步骤的失败都能清清楚楚地理解。
你必须能绝对确定地判断是否可以安全地继续下一步。
如果你确信自己能应对这些挑战，
那么在尝试进行灾难恢复之前，请先研究以下限制和陷阱：

#. 数据扫描命令无法提供其操作完成时间的预估。
   目前正在开发一项能够提供此类预估的功能。
   详情见 https://tracker.ceph.com/issues/63191 。
#. 在恢复之后、 CephFS 客户端开始使用文件系统之前，
   执行一次文件系统清理非常有必要。
#. 一般来说，我们不建议在出现问题时更改任何与 MDS 相关的设置
   （例如 ``max_mds`` ）。
#. 目前，灾难恢复是一个手动过程。我们计划通过
   Disaster Recovery Super Tool （灾难恢复超级工具）实现恢复过程的自动化。
   详情见 https://tracker.ceph.com/issues/71804 。
#. 一个众所周知的技巧（一些社区用户在用）是，
   当由于日志过长，导致 MDS 貌似卡在 ``up_replay`` 状态时，
   可以采用灾难恢复步骤（特别是 ``recover_dentries`` 步骤）。
   在 MDS 重放日志需要较长时间时，不要急于调用 ``recover_dentries`` ，
   而应该考虑按照 :ref:`cephfs_dr_stuck_during_recovery`
   中详述的步骤试着加快日志重放速度。


用另一个元数据存储池进行恢复
----------------------------
.. Using an alternate metadata pool for recovery

.. warning:: 这个方法尚未广泛测试过。我们建议按照前述过程来恢复文件系统，
   除非你有更恰当的理由。这个步骤在下手时要格外小心。

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

#. 关闭现有文件系统，以防止数据存储池被更改更多；
   卸载所有客户端。客户端们全部卸载后，
   用下列命令把这个文件系统标记为已失效：

   .. prompt:: bash #

      ceph fs fail <fs_name>

   .. note::

      ``<fs_name>`` 在这里和下文都是指最初的、损坏的文件系统。

#. 创建一个恢复文件系统。
   这个恢复文件系统将用于恢复已损坏存储池中的数据。
   首先，给这个文件系统部署一个数据存储池，
   然后，把新元数据存储池关联到新数据存储池上，
   然后，设置这个新元数据存储池的后端是旧数据存储池。

   .. prompt:: bash #

      ceph osd pool create cephfs_recovery_meta
      ceph fs new cephfs_recovery cephfs_recovery_meta <data_pool> --recover --allow-dangerous-metadata-overlay

   .. note::

      以后，你可以重命名用于恢复的元数据存储池和文件系统。
      ``--recover`` 标记会阻止所有 MDS 加入新文件系统。

#. 给这个文件系统创建初始元数据：

   .. prompt:: bash #

      cephfs-table-tool cephfs_recovery:0 reset session

   .. prompt:: bash #

      cephfs-table-tool cephfs_recovery:0 reset snap

   .. prompt:: bash #

      cephfs-table-tool cephfs_recovery:0 reset inode

   .. prompt:: bash #

      cephfs-journal-tool --rank cephfs_recovery:0 journal reset --force --yes-i-really-really-mean-it

#. 从数据存储池重建元数据存储池，执行下列命令：

   .. prompt:: bash #

      cephfs-data-scan init --force-init --filesystem cephfs_recovery --alternate-pool cephfs_recovery_meta

   .. prompt:: bash #

      cephfs-data-scan scan_extents --alternate-pool cephfs_recovery_meta --filesystem <fs_name>

   .. prompt:: bash #

      cephfs-data-scan scan_inodes --alternate-pool cephfs_recovery_meta --filesystem <fs_name> --force-corrupt

   .. prompt:: bash #

      cephfs-data-scan scan_links --filesystem cephfs_recovery

   .. note::

      上面的每一个扫描都要覆盖整个数据存储池。
      需要相当多的时间才能完成。
      看看前面的段落，把这个任务分配到多个作业进程上。

   如果损坏的文件系统包含脏日志数据，
   随后可以用如下命令恢复：

   .. prompt:: bash #

      cephfs-journal-tool --rank=<fs_name>:0 event recover_dentries list --alternate-pool cephfs_recovery_meta

#. 恢复完之后，有些恢复过来的目录其统计信息不对。
   首先确保 ``mds_verify_scatter`` 和 ``mds_debug_scatterstat``
   参数的值为 ``false`` （默认值），
   以防 MDS 检查这些统计信息：

   .. prompt:: bash #

      ceph config rm mds mds_verify_scatter

   .. prompt:: bash #

      ceph config rm mds mds_debug_scatterstat

   .. note::

      还需要核对尚未全局设置、
      或用本地 ``ceph.conf`` 文件配置的。

#. 允许 MDS 加入恢复文件系统：

   .. prompt:: bash #

      ceph fs set cephfs_recovery joinable true

#. 运行正向\ `洗刷 scrub </cephfs/scrub>` 以修复递归统计信息。\
   确保有一个 MDS 守护进程在运行，然后执行命令：

   .. prompt:: bash #

      ceph tell mds.cephfs_recovery:0 scrub start / recursive,repair,force

   .. note::

      `符号链接恢复 <https://tracker.ceph.com/issues/46166>`_
      从 Quincy 版开始支持。

      符号链接被恢复成了空的普通文件。

   建议尽快迁移已恢复文件系统上的数据。
   已恢复的文件系统可以运作后，
   不要再恢复旧文件系统。

   .. note::

      如果数据存储池也损坏了，有些文件可能没法恢复，
      因为与之相关的回溯信息丢失了。
      如果有数据对象丢失了（由于数据存储池内的归置组丢失之类的问题），
      恢复的文件里在丢失数据的位置会有空洞。


.. _符号链接恢复: https://tracker.ceph.com/issues/46166
