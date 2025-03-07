.. _cephfs-health-messages:

=================
 CephFS 健康消息
=================
.. CephFS health messages

集群健康检查
============
.. Cluster health checks

在文件系统映射图结构（以及封闭式的 MDS 映射图）变为特定状态时，
Ceph 监视器守护进程会产生健康消息。

:消息: mds rank(s) *ranks* have failed
:描述: 一或多个 MDS rank 没能分给守护进程，只有可用的替补守护进程启动后\
       集群才能恢复运转。

------

:消息: mds rank(s) *ranks* are damaged
:描述: 一或多个 MDS rank 遇到了损伤严重的元数据，只有修复这些\
       数据它才能再次启动。

------

:消息: mds cluster is degraded
:描述: 一或多个 MDS rank 现在的状态不是 up 且未在线运行，此问题解决前客户端\
       只能暂停元数据操作。此情形涉及失效、损坏的 rank ，另外也包括已分到 MDS
       但还没进入 *active* 状态的 rank （如处于 *replay* 状态的 rank ）。

------

:消息: mds *names* are laggy
:描述: 这些 MDS 守护进程至少有 ``mds_beacon_grace`` 秒（默认为 15s ）没\
       向监视器发送信标消息（ beacon message ）了，它们本来应该\
       每 ``mds_beacon_interval`` 秒（默认为 4s ）发送一次的，它们可能崩溃了。
       Ceph 监视器会自动用灾备替换掉滞后的守护进程。

------

:消息: insufficient standby daemons available
:描述: 一或多个文件系统配置的是需要一定数量的灾备守护进程
       （包括热备 standby-replay 守护进程），但是集群内却没有足够多的守护进程。
       非重放的灾备进程可算进任意文件系统（即它们可重叠）。这个警告可用
       ``ceph fs set <fs> standby_count_wanted <count>`` 来配置，
       ``count`` 配置为 0 时禁用此功能。


守护进程报告的健康检查
======================
.. Daemon-reported health checks

MDS 守护进程能定位各种各样不该出现的状况，并通过 ``ceph status``
出示给操作员。这些状况附带了人类可读的消息，另外 JSON 格式的输\
出还有一个以 MDS_HEALTH 打头的唯一代码。

.. highlight:: console

``ceph health detail`` 显示了当前情境的细节。
下面是一个集群遇到 MDS 相关的性能问题时的典型健康报告::

  $ ceph health detail
  HEALTH_WARN 1 MDSs report slow metadata IOs; 1 MDSs report slow requests
  MDS_SLOW_METADATA_IO 1 MDSs report slow metadata IOs
     mds.fs-01(mds.0): 3 slow metadata IOs are blocked > 30 secs, oldest blocked for 51123 secs
  MDS_SLOW_REQUEST 1 MDSs report slow requests
     mds.fs-01(mds.0): 5 slow requests are blocked > 30 secs

其中，例如， ``MDS_SLOW_REQUEST`` 是表达当前情境的唯一代码，
指出有一些请求花费很长时间才完成。之后的内容\
显示了此消息的级别和出现慢请求的是哪些 MDS 守护进程。

本页罗列了 MDS 守护进程抛出的健康检查消息。要看其它守护进程抛出的检查消息，
见 :ref:`health-checks` 。

``MDS_TRIM``
------------

  消息
    "Behind on trimming..."
    修剪操作落后了。
  描述
    CephFS 维护着的元数据日志是切成\ *日志片段（ log segment ）*\ 的。
    日志的长度（按片段数量算）是用 ``mds_log_max_segments`` 选项控制的，
    当片段数量超过配置时， MDS 就开始写回元数据，以便删除（裁剪、 trim ）
    最老的片段。如果回写得太慢，或者软件缺陷妨碍了裁剪，
    这样的健康消息就可能出现。此消息出现的阈值是由配置选项
    ``mds_log_warn_factor`` 控制的，默认是 2.0 。

