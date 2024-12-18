.. index:: control, commands

==========
 控制命令
==========
.. Control Commands


监视器命令
==========
.. Monitor Commands

监视器命令用 ``ceph`` 工具发出：

.. prompt:: bash $

   ceph [-m monhost] {command}

大多数情况下，命令格式如下：

.. prompt:: bash $

   ceph {subsystem} {command}


系统命令
========
.. System Commands

下列命令显示当前的集群状态：

.. prompt:: bash $

   ceph -s
   ceph status

下列命令显示集群状态的运行概况、及主要事件：

.. prompt:: bash $

   ceph -w

查看监视器法定人数状态，包括哪些监视器参与着、哪个是首领，执行下列命令：

.. prompt:: bash $

   ceph mon stat
   ceph quorum_status

查询单个监视器的状态，包括是否在法定人数里，执行下列命令：

.. prompt:: bash $

   ceph tell mon.[id] mon_status

其中， ``[id]`` 的值可以在 ``ceph -s`` 的输出里找到。


认证子系统
==========
.. Authentication Subsystem

要添加一个 OSD 的密钥环，执行下列命令：

.. prompt:: bash $

   ceph auth add {osd} {--in-file|-i} {path-to-osd-keyring}

要列出集群的密钥及其能力，执行下列命令：

.. prompt:: bash $

   ceph auth ls


归置组子系统
============
.. Placement Group Subsystem

要显示所有归置组的统计信息，执行下列命令：

.. prompt:: bash $

   ceph pg dump [--format {format}]

可用输出格式有 ``plain`` （默认）、 ``json`` 、 ``json-pretty`` 、
``xml`` 和 ``xml-pretty`` 。实现监视器和其它工具时，
最好用 ``json`` 格式。 JSON 格式分析起来\
比给人看的文本 ``plain`` 格式更具确定性，
Ceph 版本更迭时它的布局变化少得多。
从 JSON 输出中提取数据可以用 ``jq`` 工具。

要显示卡在某状态的所有归置组，
执行下列命令：

.. prompt:: bash $

   ceph pg dump_stuck inactive|unclean|stale|undersized|degraded [--format {format}] [-t|--threshold {seconds}]

``--format`` 可以是 ``plain`` （默认）、 ``json`` 、
``json-pretty`` 、 ``xml`` 或 ``xml-pretty`` 。

``--threshold`` 定义了多少秒算“卡住了（ ``stuck`` ）”，默认是 300 秒。

PG 可能卡在下列任何一个状态下：

**Inactive**

    归置组不能处理读或写操作，因为它们在等待\
    持有最新数据的 OSD 回到 ``up`` 状态。


**Unclean**

    归置组包含副本数未达期望数量的对象，
    这些 PG 还没完成恢复进程。


**Stale**

    归置组处于未知状态，相关 OSD 所在的主机有一段时间
    （由 ``mon_osd_report_timeout`` 配置）
    没向监视器集群报告了，


要删除“丢失（ ``lost`` ）”对象，或者把一个对象回退到它先前的状态，
可以是回退到前一个版本、或者删除它，
因为是刚创建的还没有之前的版本，执行下列命令：

.. prompt:: bash $

   ceph pg {pgid} mark_unfound_lost revert|delete


.. _osd-subsystem:

OSD 子系统
==========
.. OSD Subsystem

查询 OSD 子系统状态，执行下列命令：

.. prompt:: bash $

   ceph osd stat

把最新的 OSD 运行图副本写入一个文件（参见 :ref:`osdmaptool <osdmaptool>` ），
执行下列命令：

.. prompt:: bash $

   ceph osd getmap -o file

要把最新 OSD 运行图里的 CRUSH 图写入一个文件，
执行下列命令：

.. prompt:: bash $

   ceph osd getcrushmap -o file

这个命令的功能相当于下面两个\
命令的：

.. prompt:: bash $

   ceph osd getmap -o /tmp/osdmap
   osdmaptool /tmp/osdmap --export-crush file

要转储 OSD 运行图，执行下列命令：

.. prompt:: bash $

   ceph osd dump [--format {format}]

