==========
 监控集群
==========
.. Monitoring a Cluster

集群运行起来后，你可以用 ``ceph`` 工具来监控，\
典型的监控包括检查 OSD 状态、监视器状态、\
归置组状态和元数据服务器状态。

使用命令行
==========
.. Using the command line

交互模式
--------
.. Interactive mode

要在交互模式下运行 ``ceph`` ，不要带参数运行 ``ceph`` ，例如：

.. prompt:: bash $

    ceph

.. prompt:: ceph>
    :prompts: ceph>

    health
    status
    quorum_status
    mon stat

非默认的路径
------------
.. Non-default paths

如果你的配置文件或密钥环不在默认位置内，可以手动给 ``ceph`` 工具指定其位置，
执行下列命令：

.. prompt:: bash $

   ceph -c /path/to/conf -k /path/to/keyring health


检查集群的状态
==============
.. Checking a Cluster's Status

启动集群后、读写数据前，先检查下集群的健康状态。

可以用下面的命令检查集群状态：

.. prompt:: bash $

   ceph status

另外，可以执行命令：

.. prompt:: bash $

   ceph -s

在交互模式下，输入 ``status`` 再按回车 **Enter** 。

.. prompt:: ceph>
    :prompts: ceph>

    status

Ceph 就会打印出集群状态。例如，一个小型的演示集群，
各种服务都有一个例程（监视器、管理器、 OSD ），可能会打印如下的：

::

  cluster:
    id:     477e46f1-ae41-4e43-9c8f-72c918ab0a20
    health: HEALTH_OK

  services:
    mon: 3 daemons, quorum a,b,c
    mgr: x(active)
    mds: cephfs_a-1/1/1 up  {0=a=up:active}, 2 up:standby
    osd: 3 osds: 3 up, 3 in

  data:
    pools:   2 pools, 16 pgs
    objects: 21 objects, 2.19K
    usage:   546 GB used, 384 GB / 931 GB avail
    pgs:     16 active+clean


Ceph 如何计算数据量
-------------------
.. How Ceph Calculates Data Usage

``usage`` 值反映了\ *事实上*\ 已占用的原始存储空间。
``xxx GB / xxx GB`` 值则是剩余空间（较小的数）与集群总容量\
的比较。理论数值反映了所存储数据的原始尺寸，未计算其副本、\
克隆、或快照空间，所以数据存储实际占用的空间通常会超过\
理论数值，因为 Ceph 会自动创建数据副本，另外存储空间也可能\
用于克隆和快照。


观察集群
========
.. Watching a Cluster

除了各守护进程的本地日志， Ceph 集群还维护着一个 *集群日志*\ ，
它记录着事关整个系统的高级事件。此类日志记录在\
监视器服务器的磁盘上（默认为 ``/var/log/ceph/ceph.log`` ），\
也可以通过命令行监控。

要持续关注集群日志，用下列命令：

.. prompt:: bash $

   ceph -w

Ceph 会打印系统的状态，然后是正发生着的各日子消息。例如：

::

  cluster:
    id:     477e46f1-ae41-4e43-9c8f-72c918ab0a20
    health: HEALTH_OK
  
  services:
    mon: 3 daemons, quorum a,b,c
    mgr: x(active)
    mds: cephfs_a-1/1/1 up  {0=a=up:active}, 2 up:standby
    osd: 3 osds: 3 up, 3 in
  
  data:
    pools:   2 pools, 16 pgs
    objects: 21 objects, 2.19K
    usage:   546 GB used, 384 GB / 931 GB avail
    pgs:     16 active+clean
  
  
  2017-07-24 08:15:11.329298 mon.a mon.0 172.21.9.34:6789/0 23 : cluster [INF] osd.0 172.21.9.34:6806/20527 boot
  2017-07-24 08:15:14.258143 mon.a mon.0 172.21.9.34:6789/0 39 : cluster [INF] Activating manager daemon x
  2017-07-24 08:15:15.446025 mon.a mon.0 172.21.9.34:6789/0 47 : cluster [INF] Manager daemon x is now available


除了用 ``ceph -w`` 打印它们发出的日志行，还可以用
``ceph log last [n]`` 查看最近的 ``n`` 行集群日志。


监控健康检查信息
================
.. Monitoring Health Checks

