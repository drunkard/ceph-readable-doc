.. Control Commands
.. index:: control, commands

==========
 控制命令
==========


.. Monitor Commands

监视器命令
==========

监视器命令用 ``ceph`` 工具发出： ::

	ceph [-m monhost] {command}

命令格式通常是（但不总是）： ::

	ceph {subsystem} {command}


.. System Commands

系统命令
========

下列命令显示集群状态： ::

	ceph -s
	ceph status

下列命令显示集群状态的运行摘要、及主要事件： ::

	ceph -w

下列命令显示监视器法定人数状态，包括哪些监视器参与着、哪个是首\
领。 ::

	ceph quorum_status

下列命令查询单个监视器状态，包括是否在法定人数里。 ::

	ceph [-m monhost] mon_status


.. Authentication Subsystem

认证子系统
==========

要添加一个 OSD 的密钥环，执行下列命令： ::

	ceph auth add {osd} {--in-file|-i} {path-to-osd-keyring}

要列出集群的密钥及其能力，执行下列命令： ::

	ceph auth ls


.. Placement Group Subsystem

归置组子系统
============

要显示所有归置组的统计信息，执行下列命令： ::

	ceph pg dump [--format {format}]

可用输出格式有 ``plain`` （默认）和 ``json`` 。

要显示卡在某状态的所有归置组，执行下列命令： ::

	ceph pg dump_stuck inactive|unclean|stale|undersized|degraded [--format {format}] [-t|--threshold {seconds}]


``--format`` 可以是 ``plain`` （默认）或 ``json``

``--threshold`` 定义了多久算“卡住了”（默认 300 秒）

**Inactive** 归置组不能处理读或写，因为它们在等待数据及时更新的
OSD 回来。

**Unclean** 归置组包含副本数未达期望值的对象，它们应该在恢复中。

**Stale** 归置组处于未知状态——归置组所托付的 OSD 有一阵没向监\
视器报告了（由 ``mon osd report timeout`` 配置）。

删除“丢失”对象，或者恢复到其先前状态，可以是前一版本、或如果刚\
创建就干脆删除。 ::

	ceph pg {pgid} mark_unfound_lost revert|delete


.. OSD Subsystem

OSD 子系统
==========

查询 OSD 子系统状态： ::

	ceph osd stat

把最新的 OSD 运行图拷贝到一个文件，参见
:ref:`osdmaptool <osdmaptool>` ：::

	ceph osd getmap -o file

从最新 OSD 运行图拷出 CRUSH 图： ::

	ceph osd getcrushmap -o file

前述功能等价于： ::

	ceph osd getmap -o /tmp/osdmap
	osdmaptool /tmp/osdmap --export-crush file

转储 OSD 运行图， ``-f`` 的可用格式有 ``plain`` 和 ``json`` ，\
如未指定 ``--format`` 则转储为纯文本。 ::

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

设置或清空 OSD 运行图里的暂停标记。若设置了，不会有 IO 请求发\
送到任何 OSD ；用 ``unpause`` 清空此标记会导致重发未决的请求。 ::

	ceph osd pause
	ceph osd unpause

把 ``{osd-num}`` 的权重设置为 ``{weight}`` ，权重相同的两个
OSD 大致会收到相同的 I/O 请求、并存储相同数量的数据。
``ceph osd reweight`` 命令可给 OSD 设置一个增益权重，有效值在
0 和 1 之间，它使得 CRUSH 重新归置一定数量的、本应该放到此处的\
数据。它不会影响 crush 图里所分配的权重，在 CRUSH 分布算法没能\
理想地执行时，它可作为一种纠正手段。比如，假设你的某个 OSD 使\
用率达到了 90% ，但其它的大致都在 50% ，这时你就可以试着下调此\
权重来补偿它。 ::

	ceph osd reweight {osd-num} {weight}

重设所有滥用 OSD 的权重，它默认会下调达到平均利用率 120% 的那\
些 OSD ，除非你指定了阀值。 ::

	ceph osd reweight-by-utilization [threshold]

描述 reweight-by-utilization 会干什么。 ::

	ceph osd test-reweight-by-utilization

增加、删除黑名单里的地址。增加地址的时候可以指定有效期，否则有\
效期为 1 小时。黑名单里的地址不允许连接任何 OSD ，此技术常用于\
防止滞后的元数据服务器“错爱” OSD 上的数据。

这些命令大多只在故障测试时有用，因为黑名单是自动维护的，无需手\
动干涉。 ::

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
	* ``pg_num``: 归置组数量；
	* ``pgp_num``: 计算归置组存放的有效数量；
	* ``crush_rule``: 用于归置映射的规则号。

获取存储池配置值。 ::

	ceph osd pool get {pool-name} {field}

可用的 field 值有：

	* ``pg_num``: 归置组数量；
	* ``pgp_num``: 计算归置组存放的有效数量；


向 OSD ``{osd-num}`` 下达一个洗刷命令，用通配符 ``*`` 把命令下\
达到所有 OSD 。 ::

	ceph osd scrub {osd-num}