``MDS_HEALTH_CLIENT_LATE_RELEASE``, ``MDS_HEALTH_CLIENT_LATE_RELEASE_MANY``
---------------------------------------------------------------------------

  消息
    "Client *name* failing to respond to capability release"
    名为 *name* 的客户端没有回应能力释放请求。
  描述
    CephFS 客户端收到了 MDS 发出的\ *能力（ capabilities ）* ，它就像锁。
    有时候，比如一个客户端需要访问权， MDS 就会让别的客户端释放它们的能力，
    如果有客户端没响应、或者有缺陷，它就有可能没及时释放、或者根本不释放。
    如果某个客户端的响应时间超过了 ``session_timeout`` （默认为 60s ），
    这条消息就会出现。

``MDS_CLIENT_RECALL``, ``MDS_HEALTH_CLIENT_RECALL_MANY``
--------------------------------------------------------

  消息
    "Client *name* failing to respond to cache pressure"
    名为 *name* 的客户端没有回应缓存压力。
  描述
    客户端有各自的元数据缓存，客户端缓存中的条目（比如索引节点）
    也会存在于 MDS 缓存中，所以当 MDS 需要削减其缓存时
    （为了使之保持在 ``mds_cache_memory_limit`` 以下），
    它也会发消息给客户端让它们削减各自的缓存。如果有客户端没响应或者有缺陷，
    就会妨碍 MDS 将缓存保持在其限额以下， MDS 就有可能耗尽内存而后崩溃。
    如果某个客户端在最后一个 ``mds_recall_warning_decay_rate`` 秒数内都\
    没能释放到 ``mds_recall_warning_threshold``
    （以 ``mds_recall_max_decay_rate`` 为半衰期衰减）之下，这条消息就会出现。

``MDS_CLIENT_OLDEST_TID``, ``MDS_CLIENT_OLDEST_TID_MANY``
---------------------------------------------------------

  消息
    "Client *name* failing to advance its oldest client/flush tid"
    名为 *name* 的客户端没能推进它最老的客户端、回刷 tid 。
  描述
    CephFS 的客户端-MDS 协议有一个名为 *oldest tid* 的字段，\
    可让客户端通知 MDS 哪些请求全部完成了，这样的话它就可以被 MDS 遗忘。
    如果一个有缺陷的客户端未能上报这个字段，\
    那么与之相关的 MDS 就不能擅自清理这些请求所占用的资源。\
    如果某个客户端的请求在 MDS 端已完成、但尚未收到客户端上报的
    *oldest tid* 值，这样的请求数量超过 ``max_completed_requests``
    （默认为 100000 ）时，此消息就会出现。
    MDS 用于修剪已完成的客户端请求（或刷回）的最后一个 tid ，
    会包含在 `session ls` （或 `client ls` ）命令里，作为调试的辅助信息。

``MDS_DAMAGE``
--------------

  消息
    "Metadata damage detected"
    探测到了元数据损坏。
  描述
    从元数据存储池读取时，遇到了元数据损坏或丢失的情况。这\
    条消息表明损坏之处已经被妥善隔离了，以使 MDS 继续运作，\
    如此一来，若有客户端访问损坏的子树就返回 IO 错误。关于\
    损坏的细节信息可用 ``damage ls`` 管理套接字命令获取。只\
    要一遇到受损元数据，此消息就会立即出现。

``MDS_HEALTH_READ_ONLY``
------------------------

  消息
    "MDS in read-only mode"
    MDS 处于只读模式。
  描述
    MDS 已进入只读模式，任何尝试修改元数据的操作都会收到
    EROFS 错误代码。在 MDS 写入元数据存储池时遇到写错误、或\
    者管理员用 *force_readonly* 管理套接字命令强行设置时，
    MDS 会进入只读模式。

``MDS_SLOW_REQUEST``
--------------------

  消息
    "*N* slow requests are blocked"
    *N* 个慢请求被阻塞了。
  描述
    一或多个客户端请求没有及时完成，说明 MDS 要么跑得太慢、\
    要么 RADOS 集群没及时确认日志写操作、或者软件有缺陷。
    可用 ``ops`` 管理套接字命令罗列未完成的元数据操作。
    如果有客户端请求花费的时间超过 ``mds_op_complaint_time``
    （默认为 30s ），此消息就会出现。

