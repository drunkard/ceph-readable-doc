.. _health-checks:

==========
 健康检查
==========
.. Health checks

概览
====

Ceph 集群可能产生的健康消息是有限的——它们通通被定义为\
*健康检查*\ ，都有唯一标识符。

标识符（ identifier ）是一个简洁的人类可读字符串，也就是说，
标识符的可读性与典型的变量名基本相同。其目的是使工具
（比如监视器和用户界面）能够理解健康检查，并以字面含义的方式呈现健康检查结果。

本页列出了监视器和管理器守护进程产生的健康检查。此外，
您还可能看到来自 CephFS MDS 守护进程的健康检查
（见 :ref:`cephfs-health-messages` ），以及由 ``ceph-mgr`` 模块定义的健康检查。


状态定义
========
.. Definitions

监视器
------
.. Monitor

DAEMON_OLD_VERSION
__________________

一个或多个 Ceph 守护进程正在运行旧版的 Ceph 。如果检测到多个版本，
就会产生健康检查。这种情况存在的时间必须大于 ``mon_warn_older_version_delay``
（默认为一周），健康检查才会触发。这样，大多数升级就可以继续进行，
而不会引发预期和短暂的警告。如果升级暂停的时间较长，可以执行
``ceph health mute DAEMON_OLD_VERSION --sticky`` 来达到 ``health mute`` 。
不过，请务必在升级完成后运行 ``ceph health unmute DAEMON_OLD_VERSION`` ，
这样以后出现意外情况就不会被屏蔽。

MON_DOWN
________

一个或多个 Ceph 监视器守护进程宕机了。
集群要求大多数（一半以上）已装好的监视器可用。
当一个或多个监视器宕机时，客户端可能会更难与集群建立初始连接，
因为它们可能需要另外尝试其他 IP 地址\
才能连上正常运行的监视器。

应尽快恢复或重启宕机的监视器守护进程，
以降低更多监视器故障可能导致服务中断的风险。

MON_CLOCK_SKEW
______________

运行着 Ceph 监视器守护进程的主机，它们的时钟不同步。
如果集群检测到的时钟偏移大于 :confval:`mon_clock_drift_allowed` ），
就会触发这个健康检查。

解决这一问题的最佳方法是用比如传统的 ``ntpd``
或较新的 ``chrony`` 等工具同步时钟。
为提高可靠性，最好配置 NTP 守护进程与多个内部和外部时钟源同步；
此协议能自主确定最佳可用源。
另外，让 Ceph 监视器主机上的 NTP 守护进程相互同步也有好处，
与基准时间之间的误差\ **准确**\ 相比，
监视器之间相互同步甚至更重要。

如果保持时钟紧密同步不切实际，
可以增大 :confval:`mon_clock_drift_allowed` 阈值。
但是，该值必须大大低于 ``mon_lease`` 间隔，
这样监控器集群才能正常运行。
用高质量的 NTP 或 PTP 配置实现亚毫秒级的时间同步并不难，
因此，很少有必要更改此值。


MON_MSGR2_NOT_ENABLED
_____________________

启用了 :confval:`ms_bind_msgr2` 选项，但集群 monmap 中\
有一个或多个监视器没有配置绑定到 v2 端口。
这意味着 msgr2 协议特有的功能（如加密）\
在一些或所有连接上不可用。

在大多数情况下，执行下列命令可纠正这一问题： 

.. prompt:: bash $

   ceph mon enable-msgr2

运行此命令后，任何配置为监听旧默认端口（6789）
的监视器将继续监听 6789 上的 v1 连接，
并开始监听新默认端口 3300 上的 v2 连接。

如果监视器配置为在非标准端口
（即 6789 以外的端口）上监听 v1 连接，
则需要手动修改 monmap 。


MON_DISK_LOW
____________

一个或多个监视器的存储空间不足。
如果监视器数据库（通常为 ``/var/lib/ceph/<fsid>/mon.<monid>`` ）
所在文件系统上的可用空间百分比低于 :confval:`mon_data_avail_warn` 百分比数值
（默认：30%），就会触发这个健康检查。

该警报可能表明系统中有其他进程或用户正在填满\
监视器所用的文件系统。它还可能预示着\
监视器数据库过大（参阅下文 ``MON_DISK_BIG`` ）。
另一种常见情况是，之前为了排除故障提高了 Ceph 日志子系统级别，
但随后并未恢复到默认级别。
持续的详尽日志记录很容易填满 ``/var/log`` 所在的文件系统。
如果要修剪当前已打开的日志，记得重启，
或者指示 syslog 或其他守护进程重新打开这个日志文件。
另一种常见情形是用户或者进程曾向 ``/tmp`` 或 ``/var/tmp``
写入过大量数据，而它们可能位于同一个文件系统之内。

如果无法释放空间，那就可能需要\
把监视器的数据目录移到其他存储设备或文件系统
（此搬迁过程必须在监视器守护进程未运行时进行）。


MON_DISK_CRIT
_____________

一个或多个监视器的存储空间严重不足。如果监视器数据库
（通常为 ``/var/lib/ceph/<fsid>/mon.<monid>`` ）所在文件系统的\
可用空间百分比低于 :confval:`mon_data_avail_crit`
百分比值（默认：5%），就会触发这个健康检查。
参阅上文的 ``MON_DISK_LOW`` 。


MON_DISK_BIG
____________

一个或多个监视器的数据库容量非常大。
如果监控器数据库的尺寸大于
:confval:`mon_data_size_warn` （默认值： 15 GiB ）。

数据库过于大是不正常的，但并不一定表示有问题。
当有一些归置组很长时间没有达到 ``active+clean`` 状态，
或者最近发生了大量的集群恢复、扩展或拓扑变化时，
监视器数据库的尺寸可能会增大。
建议在进行大规模集群变更时，每周让集群“休整”至少数小时。

此警报还表明监视器的数据库可能没有正确压缩，
这个问题在某些旧版的 RocksDB 中也曾出现过。
用 ``ceph daemon mon.<id> compact`` 强制压缩\
可以大大降低数据库的存储空间使用。

此警报还表明监视器可能存在一个缺陷，
导致它无法修剪其存储的集群元数据。
如果问题持续存在，请报告此 bug 。

要调整警告阈值，执行下列命令： 

.. prompt:: bash $

   ceph config set global mon_data_size_warn <size>


MON_NETSPLIT
____________

Ceph 监视器集群内发生了网络分裂。基于对每个连接持续更新的评分记录，
当至少有一个监视器检测到：至少有两个 Ceph 监视器连接不上或不可达时，
就会触发此健康检查。只有集群配置了三个以上 Ceph 监视器、
并采用 ``connectivity`` :ref:`选举策略 <changing_monitor_elections>`\ 时，\
才会出现此警告。

为减少因网络瞬态问题引发的误报，
检测到的网络分裂不会立即作为健康警告报告。
相反，它们必须持续至少 :confval:`mon_netsplit_grace_period` 秒\
（默认值：9 秒）才会报告出去。
如果网络分裂在此宽限期内自动恢复，就不会发出健康警告。

网络分裂有两种报告方式：

- 对于地理位置级的网络分裂（如 "Netsplit detected between dc1 and dc2"
  ［在 dc1 和 dc2 之间检测到了网络分裂］），
  是一个位置上的所有监视器无法与另一个位置上的所有监视器通信。
- 对于单个监视器的网络分裂（如 "Netsplit detected between mon.a and mon.d"
  ［在 mon.a 和 mon.d 之间检测到网络分裂］），是个别位于其他地方的监视器断开连接。

系统会尽可能优先报告最高拓扑级别（ ``datacenter`` 、 ``rack`` 等）的问题，
以更准确地帮助运维人员确定基础设施级别的网络问题。

要调整宽限期阈值，执行下列命令：

.. prompt:: bash #

   ceph config set mon mon_netsplit_grace_period <seconds>

要完全禁用宽限期（立即报告），把此值设置为 0 ：

.. prompt:: bash #

   ceph config set mon mon_netsplit_grace_period 0


AUTH_INSECURE_GLOBAL_ID_RECLAIM
_______________________________

重新连接到监视器时，连接到集群的一个或多个客户端/守护进程没有安全地重申它们的
``global_id`` （标识集群中每个成员的唯一数字标识符）。
这样的客户端还是会被允许连接，原因是：
:confval:`auth_allow_insecure_global_id_reclaim` 选项被设置成了 ``true``
（在所有 Ceph 客户端都完成升级之前，这是必要的），
而且 :confval:`auth_expose_insecure_global_id_reclaim` 选项被设置成了 ``true``
（可以让监视器更快地检测到“不安全重申（ insecure reclaim ）”的客户端，
以便稍后完成初始身份验证后、强制这些客户端立即重连）。

要确定哪些客户端在用未打补丁的 Ceph 客户端代码，
执行下列命令： 

.. prompt:: bash $

   ceph health detail

如果收集到连入单个监控器的客户端们的转储，
并检查转储输出中的 ``global_id_status`` 字段，
你就能看到这些客户端的 ``global_id`` 重申行为。
这里的 ``reclaim_insecure`` 表示客户端未打补丁，并导致了这次健康检查。
要生成客户端转储，执行下列命令： 