向 osdN 下达修复命令，用 ``*`` 下达到所有 OSD 。 ::

	ceph osd repair N

在 osdN 上做个简单的吞吐量测试，每次写入 ``BYTES_PER_WRITE`` 、\
一共写入 ``TOTAL_DATA_BYTES`` 。默认以 4MB 增量写入 1GB 。此压\
力测试是非破坏性的，不会覆盖已有 OSD 数据，但可能会暂时影响同\
时访问此 OSD 的客户端性能。 ::

	ceph tell osd.N bench [TOTAL_DATA_BYTES] [BYTES_PER_WRITE]

要清除测试期间某个 OSD 上的缓存，用 cache drop 命令： ::

	ceph tell osd.N cache drop

要查看某一 OSD 缓存的统计信息，用 cache status 命令： ::

	ceph tell osd.N cache status


.. MDS Subsystem

MDS 子系统
==========

更改在运行 mds 的参数： ::

	ceph tell mds.{mds-id} config set {setting} {value}

例如： ::

	ceph tell mds.0 config set debug_ms 1

打开了调试消息。 ::

	ceph mds stat

显示所有元数据服务器状态。 ::

	ceph mds fail 0

把活跃 MDS 标记为失败，如果有候补此命令会触发故障转移。

.. todo:: ``ceph mds`` 子命令缺少文档：set, dump, getmap, stop, setmap


.. Mon Subsystem

监视器子系统
============

查看监视器状态： ::

	ceph mon stat

	e2: 3 mons at {a=127.0.0.1:40000/0,b=127.0.0.1:40001/0,c=127.0.0.1:40002/0}, election epoch 6, quorum 0,1,2 a,b,c

末尾的 ``quorum`` 列表列出了当前法定人数里的监视器节点。

也可以更直接地获取： ::

	ceph quorum_status

.. code-block:: javascript

	{
	    "election_epoch": 6,
	    "quorum": [
		0,
		1,
		2
	    ],
	    "quorum_names": [
		"a",
		"b",
		"c"
	    ],
	    "quorum_leader_name": "a",
	    "monmap": {
		"epoch": 2,
		"fsid": "ba807e74-b64f-4b72-b43f-597dfe60ddbc",
		"modified": "2016-12-26 14:42:09.288066",
		"created": "2016-12-26 14:42:03.573585",
		"features": {
		    "persistent": [
			"kraken"
		    ],
		    "optional": []
		},
		"mons": [
		    {
			"rank": 0,
			"name": "a",
			"addr": "127.0.0.1:40000\/0",
			"public_addr": "127.0.0.1:40000\/0"
		    },
		    {
			"rank": 1,
			"name": "b",
			"addr": "127.0.0.1:40001\/0",
			"public_addr": "127.0.0.1:40001\/0"
		    },
		    {
			"rank": 2,
			"name": "c",
			"addr": "127.0.0.1:40002\/0",
			"public_addr": "127.0.0.1:40002\/0"
		    }
		]
	    }
	}


如果法定人数未形成，上述命令会一直等待。

只看单个监视器的状态： ::

	ceph tell mon.[name] mon_status

其中， ``[name]`` 的值取自 ``ceph quorum_status`` ，其输出样本： ::

	{
	    "name": "b",
	    "rank": 1,
	    "state": "peon",
	    "election_epoch": 6,
	    "quorum": [
		0,
		1,
		2
	    ],
	    "features": {
		"required_con": "9025616074522624",
		"required_mon": [
		    "kraken"
		],
		"quorum_con": "1152921504336314367",
		"quorum_mon": [
		    "kraken"
		]
	    },
	    "outside_quorum": [],
	    "extra_probe_peers": [],
	    "sync_provider": [],
	    "monmap": {
		"epoch": 2,
		"fsid": "ba807e74-b64f-4b72-b43f-597dfe60ddbc",
		"modified": "2016-12-26 14:42:09.288066",
		"created": "2016-12-26 14:42:03.573585",
		"features": {
		    "persistent": [
			"kraken"
		    ],
		    "optional": []
		},
		"mons": [
		    {
			"rank": 0,
			"name": "a",
			"addr": "127.0.0.1:40000\/0",
			"public_addr": "127.0.0.1:40000\/0"
		    },
		    {
			"rank": 1,
			"name": "b",
			"addr": "127.0.0.1:40001\/0",
			"public_addr": "127.0.0.1:40001\/0"
		    },
		    {
			"rank": 2,
			"name": "c",
			"addr": "127.0.0.1:40002\/0",
			"public_addr": "127.0.0.1:40002\/0"
		    }
		]
	    }
	}

监视器状态的一个转储： ::

	ceph mon dump

	dumped monmap epoch 2
	epoch 2
	fsid ba807e74-b64f-4b72-b43f-597dfe60ddbc
	last_changed 2016-12-26 14:42:09.288066
	created 2016-12-26 14:42:03.573585
	0: 127.0.0.1:40000/0 mon.a
	1: 127.0.0.1:40001/0 mon.b
	2: 127.0.0.1:40002/0 mon.c

