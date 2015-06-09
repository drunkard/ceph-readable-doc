.. index:: control, commands

==========
 控制命令
==========


监视器命令
==========

监视器命令用 ``ceph`` 工具发出： ::

	ceph [-m monhost] {command}

命令格式通常是（但不总是）： ::

	ceph {subsystem} {command}


系统命令
========

下列命令显示集群状态： ::

	ceph -s
	ceph status

下列命令显示集群状态的运行摘要、及主要事件： ::

	ceph -w

下列命令显示监视器法定人数状态，包括哪些监视器参与着、哪个是首领。 ::

	ceph quorum_status

下列命令查询单个监视器状态，包括是否在法定人数里。 ::

	ceph [-m monhost] mon_status


认证子系统
==========

要添加一个 OSD 的密钥环，执行下列命令： ::

	ceph auth add {osd} {--in-file|-i} {path-to-osd-keyring}

要列出集群的密钥及其能力，执行下列命令： ::

	ceph auth list


归置组子系统
============

要显示所有归置组的统计信息，执行下列命令： ::

	ceph pg dump [--format {format}]

可用输出格式有 ``plain`` （默认）和 ``json`` 。

要显示卡在某状态的所有归置组，执行下列命令： ::

	ceph pg dump_stuck inactive|unclean|stale|undersized|degraded [--format {format}] [-t|--threshold {seconds}]


``--format`` 可以是 ``plain`` （默认）或 ``json``

``--threshold`` 定义了多久算“卡住了”（默认 300 秒）

**Inactive** 归置组不能处理读或写，因为它们在等待数据及时更新的 OSD 回来。

**Unclean** 归置组包含副本数未达期望值的对象，它们应该在恢复中。

**Stale** 归置组处于未知状态——归置组所托付的 OSD 有一阵没向监视器报告了（由 \
``mon osd report timeout`` 配置）。

删除“丢失”对象，或者恢复到其先前状态，可以是前一版本、或如果刚创建就干脆删除。 ::

	ceph pg {pgid} mark_unfound_lost revert|delete


OSD 子系统
==========

查询 OSD 子系统状态： ::

	ceph osd stat

把最新的 OSD 运行图拷贝到一个文件，参见 `osdmaptool`_ ： ::

	ceph osd getmap -o file

.. _osdmaptool: ../../man/8/osdmaptool

从最新 OSD 运行图拷出 CRUSH 图： ::

	ceph osd getcrushmap -o file

前述功能等价于： ::

	ceph osd getmap -o /tmp/osdmap
	osdmaptool /tmp/osdmap --export-crush file

转储 OSD 运行图， ``-f`` 的可用格式有 ``plain`` 和 ``json`` ，如未指定 \
``--format`` 则转储为纯文本。 ::

	ceph osd dump [--format {format}]

把 OSD 运行图转储为树，每个 OSD 一行、包含权重和状态。 ::

	ceph osd tree [--format {format}]

找出某对象在哪里或应该在哪里： ::

	ceph osd map <pool-name> <object-name>

增加或挪动一个新 OSD 条目，要给出 id/name/weight 和位置参数。 ::

	ceph osd crush set {id} {weight} [{loc1} [{loc2} ...]]

从现有 CRUSH 图删除存在的条目（ OSD ）： ::

	ceph osd crush remove {name}

从现有 CRUSH 图删除存在的空桶： ::

	ceph osd crush remove {bucket-name}

把有效的桶从分级结构里的一个位置挪到另一个。 ::

	ceph osd crush move {id} {loc1} [{loc2} ...]

设置 ``{name}`` 所指条目的权重为 ``{weight}`` 。 ::

	ceph osd crush reweight {name} {weight}

创建集群快照。 ::

	ceph osd cluster_snap {name}

把 OSD 标记为丢失，有可能导致永久性数据丢失，慎用！ ::

	ceph osd lost {id} [--yes-i-really-mean-it]

创建新 OSD 。如果未指定 ID ，有可能的话将自动分配个新 ID 。 ::

	ceph osd create [{uuid}]

删除指定 OSD 。 ::

	ceph osd rm [{id}...]

查询 OSD 运行图里的 max_osd 参数。 ::

	ceph osd getmaxosd

导入指定 CRUSH 图。 ::

	ceph osd setcrushmap -i file

设置 OSD 运行图的 ``max_osd`` 参数，扩展存储集群时有必要。 ::

	ceph osd setmaxosd

把 ID 为 ``{osd-num}`` 的 OSD 标记为 down 。 ::

	ceph osd down {osd-num}

把 OSD ``{osd-num}`` 标记为数据分布之外（即不给分配数据）。 ::

	ceph osd out {osd-num}

把 OSD ``{osd-num}`` 标记为数据分布之内（即分配了数据）。 ::

	ceph osd in {osd-num}

列出 Ceph 集群载入的类。 ::

	ceph class list

设置或清空 OSD 运行图里的暂停标记。若设置了，不会有 IO 请求发送到任何 OSD ；用 \
``unpause`` 清空此标记会导致重发未决的请求。 ::

	ceph osd pause
	ceph osd unpause

把 ``{osd-num}`` 的权重设置为 ``{weight}`` ，权重相同的两个 OSD 大致会收到相同的 \
I/O 请求、并存储相同数量的数据。 ``ceph osd reweight`` 命令可给 OSD 设置一个增益\
权重，有效值在 0 和 1 之间，它使得 CRUSH 重新归置一定数量的、本应该放到此处的数据。\
它不会影响 crush 图里所分配的权重，在 CRUSH 分布算法没能理想地执行时，它可作为一种\
纠正手段。比如，假设你的某个 OSD 使用率达到了 90% ，但其它的大致都在 50% ，这时你就\
可以试着下调此权重来补偿它。 ::

	ceph osd reweight {osd-num} {weight}

