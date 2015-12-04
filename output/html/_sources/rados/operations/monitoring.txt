==========
 监控集群
==========

集群运行起来后，你可以用 ``ceph`` 工具来监控，典型的监控包括检查 OSD 状态、监视器状\
态、归置组状态和元数据服务器状态。


交互模式
========

要在交互模式下运行 ``ceph`` ，不要带参数运行 ``ceph`` ，例如： ::

	ceph
	ceph> health
	ceph> status
	ceph> quorum_status
	ceph> mon_status


检查集群健康状况
================

启动集群后、读写数据前，先检查下集群的健康状态。你可以用下面的命令检查： ::

	ceph health

如果你的配置文件或密钥环不在默认路径下，你得指定： ::

   ceph -c /path/to/conf -k /path/to/keyring health

集群起来的时候，你也许会碰到像  ``HEALTH_WARN XXX num placement groups stale`` \
这样的健康告警，等一会再检查下。集群准备好的话 ``ceph health`` 会给出像 \
``HEALTH_OK`` 一样的消息，这时候就可以开始使用集群了。


观察集群
========

要观察集群内正发生的事件，打开一个新终端，然后输入： ::

	ceph -w

Ceph 会打印各种事件。例如一个包括 1 个监视器、和 2 个 OSD 的小型 Ceph 集群可能会打\
印出这些： ::

    cluster b370a29d-9287-4ca3-ab57-3d824f65e339
     health HEALTH_OK
     monmap e1: 1 mons at {ceph1=10.0.0.8:6789/0}, election epoch 2, quorum 0 ceph1
     osdmap e63: 2 osds: 2 up, 2 in
      pgmap v41338: 952 pgs, 20 pools, 17130 MB data, 2199 objects
            115 GB used, 167 GB / 297 GB avail
                 952 active+clean

    2014-06-02 15:45:21.655871 osd.0 [INF] 17.71 deep-scrub ok
    2014-06-02 15:45:47.880608 osd.1 [INF] 1.0 scrub ok
    2014-06-02 15:45:48.865375 osd.1 [INF] 1.3 scrub ok
    2014-06-02 15:45:50.866479 osd.1 [INF] 1.4 scrub ok
    2014-06-02 15:45:01.345821 mon.0 [INF] pgmap v41339: 952 pgs: 952 active+clean; 17130 MB data, 115 GB used, 167 GB / 297 GB avail
    2014-06-02 15:45:05.718640 mon.0 [INF] pgmap v41340: 952 pgs: 1 active+clean+scrubbing+deep, 951 active+clean; 17130 MB data, 115 GB used, 167 GB / 297 GB avail
    2014-06-02 15:45:53.997726 osd.1 [INF] 1.5 scrub ok
    2014-06-02 15:45:06.734270 mon.0 [INF] pgmap v41341: 952 pgs: 1 active+clean+scrubbing+deep, 951 active+clean; 17130 MB data, 115 GB used, 167 GB / 297 GB avail
    2014-06-02 15:45:15.722456 mon.0 [INF] pgmap v41342: 952 pgs: 952 active+clean; 17130 MB data, 115 GB used, 167 GB / 297 GB avail
    2014-06-02 15:46:06.836430 osd.0 [INF] 17.75 deep-scrub ok
    2014-06-02 15:45:55.720929 mon.0 [INF] pgmap v41343: 952 pgs: 1 active+clean+scrubbing+deep, 951 active+clean; 17130 MB data, 115 GB used, 167 GB / 297 GB avail


输出信息里包含：

- 集群唯一标识符
- 集群健康状况
- 监视器图元版本、和监视器法定人数状态
- OSD 图元版本、和 OSD 状态摘要
- 归置组图版本
- 归置组和存储池数量
- 其内存储的数据和对象数量的\ *粗略*\ 统计，以及
- 数据总量

.. topic:: Ceph 如何计算数据量

   ``used`` 值反映了\ *确实*\ 已占用的原始存储空间。 ``xxx GB / xxx GB`` 值则是剩\
   余空间（较小的数）与集群总容量的比较。理论数值反映了所存储数据的原始尺寸，未计算\
   其副本、克隆、或快照空间，所以数据存储实际占用的空间通常会超过理论数值，因为 \
   Ceph 会自动创建数据副本，另外存储空间也可能用于克隆和快照。


检查集群的使用情况
==================

要检查集群的数据用量及其在存储池内的分布情况，可以用 ``df`` 选项，它和 Linux 上的 \
``df`` 相似。如下： ::

	ceph df

输出的 **GLOBAL** 段展示了数据所占用集群存储空间的概要。

- **SIZE:** 集群的总容量；
- **AVAIL:** 集群的空闲空间总量；
- **RAW USED:** 已用存储空间总量；
- **% RAW USED:** 已用存储空间比率。用此值参照 ``full ratio`` 和 ``near full \
  ratio`` 来确保不会用尽集群空间。详情见\ `存储容量`_\ 。

输出的 **POOLS** 段展示了存储池列表及各存储池的大致使用率。本段\ **没有**\ 展示副\
本、克隆品和快照占用情况。例如，如果你把 1MB 的数据存储为对象，理论使用率将是 \
1MB ，但考虑到副本数、克隆数、和快照数，实际使用率可能是 2MB 或更多。