``--format`` 接受的参数有 ``plain`` （默认的）、
``json`` 、 ``json-pretty`` 、 ``xml`` 和 ``xml-pretty`` 。上文说过，
使用工具、脚本和其他自动化工具解析输出的是，建议用 JSON 格式。

要把 OSD 运行图转储为树，每个 OSD 一行、
显示其权重和 OSD 状态，
执行下列命令：

.. prompt:: bash $

   ceph osd tree [--format {format}]

要找出某个特定的 RADOS 对象存储在系统的哪里，
执行下列命令：

.. prompt:: bash $

   ceph osd map <pool-name> <object-name>

要增加或挪动一个新 OSD （用它的 ID 、名字、或权重指定）到指定的 CRUSH 位置，
执行下列命令：

.. prompt:: bash $

   ceph osd crush set {id} {weight} [{loc1} [{loc2} ...]]

要从 CRUSH 图删除存在的 OSD ，执行下列命令：

.. prompt:: bash $

   ceph osd crush remove {name}

要从 CRUSH 图删除存在的桶，执行下列命令：

.. prompt:: bash $

   ceph osd crush remove {bucket-name}

把一个存在的桶从 CRUSH 分级结构里的一个位置挪到另一个，
执行下列命令：

.. prompt:: bash $

   ceph osd crush move {id} {loc1} [{loc2} ...]

设置指定 OSD （用 ``{name}`` 指定）的 CRUSH 权重为 ``{weight}`` ，
执行下列命令：

.. prompt:: bash $

   ceph osd crush reweight {name} {weight}

把一个 OSD 标记为丢失（ ``lost`` ），执行下列命令：

.. prompt:: bash $

   ceph osd lost {id} [--yes-i-really-mean-it]

.. warning::
   此动作有可能导致数据永久丢失，慎用！

创建新 OSD ，执行下列命令：

.. prompt:: bash $

   ceph osd create [{uuid}]

如果执行此命令时没有指定 UUID ，
那么会在 OSD 启动时自动设置 UUID 。

删除一个或多个 OSD ，执行下列命令：

.. prompt:: bash $

   ceph osd rm [{id}...]

查询 OSD 运行图当前的 ``max_osd`` 参数，
执行下列命令：

.. prompt:: bash $

   ceph osd getmaxosd

导入指定的 CRUSH 图，执行下列命令：

.. prompt:: bash $

   ceph osd setcrushmap -i file

设置 OSD 运行图的 ``max_osd`` 参数，执行下列命令：

.. prompt:: bash $

   ceph osd setmaxosd

这个参数的默认值是 10000 ，
大多数运维人员永远不需要调整它。

把指定的 OSD 标记为 ``down`` ，执行下列命令：

.. prompt:: bash $

   ceph osd down {osd-num}

把指定的 OSD 标记为 ``out`` （这样就不会给它分配数据了），
执行下列命令：

.. prompt:: bash $

   ceph osd out {osd-num}

把指定的 OSD 标记为 ``in`` （这样数据就会分配给它），
执行下列命令：

.. prompt:: bash $

   ceph osd in {osd-num}

通过使用 OSD 运行图中的“暂停标志（ pause flag ）”，可以暂停或恢复 I/O 请求。
如果设置了这些标志，则不会向任何 OSD 发送 I/O 请求；
清除标志后，将重新发送待处理的 I/O 请求。
要设置或清除暂停标记，执行以下命令：

.. prompt:: bash $

   ceph osd pause
   ceph osd unpause

如果常规的 CRUSH 分布看起来不理想，
你可以给指定 OSD 设置覆盖权重或 ``reweight`` 权重数值。
OSD 的权重有助于确定它能承受的 I/O 请求规模和数据存储量：
权重相同的两个 OSD 大致会收到差不多数量的 I/O 请求、
也会存储差不多数量的数据。 ``ceph osd reweight`` 命令\
可给 OSD 设置一个增益权重，这个权重值的范围在 0 和 1 之间，
它会强迫 CRUSH 重新归置一定数量的（ 1 - ``weight`` ）、
本应该放到此处的数据。此命令不会影响
CRUSH 图里这个 OSD 之上的桶的权重，使用此命令只是一种纠正措施：
比如，假设你的某个 OSD 使用率达到了 90% ，
但其它的大致都在 50% ，这时你就可以下调这个外在权重来纠正这种不平衡。
要给指定 OSD 设置覆盖权重，
执行以下命令：

