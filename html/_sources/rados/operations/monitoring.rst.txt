.. Monitoring a Cluster

==========
 监控集群
==========

集群运行起来后，你可以用 ``ceph`` 工具来监控，典型的监控包括\
检查 OSD 状态、监视器状态、归置组状态和元数据服务器状态。


.. Using the command line

使用命令行
==========

.. Interactive mode

交互模式
--------

要在交互模式下运行 ``ceph`` ，不要带参数运行 ``ceph`` ，例如： ::

	ceph
	ceph> health
	ceph> status
	ceph> quorum_status
	ceph> mon_status

.. Non-default paths

非默认的路径
------------

如果你的配置文件或密钥环不在默认位置内，可以手动指定其位置： ::

   ceph -c /path/to/conf -k /path/to/keyring health


.. Checking a Cluster's Status

检查集群的状态
==============

启动集群后、读写数据前，先检查下集群的健康状态。

可以用下面的命令检查集群状态： ::

	ceph status

或者： ::

        ceph -s

在交互模式下，输入 ``status`` 再按回车 **Enter** 。 ::

	ceph> status

Ceph 就会打印出集群状态。例如，一个小型的演示集群，各种服务\
都有一个，可能会打印如下的： ::

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

.. topic:: Ceph 如何计算数据量

   ``usage`` 值反映了\ *事实上*\ 已占用的原始存储空间。
   ``xxx GB / xxx GB`` 值则是剩余空间（较小的数）与集群总容量\
   的比较。理论数值反映了所存储数据的原始尺寸，未计算其副本、\
   克隆、或快照空间，所以数据存储实际占用的空间通常会超过\
   理论数值，因为 Ceph 会自动创建数据副本，另外存储空间也可能\
   用于克隆和快照。


.. Watching a Cluster

观察集群
========

除了各守护进程的本地日志， Ceph 集群还维护着一个\
*集群日志*\ ，它记录着事关整个系统的高级事件。此类日志记录在\
监视器服务器的磁盘上（默认为 ``/var/log/ceph/ceph.log`` ），\
也可以通过命令行监控。

要持续关注集群日志，用下列命令： ::

	ceph -w

Ceph 会打印系统的状态，然后是正发生着的各日子消息。例如： ::

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


.. Monitoring Health Checks

监控健康检查信息
================

Ceph 不间断地对自身状态做\ *健康检查*\ 。查到问题时，会在
``ceph status`` （或 ``ceph health`` ）的输出中反映出来。\
另外，检查失败时、或集群恢复时，相关消息也会发往集群日志。

例如，一个 OSD 挂掉时，状态输出的 ``health`` 那段可能会更新为\
如下： ::

    health: HEALTH_WARN
            1 osds down
            Degraded data redundancy: 21/63 objects degraded (33.333%), 16 pgs unclean, 16 pgs degraded

此时，也发送了集群日志消息，以记录此次健康检查失败事件： ::

    2017-07-25 10:08:58.265945 mon.a mon.0 172.21.9.34:6789/0 91 : cluster [WRN] Health check failed: 1 osds down (OSD_DOWN)
    2017-07-25 10:09:01.302624 mon.a mon.0 172.21.9.34:6789/0 94 : cluster [WRN] Health check failed: Degraded data redundancy: 21/63 objects degraded (33.333%), 16 pgs unclean, 16 pgs degraded (PG_DEGRADED)

当这个 OSD 恢复在线时，集群日志也会记录集群已回归健康状态： ::

    2017-07-25 10:11:11.526841 mon.a mon.0 172.21.9.34:6789/0 109 : cluster [WRN] Health check update: Degraded data redundancy: 2 pgs unclean, 2 pgs degraded, 2 pgs undersized (PG_DEGRADED)
    2017-07-25 10:11:13.535493 mon.a mon.0 172.21.9.34:6789/0 110 : cluster [INF] Health check cleared: PG_DEGRADED (was: Degraded data redundancy: 2 pgs unclean, 2 pgs degraded, 2 pgs undersized)
    2017-07-25 10:11:13.535577 mon.a mon.0 172.21.9.34:6789/0 111 : cluster [INF] Cluster is now healthy


.. Network Performance Checks

网络性能检查
------------

Ceph OSDs send heartbeat ping messages amongst themselves to monitor daemon availability.  We
also use the response times to monitor network performance.
While it is possible that a busy OSD could delay a ping response, we can assume
that if a network switch fails multiple delays will be detected between distinct pairs of OSDs.