``MDS_CACHE_OVERSIZED``
-----------------------

  消息
    "Too many inodes in cache"
    缓存里的 inode 过于多了。

  描述
    MDS 没能成功削减缓存，未能降到管理员设置的上限之下。\
    如果 MDS 缓存涨得太大，守护进程可能会耗尽内存然后崩溃。\
    默认情况下，如果实际的缓存尺寸（在内存里的）比\
    ``mds_cache_memory_limit`` （默认为 4GB ）大至少 50% ，\
    这个消息就会出现。更改 ``mds_health_cache_threshold``
    可设置超出的告警比率。

``FS_WITH_FAILED_MDS``
----------------------

  消息
    "Some MDS ranks do not have standby replacements"
    有些 MDS rank 没有备胎。

  描述
    通常，失败的 MDS 会被备用 MDS 取代，这个过程是瞬间的、不会被认为是致命的。
    然而，如果没有可用的备用 MDS 来取代活跃的 MDS rank ，就会产生这条健康警告。

``MDS_INSUFFICIENT_STANDBY``
----------------------------

  消息
    "Insufficient number of available standby(-replay) MDS daemons than configured"
    热备 MDS 守护进程不够用，少于配置的。

  描述
    ``standby_count_wanted`` 配置变量可以指定热备 MDS 守护进程的最低数量。
    当可用的热备 MDS 守护进程少于配置的数量时，就会产生这个健康警报。

``FS_DEGRADED``
---------------

  消息
    "Some MDS ranks have been marked failed or damaged"
    一些 MDS rank 已经失败或损坏了。

  描述
    在一个或多个 MDS rank 由于不可恢复的错误停留在失败的或损坏的状态时出现。
    当一个（或多个） rank 离线时，这个文件系统就可能部分或完全不可用。

``MDS_UP_LESS_THAN_MAX``
----------------------------

  消息
    "Number of active ranks are less than configured number of maximum MDSs"
    活跃的 rank 数量少于配置的最大 MDS 数。

  描述
    MDS rank 的最大数量可以用 ``max_mds`` 配置变量来设置。
    当 MDS rank 少于这个配置的值时，就会产生这个健康警报。

``MDS_ALL_DOWN``
----------------------------

  消息
    "None of the MDS ranks are available (file system offline)"
    没有可用的 MDS rank （文件系统离线）。

  描述
    所有 MDS rank 都不可用，导致文件系统完全离线。

``MDS_CLIENTS_LAGGY``
----------------------------
  消息
    "Client *ID* is laggy; not evicted because some OSD(s) is/are laggy"
    客户端 *ID* 滞后了，没驱逐是因为有些 OSD 滞后了。

  描述
    如果 OSD 滞后（由于某些条件，如网络割接等），那么它可能导致客户端滞后
    （会话可能空闲或无法刷回脏数据以撤销能力）。
    如果 ``defer_client_eviction_on_laggy_osds`` 设置为 true
    （默认为 true ），就不会驱逐客户端，并因此产生这条健康警告。

``MDS_CLIENTS_BROKEN_ROOTSQUASH``
---------------------------------
  消息
    "X client(s) with broken root_squash implementation (MDS_CLIENTS_BROKEN_ROOTSQUASH)"
    X 个客户端的 root_squash 实现有问题（ MDS_CLIENTS_BROKEN_ROOTSQUASH ）

  描述
    在 root_squash 中发现了一个缺陷，它可能会丢失带有 root_squash 能力\
    的客户端所做的更改。要修正次问题需要更改协议，而且需要升级客户端。

    这是一个 HEALTH_ERR 警告，因为存在不一致和丢失数据的危险。
    建议同时升级客户端、在此期间停止使用 root_squash ，或者根据需要关闭警告。

    要驱逐并永久阻止有问题的客户端连接到集群，
    请设置 ``required_client_feature`` 位的 ``client_mds_auth_caps`` 。

``MDS_ESTIMATED_REPLAY_TIME``
-----------------------------
  消息
    "HEALTH_WARN Replay: x% complete. Estimated time remaining *x* seconds"
    HEALTH_WARN 重放： x% 已完成，预计剩余 *x* 秒。

  描述
    当一个 MDS 的日志重放耗时超过 30 秒时，此消息会显示预计完成时间。
