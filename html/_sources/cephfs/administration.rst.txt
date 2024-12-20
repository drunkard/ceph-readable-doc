.. _cephfs-administration:

CephFS 管理命令
===============

文件系统管理
------------
.. File Systems

.. note:: 文件系统、元数据存储池、和数据存储池的名字\
   只能由这些字符 [a-zA-Z0-9\_-.] 组成。

这些命令适用于 Ceph 集群的 CephFS 文件系统。注意，默认情况下，\
只允许一个文件系统；执行 ``ceph fs flag set enable_multiple true``
后才允许创建多个文件系统。

::

    fs new <file system name> <metadata pool name> <data pool name>

该命令可创建一个新文件系统，文件系统名称和元数据存储池名称如上所述。
指定的数据存储池是默认数据存储池，一旦设置就不能更改。
每个文件系统都有自己的一套 MDS 守护进程，分别管理各个 rank ，
因此要确保有足够的备用守护进程来容纳新文件系统。

::

    fs ls

按名字罗列所有文件系统。

::

    fs lsflags <file system name>

罗列一个文件系统上设置的所有标志。

::

    fs dump [epoch]

此操作将转储指定时间结（默认：当前的）的 FSMap ，
其中包括所有的文件系统设置、 MDS 守护进程及其持有的 rank ，
以及备用 MDS 守护进程列表。


::

    fs rm <file system name> [--yes-i-really-mean-it]

销毁 CephFS 文件系统。此命令会从 FSMap 中抹去有关文件系统状态的信息。
不会触及元数据存储池和数据存储池，
它们需要分开销毁。

::

    fs get <file system name>

获取指定文件系统的信息，包括配置信息和各 rank 。
这是 ``ceph fs dump`` 命令中相同信息的子集。

::

    fs set <file system name> <var> <val>

更改文件系统的配置。这些设置选项只针对指定的文件系统，
不会影响其他文件系统。集群不健康时，
只有更改 ``max_mds`` 才需要确认标记。

.. note:: 当集群不健康时，修改 FS 配置变量 ``max_mds``
   必须加上确认标志（ ``--yes-i-really-mean-it`` ）。
   这个额外的预防措施是告诉用户，
   在故障排除或恢复过程中修改 ``max_mds`` 可能于事无补。
   相反，它可能会进一步破坏集群的稳定性。

::

    fs add_data_pool <file system name> <pool name/id>

给文件系统增加一个数据存储池。此存储池可用于文件布局，
作为存储文件数据的替代位置。

::

    fs rm_data_pool <file system name> <pool name/id>

此命令从文件系统的数据存储池列表中删除指定的存储池。
如果有文件伸展到了被删除的数据存储池，
这部分文件数据将不可用。无法删除默认数据存储池
（创建文件系统时的）。

::

    fs rename <file system name> <new file system name> [--yes-i-really-mean-it]

重命名一个 Ceph 文件系统。这个操作同时会把文件系统的数据存储池和\
元数据存储池上的应用标签（ application tag ）更改为新的文件系统名称。
授权给旧文件系统名字的 CephX ID 需要重新授权到新名字。
正在使用这些 ID 的客户端们正在进行的所有操作都可能会中断。
文件系统上的镜像应该禁用掉。

::

    fs swap <fs1-name> <fs1_id> <fs2-name> <fs2_id> [--swap-fscids=yes|no] [--yes-i-really-mean-it]

交换两个 Ceph 文件系统的名称，并相应更新两个文件系统所有存储池上的应用程序标签。
某些工具跟踪的是文件系统 FSCID ，而不是名字，
可能会因为这一操作而错乱。因此，
系统提供了 ``--swap-fscids`` 强制选项，
用以决定是否必须交换 FSCID 。

.. note:: FSCID 表示文件系统集群 ID （File System Cluster ID ）。

在交换之前，应禁用两个 CephFS 上的镜像功能
（因为 cephfs-mirror 守护进程内部使用的是 fscid ，
在它运行时更改 fscid 可能会导致不可预知的行为），
两个 CephFS 都应处于离线状态，并且必须为两个 CephFS
设置文件系统标志 ``refuse_client_sessions`` 。

这个 API 的功能是为了方便灾难恢复，
在灾难恢复时，从先前的文件系统重建的新文件系统准备好了，
可以随时接替可能受损的文件系统。
操作员可以使用交换操作来代替两次 ``fs rename`` 重命名操作，
这样就不会出现通过名字调用的主（或生产环境）文件系统不存在的 FSMap 时间结。
当 Ceph 被（ Rook ）这样的自动化存储操作员监控时，
这一点非常重要，因为这些操作员会尝试持续调节存储系统。
一旦发现文件系统不存在，操作员就会尝试重新创建文件系统。

交换后，如果现有的挂载要“跟着”旧文件系统到它的新名字，
那么可能需要重新授权 CephX 凭据。一般来说，
为了实现灾难恢复，现有挂载最好继续使用相同的文件系统名字。
所有已经挂载的 CephFS 文件系统都必须重新挂载。
还存在的未刷入操作将丢失。当判断出\
其中一个已交换的文件系统已经准备好供客户端使用时，运行： ::

    ceph fs set <fs> joinable true
    ceph fs set <fs> refuse_client_sessions false