.. prompt:: bash $

   ceph tell mon.\* sessions

我们强烈建议将系统中的所有客户端升级到\
能正确重申 ``global_id`` 值的 Ceph 更高版本。
升级所有客户端后，执行下列命令，不再允许不安全的重连：

.. prompt:: bash $

   ceph config set mon auth_allow_insecure_global_id_reclaim false

如果立即升级所有客户端办不到，可以临时把这个警告\
:ref:`消音 <rados-monitoring-muting-health-checks>`\ ，
执行下列命令：

.. prompt:: bash $

   ceph health mute AUTH_INSECURE_GLOBAL_ID_RECLAIM 1w   # 1 week

虽然我们\ **不建议**\ 你这样做，但是，确实可以永久禁用此警告，
执行下列命令：

.. prompt:: bash $

   ceph config set mon mon_warn_on_insecure_global_id_reclaim false


AUTH_INSECURE_GLOBAL_ID_RECLAIM_ALLOWED
_______________________________________

Ceph 目前的配置允许客户端们通过不安全的过程重连监视器，
并重申它们之前的 ``global_id`` 。之所以允许这种重申，
是因为 :confval:`auth_allow_insecure_global_id_reclaim` 默认被设置为 ``true`` 。
即使现有的 Ceph 客户端升级到能正确、
安全地重申其 ``global_id`` 的 Ceph 更新版本后，
可能还是有必要保留此设置。

如果 ``AUTH_INSECURE_GLOBAL_ID_RECLAIM`` 健康检查未被触发，
并且 :confval:`auth_expose_insecure_global_id_reclaim` 设置未被禁用
（默认已启用），那就说明当前没有连入需要升级的客户端。
在这种情况下，可以安全地禁用不安全的 ``global_id`` 重申，
执行下列命令： 

.. prompt:: bash $

   ceph config set mon auth_allow_insecure_global_id_reclaim false

相反，如果还有客户端需要升级，
那就可以临时\ :ref:`消音 <rados-monitoring-muting-health-checks>`\ 此警告，
执行下列命令：

.. prompt:: bash $

   ceph health mute AUTH_INSECURE_GLOBAL_ID_RECLAIM_ALLOWED 1w   # 1 week

虽然我们\ **不建议**\ 这样做，但是，确实可以永久禁用此警告，
执行下列命令：

.. prompt:: bash $

   ceph config set mon mon_warn_on_insecure_global_id_reclaim_allowed false


管理器
------
.. Manager

MGR_DOWN
________

现在，所有 Ceph 管理器守护进程都已关闭。通常情况下，
集群应该有至少一个运行着的管理器（ ``ceph-mgr`` ）守护进程。
如果没有管理器守护进程在运行，集群监控自身的能力将受到影响，
一部分管理 API 功能将不可用（例如，仪表盘将无法工作，
报告指标或运行时状态的大多数 CLI 命令将阻塞）。
不过，集群仍能执行客户端 I/O 操作并从故障中恢复。

应尽快重启宕机的管理器守护进程，
以确保可以监控集群（例如，这样 ``ceph -s`` 信息\
就可用且是最新的，这样 Prometheus 也能抓取指标）。


MGR_MODULE_DEPENDENCY
_____________________

已启用的管理器模块，其依赖性检查失败。
这种健康检查通常会附带模块对问题的解释信息。

例如，模块可能报告未安装所需软件包：
在这种情况下，应安装所需软件包并重启\
管理器守护进程。

这个健康检查仅适用于已启用的模块。
如果模块未启用，可以从 ``ceph module ls`` 的输出中查看\
它是否存在依赖问题。


MGR_MODULE_ERROR
________________

管理器模块出现意外错误。通常，
这意味着模块的 ``serve()`` 函数出现了一个未处理的异常。
如果异常自身没有提供有用的描述，
那么，给人看的错误描述可能也是措辞模糊。

这样的健康检查说明可能存在缺陷：如果您认为遇到了 bug ，
请开一个 Ceph bug 报告。

不过，如果您认为这个错误是暂时的，可以重新启动管理器守护进程，
或者在活跃守护进程上用 ``ceph mgr fail`` 强制切换到另一个守护进程。


OSDs
----

OSD_DOWN
________

至少有一个 OSD 被标记成了 ``down`` 状态，其 ``ceph-osd`` 守护进程\
或者它们所在主机可能已经奔溃或关闭了、
或者是对端 OSD 与此 OSD 之间的公共网/私有网不通。
常见起因有守护进程停止或崩溃、主机挂了、或者网络故障。

核实一下此主机是否健康、守护进程是否启动、网络是否正常。
如果那个守护进程崩溃了，其守护进程日志文件
（ ``/var/log/ceph/ceph-osd.*`` ）里会包含调试信息。


OSD_<crush type>_DOWN
_____________________

(例如 OSD_HOST_DOWN, OSD_ROOT_DOWN)

某一个 CRUSH 子树里的所有 OSD 都被标记成了 down （例如，一台主机上的所有 OSD ）。


OSD_ORPHAN
__________

CRUSH 图分级结构里提到了这个 OSD ，但它并不存在。

CRUSH 图分级结构里的这个 OSD 可以用以下命令删除：

.. prompt:: bash $

  ceph osd crush rm osd.<id>


OSD_OUT_OF_ORDER_FULL
_____________________

有关 ``nearfull`` 、 ``backfillfull`` 、 ``full`` 、和/或 ``failsafe_full``
的利用率阈值不再增长。特别是，出现了以下情形： ``nearfull`` < ``backfillfull`` 、
``backfillfull`` < ``full`` 、和 ``full`` < ``failsafe_full`` 。
这样集群会出现意外行为。

要调整这些利用率阈值，执行下列命令：

.. prompt:: bash $

   ceph osd set-nearfull-ratio <ratio>
   ceph osd set-backfillfull-ratio <ratio>
   ceph osd set-full-ratio <ratio>


OSD_FULL
________

至少一个 OSD 已经超过了 ``full`` 阈值，并且导致集群不再允许写入。

要按存储池检查利用率，执行下列命令：

.. prompt:: bash #

   ceph df

要查看当前设置的 ``full`` 比率，执行下列命令：

.. prompt:: bash #

   ceph osd dump | grep full_ratio

恢复写入服务可用性的短期解决方法是把 full 阈值提高一点。
可以执行下列命令：

.. prompt:: bash #

   ceph osd set-full-ratio <ratio>

关于排查 OSD 空余空间问题的详细讨论，参见
:ref:`OSD 故障排查 <no-free-drive-space>`\ 。

应该在恰当的 CRUSH 故障域里面额外部署一些 OSD ，
以增加容量，并/或删除现有数据，以释放集群空间。
一个不易察觉的情况是，之前用 ``rados bench`` 工具测试一个或多个存储池的性能，
而由此产生的 RADOS 对象随后却没有清理。
你可以对每个存储池调用 ``rados ls`` 来检查这种情况，
并查找以 ``bench`` 或其他任务名字打头的对象。
可以手动删除这些对象，以回收容量，但需要非常非常谨慎。


OSD_BACKFILLFULL
________________

一个或多个 OSD 已超过 ``backfillfull`` 阈值，
或当前已映射好的回填完成后\ *会*\ 超过此阈值，
它会妨碍数据重均衡到这个 OSD 。这个警报是一个预先警告，
表明重均衡可能无法完成，而且集群已接近用满。

要查看每个存储池的利用率，执行下列命令：

.. prompt:: bash $

   ceph df

关于排查 OSD 空余空间问题的详细讨论，参见
:ref:`OSD 故障排查 <no-free-drive-space>`\ 。


OSD_NEARFULL
____________

一个或多个 OSD 已超过 ``nearfull`` 阈值。
这个警报是一个预先警告，表明集群快满了。

要查看每个存储池的利用率，执行下列命令：

.. prompt:: bash $

   ceph df

关于排查 OSD 空余空间问题的详细讨论，参见
:ref:`OSD 故障排查 <no-free-drive-space>`\ 。


OSDMAP_FLAGS
____________

已经设置了一个或多个需要关注的集群标志。这些标志包括：

* ``full`` - 此集群被标记为满了，且不能提供写入服务。
* ``pauserd``, ``pausewr`` - 有暂停的读出和写入
* ``noup`` - 不允许 OSD 们启动
* ``nodown`` - OSD 故障报告正被忽略，
  这意味着监视器不会把 OSD 标记为 ``down`` 。
* ``noin`` - 之前被标记成 ``out`` 的 OSD 们，
  重启之后不会重新被标记为 ``in`` 。
* ``noout`` - 处于 ``down`` 状态的 OSD 们，
  在经过配置的时间间隔后不会被标记为 ``out`` 。
* ``nobackfill``, ``norecover``, ``norebalance`` - 恢复或数据重均衡被暂停了。
* ``noscrub``, ``nodeep_scrub`` - 洗刷被禁用了
* ``notieragent`` - cache-tiering （缓存分级）活动被暂停了