Ceph 不间断地对自身状态做\ *健康检查*\ 。查到问题时，
会在 ``ceph status`` （或 ``ceph health`` ）的输出中反映出来。\
另外，检查失败时、或集群恢复时，相关消息也会发往集群日志。

例如，一个 OSD 挂掉时，状态输出的 ``health`` 那段可能会更新为如下：

::

    health: HEALTH_WARN
            1 osds down
            Degraded data redundancy: 21/63 objects degraded (33.333%), 16 pgs unclean, 16 pgs degraded

此时，也发送了集群日志消息，以记录此次健康检查失败事件：

::

    2017-07-25 10:08:58.265945 mon.a mon.0 172.21.9.34:6789/0 91 : cluster [WRN] Health check failed: 1 osds down (OSD_DOWN)
    2017-07-25 10:09:01.302624 mon.a mon.0 172.21.9.34:6789/0 94 : cluster [WRN] Health check failed: Degraded data redundancy: 21/63 objects degraded (33.333%), 16 pgs unclean, 16 pgs degraded (PG_DEGRADED)

当这个 OSD 恢复在线时，集群日志也会记录集群已回归健康状态：

::

    2017-07-25 10:11:11.526841 mon.a mon.0 172.21.9.34:6789/0 109 : cluster [WRN] Health check update: Degraded data redundancy: 2 pgs unclean, 2 pgs degraded, 2 pgs undersized (PG_DEGRADED)
    2017-07-25 10:11:13.535493 mon.a mon.0 172.21.9.34:6789/0 110 : cluster [INF] Health check cleared: PG_DEGRADED (was: Degraded data redundancy: 2 pgs unclean, 2 pgs degraded, 2 pgs undersized)
    2017-07-25 10:11:13.535577 mon.a mon.0 172.21.9.34:6789/0 111 : cluster [INF] Cluster is now healthy


网络性能检查
------------
.. Network Performance Checks

Ceph OSD 会相互发送心跳 ping 消息，
以监视守护进程的可用性和网络性能。
如果只是探测到了单次延迟的响应，这表明可能仅仅是 OSD 很忙碌。
但是如果在不同 OSD 对之间都探测到了多次延迟，
这表明可能是网络交换机故障、 NIC 故障、或者一个底层故障。

默认情况下，超过 1 秒（ 1000 毫秒）的心跳时间会产生一个健康检查，
一个 ``HEALTH_WARN`` 。例如：

::

    HEALTH_WARN Slow OSD heartbeats on back (longest 1118.001ms)

在 ``ceph health detail`` 命令的输出中，您可以看到\
哪些 OSD 出现了延迟以及延迟时间有多长。
``ceph health detail`` 的输出限制为 10 行。
下面是 ``ceph health detail`` 命令的输出示例：

::

    [WRN] OSD_SLOW_PING_TIME_BACK: Slow OSD heartbeats on back (longest 1118.001ms)
        Slow OSD heartbeats on back from osd.0 [dc1,rack1] to osd.1 [dc1,rack1] 1118.001 msec possibly improving
        Slow OSD heartbeats on back from osd.0 [dc1,rack1] to osd.2 [dc1,rack2] 1030.123 msec
        Slow OSD heartbeats on back from osd.2 [dc1,rack2] to osd.1 [dc1,rack1] 1015.321 msec
        Slow OSD heartbeats on back from osd.1 [dc1,rack1] to osd.0 [dc1,rack1] 1010.456 msec

要查看更多细节并收集完整的网络性能信息转储，
用 ``dump_osd_network`` 命令。
该命令通常发送到 Ceph 管理器守护进程，
但也可用于收集特定 OSD 的交互信息，方法是将其发送到这个 OSD 。
慢心跳的默认阈值是 1 秒（ 1000 毫秒），
但可以用毫秒数作为参数来覆盖该阈值。

要显示指定阈值为 0 的所有网络性能数据，向 mgr 发送以下命令：

.. prompt:: bash $

   ceph daemon /var/run/ceph/ceph-mgr.x.asok dump_osd_network 0