记住，如果正在做灾难恢复交换，其中一个被交换的文件系统\
可以保持离线，以便将来进行分析。


可配置选项
----------
.. Settings

::

    fs set <fs name> max_file_size <size in bytes>

CephFS 允许的最大文件尺寸是可以配置的，默认是 1TB 。如果你想在
CephFS 里存储大文件，也许得把这个限量设置得高些。它是个 64 位的字段。

把 ``max_file_size`` 设置为 0 并不意味着取消这个限量，
而是限制客户端只能创建空文件。


最大文件尺寸以及性能
--------------------
.. Maximum file sizes and performance

在追加到文件、或设置其尺寸时， CephFS 将确保不会超过最大文件\
尺寸限量；但不会影响（数据）是怎样存储的。

有用户创建了一个硕大的文件时（没必要写入什么数据），
某些操作（像删除）会让 MDS 不得不做大量操作，
去检查此文件的尺寸范围内是否有本应存在
（据文件尺寸计算出的）、实际上却不存在的 RADOS 对象。

``max_file_size`` 选项可防止用户创建巨型文件，如 EB 级的文件，\
导致在遇到类似查询状态或删除操作时产生无谓的 MDS 负载。


关闭集群
--------
.. Taking the cluster down

关闭一个 CephFS 集群需要设置 down 标志：

::

    fs set <fs_name> down true

让集群重新上线：

::

    fs set <fs_name> down false

此命令还会恢复 max_mds 以前的值。
MDS 守护进程离线时，会将日志刷入元数据存储池，并停止所有客户端 I/O 。


快速关闭集群以进行删除或灾难恢复
--------------------------------
.. Taking the cluster down rapidly for deletion or disaster recovery

要快速删除文件系统（用于测试）或者快速关闭文件系统和
MDS 守护进程，可以用 ``ceph fs fail`` 命令：

::

    ceph fs fail <fs_name> {--yes-i-really-mean-it}

.. note:: 注意，确认标记是可选的，
   因为只有当 MDS 处于活动状态而且有健康警告
   MDS_TRIM 或 MDS_CACHE_OVERSIZED 时才需要。

此命令会设置一个文件系统标志，
以防止备用机激活这个文件系统（ ``joinable`` 标志）。

也可以通过以下操作手动完成此过程：

::

    fs set <fs_name> joinable false

然后，操作员就可以让所有 rank 失效，这将导致相关的
MDS 守护进程重生为热备。文件系统将处于降级状态。

::

    # For all ranks, 0-N:
    mds fail <fs_name>:<n>

.. note:: 注意，确认标记是可选的，
   因为只有当 MDS 处于活动状态而且有健康警告
   MDS_TRIM 或 MDS_CACHE_OVERSIZED 时才需要。

一旦所有 rank 都处于不活动状态，文件系统就可以删除或者\
留在这个状态以用于其他目的（或许灾难恢复）。

要恢复集群，只需设置 joinable 标志即可：

::

    fs set <fs_name> joinable true


守护进程管理
------------
.. Daemons

大多数可操纵 MDS 的命令都需要一个 ``<role>`` 参数，
它必须是以下三种格式之一：

::

    <fs_name>:<rank>
    <fs_id>:<rank>
    <rank>

可操纵 MDS 守护进程的命令：

::

    ceph mds fail <gid/name/role>

把一个 MDS 守护进程标记为已失效。假如一个 MDS 守护进程在
``mds_beacon_grace`` 秒内都没向监视器发送一条消息，
这个操作就等价于集群自己的操作。如果此守护进程之前是活跃的，
而且有可用的备机，用命令 ``ceph mds fail`` 将迫使业务转移到备机。

如果此 MDS 守护进程事实上仍在运行，那么执行 ``ceph mds fail``
将使之重启；如果它之前是活跃的、并且还有可用的备机，
那么这个“已失效”的守护进程回来后将作为备机。


::

    ceph tell mds.<daemon name> command ...

向 MDS 守护进程发出一个命令，指定 ``mds.*`` 可向所有守护进程\
发送命令。用 ``ceph tell mds.* help`` 命令获取所有可用命令。

::

    ceph mds metadata <gid/name/role>

获取指定 MDS （监视器知道它）的元数据。

::

    ceph mds repaired <role>

把文件系统 rank 标记为已修复。这里不像名字说明的那样，这个命令\
不会更改 MDS ，它操纵的是先前被标记为已损坏的文件系统 rank 。

::

    ceph mds last-seen <name>

了解名字是 ``name`` 的 MDS 上次出现在 FSMap 中的时间。
JSON 格式的输出包括 MDS 最后出现的时间结。
历史信息受到以下 ``mon`` 配置选项的限制：

.. confval:: mon_fsmap_prune_threshold


客户端必须具备的功能
--------------------
.. Required Client Features

有时需要设置门槛，客户端必须支持哪些功能才能与 CephFS 通信。
没有这些功能的客户端可能会干扰其他客户端或出现意料之外的行为。
或者，您可能希望必须具备较新的功能，
以防止较旧且可能存在漏洞的客户端连接。