除了 ``full`` 以外，这些标记都可以用下列命令设置或清除：

.. prompt:: bash $

   ceph osd set <flag>
   ceph osd unset <flag>


OSD_FLAGS
_________

至少一个 OSD 或 CRUSH {节点、设备类别} 设置了需要关注的标记。
这些标记包括：

* ``noup``: 不允许启动这些 OSD 们
* ``nodown``: 这些 OSD 的故障报告会被忽略
* ``noin``: 在之前的一次故障中，如果这些 OSD 被自动标记成了 ``out`` ，
  在它们启动后不会被标记为 ``in``
* ``noout`` - 处于 ``down`` 状态的 OSD 们，
  在经过配置的时间间隔后不会被标记为 ``out`` 。

这些标记可以批量设置和清除，执行下列命令：

.. prompt:: bash $

  ceph osd set-group <flags> <who>
  ceph osd unset-group <flags> <who>

例如：

.. prompt:: bash $

  ceph osd set-group noup,noout osd.0 osd.1
  ceph osd unset-group noup,noout osd.0 osd.1
  ceph osd set-group noup,noout host-foo
  ceph osd unset-group noup,noout host-foo
  ceph osd set-group noup,noout class-hdd
  ceph osd unset-group noup,noout class-hdd


OLD_CRUSH_TUNABLES
__________________

CRUSH 图在使用很老的选项，应该更新它。还能使用（即，可连接此集群的\
最老客户端版本号）而不会触发此健康告警的最老可调选项由
:confval:`mon_crush_min_required_version` 配置选项决定。
详情见 ref:`crush-map-tunables` 。


OLD_CRUSH_STRAW_CALC_VERSION
____________________________

CRUSH 图在使用一个比较老的、非最优方法为 ``straw`` 桶计算中间权重值。

应该更新 CRUSH 图以使用较新的方法（即： ``straw_calc_version=1`` ）。
详情参阅 :ref:`crush-map-tunables` 。


CACHE_POOL_NO_HIT_SET
_____________________

至少有一个缓存存储池没有配置 *hit set* 来跟踪利用率。
这个问题妨碍分层代理无法找出需要从缓存中刷回和驱逐的冷对象。

要配置缓存存储池的 hit set ，执行下列命令：

.. prompt:: bash $

   ceph osd pool set <poolname> hit_set_type <type>
   ceph osd pool set <poolname> hit_set_period <period-in-seconds>
   ceph osd pool set <poolname> hit_set_count <number-of-hitsets>
   ceph osd pool set <poolname> hit_set_fpp <target-false-positive-rate>


OSD_NO_SORTBITWISE
__________________

没有在跑 luminous v12.y.z 之前的 OSD ，但却没有设置 ``sortbitwise`` 标记。

运行 Luminous v12.y.z 或更高版软件的 OSD 要想启动，必须设置 ``sortbitwise`` 标记。
要想安全地设置这个标记，执行下列命令：

.. prompt:: bash $

   ceph osd set sortbitwise


OSD_FILESTORE
_____________

如果 OSD 正在运行旧的 Filestore 后端，就发出警告。
Filestore OSD 后端已废弃；从 Ceph Luminous 版开始，
BlueStore 后端就成了默认的对象存储库。

Filestore OSD 不支持 mClock 调度器。因此，
Filestore OSD 的默认 ``osd_op_queue`` 设置成了 ``wpq`` ，
即使用户试图更改也会强制施行。

.. prompt:: bash $

   ceph report | jq -c '."osd_metadata" | .[] | select(.osd_objectstore | contains("filestore")) | {id, osd_objectstore}'

.. important:: 要升级到 Reef 或者之后的版本，
   必须首先把 Filestore OSD 迁移到 BlueStore 。

如果您正在把 Reef 之前的版本升级到 Reef 或更高版本，
但不方便立即\
:ref:`把 Filestore OSD 迁移到 BlueStore <rados_operations_bluestore_migration>`\ ，
可以执行下列命令，暂时\
:ref:`消音 <rados-monitoring-muting-health-checks>` 此警报： 

.. prompt:: bash $

   ceph health mute OSD_FILESTORE

由于从 Filestore OSD 迁移到 BlueStore 需要相当长的时间才能完成，
我们建议您在更新 Reef 或后续版本之前尽早开始此步骤。


OSD_UNREACHABLE
_______________

至少有一个 OSD 注册的 v1/v2 公网地址\
不在定义的 :confval:`public_network` 子网内，
导致这些无法访问的 OSD 无法与 ceph 客户端正常通信。

即使这些无法访问的 OSD 处于 up 状态，
rados 客户端也会由于这种不一致而挂起，直到 TCP 超时才出错。


POOL_FULL
_________

至少有一个存储池已经达到了配额，且不再允许写入。

要查看存储池配额和利用率，执行下列命令：

.. prompt:: bash $

   ceph df detail

关于 ``ceph df`` 命令的更多细节，
见\ :ref:`rados-monitoring-pool-usage` 。

如果你想提高此存储池的配额，执行下列命令：

.. prompt:: bash $

   ceph osd pool set-quota <poolname> max_objects <num-objects>
   ceph osd pool set-quota <poolname> max_bytes <num-bytes>

如果不想，可以删除一些现有数据，以降低利用率。


BLUEFS_SPILLOVER
________________

至少有一个使用 BlueStore 后端的 OSD 已分配了 DB 分区
（就是用于存储元数据的存储空间，通常位于速度较快的设备上），
但由于这部分空间已被填满，元数据已经“溢出（ spilled over ）”到速度较慢的设备上。
这未必是出错的情况，甚至也不是意料之外的行为，但可能导致性能下降。
如果管理员规划了让所有元数据都能装到速度较快的设备上，
那么此警报说明提供的空间不足。

要在所有 OSD 上禁用此警报，执行下列命令：

.. prompt:: bash $

   ceph config set osd bluestore_warn_on_bluefs_spillover false

另外，还能禁用某一个指定 OSD 的警告，执行下列命令：

.. prompt:: bash $

   ceph config set osd.123 bluestore_warn_on_bluefs_spillover false

为了保证有更多的元数据空间，可以销毁并重新配置相关的 OSD 。
这个过程需要数据迁移和恢复。

也可能需要扩展支持 DB 存储所在的 LVM 逻辑卷。
如果底层 LV 已经扩展，则必须停止 OSD 守护进程，
并通知 BlueFS 设备大小变化了，执行下列命令： 

.. prompt:: bash $

   ceph-bluestore-tool bluefs-bdev-expand --path /var/lib/ceph/osd/ceph-$ID


BLUEFS_AVAILABLE_SPACE
______________________

要查看 BlueFS 还有多少空闲空间，执行下列命令：

.. prompt:: bash $

   ceph daemon osd.123 bluestore bluefs available

此命令顶多输出这三个值： ``BDEV_DB free`` 、 ``BDEV_SLOW free`` 和
``available_from_bluestore`` 。 ``BDEV_DB`` 和 ``BDEV_SLOW`` 报告的是
BlueFS 已申请到、但目前还被认为是空闲的空间尺寸。
``available_from_bluestore`` 的数值表示 BlueStore 向 BlueFS 释放更多空间的能力。
由于 BlueFS 分配单元通常大于 BlueStore 分配单元，
因此该值与 BlueStore 可用空间的数值不同是正常的。
这意味着 BlueStore 空闲空间里只有一部分可供 BlueFS 使用。


BLUEFS_LOW_SPACE
_________________

如果 BlueFS 可用的空闲空间不足，而且 BlueStore 可用空闲空间不多
（换句话说， ``bluestore max free`` 的数值较低），
可以试试减少 BlueFS 分配单元的尺寸。
要模拟分配单元不同时的可用空间，执行下列命令： 

.. prompt:: bash $

   ceph daemon osd.123 bluestore bluefs available <alloc-unit-size>


BLUESTORE_FRAGMENTATION
_______________________

``BLUESTORE_FRAGMENTATION`` 表示 BlueStore 底层的可用空间都是碎片化的。
这是正常的、不可避免的现象，但过度碎片化会导致速度变慢。
要检查 BlueStore 碎片化情况，执行下列命令： 

.. prompt:: bash $

   ceph daemon osd.123 bluestore allocator score block

碎片化的评分在 [0-1] 这个范围内。

- [0.0 .. 0.4] 轻微碎片化
- [0.4 .. 0.7] 小的、可接受的碎片化
- [0.7 .. 0.9] 比较多，但还算安全的碎片化
- [0.9 .. 1.0] 严重碎片化，可能影响 BlueFS 从 BlueStore 获取空间的能力

要查看空闲空间的碎片化详情，执行下列命令：

.. prompt:: bash #

   ceph daemon osd.123 bluestore allocator dump block

对于当前进程没在运行的 OSD ，可以用 :program:`ceph-bluestore-tool` 检查碎片情况。
要查看碎片评分，执行下列命令： 

.. prompt:: bash #

   ceph-bluestore-tool --path /var/lib/ceph/osd/ceph-123 --allocator block free-score