By default we will warn about ping times which exceed 1 second (1000 milliseconds).

::

    HEALTH_WARN Slow OSD heartbeats on back (longest 1118.001ms)

The health detail will add the combination of OSDs are seeing the delays and by how much.  There is a limit of 10
detail line items.

::

    [WRN] OSD_SLOW_PING_TIME_BACK: Slow OSD heartbeats on back (longest 1118.001ms)
        Slow OSD heartbeats on back from osd.0 [dc1,rack1] to osd.1 [dc1,rack1] 1118.001 msec possibly improving
        Slow OSD heartbeats on back from osd.0 [dc1,rack1] to osd.2 [dc1,rack2] 1030.123 msec
        Slow OSD heartbeats on back from osd.2 [dc1,rack2] to osd.1 [dc1,rack1] 1015.321 msec
        Slow OSD heartbeats on back from osd.1 [dc1,rack1] to osd.0 [dc1,rack1] 1010.456 msec

To see even more detail and a complete dump of network performance information the ``dump_osd_network`` command can be used.  Typically, this would be
sent to a mgr, but it can be limited to a particular OSD's interactions by issuing it to any OSD.  The current threshold which defaults to 1 second
(1000 milliseconds) can be overridden as an argument in milliseconds.

The following command will show all gathered network performance data by specifying a threshold of 0 and sending to the mgr.

::

    $ ceph daemon /var/run/ceph/ceph-mgr.x.asok dump_osd_network 0
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



.. Muting health checks

屏蔽健康检查
------------

Health checks can be muted so that they do not affect the overall
reported status of the cluster.  Alerts are specified using the health
check code (see :ref:`health-checks`)::

  ceph health mute <code>

For example, if there is a health warning, muting it will make the
cluster report an overall status of ``HEALTH_OK``.  For example, to
mute an ``OSD_DOWN`` alert,::

  ceph health mute OSD_DOWN

Mutes are reported as part of the short and long form of the ``ceph health`` command.
For example, in the above scenario, the cluster would report::

  $ ceph health
  HEALTH_OK (muted: OSD_DOWN)
  $ ceph health detail
  HEALTH_OK (muted: OSD_DOWN)
  (MUTED) OSD_DOWN 1 osds down
      osd.1 is down

A mute can be explicitly removed with::

  ceph health unmute <code>

For example,::

  ceph health unmute OSD_DOWN

A health check mute may optionally have a TTL (time to live)
associated with it, such that the mute will automatically expire
after the specified period of time has elapsed.  The TTL is specified as an optional
duration argument, e.g.::

  ceph health mute OSD_DOWN 4h    # mute for 4 hours
  ceph health mute MON_DOWN 15m   # mute for 15  minutes

Normally, if a muted health alert is resolved (e.g., in the example
above, the OSD comes back up), the mute goes away.  If the alert comes
back later, it will be reported in the usual way.

It is possible to make a mute "sticky" such that the mute will remain even if the
alert clears.  For example,::

  ceph health mute OSD_DOWN 1h --sticky   # ignore any/all down OSDs for next hour

Most health mutes also disappear if the extent of an alert gets worse.  For example,
if there is one OSD down, and the alert is muted, the mute will disappear if one
or more additional OSDs go down.  This is true for any health alert that involves
a count indicating how much or how many of something is triggering the warning or
error.


.. Detecting configuration issues

检测配置问题
============

除了 Ceph 持续运行时进行的自我健康检查，还有一些配置问题只能用\
外部工具探测。

可以用 `ceph-medic`_ 工具另行检查你的 Ceph 集群配置。


.. Checking a Cluster's Usage Stats

检查集群的使用情况
==================

要检查集群的数据用量及其在存储池内的分布情况，可以用 ``df``
选项，它和 Linux 上的 ``df`` 相似。如下： ::

	ceph df

输出中的 **RAW STORAGE** 段概述了你的集群管理着的存储空间。

- **CLASS:** The class of OSD device (or the total for the cluster)
- **SIZE:** 集群管理着的存储容量；
- **AVAIL:** 集群的空闲空间总量；
- **USED:** 用户数据消耗的原始存储空间；
- **RAW USED:** 已使用的原始存储空间总量，包括用户数据、内部开销、\
  或保留容量；
- **% RAW USED:** 已用原始存储空间比率。用此值参照 ``full ratio``
  和 ``near full ratio`` 来确保不会用尽集群空间。详情见\
  `存储容量`_\ 。