- **NAME:** 存储池名字；
- **ID:** 存储池唯一标识符；
- **USED:** 大概数据量，单位为 KB 、 MB 或 GB ；
- **%USED:** 各存储池的大概使用率；
- **Objects:** 各存储池内的大概对象数。

.. note:: **POOLS** 段内的数字是理论值，它们不包含副本、快照或克隆。因此，它与 \
   **USED** 和 **%USED** 数量之和不会达到 **GLOBAL** 段中的 **RAW USED** 和 \
   **%RAW USED** 数量。


检查集群状态
============

要检查集群的状态，执行下面的命令： ::

	ceph status

或者： ::

	ceph -s

在交互模式下，输入 ``status`` 然后按\ **回车**\ ： ::

	ceph> status

Ceph 将打印集群状态，例如一个包括 1 个监视器、和 2 个 OSD 的小型 Ceph 集群可能打印： ::

    cluster b370a29d-9287-4ca3-ab57-3d824f65e339
     health HEALTH_OK
     monmap e1: 1 mons at {ceph1=10.0.0.8:6789/0}, election epoch 2, quorum 0 ceph1
     osdmap e63: 2 osds: 2 up, 2 in
      pgmap v41332: 952 pgs, 20 pools, 17130 MB data, 2199 objects
            115 GB used, 167 GB / 297 GB avail
                   1 active+clean+scrubbing+deep
                 951 active+clean


检查 OSD 状态
=============

你可以执行下列命令来确定 OSD 状态为 ``up`` 且 ``in`` ： ::

	ceph osd stat

或者： ::

	ceph osd dump

你也可以根据 OSD 在 CRUSH 图里的位置来查看： ::

	ceph osd tree

Ceph 会打印 CRUSH 的树状态、它的 OSD 例程、状态、权重： ::

	# id	weight	type name	up/down	reweight
	-1	3	pool default
	-3	3		rack mainrack
	-2	3			host osd-host
	0	1				osd.0	up	1
	1	1				osd.1	up	1
	2	1				osd.2	up	1

个中详情见\ `监控 OSD 和归置组`_\ 。


检查监视器状态
==============

如果你有多个监视器（很可能），你启动集群后、读写数据前应该检查监视器法定人数状态。\
运行着多个监视器时必须形成法定人数，最好周期性地检查监视器状态来确定它们在运行。

要查看监视器图，执行下面的命令： ::

	ceph mon stat

或者： ::

	ceph mon dump

要检查监视器的法定人数状态，执行下面的命令： ::

	ceph quorum_status

Ceph 会返回法定人数状态，例如，包含 3 个监视器的 Ceph 集群可能返回下面的：

.. code-block:: javascript

	{ "election_epoch": 10,
	  "quorum": [
	        0,
	        1,
	        2],
	  "monmap": { "epoch": 1,
	      "fsid": "444b489c-4f16-4b75-83f0-cb8097468898",
	      "modified": "2011-12-12 13:28:27.505520",
	      "created": "2011-12-12 13:28:27.505520",
	      "mons": [
	            { "rank": 0,
	              "name": "a",
	              "addr": "127.0.0.1:6789\/0"},
	            { "rank": 1,
	              "name": "b",
	              "addr": "127.0.0.1:6790\/0"},
	            { "rank": 2,
	              "name": "c",
	              "addr": "127.0.0.1:6791\/0"}
	           ]
	    }
	}


检查 MDS 状态
=============

元数据服务器为 Ceph 文件系统提供元数据服务，元数据服务器有两种状态： \
``up | down`` 和 ``active | inactive`` ，执行下面的命令查看元数据服务器状态\
为 ``up`` 且 ``active`` ： ::

	ceph mds stat

要展示元数据集群的详细状态，执行下面的命令： ::

	ceph mds dump


检查归置组状态
==============

归置组把对象映射到 OSD 。监控归置组时，我们希望它们的状态是 ``active`` 且 \
``clean`` 。个中详情见\ `监控 OSD 和归置组`_\ 。

.. _监控 OSD 和归置组: ../monitoring-osd-pg


使用管理套接字
==============

Ceph 管理套接字允许你通过套接字接口查询守护进程，它们默认存在于 ``/var/run/ceph`` \
下。要通过管理套接字访问某个守护进程，先登录它所在的主机、再执行下列命令： ::

	ceph daemon {daemon-name}
	ceph daemon {path-to-socket-file}

比如，这是下面这两种用法是等价的： ::

	ceph daemon osd.0 foo
	ceph daemon /var/run/ceph/ceph-osd.0.asok foo

用下列命令查看可用的管理套接字命令： ::

	ceph daemon {daemon-name} help

管理套接字命令允许你在运行时查看和修改配置，见\ `查看运行时配置`_\ 。

另外，你可以在运行时直接修改配置选项（也就是说管理套接字会绕过监视器，不要求你直接登\
录宿主主机，不像 ``ceph {daemon-type} tell {id} injectargs`` 依赖监视器。

.. _查看运行时配置: ../../configuration/ceph-conf#ceph-runtime-config
.. _存储容量: ../../configuration/mon-config-ref#storage-capacity