.. prompt:: bash $

   ceph osd reweight {osd-num} {weight}

.. note:: 任何指定的覆盖权重值都会与平衡器有冲突。
   这意味着，如果正在使用平衡器，所有覆盖权重值都应为 ``1.0000`` ，
   以避免集群出现不太好的行为。

如果某些 OSD 的利用率过高，为了维持平衡，
可以对集群 OSD 的权重进行调整。注意，
覆盖权重或 ``reweight`` 权重只是相互之间的相对值（默认是 1.00000 ），
不是绝对值，一定要把它们与 CRUSH 权重区别开来，
CRUSH 权重反映的是一个桶以 TiB 计算的绝对容量。
要按利用率调整 OSD 的权重，执行以下命令：

.. prompt:: bash $

   ceph osd reweight-by-utilization [threshold [max_change [max_osds]]] [--no-increasing]

默认情况下，这个命令调整 OSD 的覆盖权重时会选择\
比平均利用率大或小 20% 的 OSD 们，但是，
可以用 ``threshold`` 参数指定别的百分比。

要限制 OSD 权重调整的幅度，可以用 ``max_change`` 参数，
默认为 0.05 。要限制一次可以调整的 OSD 数量，
用 ``max_osds`` 参数，默认是 4 。
增大这些参数可以加速 OSD 利用率的均衡，
也会潜在地增加对客户端操作的影响，因为会导致的数据挪动更多。

在正式运行 ``osd reweight-by-utilization`` 命令前，可以先测试一下。
要想确定调用 ``osd reweight-by-utilization`` 命令时哪些以及有多少
PG 和 OSD 会受影响，执行下列命令：

.. prompt:: bash $

   ceph osd test-reweight-by-utilization [threshold [max_change max_osds]] [--no-increasing]

给 ``reweight-by-utilization`` 和 ``test-reweight-by-utilization`` 命令\
加上 ``--no-increasing`` 选项可以防止\
当前 < 1.00000 的覆盖权重被增大。
这个选项在特定环境下有用：比如，
在均衡一个需要急需补救的 ``full`` 或 ``nearful`` 的 OSD 时、
或者一些 OSD 正在维修、或者正在慢慢进入工作状态时。

用 Nautilus 或更新版本部署的（或者 Luminous 和 Mimic 的后期修订版）
它们没有 Luminous 之前的客户端，可以转而启用
``ceph-mgr`` 的 `balancer` 模块。

黑名单（ blocklist ）是可以更改的，可以增加、删除黑名单里的一个 IP 地址，
或者一个 CIDR 网段。如果一个地址进了黑名单，它将无法连接到任何 OSD 。
如果一个 OSD 包含在一个已被列入黑名单的 IP 地址或 CIDR 范围内，
那么该 OSD 作为客户端时将无法对它的互联 OSD 执行操作：
此类被封禁的操作包括分层（ tiering ）和 copy-from 功能。
要在黑名单中添加或删除 IP 地址、 CIDR 网段，
执行以下命令之一：

.. prompt:: bash $

   ceph osd blocklist ["range"] add ADDRESS[:source_port][/netmask_bits] [TIME]
   ceph osd blocklist ["range"] rm ADDRESS[:source_port][/netmask_bits]

用上面的 ``add`` 命令加进黑名单的时候，
可以用 ``TIME`` 关键字指定屏蔽时长（单位为秒），
就是它在黑名单里呆的时间，默认 1 小时。
增加或删除 CIDR 网段时，要加上面命令里的 ``range`` 关键字。

注意，这些命令主要用于故障测试。
正常情况下，黑名单是系统自动维护的，
不需要任何手动干预。

要创建或删除指定存储池的快照，
执行以下命令之一：

.. prompt:: bash $

   ceph osd pool mksnap {pool-name} {snap-name}
   ceph osd pool rmsnap {pool-name} {snap-name}

创建/删除/重命名指定的存储池，
执行以下命令：

.. prompt:: bash $

   ceph osd pool create {pool-name} [pg_num [pgp_num]]
   ceph osd pool delete {pool-name} [{pool-name} --yes-i-really-really-mean-it]
   ceph osd pool rename {old-name} {new-name}