输出的 **POOLS** 段展示了存储池列表及各存储池的大致使用率。\
本段\ **没有**\ 展示副本、克隆品和快照占用情况。例如，如果你把
1MB 的数据存储为对象，理论使用率将是 1MB ，但考虑到副本数、克\
隆数、和快照数，实际使用率可能是 2MB 或更多。

- **NAME:** 存储池名字；
- **ID:** 存储池唯一标识符；
- **USED:** 大概数据量，单位为 KB 、 MB 或 GB ；
- **%USED:** 各存储池的大概使用率；
- **MAX AVAIL:** 这个存储池还能容纳的数据量估值；
- **OBJECTS:** 各存储池内的大概对象数。

.. note:: **POOLS** 段内的数字是理论值，它们不包含副本、快照或\
   克隆。因此，它与 **USED** 和 **%USED** 数量之和不会达到
   **GLOBAL** 段中的 **RAW USED** 和 **%RAW USED** 数量。

.. note:: **MAX AVAIL** 数值计算很复杂，涉及到存储池是副本的还\
   是纠删码的、映射存储与设备的 CRUSH 规则、那些设备的利用率、\
   还有配置的 mon_osd_full_ratio 。


.. Checking OSD Status

检查 OSD 状态
=============

你可以执行下列命令来确定 OSD 状态为 ``up`` 且 ``in`` ： ::

	ceph osd stat

或者： ::

	ceph osd dump

你也可以根据 OSD 在 CRUSH 图里的位置来查看： ::

	ceph osd tree

Ceph 会打印 CRUSH 的树状态、它的 OSD 例程、状态、权重： ::

	#ID CLASS WEIGHT  TYPE NAME             STATUS REWEIGHT PRI-AFF
	 -1       3.00000 pool default
	 -3       3.00000 rack mainrack
	 -2       3.00000 host osd-host
	  0   ssd 1.00000         osd.0             up  1.00000 1.00000
	  1   ssd 1.00000         osd.1             up  1.00000 1.00000
	  2   ssd 1.00000         osd.2             up  1.00000 1.00000

个中详情见\ `监控 OSD 和归置组`_\ 。


.. Checking Monitor Status

检查监视器状态
==============

如果你有多个监视器（很可能），你启动集群后、读写数据前应该检查\
监视器法定人数状态。运行着多个监视器时必须形成法定人数，最好\
周期性地检查监视器状态来确定它们在运行。

要查看监视器图，执行下面的命令： ::

	ceph mon stat

或者： ::

	ceph mon dump

要检查监视器的法定人数状态，执行下面的命令： ::

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


.. Checking MDS Status

检查 MDS 状态
=============

元数据服务器为 Ceph 文件系统提供元数据服务，元数据服务器有两种\
状态： ``up | down`` 和 ``active | inactive`` ，执行下面的命令\
查看元数据服务器状态为 ``up`` 且 ``active`` ： ::

	ceph mds stat

要展示元数据集群的详细状态，执行下面的命令： ::

	ceph fs dump


.. Checking Placement Group States

检查归置组状态
==============

归置组把对象映射到 OSD 。监控归置组时，我们希望它们的状态是
``active`` 且 ``clean`` 。个中详情见\ `监控 OSD 和归置组`_\ 。

.. _监控 OSD 和归置组: ../monitoring-osd-pg


.. Using the Admin Socket
.. _rados-monitoring-using-admin-socket:

使用管理套接字
==============

Ceph 管理套接字允许你通过套接字接口查询守护进程，它们默认存在于
``/var/run/ceph`` 下。要通过管理套接字访问某个守护进程，先登录\
它所在的主机、再执行下列命令： ::

	ceph daemon {daemon-name}
	ceph daemon {path-to-socket-file}

比如，这是下面这两种用法是等价的： ::

	ceph daemon osd.0 foo
	ceph daemon /var/run/ceph/ceph-osd.0.asok foo

用下列命令查看可用的管理套接字命令： ::

	ceph daemon {daemon-name} help

管理套接字命令允许你在运行时查看和修改配置，见\
`查看运行时配置`_\ 。

另外，你可以在运行时直接修改配置选项（也就是说管理套接字会绕过\
监视器，不要求你直接登录宿主主机，不像
``ceph {daemon-type} tell {id} config set`` 依赖监视器。

.. _查看运行时配置: ../../configuration/ceph-conf#viewing-a-configuration-at-runtime
.. _存储容量: ../../configuration/mon-config-ref#storage-capacity
.. _ceph-medic: http://docs.ceph.com/ceph-medic/master/