要转储出详细的空闲块，执行下列命令：

.. prompt:: bash #

   ceph-bluestore-tool --path /var/lib/ceph/osd/ceph-123 --allocator block free-dump


BLUESTORE_LEGACY_STATFS
_______________________

至少有一个 OSD 的 BlueStore 卷是在 Nautilus 版本之前创建的。
(在 Nautilus 中， BlueStore 按每个存储池的粒度跟踪它的内部使用情况统计信息）。

如果\ *所有*\ 的 OSD 都早于 Nautilus ，这就意味着根本无法获得每个存储池的指标。
但如果 Nautilus 之前的 OSD 和 Nautilus 之后的 OSD 混在一起，
``ceph df`` 报告的集群利用率统计数据就会不准确。

可以更新一下老的 OSD 们，就可以用新的利用率跟踪方案了，这样操作：
停止每个 OSD 、运行修复操作，然后重启这个 OSD 。
例如，要更新 ``osd.123`` ，执行下列命令：

.. prompt:: bash #

   systemctl stop ceph-osd@123
   ceph-bluestore-tool repair --path /var/lib/ceph/osd/ceph-123
   systemctl start ceph-osd@123

要禁用此警报，执行下列命令：

.. prompt:: bash #

  ceph config set global bluestore_warn_on_legacy_statfs false


BLUESTORE_NO_PER_POOL_OMAP
__________________________

至少有一个 OSD 的卷是在 Octopus 版本之前创建的。
(在 Octopus 及其后续版本中， BlueStore 按存储池跟踪 omap 空间利用率）。

如果有 BlueStore OSD 没启用新的跟踪方式，
集群将根据最近的深度洗刷结果报告\
每个存储池 omap 使用情况的近似值。

可以更新 OSD 们做到按存储池跟踪，这样操作：
停止每个 OSD 、运行修复操作，然后重启这个 OSD 。
例如，要更新 ``osd.123`` ，执行下列命令：

.. prompt:: bash #

   systemctl stop ceph-osd@123
   ceph-bluestore-tool repair --path /var/lib/ceph/osd/ceph-123
   systemctl start ceph-osd@123

要禁用此警报，执行下列命令：

.. prompt:: bash #

  ceph config set global bluestore_warn_on_no_per_pool_omap false


BLUESTORE_NO_PER_PG_OMAP
__________________________

至少有一个 OSD 的卷是在 Pacific 之前创建的。
(在 Pacific 及其后续版本中， Bluestore 按归置组 (PG)
跟踪 omap 空间利用率）。

在 PG 迁移时，按 PG 划分的 omap 可以使 PG 删除速度更快。

可以更新一下老的 OSD ，让它按 PG 更新，这样操作：
停止每个 OSD 、运行修复操作，然后重启 OSD 。
例如，要更新 ``osd.123`` ，执行下列命令：

.. prompt:: bash #

   systemctl stop ceph-osd@123
   ceph-bluestore-tool repair --path /var/lib/ceph/osd/ceph-123
   systemctl start ceph-osd@123

要禁用此警报，执行下列命令：

.. prompt:: bash #

  ceph config set global bluestore_warn_on_no_per_pg_omap false


BLUESTORE_DISK_SIZE_MISMATCH
____________________________

至少有一个 BlueStore OSD 存在内部不一致，
在物理设备大小与跟踪其大小的元数据之间。
这种不一致可能导致 OSD 以后发生崩溃。

存在这种不一致的 OSD 应该销毁并重新配置。
要非常小心，每次只在一个 OSD 上执行此过程，
以最小化丢失数据的风险。要完成此步骤，
其中 ``$N`` 是存在不一致的 OSD ，执行下列命令： 

.. prompt:: bash #

   ceph osd out osd.$N
   while ! ceph osd safe-to-destroy osd.$N ; do sleep 1m ; done
   ceph osd destroy osd.$N
   ceph-volume lvm zap /path/to/device
   ceph-volume lvm create --osd-id $N --data /path/to/device

.. note::

   等着这个恢复过程在一个 OSD 上完成，再处理下一个。


BLUESTORE_NO_COMPRESSION
________________________

至少有一个 OSD 无法加载 BlueStore 压缩插件。
造成这一问题的原因可能是安装中断，
其中的 ``ceph-osd`` 二进制文件与压缩插件不匹配。
也可能是最近一次升级造成的，升级过程中未重新启动 ``ceph-osd`` 守护进程。

要解决此问题，请确认运行受影响 OSD 的主机上的\
所有软件包都已正确安装，并且 OSD 守护进程已重启。
如果此问题仍然存在，请检查 OSD 日志，查看有关问题源的信息。


BLUESTORE_SPURIOUS_READ_ERRORS
______________________________

至少有一个 BlueStore OSD 在主设备上检测到了读取错误。
BlueStore 已重试磁盘读取，从这些错误中恢复过来了。
该警报可能表明底层硬件有问题、 I/O 子系统有问题、
或类似的问题。此类问题可能导致永久性数据损坏。
有关虚假读取错误根本原因的一些观察结果可在此处找到：
https://tracker.ceph.com/issues/22464 。

此警报不需要立即响应，但受影响的主机可能需要多加小心：
例如，将主机升级到最新的操作系统/内核版本，
并进行硬件资源利用率监视器。

要在所有 OSD 上禁用此警报，执行下列命令：

.. prompt:: bash #

   ceph config set osd bluestore_warn_on_spurious_read_errors false

或者，要在指定 OSD 上禁用此警报，执行下列命令：

.. prompt:: bash #

   ceph config set osd.123 bluestore_warn_on_spurious_read_errors false


BLOCK_DEVICE_STALLED_READ_ALERT
_______________________________

出现了 BlueStore 日志消息，显示存储驱动器问题，
这些问题可能会导致性能下降，并可能导致数据不可用或丢失。
这些信息表明存储驱动器即将出现故障，应进行评估，
而且有可能需要移除并更换。

.. code-block:: console

   read stalled read 0x29f40370000~100000 (buffered) since 63410177.290546s, timeout is 5.000000s

不过，这个很难发现，因为没有明显的警告
（例如， ``ceph health detail`` 中的健康警告或信息）。
更多观察结果请见： https://tracker.ceph.com/issues/62500

还有可能会出现假阳性的 ``stalled read`` 实例，因此增加了一个机制来提高准确性。
如果在最近的 :confval:`bdev_stalled_read_warn_lifetime` 秒内，
发现某个 BlueStore 块设备的 ``stalled read`` 事件数量大于或等于
:confval:`bdev_stalled_read_warn_threshold` ，就会在 ``ceph health detail`` 中报告此警告。
警告状态将在条件消除后移除。

:confval:`bdev_stalled_read_warn_lifetime` 和
:confval:`bdev_stalled_read_warn_threshold` 的默认值\
可以全局地或对特定 OSD 进行覆盖。

若要更改，执行下列命令： 

.. prompt:: bash $

   ceph config set global bdev_stalled_read_warn_lifetime 10
   ceph config set global bdev_stalled_read_warn_threshold 5

可以给某些 OSD 或者一个类设置。例如，只给 SSD OSD 设置：

.. prompt:: bash $

   ceph config set osd.123 bdev_stalled_read_warn_lifetime 10
   ceph config set osd.123 bdev_stalled_read_warn_threshold 5
   ceph config set class:ssd bdev_stalled_read_warn_lifetime 10
   ceph config set class:ssd bdev_stalled_read_warn_threshold 5


WAL_DEVICE_STALLED_READ_ALERT
_____________________________

警告状态 ``WAL_DEVICE_STALLED_READ_ALERT`` 用于指示指定 BlueStore OSD 的
``WAL_DEVICE`` 上的 ``stalled read`` （读取已停滞）实例。
该警告可通过 :confval:`bdev_stalled_read_warn_lifetime` 和
:confval:`bdev_stalled_read_warn_threshold` 选项进行配置，
配置命令类似于 ``BLOCK_DEVICE_STALLED_READ_ALERT`` 警告小节所述的命令。

DB_DEVICE_STALLED_READ_ALERT
____________________________

警告状态 ``DB_DEVICE_STALLED_READ_ALERT`` 用于指示指定 BlueStore OSD 的
``DB_DEVICE`` 上的 ``stalled read`` （读取已停滞）实例。
该警告可通过 :confval:`bdev_stalled_read_warn_lifetime` 和
:confval:`bdev_stalled_read_warn_threshold` 选项进行配置，
配置命令类似于 ``BLOCK_DEVICE_STALLED_READ_ALERT`` 警告小节所述的命令。


BLUESTORE_SLOW_OP_ALERT
_______________________

BlueStore 日志信息会显示存储硬盘问题，这些问题可能导致性能下降、
数据不可用或丢失。这些信息表明存储驱动器可能即将发生故障，
应进行检查，有必要的话更换掉。

.. code-block:: console

   log_latency_fn slow operation observed for _txc_committed_kv, latency = 12.028621219s, txc = 0x55a107c30f00
   log_latency_fn slow operation observed for upper_bound, latency = 6.25955s
   log_latency slow operation observed for submit_transaction..

此情形也可能由 ``BLUESTORE_SLOW_OP_ALERT`` 集群健康标记反映。

由于可能会出现误报的 ``slow ops`` 实例，因此增加了一种机制以提高可靠性。
如果在最近的 :confval:`bluestore_slow_ops_warn_lifetime` 秒内，
发现指定 BlueStore OSD 的 ``slow ops`` 指示数量\
大于或等于 :confval:`bluestore_slow_ops_warn_threshold` ，
就会在 ``ceph health detail`` 中报告该警告。
警告状态会在条件消除后清除。

可以全局地、或更改指定 OSD 的
:confval:`bluestore_slow_ops_warn_lifetime` 和
:confval:`bluestore_slow_ops_warn_threshold` 默认值。

若要更改，执行下列命令： 

.. prompt:: bash $

   ceph config set global bluestore_slow_ops_warn_lifetime 10
   ceph config set global bluestore_slow_ops_warn_threshold 5

此操作可以针对指定 OSD 或者指定掩码（类别）。例如：

.. prompt:: bash $

   ceph config set osd.123 bluestore_slow_ops_warn_lifetime 10
   ceph config set osd.123 bluestore_slow_ops_warn_threshold 5
   ceph config set class:ssd bluestore_slow_ops_warn_lifetime 10
   ceph config set class:ssd bluestore_slow_ops_warn_threshold 5


设备健康
--------
.. Device health

DEVICE_HEALTH
_____________

预计至少有一个 OSD 设备即将发生故障，
警告阈值由 ``mgr/devicehealth/warn_threshold`` 配置选项决定。

由于此警报仅适用于当前标记为 ``in`` 的 OSD ，因此对这种预期故障的恰当响应是：
(1) 标记 OSD 为 ``out`` ，以便把数据从这个 OSD 迁移出去，
然后 (2) 将硬件从系统中拆除。注意，如果启用了 ``mgr/devicehealth/self_heal``
（由 ``mgr/devicehealth/mark_out_threshold`` 决定），
标记 ``out`` 这个操作通常会自动完成。如果一个 OSD 设备已经损坏，
但使用着这个设备的 OSD 其状态仍然是 ``up`` ，那么恢复会降级。
在这种情况下，强制停止有关 OSD 守护进程可能更好，
这样就可以从存活的健康 OSD 开始恢复。
这样做时必须格外小心，注意故障域，以免影响数据可用性。

要检查设备健康状况，执行下列命令： 

.. prompt:: bash $

   ceph device info <device-id>

设备预期寿命用 Ceph 管理器运行的预测模型设置、
或可以执行下列命令的外部工具设置： 

.. prompt:: bash $

   ceph device set-life-expectancy <device-id> <from> <to>

你可以手动更改已经存储的预期寿命，但这种更改通常不会产生任何效果。
原因在于，最初设置了预期寿命的那个工具，
可能还会再次重设你的更改，
而且更改存储的数值并不会影响硬件设备实际的健康状况。


DEVICE_HEALTH_IN_USE
____________________

至少有一个设备（即 OSD ）预计很快就会发生故障，
并已被标记为 ``out`` 状态（不在集群内，
由 ``mgr/devicehealth/mark_out_threshold`` 控制），
但它们仍参与一个或多个归置组。这可能是因为 OSD 最近才被标记为 ``out`` ，
而数据仍在迁移，也可能是因为某些原因导致数据无法从 OSD 迁出
（例如，集群已经快满了，或 CRUSH 分级结构里\
没有其他合适的 OSD 可以承接数据）。

要消除此消息，可以禁用自愈行为
（就是把 ``mgr/devicehealth/self_heal`` 设置为 ``false`` ）、
调整 ``mgr/devicehealth/mark_out_threshold`` 、
或找出是什么因素导致数据不能从即将发生故障的 OSD 迁出。


.. _rados_health_checks_device_health_toomany:

DEVICE_HEALTH_TOOMANY
_____________________

预计有太多设备（即 OSD ）很快将发生故障，
而且由于启用了 ``mgr/devicehealth/self_heal`` 行为，
把所有嫌疑 OSD 标记为 ``out``
会超出集群的 ``mon_osd_min_in_ratio`` 比率。
此比率可防止过多的 OSD 被自动标记为 ``out`` 。

你应该及时向集群添加新 OSD 以防止数据丢失，
或逐步替换即将故障的 OSD 。

或者，你可以静默这个健康检查，可以调整选项，
包括 ``mon_osd_min_in_ratio`` 或 ``mgr/devicehealth/mark_out_threshold`` 。
但是，特别注意，这将导致不可恢复的数据丢失的可能性增大。


数据健康（存储池和归置组们）
----------------------------
.. Data health (pools & placement groups)

PG_AVAILABILITY
_______________

数据可用性降低。换句话说，
集群无法为集群中一部分数据的潜在读取或写入请求提供服务。
更确切地说，至少有一个归置组 (PG) 处于无法为 I/O 请求提供服务的状态。
以下任何一种 PG 状态，如果不能快速清除，都是有问题的：
*peering* 、 *stale* 、 *incomplete* 和 *active* 缺失。

有关受影响 PG 的详细信息，
执行下列命令：

.. prompt:: bash $

   ceph health detail

大多数情况下，此问题的根源都是当前至少有一个 OSD 处于 ``down`` 状态：
见前文的 ``OSD_DOWN`` 。

要查看指定嫌疑 PG 的状态，执行下列命令：

.. prompt:: bash $

   ceph tell <pgid> query


PG_DEGRADED
___________

某些数据的冗余度降低：换句话说，
集群中不是所有数据（在多副本存储池的情况下）
或纠删码片段（在纠删码存储池的情况下）都达到了应有的副本数。
更确切地说，至少有一个归置组 (PG) ：

* 已设置 *degraded* 或 *undersized* 标志，
  这意味着集群中那个 PG 没有足够的数量；或者
* 很长时间都没能达到 *clean* 状态。

有关受影响 PG 的详细信息，
执行下列命令： 

.. prompt:: bash $

   ceph health detail

大多数情况下，此问题的根源都是当前至少有一个 OSD 处于 ``down`` 状态：
见前文的 ``OSD_DOWN`` 。

要查看指定嫌疑 PG 的状态，执行下列命令：

.. prompt:: bash $

   ceph tell <pgid> query


PG_RECOVERY_FULL
________________

由于集群缺乏可用空间，数据冗余度可能会降低，
甚至使某些数据面临风险。更确切地说，
至少有一个归置组设置了 *recovery_toofull* 标志，
这意味着集群无法迁移或恢复数据，因为至少有一个 OSD 超过了 ``full`` 阈值。

有关应对这种情况的步骤，请参阅上文的 *OSD_FULL* 。


PG_BACKFILL_FULL
________________

由于集群缺乏可用空间，数据冗余度可能会降低，
甚至使某些数据面临风险。更确切地说，
至少有一个归置组设置了 *backfill_toofull* 标志，
这意味着集群无法迁移或恢复数据，因为至少有一个 OSD 超过了 ``backfillfull`` 阈值。

有关应对这种情况的步骤，请参阅上文的 *OSD_BACKFILLFULL* 。


PG_DAMAGED
__________

数据洗刷操作发现了集群中存在数据一致性问题。
更确切地说，至少有一个归置组 (1) 已设置 *inconsistent* 或
``snaptrim_error`` 标志，它表明先前的数据洗刷操作发现了问题，
或 (2) 已设置 *repair* 标志，这意味着当前正在对这样的不一致进行修复。

详情见 :doc:`../troubleshooting/troubleshooting-pg` 。


OSD_SCRUB_ERRORS
________________

近期的 OSD 洗刷出现了明显不一致的地方。这个错误一般和
*PG_DAMAGED* （见上文）成对出现。

详情见 :doc:`../troubleshooting/troubleshooting-pg` 。


OSD_TOO_MANY_REPAIRS
____________________

读修复次数已超过配置的阈值数值
``mon_osd_warn_num_repaired`` （默认值： ``10`` ）。
由于洗刷操作只处理静态数据的错误，
而且当另一个副本可用时发生的任何读取错误都会立即修复，
客户端也可以获取到对象数据，
有可能存在即将发生故障的磁盘，它们没有注册任何洗刷错误。
这个修复计数的存在就是为了识别此类故障磁盘。

为了清除此警告，我们加了一条新命令
``ceph tell osd.# clear_shards_repaired [count]`` 。
默认情况下，它会将修复计数设为 0 。可以给此命令加 `count` 数值。
这样，管理员就可以向命令传递 ``mon_osd_warn_num_repaired``
（或提高）的数值来重新启用这条警告。
除了用 `clear_shards_repaired` 之外，还可以用
`ceph health mute` 来给 `OSD_TOO_MANY_REPAIRS` 警报静音。


LARGE_OMAP_OBJECTS
__________________

至少有一个存储池包含巨型 omap 对象，由
``osd_deep_scrub_large_omap_object_key_threshold`` （用于确定大型 omap 对象的键数阈值）
或 ``osd_deep_scrub_large_omap_object_value_sum_threshold``
（用于确定是否为大型 omap 对象的所有键值总字节数的阈值）或两者决定。
要查找有关对象名称、键值数和字节数的更多信息，
请在群集日志中搜索 “Large omap object found”（发现大型 omap 对象）。
未启用自动重分片的 RGW-Bucket 索引对象可能会导致此问题。
有关重分片的更多信息，请参阅
:ref:`RGW Dynamic Bucket Index Resharding <rgw_dynamic_bucket_index_resharding>` 。

要调整上述阈值，执行下列命令：

.. prompt:: bash $

   ceph config set osd osd_deep_scrub_large_omap_object_key_threshold <keys>
   ceph config set osd osd_deep_scrub_large_omap_object_value_sum_threshold <bytes>


CACHE_POOL_NEAR_FULL
____________________

缓存层存储池即将填满，该状态由缓存池的 ``target_max_bytes`` 和
``target_max_objects`` 属性决定。当存储池达到目标阈值时，
此存储池的写入请求可能阻塞，同时，缓存中的数据会被刷回和驱逐。
此状态通常会导致极高的延迟和性能下降。

要调整缓存存储池的目标尺寸，执行下列命令：

.. prompt:: bash $

   ceph osd pool set <cache-pool-name> target_max_bytes <bytes>
   ceph osd pool set <cache-pool-name> target_max_objects <objects>

正常的缓存刷回和驱逐活动被抑制可能还有其他原因：例如，
基础层（ base tier ）可用性降低、基础层性能下降或集群整体负载过高。


TOO_FEW_PGS
___________

集群中正在使用的归置组（PG）数量低于可配置阈值，
每个 OSD 最少 ``mon_pg_warn_min_per_osd`` 个 PG 。
这可能导致数据在集群 OSD 间分布不均、
平衡性不足，进而降低整体性能。

如果数据存储池尚未创建，此情况属正常现象。

要解决此问题，可增加现有数据池的 PG 数量或创建新存储池。
更多信息请参阅 :ref:`choosing-number-of-placement-groups` 。


POOL_PG_NUM_NOT_POWER_OF_TWO
____________________________

至少有一个存储池的 ``pg_num`` 值不是 2 的幂次方。虽然这不是致命错误，
但会造成数据分布失衡——某些归置组将承载远超其他组的数据量。

此问题可轻松纠正，把受影响存储池的 ``pg_num`` 值调整为最接近的 2 的幂次方即可。
可以启用 PG Autoscaler ，或执行如下命令：

.. prompt:: bash $

   ceph osd pool set <pool-name> pg_num <value>

要禁用此健康检查，执行下列命令：

.. prompt:: bash $

   ceph config set global mon_warn_on_pool_pg_num_not_power_of_two false

注意，不建议禁用此健康检查。


POOL_TOO_FEW_PGS
________________

鉴于当前存储在存储池中的数据量，
一个或多个存储池可能需要更多归置组（ PG ）。
此问题可能导致集群中 OSD 的数据分布和平衡性不佳，从而降低整体性能。
只有当存储池的 ``pg_autoscale_mode`` 属性设置为 ``warn`` 时才会触发此警报。

如果要禁用此警报，可执行以下命令，
完全禁用该存储池的 PG 自动伸缩功能：

.. prompt:: bash $

   ceph osd pool set <pool-name> pg_autoscale_mode off

要允许集群自动调整存储池的 PG 数量，
执行下列命令：

.. prompt:: bash $

   ceph osd pool set <pool-name> pg_autoscale_mode on

另外，要想手动把一个存储池的 PG 数量设置成建议的数值，
执行下列命令：

.. prompt:: bash $

   ceph osd pool set <pool-name> pg_num <new-pg-num>

详情见 :ref:`choosing-number-of-placement-groups` 和
:ref:`pg-autoscaler` 。


TOO_MANY_PGS
____________

集群中在用的归置组（ PG ）数量已超过可配置的阈值：
每 OSD ``mon_max_pg_per_osd`` 个 PG 。
若超过此阈值，集群会禁止新建存储池、增加池 `pg_num` 数量或提高存储池的副本数
（上述任一操作均会导致集群 PG 数量增加）。
过多的 PG 会导致 OSD 守护进程的内存占用率升高，
集群状态变更后（如 OSD 重启、新增或移除）的互联速度变慢，
并增加管理器和监视器守护进程的负载。

要缓解此问题，最简单的方案是新增硬件来提升集群中 OSD 节点的数量。
注意：由于本健康检查统计的是处于 ``in`` 状态的 OSD 节点数量，
如果有可用的 ``out`` OSD 节点，把它们标记为 ``in`` 同样有效。
真要标记的话，执行下列命令： 

.. prompt:: bash $

   ceph osd in <osd id(s)>

详情见 :ref:`choosing-number-of-placement-groups` 。


POOL_TOO_MANY_PGS
_________________

鉴于当前存储在存储池中的数据量，
一个或多个存储池应减少其归置组（PG）数量。
此问题可能导致 OSD 守护进程内存利用率偏高、
集群状态变更后（例如 OSD 重启、新增或移除）互联速度变慢，
以及管理器和监视器守护进程负载增加。
只有当存储池的 ``pg_autoscale_mode`` 属性设置为 ``warn`` 时才会触发此警报。

要禁用该警报，可以完全关闭存储池的 PG 自动伸缩功能，
执行下列命令： 

.. prompt:: bash $

   ceph osd pool set <pool-name> pg_autoscale_mode off

要允许集群自动调整某个存储池的 PG 数量，
执行下列命令：

.. prompt:: bash $

   ceph osd pool set <pool-name> pg_autoscale_mode on

另外，可以手动把某个存储池的 PG 数设置为建议值，
执行下列命令：

.. prompt:: bash $

   ceph osd pool set <pool-name> pg_num <new-pg-num>

详情见 :ref:`choosing-number-of-placement-groups` 和
:ref:`pg-autoscaler` 。


POOL_TARGET_SIZE_BYTES_OVERCOMMITTED
____________________________________

至少有一个存储池明确设置了 ``target_size_bytes`` 属性，
用于估算存储池的预期大小，
但该属性的值（单独或与其他存储池组合）超过了总的可用存储空间。

此警报通常表明存储池的 ``target_size_bytes`` 值过于大了，
应该减小或设为零。要减小 ``target_size_bytes`` 数值或将其设为零，
执行下列命令： 

.. prompt:: bash $

   ceph osd pool set <pool-name> target_size_bytes 0

上述命令把 ``target_size_bytes`` 的数值设置成了 0 。
要把 ``target_size_bytes`` 的数值设置成非 0 的值，
把 ``0`` 换成你想要的非 0 数值即可。

更详细的内容见  :ref:`specifying_pool_target_size` 。


POOL_HAS_TARGET_SIZE_BYTES_AND_RATIO
____________________________________

至少有一个存储池同时设置了 ``target_size_bytes`` 和
``target_size_ratio`` 属性时，系统将据此估算存储池的预期大小。
这两个属性中仅允许一个是非零值。若两者均设置为非零值，
则优先采用 ``target_size_ratio`` ，忽略 ``target_size_bytes`` 。

要把 ``target_size_bytes`` 重置为 0 ，执行下列命令：

.. prompt:: bash $

   ceph osd pool set <pool-name> target_size_bytes 0

详情见 :ref:`specifying_pool_target_size` 。


TOO_FEW_OSDS
____________

集群中的 OSD 数量低于可配置阈值 ``osd_pool_default_size`` 。
这意味着部分或全部数据可能无法满足 CRUSH 规则和存储池配置中指定的数据保护策略。


SMALLER_PGP_NUM
_______________

至少有一个存储池的 ``pgp_num`` 数值小于 ``pg_num`` 。
此警报通常表明在归置行为没有增加的情况下，
归置组（ PG ）计数被增加了。

这种差异有时是有意引进的，目的是分离出 `split` 步骤，
因为在 ``pgp_num`` 更改后，会引起数据迁移，
进而导致 PG 计数被调整。

解决此问题的方法，通常是把 ``pgp_num`` 设置为与 ``pg_num`` 相匹配的数值，
从而触发数据迁移。执行下列命令：

.. prompt:: bash $

   ceph osd pool set <pool> pgp_num <pg-num-value>


MANY_OBJECTS_PER_PG
___________________

至少有一个存储池里每个归置组（ PG ）的\
平均对象数量远高于整个集群的平均值。
具体的阈值由 ``mon_pg_warn_max_object_skew`` 配置值决定。

此警报通常表明：包含集群大部分数据的存储池，
它的 PG 数量过少，或者是数据较少的其他存储池它们的 PG 数量过多。
见上文 *TOO_MANY_PGS* 。

在管理器上调高 ``mon_pg_warn_max_object_skew`` 配置选项的阈值\
可以消除此健康告警。

对于某个指定存储池，如果把 ``pg_autoscale_mode`` 设置为 ``on`` ，
它的健康告警就能消除。


POOL_APP_NOT_ENABLED
____________________

一个存储池存在，但是这个存储池没打标签，标记它用于哪个应用程序。

要解决此问题，给这个存储池打个标签用于一个应用程序即可。
例如，如果此存储池用于 RBD ，执行下列命令：

.. prompt:: bash $

   rbd pool init <poolname>

另外，如果这个存储池在被一个定制应用程序使用（这里是 foo ），
你可以执行下面的低级命令来给存储池打标：

.. prompt:: bash $

   ceph osd pool application enable foo

详情见  :ref:`associate-pool-to-application` 。


POOL_FULL
_________

至少有一个存储池已达到（或即将达到）其配额上限。
触发此健康检查的阈值由 ``mon_pool_quota_crit_threshold`` 配置选项决定。

可通过执行下列命令来调高或调低（或去除）存储池配额： 

.. prompt:: bash $

   ceph osd pool set-quota <pool> max_bytes <bytes>
   ceph osd pool set-quota <pool> max_objects <objects>

要禁用配额，可以把配额数值设置为 ``0`` 。


POOL_NEAR_FULL
______________

至少有一个存储池即将达到配置的用满阈值。

有好几个阈值能触发此健康检查，其中之一由
``mon_pool_quota_warn_threshold`` 配置选项决定。

可通过执行下列命令来调高或调低（或去除）存储池配额： 

.. prompt:: bash $

   ceph osd pool set-quota <pool> max_bytes <bytes>
   ceph osd pool set-quota <pool> max_objects <objects>

要禁用配额，可以把配额数值设置为 0 。

能触发上述两个健康检查的其它阈值有
``mon_osd_nearfull_ratio`` 和 ``mon_osd_full_ratio`` 。
细节及解决方法见 :ref:`storage-capacity` 和 :ref:`no-free-drive-space` 。


OBJECT_MISPLACED
________________

集群内一个或多个对象未存储在 CRUSH 算法推荐的位置。
此警报表明，近期集群变更引发的数据迁移尚未完成。

数据归置错误本身并非危险状态；数据一致性从未有风险，
并且新副本都创建好且数量达到期望值、位于期望的位置之前，
旧对象副本不会被清除。


OBJECT_UNFOUND
______________

集群内至少有一个对象找不到。更准确地说，
OSD 们都知道一个对象的较新或更新后的副本应该存在，
但是从目前在线的 OSD 上却找不到这个对象副本。

对未找到对象的读出或写入请求会阻塞。

理想情况下，持有未找到对象较新副本的 "down" OSD 能够重新上线。
要识别候选 OSD ，需检查负责这个未找到对象的 PG(s) 互联状态。
要查看互联状态，执行下列命令： 

.. prompt:: bash $

   ceph tell <pgid> query

另一方面，如果对象的最新副本不可用，
可以让集群回滚到该对象的先前版本。
详情见 :ref:`failures-osd-unfound` 。


SLOW_OPS
________

至少有一个 OSD 请求或监视器请求在处理时花费的时间过长。
此警报可能表明系统负载过高、存储设备速度缓慢或存在软件缺陷。

要向守护进程查询导致速度缓慢的请求队列，
在这个守护进程所在主机上运行以下命令： 

.. prompt:: bash $

   ceph daemon osd.<id> ops

要查看最慢的近期请求的梗概，执行下列命令：

.. prompt:: bash $

   ceph daemon osd.<id> dump_historic_ops

要查看某一 OSD 的位置，执行下列命令：

.. prompt:: bash $

  ceph osd find osd.<id>


PG_NOT_SCRUBBED
_______________

至少有一个归置组（ PG ）最近没有进行进行洗刷。
PG 通常会在由全局变量 :confval:`osd_scrub_max_interval`
确定的时间间隔内进行洗刷。
这个间隔可通过修改存储池的变量 :confval:`scrub_max_interval` 的值覆盖掉。
当洗刷计划发起后，若超过设定的时间间隔百分比
（由 ``mon_warn_pg_not_scrubbed_ratio`` 决定），
却仍未执行洗刷，就触发此健康检查。

只有在 PG 被标记为 ``clean`` （意思是它们将是干净的，
而不是说它们已经被检查过、发现是干净的）时才会被洗刷。
归置错的或降级的 PG 不会被标记为 ``clean``
（参见上文 *PG_AVAILABILITY* 和 *PG_DEGRADED* ）。

如果需要手动对干净 PG 进行洗刷，执行下列命令： 

.. prompt:: bash $

   ceph pg scrub <pgid>


PG_NOT_DEEP_SCRUBBED
____________________

一个或多个归置组（ PG ）近期未进行深度洗刷。
PG 通常每隔最多 :confval:`osd_deep_scrub_interval` 秒进行一次洗刷。
当洗刷计划发起后，时间间隔超过
:confval:`mon_warn_pg_not_deep_scrubbed_ratio` 比例仍未执行洗刷时，
就会触发此健康检查告警。

只有当 PG 被标记为 ``clean`` （意思是它们将是干净的，
而不是说它们已经被检查过、发现是干净的）时才会被深度洗刷。
归置错误或降级的 PG 可能不会被标记为 ``clean``
（参见上文 *PG_AVAILABILITY* 和 *PG_DEGRADED* ）。

本文档提供两种设置 :confval:`osd_deep_scrub_interval` 数值的方法。
第一种方法能够全局更改 :confval:`osd_deep_scrub_interval` 的数值，
第二种方法则分别更改 OSD 和管理器守护进程的
:confval:`osd_deep_scrub_interval` 数值。

方法一
~~~~~~
.. First Method

要手动发起对一个干净 PG 的深度洗刷，执行下列命令：

.. prompt:: bash $

   ceph pg deep-scrub <pgid>

在特定条件下，会出现警告信息 ``PGs not deep-scrubbed in time``
（PG 未及时深度洗刷）。这可能是因为集群中包含很多大型 PG ，
其深度洗刷过程耗时较长。为解决此问题，
必须更改全局的 :confval:`osd_deep_scrub_interval` 数值。

#. 确认 ``ceph health detail`` 返回了 ``pgs not deep-scrubbed in time``
   警告信息::

      # ceph health detail
      HEALTH_WARN 1161 pgs not deep-scrubbed in time
      [WRN] PG_NOT_DEEP_SCRUBBED: 1161 pgs not deep-scrubbed in time
      pg 86.fff not deep-scrubbed since 2024-08-21T02:35:25.733187+0000

#. 更改全局的 ``osd_deep_scrub_interval`` ：

   .. prompt:: bash #

      ceph config set global osd_deep_scrub_interval 1209600

上述步骤是 Eugen Block 在 2024 年 9 月开发的。

细节见 `Eugen Block 的博客 <https://heiterbiswolkig.blogs.nde.ag/2024/09/06/pgs-not-deep-scrubbed-in-time/>`_ 。

见 `Redmine tracker issue #44959 <https://tracker.ceph.com/issues/44959>`_ 。

方法二
~~~~~~
.. Second Method

要对一个干净的 PG 手动启动深度洗刷，执行下列命令：

.. prompt:: bash $

   ceph pg deep-scrub <pgid>

在特定情况下，会出现 ``PGs not deep-scrubbed in time`` 警告信息。
可能是因为此集群包含很多大型 PG ，它们需要较长的时间才能完成深度洗刷。
要应对此情形，必须更改 OSD 和管理器守护进程的
:confval:`osd_deep_scrub_interval` 数值。

#. 确认 ``ceph health detail`` 会返回 ``pgs not deep-scrubbed in time``
   警告信息::

      # ceph health detail
      HEALTH_WARN 1161 pgs not deep-scrubbed in time
      [WRN] PG_NOT_DEEP_SCRUBBED: 1161 pgs not deep-scrubbed in time
      pg 86.fff not deep-scrubbed since 2024-08-21T02:35:25.733187+0000

#. 更改 OSD 的 ``osd_deep_scrub_interval`` ：

   .. prompt:: bash #

      ceph config set osd osd_deep_scrub_interval 1209600

#. 更改管理器的 ``osd_deep_scrub_interval`` ：

   .. prompt:: bash #

      ceph config set mgr osd_deep_scrub_interval 1209600

上述步骤是 Eugen Block 在 2024 年 9 月份开发的。

细节见 `Eugen Block 的博客 <https://heiterbiswolkig.blogs.nde.ag/2024/09/06/pgs-not-deep-scrubbed-in-time/>`_ 。

见 `Redmine tracker issue #44959 <https://tracker.ceph.com/issues/44959>`_ 。


PG_SLOW_SNAP_TRIMMING
_____________________

至少有一个 PG 的快照修剪队列已超过配置的警告阈值。
此警报表明，近期删除了数量巨大的快照、
或者是 OSD 修剪快照的速度跟不上删除新快照的速度。

警告阈值由 :confval:`mon_osd_snap_trim_queue_warn_on` 选项配置
（默认值： ``32768`` ）。

当 OSD 负载过高导致后台任务处理滞后，
或 OSD 内部元数据数据库严重碎片化而无法正常运作时，
可能触发此警报。
该警报也可能预示着 OSD 存在其他的性能问题。

快照修剪队列的准确大小可通过 ``ceph pg ls -f json-detail`` 命令中的
``snaptrimq_len`` 字段获取。


跨区域模式
----------
.. Stretch Mode

INCORRECT_NUM_BUCKETS_STRETCH_MODE
__________________________________

跨区域模式（ stretch mode ）目前仅支持两个隔离（ dividing ） CRUSH 桶，
桶内配置若干 OSD 。此警告表明，启用跨区域模式后，隔离的桶数量不是两个。
在修复此状况前，系统可能出现不可预测的故障以及监视器断言。

建议通过去除多余的隔离 CRUSH 桶或将隔离桶数量增至两个来修复此问题。
更多信息参阅 :ref:`stretch_mode` 。


UNEVEN_WEIGHTS_STRETCH_MODE
___________________________

跨区域模式启用后，两个隔离 CRUSH 桶的权重必须相等。此警告表明，
启用跨区域模式后，两个隔离桶的权重不相等。虽然这不是紧要的致命错误，
但是当 Ceph 尝试在隔离桶之间切换时，可能会出现混乱。

我们建议您修复此问题，把两个隔离 CRUSH 桶的权重设置得相等即可。
具体就是确保各个隔离桶内 OSD 的总权重保持相同。详情见 :ref:`stretch_mode` 。


NONEXISTENT_MON_CRUSH_LOC_STRETCH_MODE
______________________________________

当启用了跨区域模式后，给监视器指定的 CRUSH 位置必须分布在隔离的 CRUSH 桶中。
唯一的例外是 tiebreaker 监视器，它必须位于隔离桶之外。

此警告表明，在跨区域模式下，至少有一个监视器的
CRUSH 位置不在任何一个隔离桶内。

建议您解决此问题，确保所有监视器（除了 tiebreaker 监视器以外）的
CRUSH 位置均位于隔离桶内。详情参阅：:ref:`stretch_mode` 。


NVMeoF 网关
-----------
.. NVMeoF Gateway

NVMEOF_SINGLE_GATEWAY
_____________________

有一个网关组只有一个网关。这种情况并不理想，
因为在一个组内只有一个网关时，无法实现高可用性（ HA ）。
这可能会导致 NVMeoF 网关的故障转移和故障恢复操作出现问题。

建议在一个组中配置多个 NVMeoF 网关。


NVMEOF_GATEWAY_DOWN
___________________

部分网关处于 ``GW_UNAVAILABLE`` 状态。如果 NVMeoF 守护进程崩溃，
这个守护进程的日志文件（位于 ``/var/log/ceph/`` 目录下）可能包含故障排除信息。


NVMEOF_GATEWAY_DELETING
_______________________

部分网关当前处于 ``GW_DELETING`` 状态。它们将一直保持此状态，
直至网关负载均衡组下的所有命名空间都被转移到另一个负载均衡组 ID 之下。
这一过程由负载均衡进程自动完成。如果此警报持续存在很长时间，
说明该进程可能存在问题。


杂项
----
.. Miscellaneous

RECENT_CRASH
____________

最近，至少有一个 Ceph 守护进程崩溃了，而且管理员尚未确认并存档这些崩溃事件。
此警报表明，可能存在软件错误、
硬件问题（例如，即将发生故障的磁盘）或其他问题。

要罗列最近的崩溃事件，执行下列命令：

.. prompt:: bash $

   ceph crash ls-new

要检查某一次崩溃的信息，执行下列命令：

.. prompt:: bash $

   ceph crash info <crash-id>

要把这个警告消音，你可以归档此次崩溃（可能得在管理员检查过此崩溃之后），
执行下列命令：

.. prompt:: bash $

   ceph crash archive <crash-id>

类似地，要归档近期的所有崩溃，执行下列命令：

.. prompt:: bash $

   ceph crash archive-all

归档起来的崩溃仍然能用 ``ceph crash ls`` 命令看到，
但是用 ``ceph crash ls-new`` 命令看不到。

最近这个时间段有多长由选项 :confval:`mgr/crash/warn_recent_interval`
（默认值：两周）定义。

要完全禁用此警告，执行下列命令：

.. prompt:: bash $

   ceph config set mgr/crash/warn_recent_interval 0


RECENT_MGR_MODULE_CRASH
_______________________

最近，至少有一个管理器模块崩溃过，而且管理员尚未确认并存档这些崩溃。
此警报通常表明，在管理器（ ``ceph-mgr`` ）守护进程中\
运行的某个软件模块存在软件缺陷。
出现问题的模块可能会因此被禁用，
但其他模块不受影响，仍能按预期运行。

与 `RECENT_CRASH`_ 健康检查一样，可以调查某个特定的崩溃，
执行下列命令：

.. prompt:: bash $

   ceph crash info <crash-id>

要把这个警告消音，你可以归档此次崩溃（可能得在管理员检查过此崩溃之后），
执行下列命令：

.. prompt:: bash $

   ceph crash archive <crash-id>

类似地，要归档近期的所有崩溃，执行下列命令：

.. prompt:: bash $

   ceph crash archive-all

归档起来的崩溃仍然能用 ``ceph crash ls`` 命令看到，
但是用 ``ceph crash ls-new`` 命令看不到。

最近这个时间段有多长由选项 :confval:`mgr/crash/warn_recent_interval`
（默认值：两周）定义。

要完全禁用此警告，执行下列命令：

.. prompt:: bash $

   ceph config set mgr/crash/warn_recent_interval 0


TELEMETRY_CHANGED
_________________

Telemetry （遥测）功能已经启用，但同时由于 telemetry 报告的内容被更改了，
所以 telemetry 报告不会发出去。

Ceph 开发人员偶尔会修订 telemetry 功能，以纳入新的有用信息，
或去除后来发现无用或敏感的信息。
如果报告中包含了任意一点新信息，
Ceph 都会要求管理员重新启用 telemetry 功能。
这一要求确保管理员有机会（重新）审查将要共享的信息。

要审查 telemetry 报告的内容，执行下列命令：

.. prompt:: bash $

   ceph telemetry show

注意， telemetry 报告包含多个通道，这些通道可以独立启用或禁用。
详情参阅 :ref:`telemetry` 。

要重新启用 telemetry （并消除这条警报），执行下列命令：

.. prompt:: bash $

  ceph telemetry on

要禁用 telemetry （并消除这条警报），执行下列命令：

.. prompt:: bash $

  ceph telemetry off


AUTH_BAD_CAPS
_____________

至少有一个授权用户具有监视器无法解析的能力。通常，
此警报表示有一个或多个守护进程正在用未被授权执行任何操作的用户进行身份验证。

此警报最有可能在升级后触发，如果
（1）这些能力是在旧版本的 Ceph 中设置的，
而旧版本 Ceph 没有正确验证这些功能的语法，
或者（2）能力的语法变了。

要删除可疑用户，执行下列命令：

.. prompt:: bash $

   ceph auth rm <entity-name>

（这解决了健康检查的问题，但同时也使得客户端无法\
以被删除用户的身份进行身份验证。）

另外，要更新用户的能力，
执行下列命令：

.. prompt:: bash $

   ceph auth <entity-name> <daemon-type> <caps> [<daemon-type> <caps> ...]

关于认证能力的详情，见 :ref:`user-management` 。


OSD_NO_DOWN_OUT_INTERVAL
________________________

:confval:`mon_osd_down_out_interval` 选项设置成了零，
这意味着当 OSD 发生故障时，系统不会自动执行任何修复或恢复操作。
相反，管理员或外部编排器必须手动将状态为 ``down`` 的 OSD 标记为 ``out``
（用命令 ``ceph osd out <osd-id>`` ），以触发系统恢复。

这个选项通常设置成 5 或 10 分钟 - 足够一台主机更换电源或重启。

要消除此警报，可以把 :confval:`mon_warn_on_osd_down_out_interval_zero`
设置成 ``false`` ，执行下列命令：

.. prompt:: bash $

  ceph config global mon mon_warn_on_osd_down_out_interval_zero false


DASHBOARD_DEBUG
_______________

打开了 Dashboard 调试模式。这意味着，如果在处理一个 REST API 请求时出错了，
HTTP 错误响应会包含 Python 的追溯信息（ traceback ）。
这个行为在生产环境下应该禁用，因为这样的回溯信息可能包含并暴露敏感信息。

要禁用调试模式，执行下列命令：

.. prompt:: bash $

  ceph dashboard debug disable


BLAUM_ROTH_W_IS_NOT_PRIME
_________________________

一个纠删码存储池正在用 ``blaum_roth`` 技术，且 ``w + 1`` 不是素数。
如果这个存储池需要回填或恢复，可能会导致数据损坏。

要罗列出纠删码配置，执行下列命令：

.. prompt:: bash #

   ceph osd erasure-code-profile ls

然后，查看一份指定配置的 ``w`` 值，执行下列命令：

.. prompt:: bash #

   ceph osd erasure-code-profile get <name of profile>

理想的解决方案是用正确的 ``w`` 值新建一个存储池，并复制所有对象。
然后，在数据损坏发生前删掉旧的存储池。