::

    {
        "threshold": 0,
        "entries": [
            {
                "last update": "Wed Sep  4 17:04:49 2019",
                "stale": false,
                "from osd": 2,
                "to osd": 0,
                "interface": "front",
                "average": {
                    "1min": 1.023,
                    "5min": 0.860,
                    "15min": 0.883
                },
                "min": {
                    "1min": 0.818,
                    "5min": 0.607,
                    "15min": 0.607
                },
                "max": {
                    "1min": 1.164,
                    "5min": 1.173,
                    "15min": 1.544
                },
                "last": 0.924
            },
            {
                "last update": "Wed Sep  4 17:04:49 2019",
                "stale": false,
                "from osd": 2,
                "to osd": 0,
                "interface": "back",
                "average": {
                    "1min": 0.968,
                    "5min": 0.897,
                    "15min": 0.830
                },
                "min": {
                    "1min": 0.860,
                    "5min": 0.563,
                    "15min": 0.502
                },
                "max": {
                    "1min": 1.171,
                    "5min": 1.216,
                    "15min": 1.456
                },
                "last": 0.845
            },
            {
                "last update": "Wed Sep  4 17:04:48 2019",
                "stale": false,
                "from osd": 0,
                "to osd": 1,
                "interface": "front",
                "average": {
                    "1min": 0.965,
                    "5min": 0.811,
                    "15min": 0.850
                },
                "min": {
                    "1min": 0.650,
                    "5min": 0.488,
                    "15min": 0.466
                },
                "max": {
                    "1min": 1.252,
                    "5min": 1.252,
                    "15min": 1.362
                },
            "last": 0.791
        },
        ...


.. _rados-monitoring-muting-health-checks:

屏蔽健康检查
------------
.. Muting health checks

健康检查可以屏蔽掉（ mute ），这样就不会影响集群的整体报告状态。
例如，如果集群产生了单个健康检查，然后您将该健康检查屏蔽掉，
那么集群将报告 ``HEALTH_OK`` 状态。要屏蔽特定的健康检查，
用与这个健康检查相对应的健康检查代码（请参阅 :ref:`health-checks` ），
并执行以下命令：

.. prompt:: bash $

   ceph health mute <code>

例如，要屏蔽 ``OSD_DOWN`` 健康检查，执行下列命令：

.. prompt:: bash $

   ceph health mute OSD_DOWN

屏蔽掉的也会展示在 ceph 健康检查命令的简报、和详情输出里。
例如，在上述场景下，集群将报告：

.. prompt:: bash $

   ceph health

::

   HEALTH_OK (muted: OSD_DOWN)

.. prompt:: bash $

   ceph health detail

::

   HEALTH_OK (muted: OSD_DOWN)
   (MUTED) OSD_DOWN 1 osds down
       osd.1 is down

取消屏蔽，执行下列命令：

.. prompt:: bash $

   ceph health unmute <code>

例如：

.. prompt:: bash $

   ceph health unmute OSD_DOWN

"health mute" （健康消息屏蔽）可以设置一个 TTL
（生存时间， **T**\ime **T**\o **L**\ive ）：
这意味着屏蔽将在指定时间后自动失效。
TTL 是可选的时间段参数，如下所示：

.. prompt:: bash $

   ceph health mute OSD_DOWN 4h    # mute for 4 hours
   ceph health mute MON_DOWN 15m   # mute for 15 minutes

通常情况下，如果之前屏蔽掉的健康检查已解决（例如，
在上述示例中引发 ``OSD_DOWN`` 健康检查的 OSD 已恢复正常），屏蔽就会失效。
如果同样的健康检查之后再次出现，还会以常规方式报告。

可以将健康静音设置为 sticky （有粘性）：意思是即使健康检查已经清除，屏蔽依然保持。
例如，要让健康静音成为“粘性”屏蔽，执行下列命令：

.. prompt:: bash $

   ceph health mute OSD_DOWN 1h --sticky   # ignore any/all down OSDs for next hour

如果触发健康检查的不健康状况恶化，大多数健康检查屏蔽会失效。
例如，假设有一个 OSD 出现故障，而它的健康检查屏蔽掉了。在这种情况下，
如果又有一个或多个 OSD 出现故障，那么这个健康屏蔽就会失效。
所有带有阈值的健康检查都会出现这种行为。


.. _rados-monitoring-pool-usage:

检查集群的使用情况
==================
.. Checking a Cluster's Usage Stats

要检查集群的数据用量及其在存储池内的分布情况，可以用 ``df`` 选项，
它和 Linux 上的 ``df`` 命令相似。执行下列命令：

.. prompt:: bash $

   ceph df

``ceph df`` 的输出像这样： ::

   --- RAW STORAGE ---
   CLASS     SIZE    AVAIL     USED  RAW USED  %RAW USED
   hdd    5.4 PiB  1.2 PiB  4.3 PiB   4.3 PiB      78.58
   ssd     22 TiB   19 TiB  2.7 TiB   2.7 TiB      12.36
   TOTAL  5.5 PiB  1.2 PiB  4.3 PiB   4.3 PiB      78.32

   --- POOLS ---
   POOL                         ID   PGS   STORED  OBJECTS     USED  %USED  MAX AVAIL
   .mgr                         11     1  558 MiB      141  1.6 GiB      0    5.8 TiB
   cephfs_meta                  13  1024  166 GiB   14.59M  499 GiB   2.74    5.8 TiB
   cephfs_data                  14  1024      0 B    1.17G      0 B      0    5.8 TiB
   cephfsECvol                  19  2048  2.8 PiB    1.81G  3.5 PiB  83.79    561 TiB
   .nfs                         20    32  9.7 KiB       61  118 KiB      0    5.8 TiB
   testbench                    71    32   12 GiB    3.14k   37 GiB      0    234 TiB
   default.rgw.buckets.data     76  2048  482 TiB  132.09M  643 TiB  47.85    526 TiB
   .rgw.root                    97     1  1.4 KiB        4   48 KiB      0    5.8 TiB
   default.rgw.log              98   256  3.6 KiB      209  408 KiB      0    5.8 TiB
   default.rgw.control          99     1      0 B        8      0 B      0    5.8 TiB
   default.rgw.meta            100   128  3.8 KiB       20  194 KiB      0    5.8 TiB
   default.rgw.buckets.index   101   256  4.2 MiB       33   13 MiB      0    5.8 TiB
   default.rgw.buckets.non-ec  102   128  5.6 MiB       13   17 MiB      0    5.8 TiB
   kubedata                    104   256   63 GiB   17.65k  188 GiB   0.03    234 TiB
   kubemeta                    105   256  241 MiB      166  724 MiB      0    5.8 TiB

- **CLASS:** 现有 CRUSH 设备类别的统计信息，例如， ``ssd`` 和 ``hdd`` 。
- **SIZE:** 集群管理着的存储容量；
- **AVAIL:** 集群的空闲空间总量；
- **USED:** 用户数据消耗的原始存储空间，包括 BlueStore 的数据库。
- **RAW USED:** 用户数据、内部开销、和保留容量占用的原始存储空间。
- **% RAW USED:** 已用原始存储空间比率。盯着这个数值，
  加上 ``backfillfull ratio`` 和 ``near full ratio`` 来防范集群达到用满阈值。
  详情见\ :ref:`storage-capacity` 。

更加详细的信息可以用如下命令显示：

.. prompt:: bash #

   ceph df detail

其输出类似于下面的实例： ::

   --- RAW STORAGE ---
   CLASS     SIZE    AVAIL     USED  RAW USED  %RAW USED
   hdd    5.4 PiB  1.2 PiB  4.3 PiB   4.3 PiB      78.58
   ssd     22 TiB   19 TiB  2.7 TiB   2.7 TiB      12.36
   TOTAL  5.5 PiB  1.2 PiB  4.3 PiB   4.3 PiB      78.32

   --- POOLS ---
   POOL                         ID   PGS   STORED   (DATA)   (OMAP)  OBJECTS     USED   (DATA)   (OMAP)  %USED  MAX AVAIL  QUOTA OBJECTS  QUOTA BYTES  DIRTY  USED COMPR  UNDER COMPR
   .mgr                         11     1  558 MiB  558 MiB      0 B      141  1.6 GiB  1.6 GiB      0 B      0    5.8 TiB            N/A          N/A    N/A         0 B          0 B
   cephfs_meta                  13  1024  166 GiB  206 MiB  166 GiB   14.59M  499 GiB  618 MiB  498 GiB   2.74    5.8 TiB            N/A          N/A    N/A         0 B          0 B
   cephfs_data                  14  1024      0 B      0 B      0 B    1.17G      0 B      0 B      0 B      0    5.8 TiB            N/A          N/A    N/A         0 B          0 B
   cephfsECvol                  19  2048  2.8 PiB  2.8 PiB   17 KiB    1.81G  3.5 PiB  3.5 PiB   21 KiB  83.79    561 TiB            N/A          N/A    N/A         0 B          0 B
   .nfs                         20    32  9.7 KiB  2.2 KiB  7.5 KiB       61  118 KiB   96 KiB   22 KiB      0    5.8 TiB            N/A          N/A    N/A         0 B          0 B
   testbench                    71    32   12 GiB   12 GiB  2.3 KiB    3.14k   37 GiB   37 GiB  6.9 KiB      0    234 TiB            N/A          N/A    N/A         0 B          0 B
   default.rgw.buckets.data     76  2048  482 TiB  482 TiB      0 B  132.09M  643 TiB  643 TiB      0 B  47.85    526 TiB            N/A          N/A    N/A     312 MiB      623 MiB
   .rgw.root                    97     1  1.4 KiB  1.4 KiB      0 B        4   48 KiB   48 KiB      0 B      0    5.8 TiB            N/A          N/A    N/A         0 B          0 B
   default.rgw.log              98   256  3.6 KiB  3.6 KiB      0 B      209  408 KiB  408 KiB      0 B      0    5.8 TiB            N/A          N/A    N/A         0 B          0 B
   default.rgw.control          99     1      0 B      0 B      0 B        8      0 B      0 B      0 B      0    5.8 TiB            N/A          N/A    N/A         0 B          0 B
   default.rgw.meta            100   128  3.8 KiB  3.2 KiB    671 B       20  194 KiB  192 KiB  2.0 KiB      0    5.8 TiB            N/A          N/A    N/A         0 B          0 B
   default.rgw.buckets.index   101   256  4.2 MiB      0 B  4.2 MiB       33   13 MiB      0 B   13 MiB      0    5.8 TiB            N/A          N/A    N/A         0 B          0 B
   default.rgw.buckets.non-ec  102   128  5.6 MiB      0 B  5.6 MiB       13   17 MiB      0 B   17 MiB      0    5.8 TiB            N/A          N/A    N/A         0 B          0 B
   kubedata                    104   256   63 GiB   63 GiB      0 B   17.65k  188 GiB  188 GiB      0 B   0.03    234 TiB            N/A       20 TiB    N/A         0 B          0 B
   kubemeta                    105   256  241 MiB  241 MiB  278 KiB      166  723 MiB  722 MiB  833 KiB      0    5.8 TiB            N/A          N/A    N/A         0 B          0 B


**POOLS:**  

输出的 **POOLS** 段展示了存储池列表及各存储池的\ *名义*\ 使用率。\
本段\ **没有**\ 展示副本、克隆品和快照占用情况。
例如，如果你把 1MB 的数据存储为对象，
那么名义使用率将是 1MB ，但考虑到副本数、克隆数、和快照数，
实际使用率可能是 2MB 或更多。

- **ID:** 存储池内指定节点的编号。
- **STORED:** 用户存储在存储池中的实际数据量。
  这与 Ceph 早期版本中的 USED 列类似，
  但计算（对于 BlueStore ！）更精确
  （因为间隙得到了正确处理）。

  - **(DATA):** RBD （RADOS 块设备）、 CephFS 文件数据、和
    RGW （RADOS 网关）对象数据占用的空间。
  - **(OMAP):** 键值对。主要是 CephFS 和 RGW （RADOS 网关）
    用来存储元数据。

- **OBJECTS:** 每个存储池所存储对象的名义数量
  （即除副本、克隆或快照外的对象数量）。
- **USED:** 在所有 OSD 上为一个存储池分配的空间。
  这包括复制空间、分配粒度空间以及与纠删码相关的开销空间。
  压缩节省的空间和对象内容间隙也计算在内。
  不过， BlueStore 的数据库不包括在
  USED 项下的报告中。

  - **(DATA):** RBD （RADOS 块设备）、 CephFS 文件数据、
    和 RGW （RADOS 网关）对象数据的对象使用情况。
  - **(OMAP):** 对象的键值对。主要是 CephFS 和 RGW
    （RADOS 网关）在用，用来做元数据存储。

- **%USED:** 每个存储池已用存储空间的名义百分比。
- **MAX AVAIL:** 可写入此存储池的名义数据量\
  的估计值。
- **QUOTA OBJECTS:** 配额对象的数量。
- **QUOTA BYTES:** 配额对象的字节数。
- **DIRTY:** 缓存池中已写入缓存池但\
  尚未刷回到后端存储池的对象数量。
  此字段仅在使用分级缓存时可用。
- **USED COMPR:** 为压缩数据分配的空间大小。
  除了已压缩的数据，还包括复制、分配粒度和\
  纠删码开销所需的所有空间。
- **UNDER COMPR:** 压缩过的（所有副本的总和）、
  以及值得以压缩形式存储的数据量。


.. note:: POOLS 段内的数值是名义上的，
   它们不包含副本、克隆、或快照。因此，
   输出里 POOLS 段中的 USED 和 %USED 数量之和不会等于
   RAW 段中的 USED 和 USED 数量之和。

.. note:: MAX AVAIL 数值是个复杂的函数，
   取决于所用的是多副本还是纠删码、
   映射存储与设备的 CRUSH 规则、那些设备的利用率、\
   还有配置的 ``mon_osd_full_ratio`` 选项。


检查 OSD 状态
=============
.. Checking OSD Status

要确定 OSD 状态是否为 ``up`` 且 ``in`` ，执行下列命令：

.. prompt:: bash #

	ceph osd stat

或者，执行下列命令：

.. prompt:: bash #

	ceph osd dump

根据 OSD 在 CRUSH 图里的位置来查看，执行下列命令：

.. prompt:: bash #

	ceph osd tree

打印出 CRUSH 树，显示主机、及其内的 OSD ， OSD 状态是否为 ``up`` 、
还有 OSD 的权重，执行下列命令：

.. code-block:: bash

   #ID CLASS WEIGHT  TYPE NAME             STATUS REWEIGHT PRI-AFF
    -1       3.00000 pool default
    -3       3.00000 rack mainrack
    -2       3.00000 host osd-host
     0   ssd 1.00000         osd.0             up  1.00000 1.00000
     1   ssd 1.00000         osd.1             up  1.00000 1.00000
     2   ssd 1.00000         osd.2             up  1.00000 1.00000

个中详情见\ :ref:`rados_operations_monitoring_osd_pg` 。


检查监视器状态
==============
.. Checking Monitor Status

如果你的集群有多个监视器，则需要执行某些“监视器状态”（ monitor status ）检查。
在启动集群后、读写数据前，应该检查法定人数状态。
运行着多个监视器时必须形成法定人数，才能保证集群是正常运行的。
最好周期性地检查监视器状态来确定它们在运行。

.. _display-mon-map:

要查看监视器图，执行下列命令：

.. prompt:: bash $

   ceph mon stat

或者，执行下列命令：

.. prompt:: bash $

   ceph mon dump

要检查监视器集群的法定人数状态，执行下列命令：

.. prompt:: bash $

   ceph quorum_status

Ceph 会返回法定人数状态，例如，包含 3 个监视器的 Ceph 集群可能\
返回下面的：

.. code-block:: javascript

	{ "election_epoch": 10,
	  "quorum": [
	        0,
	        1,
	        2],
	  "quorum_names": [
		"a",
		"b",
		"c"],
	  "quorum_leader_name": "a",
	  "monmap": { "epoch": 1,
	      "fsid": "444b489c-4f16-4b75-83f0-cb8097468898",
	      "modified": "2011-12-12 13:28:27.505520",
	      "created": "2011-12-12 13:28:27.505520",
	      "features": {"persistent": [
				"kraken",
				"luminous",
				"mimic"],
		"optional": []
	      },
	      "mons": [
	            { "rank": 0,
	              "name": "a",
	              "addr": "127.0.0.1:6789/0",
		      "public_addr": "127.0.0.1:6789/0"},
	            { "rank": 1,
	              "name": "b",
	              "addr": "127.0.0.1:6790/0",
		      "public_addr": "127.0.0.1:6790/0"},
	            { "rank": 2,
	              "name": "c",
	              "addr": "127.0.0.1:6791/0",
		      "public_addr": "127.0.0.1:6791/0"}
	           ]
	  }
	}


检查 MDS 状态
=============
.. Checking MDS Status

元数据服务器为 CephFS 提供元数据服务。元数据服务器有两组\
状态： ``up | down`` 和 ``active | inactive`` 。
要查看元数据服务器状态为 ``up`` 且 ``active`` ，执行下列命令：

.. prompt:: bash $

   ceph mds stat

要展示元数据服务器们的详细状态，执行下列命令：

.. prompt:: bash $

   ceph fs dump


检查归置组状态
==============
.. Checking Placement Group States

归置组（ PG ）把对象映射到 OSD 。归置组处于监控下，以确保它们的状态是
``active`` 且 ``clean`` 。参见\ :ref:`rados_operations_monitoring_osd_pg`\ 。


.. _rados-monitoring-using-admin-socket:

使用管理套接字
==============
.. Using the Admin Socket

Ceph 管理套接字允许你通过套接字接口查询守护进程，
它们默认存在于 ``/var/run/ceph`` 下。
要通过管理套接字访问某个守护进程，先登录它所在的主机、再执行下列命令：

.. prompt:: bash $

   ceph daemon {daemon-name}
   ceph daemon {path-to-socket-file}

比如，这是下面这两种用法是等价的：

.. prompt:: bash $

   ceph daemon osd.0 foo
   ceph daemon /var/run/ceph/ceph-osd.0.asok foo

运行管理员套接字命令有两种方法：(1) 如上所述，用 ``ceph daemon`` ，
这种方法绕过了监视器，假定已经直接登录守护进程所在主机；
(2) 用 ``ceph tell {daemon-type}.{id}`` 命令，这种方法由监视器转发，
不需要访问那个守护进程所在的主机。

用 ``raise`` 命令向守护进程发送信号，效果和运行 ``kill -X {daemon.pid}`` 命令一样。
通过 ``ceph tell`` 执行命令时，可以向守护进程发送信号，而无需访问其主机：

.. prompt:: bash $

   ceph daemon {daemon-name} raise HUP
   ceph tell {daemon-type}.{id} raise -9

查看可用的管理套接字命令，执行下列命令：

.. prompt:: bash $

   ceph daemon {daemon-name} help

管理套接字命令允许你在运行时查看和修改配置。
关于查看配置信息的更多内容，见\ :ref:`configuring_ceph_runtime_view`\ 。


信使状态
========
.. Messenger Status

Ceph 守护进程和 librados 客户端支持管理员套接字命令 ``messenger dump`` ，
该命令可显示有关网络连接、套接字、绑定地址和内核 TCP 统计信息
（通过 tcp(7) TCP_INFO ）这些运行时快照。

.. note:: 在创建快照时，被查询的信使需要锁定连接数据结构。
   这个锁定的持续时间大约为几十毫秒。
   这可能会干扰正常运行。
   可以用 ``dumpcontents`` 参数限制转储的数据结构。


实例
----
.. Examples

执行命令却没指定要转储的信使时，会返回可用的信使列表：

.. prompt:: bash $

   ceph tell osd.0 messenger dump

.. code-block:: json

 {
    "messengers": [
        "client",
        "cluster",
        "hb_back_client",
        "hb_back_server",
        "hb_front_client",
        "hb_front_server",
        "ms_objecter",
        "temp_mon_client"
    ]
  }

``client`` 和 ``cluster`` 信使对应配置的客户端网络、集群网络
（见 :doc:`/rados/configuration/network-config-ref` ）。
带 ``hb_`` 前缀的信使是心跳系统的一部分。

罗列出客户端信使上当前的所有连接：

.. code-block:: bash

        ceph tell osd.0 messenger dump client \
            | jq -r '.messenger.connections[].async_connection |
                [.conn_id, .socket_fd, .worker_id,
                    if .status.connected then "connected" else "disconnected" end,
                    .state,
                    "\(.peer.type).\(.peer.entity_name.id).\(.peer.id)",
                    .protocol.v2.con_mode, .protocol.v2.crypto.rx, .protocol.v2.compression.rx] |
                @tsv'

::

  249     102     0       connected       STATE_CONNECTION_ESTABLISHED    client.admin.6407       crc    PLAIN   UNCOMPRESSED
  242     99      1       connected       STATE_CONNECTION_ESTABLISHED    client.rgw.8000.4473    crc     PLAIN   UNCOMPRESSED
  248     89      1       connected       STATE_CONNECTION_ESTABLISHED    mgr..-1 secure  AES-128-GCM     UNCOMPRESSED
  32      101     2       connected       STATE_CONNECTION_ESTABLISHED    client.rgw.8000.4483    crc     PLAIN   UNCOMPRESSED
  3       86      2       connected       STATE_CONNECTION_ESTABLISHED    mon..-1 secure  AES-128-GCM     UNCOMPRESSED
  244     102     0       connected       STATE_CONNECTION_ESTABLISHED    client.admin.6383       crc     PLAIN   UNCOMPRESSED

打印活跃连接及其 TCP 往返时间和重传计数器：

.. code-block:: bash

        ceph tell osd.0 messenger dump client --tcp-info \
            | jq -r '.messenger.connections[].async_connection |
                select(.status.connected) |
                select(.peer.type != "client") |
                [.conn_id, .socket_fd, .worker_id,
                    "\(.peer.type).\(.peer.global_id)",
                    .tcp_info.tcpi_rtt_us, .tcp_info.tcpi_rttvar_us, .tcp_info.tcpi_total_retrans] |
                @tsv'

::

	248     89      1       mgr.0   863     1677    0
	3       86      2       mon.0   230     278     0


.. _data_availability_score:

跟踪一个集群的数据可用性评分
============================
.. Tracking Data Availability Score of a Cluster

Ceph 集群内部会跟踪每个存储池的数据可用性。
要检查集群中每个存储池的数据可用性评分，
执行下列命令：

.. prompt:: bash #

   ceph osd pool availability-status

输出实例：

::

   POOL         UPTIME  DOWNTIME  NUMFAILURES  MTBF  MTTR  SCORE     AVAILABLE
   rbd             2m     21s           1        2m   21s  0.888889     1
   .mgr             86s     0s          0        0s   0s        1       1
   cephfs.a.meta    77s     0s          0        0s   0s        1       1
   cephfs.a.data    76s     0s          0        0s   0s        1       1

上述时间值已进行四舍五入以方便阅读。要查看精确到秒的值，
可以用 ``--format`` 选项再加上 ``json`` 或 ``json-pretty`` 值。

当存储池中至少有一个 PG 变得不可用、或者存储池中至少有一个对象找不到时，
这个存储池状态就会变成 ``unavailable`` （不可用）。
除此之外，这个存储池的状态都是 ``available`` （可用的）。
根据存储池当前和先前的状态，我们更新
``uptime`` （运行时间）和 ``downtime`` （停机时间）的数值：

================ =============== =============== =================
  先前的状态       当前的状态      Uptime 更新     Downtime 更新
================ =============== =============== =================
 Available        Available       +diff time      no update
 Available        Unavailable     +diff time      no update
 Unavailable      Available       +diff time      no update
 Unavailable      Unavailable     no update       +diff time
================ =============== =============== =================

用更新过的 ``uptime`` 和 ``downtime`` 数值，
我们会计算出每个存储池的平均故障时间（ MTBF ）
和平均恢复时间（ MTTR ）。然后，
用 MTBF 与总时间的比值来算出可用性得分。

这个评分每秒更新一次。
对于连续的成功更新之间发生并撤销的瞬时存储池变更，
它们不会被捕捉到。这个间隔时间长短可以配置，
执行下列命令：

.. prompt:: bash #

   ceph config set mon pool_availability_update_interval 2

此命令把更新间隔设置为 2 秒。
注意，这个间隔时间不能比
``paxos_propose_interval`` 配置的数值小。


这个功能默认是打开的。要关闭此功能，比如在计划内的停机时间，
可以把 ``enable_availability_tracking`` 配置选项设置为 ``false`` 。

.. prompt:: bash #

   ceph config set mon enable_availability_tracking false

此功能关闭后，最后计算出的评分将会保留下来。
等此功能再次打开后，这个评分也会重新开始更新。

如果需要，系统也允许清除指定存储池的数据可用性评分，
执行下列命令：

.. prompt:: bash #

   ceph osd pool clear-availability-status <pool-name>

.. note:: 在这个功能被禁用时，清除评分命令也不可用。