重设所有滥用 OSD 的权重，它默认会下调达到平均利用率 120% 的那些OSD ，除非你指定了阀\
值。 ::

	ceph osd reweight-by-utilization [threshold]

增加、删除黑名单里的地址。增加地址的时候可以指定有效期，否则有效期为 1 小时。黑名单\
里的地址不允许连接任何 OSD ，此技术常用于防止滞后的元数据服务器“错爱” OSD 上的数据。

这些命令大多只在故障测试时有用，因为黑名单是自动维护的，无需手动干涉。 ::

	ceph osd blacklist add ADDRESS[:source_port] [TIME]
	ceph osd blacklist rm ADDRESS[:source_port]

创建/删除存储池快照。 ::

	ceph osd pool mksnap {pool-name} {snap-name}
	ceph osd pool rmsnap {pool-name} {snap-name}

创建/删除/重命名存储池。 ::

	ceph osd pool create {pool-name} pg_num [pgp_num]
	ceph osd pool delete {pool-name} [{pool-name} --yes-i-really-really-mean-it]
	ceph osd pool rename {old-name} {new-name}

更改存储池设置。 ::

	ceph osd pool set {pool-name} {field} {value}

可用的 field 值有：

	* ``size``: 设置存储池内数据的副本数；
	* ``crash_replay_interval``: 允许客户端重放确认而未提交的请求前等待的时间，秒；
	* ``pg_num``: 归置组数量；
	* ``pgp_num``: 计算归置组存放的有效数量；
	* ``crush_ruleset``: 用于归置映射的规则号。

获取存储池配置值。 ::

	ceph osd pool get {pool-name} {field}

可用的 field 值有：

	* ``pg_num``: 归置组数量；
	* ``pgp_num``: 计算归置组存放的有效数量；
	* ``lpg_num``: 本地归置组数量；
	* ``lpgp_num``: 用于存放本地归置组的数量。


向 OSD ``{osd-num}`` 下达一个洗刷命令，用通配符 ``*`` 把命令下达到所有 OSD 。 ::

	ceph osd scrub {osd-num}

向 osdN 下达修复命令，用 ``*`` 下达到所有 OSD 。 ::

	ceph osd repair N

在 osdN 上进行个简单的吞吐量测试，每次写入 ``BYTES_PER_WRITE`` 、一共写入 \
``TOTAL_BYTES`` 。默认以 4MB 增量写入 1GB 。
此压力测试是非破坏性的，不会覆盖已有 OSD 数据，但可能会暂时影响同时访问此 \
OSD 的客户端性能。 ::

	ceph tell osd.N bench [NUMER_OF_OBJECTS] [BYTES_PER_WRITE]


MDS 子系统
==========

更改在运行 mds 的参数： ::

	ceph tell mds.{mds-id} injectargs --{switch} {value} [--{switch} {value}]

例如： ::

	ceph tell mds.0 injectargs --debug_ms 1 --debug_mds 10

打开了调试消息。 ::

	ceph mds stat

显示所有元数据服务器状态。 ::

	ceph mds fail 0

把活跃 MDS 标记为失败，如果有候补此命令会触发故障转移。

.. todo:: ``ceph mds`` 子命令缺少文档：set, dump, getmap, stop, setmap


监视器子系统
============

查看监视器状态： ::

	ceph mon stat

	2011-12-14 10:40:59.044395 mon {- [mon,stat]
	2011-12-14 10:40:59.057111 mon.1 -} 'e3: 5 mons at {a=10.1.2.3:6789/0,b=10.1.2.4:6789/0,c=10.1.2.5:6789/0,d=10.1.2.6:6789/0,e=10.1.2.7:6789/0}, election epoch 16, quorum 0,1,2,3' (0)

末尾的 ``quorum`` 列表列出了当前法定人数里的监视器节点。

也可以更直接地获取： ::

	$ ./ceph quorum_status

	2011-12-14 10:44:20.417705 mon {- [quorum_status]
	2011-12-14 10:44:20.431890 mon.0 -} 

.. code-block:: javascript

	'{ "election_epoch": 10,
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
	              "addr": "127.0.0.1:6791\/0"}]}}' (0)

如果法定人数未形成，上述命令会一直等待。

你刚刚连接的监视器的状态（用 ``-m HOST:PORT`` 另外指定）： ::

	ceph mon_status


	2011-12-14 10:45:30.644414 mon {- [mon_status]
	2011-12-14 10:45:30.644632 mon.0 -} 

.. code-block:: javascript

	'{ "name": "a",
	  "rank": 0,
	  "state": "leader",
	  "election_epoch": 10,
	  "quorum": [
	        0,
	        1,
	        2],
	  "outside_quorum": [],
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
	              "addr": "127.0.0.1:6791\/0"}]}}' (0)

监视器状态转储： ::

	ceph mon dump

	2011-12-14 10:43:08.015333 mon {- [mon,dump]
	2011-12-14 10:43:08.015567 mon.0 -} 'dumped monmap epoch 1' (0)
	epoch 1
	fsid 444b489c-4f16-4b75-83f0-cb8097468898
	last_changed 2011-12-12 13:28:27.505520
	created 2011-12-12 13:28:27.505520
	0: 127.0.0.1:6789/0 mon.a
	1: 127.0.0.1:6790/0 mon.b
	2: 127.0.0.1:6791/0 mon.c

