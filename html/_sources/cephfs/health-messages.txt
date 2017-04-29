=================
 CephFS 健康消息
=================

.. _Cluster health checks:

集群健康检查
============

在文件系统映射图结构（以及封闭式的 MDS 映射图）变为特定状态时，
Ceph 监视器守护进程会产生健康消息。

:消息: mds rank(s) *ranks* have failed
:描述: 一或多个 MDS rank 没能分给守护进程，只有可用的替补守护\
       进程启动后集群才能恢复运转。

------

:消息: mds rank(s) *ranks* are damaged
:描述: 一或多个 MDS rank 遇到了损伤严重的元数据，只有修复这些\
       数据它才能再次启动。

------

:消息: mds cluster is degraded
:描述: 一或多个 MDS rank 现在的状态不是 up 且未在线运行，此\
       问题解决前客户端只能暂停元数据操作。此情形涉及失效、损\
       坏的 rank ，另外也包括已分到 MDS 但还没进入 *active* 状\
       态的 rank （如处于 *replay* 状态的 rank ）。

------

:消息: mds *names* are laggy
:描述: 这些 MDS 守护进程至少有 ``mds_beacon_grace`` 秒（默认为
       15s ）没向监视器发送信标消息（ beacon message ）了，它\
       们本来应该每 ``mds_beacon_interval`` 秒（默认为 4s ）发\
       送一次的，它们可能崩溃了。 Ceph 监视器会自动用灾备替换\
       掉滞后的守护进程。

------

:消息: insufficient standby daemons available
:描述: 一或多个文件系统配置的是需要一定数量的灾备守护进程（包\
       括灾备重放 standby-replay 守护进程），但是集群内却没有\
       足够多的守护进程。非重放的灾备进程可算进任意文件系统（\
       即它们可重叠）。这个警告可用
       ``ceph fs set <fs> standby_count_wanted <count>`` 来配\
       置， ``count`` 配置为 0 时禁用此功能。


.. _Daemon-reported health checks:

守护进程报告的健康检查
======================

MDS 守护进程能定位各种各样不该出现的状况，并通过 ``ceph status``
出示给操作员。这些状况附带了人类可读的消息，另外 JSON 格式的输\
出还有一个以 MDS_HEALTH 打头的唯一代码。

:消息: "Behind on trimming..."
:代码: MDS_HEALTH_TRIM
:描述: CephFS 维护着的元数据日志是切成\
       *日志片段（ log segment ）*\ 的。日志的长度（按片段数量\
       算）是用 ``mds_log_max_segments`` 选项控制的，当片段数\
       量超过配置时， MDS 就开始写回元数据，以便删除（裁剪、
       trim ）最老的片段。如果回写得太慢，或者软件缺陷妨碍了裁\
       剪，这样的健康消息就可能出现。此消息出现的阈值是片段数\
       量达到 ``mds_log_max_segments`` 的两倍。

------

:消息: "Client *name* failing to respond to capability release"
:代码: MDS_HEALTH_CLIENT_LATE_RELEASE, MDS_HEALTH_CLIENT_LATE_RELEASE_MANY
:描述: CephFS 客户端收到了 MDS 发出的\
       *能力（ capabilities ）* ，它就像锁。有时候，比如一个客\
       户端需要访问权， MDS 就会让别的客户端释放它们的能力，如\
       果有客户端没响应、或者有缺陷，它就有可能没及时释放、或\
       者根本不释放。如果某个客户端的响应时间超过了
       ``mds_revoke_cap_timeout`` （默认为 60s ），这条消息就\
       会出现。

------

:消息: "Client *name* failing to respond to cache pressure"
:代码: MDS_HEALTH_CLIENT_RECALL, MDS_HEALTH_CLIENT_RECALL_MANY
:描述: 客户端有各自的元数据缓存，客户端缓存中的条目（比如索引\
       节点）也会存在于 MDS 缓存中，所以当 MDS 需要削减其缓存\
       时（保持在 ``mds_cache_size`` 以下），它也会发消息给客\
       户端让它们削减自己的缓存。如果有客户端没响应或者有缺陷，\
       就会妨碍 MDS 将缓存保持在 ``mds_cache_size`` 以下， MDS
       就有可能耗尽内存而后崩溃。如果某个客户端的响应时间超过了
       ``mds_recall_state_timeout`` （默认为 60s ），这条消息\
       就会出现。

------

:消息: "Client *name* failing to advance its oldest client/flush tid"
:代码: MDS_HEALTH_CLIENT_OLDEST_TID, MDS_HEALTH_CLIENT_OLDEST_TID_MANY
:描述: CephFS 的客户端-MDS 协议有一个名为 *oldest tid* 的字段，\
       可让客户端通知 MDS 哪些请求全部完成了，这样的话它就有可\
       能被 MDS 遗忘。如果一个有缺陷的客户端未能上报这个字段，\
       那么与之相关的 MDS 就不能擅自清理这些请求所占用的资源。\
       如果某个客户端的请求在 MDS 端已完成、但尚未收到客户端上\
       报的 *oldest tid* 值，这样的请求数量超过
       ``max_completed_requests`` （默认为 100000 ）时，此消息\
       就会出现。

------

:消息: "Metadata damage detected"
:代码: MDS_HEALTH_DAMAGE,
:描述: 从元数据存储池读取时，遇到了元数据损坏或丢失的情况。这\
       条消息表明损坏之处已经被妥善隔离了，以使 MDS 继续运作，\
       如此一来，若有客户端访问损坏的子树就返回 IO 错误。关于\
       损坏的细节信息可用 ``damage ls`` 管理套接字命令获取。只\
       要一遇到受损元数据，此消息就会立即出现。

------

:消息: "MDS in read-only mode"
:代码: MDS_HEALTH_READ_ONLY,
:描述: MDS 已进入只读模式，任何尝试修改元数据的操作都会收到
       EROFS 错误代码。在 MDS 写入元数据存储池时遇到写错误、或\
       者管理员用 *force_readonly* 管理套接字命令强行设置时，
       MDS 会进入只读模式。

------

:消息: *N* slow requests are blocked"
:代码: MDS_HEALTH_SLOW_REQUEST,
:描述: 一或多个客户端请求没有及时完成，说明 MDS 要么跑得太慢、\
       要么 RADOS 集群没及时确认日志写操作、或者软件有缺陷。可\
       用 ``ops`` 管理套接字命令罗列未完成的元数据操作。如果有\
       客户端请求花费的时间超过 ``mds_op_complaint_time`` （默\
       认为 30s ），此消息就会出现。

------

:消息: "Too many inodes in cache"
:代码: MDS_HEALTH_CACHE_OVERSIZED
:描述: MDS 没能成功削减缓存，未能降到管理员设置的上限之下。如\
       果 MDS 缓存涨得太大，守护进程可能会耗尽内存然后崩溃。如\
       果实际的缓存尺寸（按索引节点算）比 ``mds_cache_size``
       （默认为 100000 ）大至少 50% ，这个消息就会出现。