修改一个文件系统，要求客户端必须具备哪些功能的命令：

::

    fs required_client_features <fs name> add reply_encoding
    fs required_client_features <fs name> rm reply_encoding

罗列所有的 CephFS 功能：

::

    fs feature ls

缺少新增加功能的客户端将被自动驱逐。

以下是当前的 CephFS 功能及其首次出现的版本：

+----------------------------+--------------+-----------------+
| 功能                       | Ceph 版本    | 上游内核        |
+============================+==============+=================+
| jewel                      | jewel        | 4.5             |
+----------------------------+--------------+-----------------+
| kraken                     | kraken       | 4.13            |
+----------------------------+--------------+-----------------+
| luminous                   | luminous     | 4.13            |
+----------------------------+--------------+-----------------+
| mimic                      | mimic        | 4.19            |
+----------------------------+--------------+-----------------+
| reply_encoding             | nautilus     | 5.1             |
+----------------------------+--------------+-----------------+
| reclaim_client             | nautilus     | N/A             |
+----------------------------+--------------+-----------------+
| lazy_caps_wanted           | nautilus     | 5.1             |
+----------------------------+--------------+-----------------+
| multi_reconnect            | nautilus     | 5.1             |
+----------------------------+--------------+-----------------+
| deleg_ino                  | octopus      | 5.6             |
+----------------------------+--------------+-----------------+
| metric_collect             | pacific      | N/A             |
+----------------------------+--------------+-----------------+
| alternate_name             | pacific      | 6.5             |
+----------------------------+--------------+-----------------+
| notify_session_state       | quincy       | 5.19            |
+----------------------------+--------------+-----------------+
| op_getvxattr               | quincy       | 6.0             |
+----------------------------+--------------+-----------------+
| 32bits_retry_fwd           | reef         | 6.6             |
+----------------------------+--------------+-----------------+
| new_snaprealm_info         | reef         | UNKNOWN         |
+----------------------------+--------------+-----------------+
| has_owner_uidgid           | reef         | 6.6             |
+----------------------------+--------------+-----------------+
| client_mds_auth_caps       | squid+bp     | PLANNED         |
+----------------------------+--------------+-----------------+

..
    Comment: use `git describe --tags --abbrev=0 <commit>` to lookup release


CephFS 功能描述


::

    reply_encoding

如果客户端支持此功能，那么 MDS 会将请求回复编码为可扩展格式。


::

    reclaim_client

MDS 允许新客户端承袭另一个（已死亡）客户端的状态。
NFS-Ganesha 使用了这一功能。


::

    lazy_caps_wanted

当落伍的客户端恢复时，如果客户端支持此功能，
那么 mds 只需要重新发放它明确要求的能力即可。


::

    multi_reconnect

当 mds 故障切换时，客户端会向 mds 发送重连消息，
以重建缓存状态。如果 MDS 支持此功能，
客户端就可以将大的重新连接信息分成多条。


::

    deleg_ino

如果客户端支持此功能， MDS 就把 inode 号委托（ delegate ）给客户端。
拥有委托的 inode 号是先决条件，而后客户端才能异步地创建文件。


::

    metric_collect

如果 MDS 支持性能指标功能，客户端可以发给它。


::

    alternate_name

客户端可以设置并理解目录的“别名（ alternate names ）”。
此功能用于支持加密的文件名。

::

    client_mds_auth_caps

要在客户端的 ``mds`` caps 中有效执行 ``root_squash`` ，
客户端必须了解它正在执行 ``root_squash`` 和其他 cap 元数据。
没有此功能的客户端存在风险，会丢弃文件的更新。
建议设置此功能位。


全局配置选项
------------
.. Global settings

::

    ceph fs flag set <flag name> <flag val> [<confirmation string>]

设置全局的 CephFS 标记（即不是特定于某个文件系统的）。
当前，仅有的标记是 enable_multiple ，启用它就可以支持多个
CephFS 文件系统。

有些标志会强迫你用 ``--yes-i-really-mean-it`` 或者类似的语句
（执行时会提示）来确认你的意图。运行这类命令时要三思而后行，
它们通常用于提示非常危险的动作。


.. _advanced-cephfs-admin-settings:

高级选项
--------
.. Advanced

以下这些命令在常规操作中用不到，在遇到异常时才需要。
这些命令若使用不当会产生严重问题，
甚至会导致文件系统无法访问。

::

    ceph mds rmfailed

从失效集合中删除一个 rank 。

::

    ceph fs reset <file system name>

此命令可把文件系统状态（除名字和存储池以外的）重置为默认值。\
所有非 0 rank 都会保存在已停止集里面。

::

    ceph fs new <file system name> <metadata pool name> <data pool name> --fscid <fscid> --force

此命令将创建一个具有指定 **fscid** （文件系统集群 ID ）的文件系统。
当应用程序希望文件系统的 ID 在恢复后仍能保持稳定时，
你可能想要这样做，例如在监视器数据库丢失并重建后。
因此，文件系统 ID 并不总是随着较新的文件系统出现\
而不断增加。