更改存储池设置，执行以下命令：

.. prompt:: bash $

   ceph osd pool set {pool-name} {field} {value}

下面是可用的 field 值：

   * ``size``: 设置存储池内数据的副本数；
   * ``pg_num``: 归置组数量；
   * ``pgp_num``: 计算归置组存放的有效数量；
   * ``crush_rule``: 用于归置映射的规则号。

查看存储池配置值，执行以下命令：

.. prompt:: bash $

   ceph osd pool get {pool-name} {field}

可用的 field 值有：

   * ``pg_num``: 归置组数量；
   * ``pgp_num``: 计算归置存放时的有效 PG 数量；

向指定 OSD 或所有 OSD （用 ``*`` 指定）下达一个洗刷命令，
执行以下命令：

.. prompt:: bash $

   ceph osd scrub {osd-num}

向指定 OSD 或所有 OSD （用 ``*`` 指定）下达修复命令，
执行下列命令：

.. prompt:: bash $

   ceph osd repair N

你可以在指定 OSD 上做个简单的吞吐量压力测试。
此测试每次写入的尺寸是 ``BYTES_PER_WRITE`` （默认 4 MB ），
增量地写入，一共写入 ``TOTAL_DATA_BYTES`` （默认 1 GB ）。
此测试没有破坏性，不会覆盖已有的 OSD 数据，
但可能会暂时影响同时访问此 OSD 的客户端性能。
要启动这个压力测试，执行下列命令：

.. prompt:: bash $

   ceph tell osd.N bench [TOTAL_DATA_BYTES] [BYTES_PER_WRITE]

要清除两次压力测试之间一个指定 OSD 上的缓存，
执行下列命令：

.. prompt:: bash $

   ceph tell osd.N cache drop

要查看某一 OSD 缓存的统计信息，执行下列命令：

.. prompt:: bash $

   ceph tell osd.N cache status


MDS 子系统
==========
.. MDS Subsystem

一个元数据服务器正在运行时，要想更改它的配置参数，可执行下列命令：

.. prompt:: bash $

   ceph tell mds.{mds-id} config set {setting} {value}

例如：想要打开调试消息，执行下列命令：

.. prompt:: bash $

   ceph tell mds.0 config set debug_ms 1

要查看所有元数据服务器的状态，执行下列命令：

.. prompt:: bash $

   ceph mds stat

把活跃的 MDS 标记为失败（是为了触发故障转移到备机，
如果有备机的话），执行下列命令：

.. prompt:: bash $

   ceph mds fail 0

.. todo:: ``ceph mds`` 子命令缺少文档：set, dump, getmap, stop, setmap


监视器子系统
============
.. Mon Subsystem

查看监视器统计信息，执行下列命令：

.. prompt:: bash $

   ceph mon stat

此命令的输出和下面的内容相似：

::

   e2: 3 mons at {a=127.0.0.1:40000/0,b=127.0.0.1:40001/0,c=127.0.0.1:40002/0}, election epoch 6, quorum 0,1,2 a,b,c

输出的末尾有个 ``quorum`` 列表，它列出了当前法定人数里的监视器节点。

也可以更直接地获取此信息，执行下列命令：

.. prompt:: bash $

   ceph quorum_status -f json-pretty

此命令的输出形似如下：

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

只看单个监视器的状态，执行下列命令：

.. prompt:: bash $

   ceph tell mon.[name] mon_status

其中， ``[name]`` 的值可以从
``ceph quorum_status`` 命令的输出里找到，
此命令的输出样本：

::

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

要看看监视器状态的一个转储，执行下列命令：

.. prompt:: bash $

   ceph mon dump

此命令的输出和如下内容相似：

::

   dumped monmap epoch 2
   epoch 2
   fsid ba807e74-b64f-4b72-b43f-597dfe60ddbc
   last_changed 2016-12-26 14:42:09.288066
   created 2016-12-26 14:42:03.573585
   0: 127.0.0.1:40000/0 mon.a
   1: 127.0.0.1:40001/0 mon.b
   2: 127.0.0.1:40002/0 mon.c
