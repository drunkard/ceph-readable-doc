============
 归置组排障
============

归置组总不整洁
==============
.. Placement Groups Never Get Clean

如果你新建了一套集群，但集群一直处于 ``active`` 、 ``active+remapped``
或者 ``active+degraded`` 状态，一直不能进入 ``active+clean`` 状态，
那就可能是配置有问题。

你也许得仔细阅读 `存储池、归置组和 CRUSH 配置参考`_ 里的配置选项，做一些调整。

一般来说，你的集群应该有一个以上 OSD 、并且存储池的对象副本数大于 1 。

.. _one-node-cluster:

单节点集群
----------
.. One Node Cluster

Ceph 不再提供单节点的运营文档，
因为你不可能在单个节点上设计、部署一套分布式计算系统。
另外，在运行 Ceph 守护进程的节点上同时用内核客户端挂载会导致死机，
这是 Linux 内核自身的问题（除非你在 VM 里跑客户端）。
抛开这里提到的诸多限制，用单节点配置体验 Ceph 是可以的。

如果你正尝试在单个节点上创建一个集群，在创建监视器和 OSD 前，
必须把配置文件里的 ``osd crush chooseleaf type`` 的默认值从
``1`` （表示 ``host`` 或 ``node`` ）改成 ``0`` （表示 ``osd`` ）。
就是告诉 Ceph ， OSD 可以和同一主机上的其它 OSD 互联。
如果你想要建一套单节点集群，
而 ``osd crush chooseleaf type`` 却大于 ``0`` ，
根据你的配置， Ceph 就会让一个 OSD 上的 PG 与\
另一节点、机箱、机架、行、甚至是数据中心里面的 PG 互联。

.. tip:: **不要** 在运行 Ceph 存储集群的同一主机上\
   用内核客户端直接挂载，因为会有内核冲突。
   但是，你可以在同一节点上的虚拟机里面用内核客户端挂载。

如果你在用单个磁盘创建 OSD ，
你必须先创建好数据目录。


OSD 数量小于副本数
------------------
.. Fewer OSDs than Replicas

如果你已经把两个 OSD 带到了 ``up`` 且 ``in`` 状态，
却还没见到处于 ``active + clean`` 状态的归置组，
可能是因为你的 ``osd pool default size`` 大于 ``2`` 。

有几种办法解决这个问题。如果你想让\
一个双副本集群进入 ``active + degraded`` 状态，
你可以把 ``osd pool default min size`` 设置为 ``2`` ，
这样就能在 ``active + degraded`` 状态下写入对象了。
你还可以把 ``osd pool default size`` 设置成 ``2`` ，
这样你就有两份副本了（原始的和一个副本），
这种情况下，集群还能达到 ``active + clean`` 状态。

.. note:: 你可以在运行时做出更改。如果在配置文件里更改，
   就需要重启集群。


存储池副本数为 1
----------------
.. Pool Size = 1

如果把 ``osd pool default size`` 设置成 ``1`` ，你就只有一份对象副本。
OSD 靠其它 OSD 们告诉它，它应该持有哪些对象。
如果第一个 OSD 有某个对象的一个副本、并且没有第二个副本，
那么就没有第二个 OSD 能告诉第一个 OSD 它应该有那个副本。
对于映射到第一个 OSD 的每个归置组（见 ``ceph pg dump`` ），
你可以强制第一个 OSD 通知它需要的归置组，用命令： ::

   	ceph osd force-create-pg <pgid>


CRUSH 图错误
------------
.. CRUSH Map Errors

归置组仍然不整洁的另一个原因可能是 CRUSH 图中的错误。


卡住的归置组
============
.. Stuck Placement Groups

有失败时归置组会进入“degraded”（降级）或“peering”（连接建立中）状态，
这事时有发生，通常这些状态意味着正常的失败恢复正在进行。
然而，如果一个归置组长时间处于某个这些状态就意味着有更大的问题，
因此监视器在归置组卡 （stuck） 在非最优状态时会警告。
我们具体检查：

* ``inactive`` （不活跃）——归置组长时间无活跃
  （即它不能提供读写服务了）；
  
* ``unclean`` （不干净）——归置组长时间不干净
  （例如它未能从前面的失败完全恢复）；

* ``stale`` （不新鲜）——归置组状态没有被 ``ceph-osd`` 更新，
  表明存储这个归置组的所有节点可能都挂了。

你可以摆出卡住的归置组，用下列命令之一： ::

	ceph pg dump_stuck stale
	ceph pg dump_stuck inactive
	ceph pg dump_stuck unclean

卡在 ``stale`` 状态的归置组通过修复 ``ceph-osd`` 进程通常可以修复；
卡在 ``inactive`` 状态的归置组通常是互联问题
（参见 :ref:`failures-osd-peering` ）；
卡在 ``unclean`` 状态的归置组通常是由于某些原因\
阻止了恢复的完成，像未找到的对象
（参见 :ref:`failures-osd-unfound` ）。


.. _failures-osd-peering:

归置组挂了——互联失败
========================
.. Placement Group Down - Peering Failure

在某些情况下， ``ceph-osd`` 的 `互联` 进程会遇到问题，
使 PG 不能活跃、可用。
例如 ``ceph health`` 也许显示： ::

	ceph health detail
	HEALTH_ERR 7 pgs degraded; 12 pgs down; 12 pgs peering; 1 pgs recovering; 6 pgs stuck unclean; 114/3300 degraded (3.455%); 1/3 in osds are down
	...
	pg 0.5 is down+peering
	pg 1.4 is down+peering
	...
	osd.1 is down since epoch 69, last address 192.168.106.220:6801/8651

可以查询到 PG 为何被标记为 ``down`` ： ::

	ceph pg 0.5 query

.. code-block:: javascript

 { "state": "down+peering",
   ...
   "recovery_state": [
        { "name": "Started\/Primary\/Peering\/GetInfo",
          "enter_time": "2012-03-06 14:40:16.169679",
          "requested_info_from": []},
        { "name": "Started\/Primary\/Peering",
          "enter_time": "2012-03-06 14:40:16.169659",
          "probing_osds": [
                0,
                1],
          "blocked": "peering is blocked due to down osds",
          "down_osds_we_would_probe": [
                1],
          "peering_blocked_by": [
                { "osd": 1,
                  "current_lost_at": 0,
                  "comment": "starting or marking this osd lost may let us proceed"}]},
        { "name": "Started",
          "enter_time": "2012-03-06 14:40:16.169513"}
    ]
 }

``recovery_state`` 段告诉我们因 ``ceph-osd`` 守护进程挂了\
而导致互联被阻塞，本例是 ``osd.1`` 挂了，
启动这个 ``ceph-osd`` 应该就可以恢复。

另外，如果 ``osd.1`` 是灾难性的失败（如硬盘损坏），
我们可以告诉集群它丢失（ ``lost`` ）了，
让集群尽力完成副本拷贝。

.. important:: 假如集群不能保证\
   其它数据副本是一致且最新就危险了！

让 Ceph 无论如何都继续： ::

	ceph osd lost 1

恢复将继续。



.. _failures-osd-unfound:

未找到的对象
============
.. Unfound Objects

某几种失败相组合可能导致 Ceph 抱怨有找不到（ ``unfound`` ）的\
对象： ::

	ceph health detail
	HEALTH_WARN 1 pgs degraded; 78/3778 unfound (2.065%)
	pg 2.4 is active+degraded, 78 unfound

这意味着存储集群知道一些对象
（或者现有对象的较新副本）存在，\
却没有找到它们的副本。下例展示了这种情况是如何发生的，
一个 PG 的数据存储在 ceph-osd 1 和 2 上：

* 1 挂了；
* 2 独自处理一些写动作；
* 1 起来了；
* 1 和 2 重新互联， 1 上面丢失的对象加入队列准备恢复；
* 新对象还未拷贝完， 2 挂了。

这时， 1 知道这些对象存在，但是活着的 ``ceph-osd`` 都没有副本，\
这种情况下，读写这些对象的 IO 就会被阻塞，
集群只能指望节点早点恢复。
这时我们假设用户希望先得到一个 IO 错误。

首先，你应该确认哪些对象找不到了： ::

	ceph pg 2.4 list_unfound [starting offset, in json]

.. code-block:: javascript

  {
    "num_missing": 1,
    "num_unfound": 1,
    "objects": [
        {
            "oid": {
                "oid": "object",
                "key": "",
                "snapid": -2,
                "hash": 2249616407,
                "max": 0,
                "pool": 2,
                "namespace": ""
            },
            "need": "43'251",
            "have": "0'0",
            "flags": "none",
            "clean_regions": "clean_offsets: [], clean_omap: 0, new_object: 1",
            "locations": [
                "0(3)",
                "4(2)"
            ]
        }
    ],
    "state": "NotRecovering",
    "available_might_have_unfound": true,
    "might_have_unfound": [
        {
            "osd": "2(4)",
            "status": "osd is down"
        }
    ],
    "more": false
  }

如果在一次查询里列出的对象太多，
``more`` 这个字段将为 ``true`` ，因此你可以查询更多。
（命令行工具可能隐藏了，但这里没有）

其次，你可以找出哪些 OSD 上探测到了、
或可能包含数据。

在这份罗列结果的末尾（ ``more`` 值为 false 之前），
``available_might_have_unfound`` 是 true 的时候会有 ``might_have_unfound`` 。
这个和 ``ceph pg #.# query`` 的输出是等价的，
只是这个结果让我们省去了直接查询（ ``query`` ）的必要。
``might_have_unfound`` 信息的提供方式和下文 ``query`` 的相同，
仅有的差异是有 ``already probed`` 状态的 OSD 会被忽略。

``query`` 的用法： ::

	ceph pg 2.4 query

.. code-block:: javascript

   "recovery_state": [
        { "name": "Started\/Primary\/Active",
          "enter_time": "2012-03-06 15:15:46.713212",
          "might_have_unfound": [
                { "osd": 1,
                  "status": "osd is down"}]},

本案例中，集群知道 ``osd.1`` 可能有数据，但它挂了（ ``down`` ）。\
所有可能的状态有：

* 已经探测到了
* 在查询
* OSD 挂了
* 尚未查询

有时候集群要花一些时间来查询可能的位置。

还有一种可能性，对象存在于其它位置却未被列出，
例如，集群里的一个 ``ceph-osd`` 停止且被剔出，
然后完全恢复了；后来的失败、恢复后仍有未找到的对象，
它也不会觉得早已死亡的 ``ceph-osd`` 上仍可能包含这些对象。
（这种情况几乎不太可能发生）。

如果所有位置都查询过了仍有对象丢失，
那就得放弃丢失的对象了。
这仍可能是罕见的失败组合导致的，
集群在写入完成前，未能得知写入是否已执行。
以下命令把未找到的（ unfound ）对象\
标记为丢失（ lost ）。 ::

	ceph pg 2.5 mark_unfound_lost revert|delete

上述最后一个参数告诉集群应如何处理丢失的对象。

delete 选项将导致完全删除它们。

revert 选项（纠删码存储池不可用）会回滚到前一个版本或者
（如果它是新对象的话）删除它。要慎用，
它可能迷惑那些期望对象存在的应用程序。


无根归置组
==========
.. Homeless Placement Groups

拥有归置组拷贝的 OSD 都可以失败，
在这种情况下，那一部分的对象存储不可用，
监视器就不会收到那些归置组的状态更新了。
为检测这种情况，监视器把任何主 OSD 失败的归置组\
标记为 ``stale`` （不新鲜），例如： ::

	ceph health
	HEALTH_WARN 24 pgs stale; 3/300 in osds are down

你能找出哪些归置组 ``stale`` 、和最后存储这些归置组的 OSD ，
命令如下： ::

	ceph health detail
	HEALTH_WARN 24 pgs stale; 3/300 in osds are down
	...
	pg 2.5 is stuck stale+active+remapped, last acting [2,0]
	...
	osd.10 is down since epoch 23, last address 192.168.106.220:6800/11080
	osd.11 is down since epoch 13, last address 192.168.106.220:6803/11539
	osd.12 is down since epoch 24, last address 192.168.106.220:6806/11861

如果想使归置组 2.5 重新在线，例如，
上面的输出告诉我们它最后由 ``osd.0`` 和 ``osd.2`` 处理，
重启这些 ``ceph-osd`` 将恢复那个归置组（还有其它的很多 PG ）。


只有几个 OSD 接收数据
=====================
.. Only a Few OSDs Receive Data

如果你的集群有很多节点，但只有其中几个接收数据，
`检查`_\ 下存储池里的归置组数量。因为归置组是映射到多个 OSD 的，
这样少量的归置组将不能分布于整个集群。
试着创建个新存储池，其归置组数量是 OSD 数量的若干倍。详情见\ `归置组`_\ ，
存储池的默认归置组数量没多大用，你可以参考\ `这里`_\ 更改它。


不能写入数据
============
.. Can't Write Data

如果你的集群已启动，但一些 OSD 没起来，导致不能写入数据，
确认下运行的 OSD 数量满足归置组要求的最低 OSD 数。
如果不能满足， Ceph 就不会允许你写入数据，
因为 Ceph 不能保证复制能如愿进行。
详情参见\ `存储池、归置组和 CRUSH 配置参考`_\ 里的
``osd pool default min size`` 。


归置组不一致
============
.. PGs Inconsistent

如果你看到状态变成了 ``active + clean + inconsistent`` ，
可能是洗刷时遇到了错误。与往常一样，我们可以这样找出不一致的归置组： ::

    $ ceph health detail
    HEALTH_ERR 1 pgs inconsistent; 2 scrub errors
    pg 0.6 is active+clean+inconsistent, acting [0,1,2]
    2 scrub errors

或者这样，如果你喜欢程序化的输出： ::

    $ rados list-inconsistent-pg rbd
    ["0.6"]

一致的状态只有一种，然而在最坏的情况下，
我们可能会遇到多个对象产生了各种各样的不一致。假设\
在 PG ``0.6`` 里的一个名为 ``foo`` 的对象被截断了，我们可以这样查看： ::

    $ rados list-inconsistent-obj 0.6 --format=json-pretty

.. code-block:: javascript

    {
        "epoch": 14,
        "inconsistents": [
            {
                "object": {
                    "name": "foo",
                    "nspace": "",
                    "locator": "",
                    "snap": "head",
                    "version": 1
                },
                "errors": [
                    "data_digest_mismatch",
                    "size_mismatch"
                ],
                "union_shard_errors": [
                    "data_digest_mismatch_info",
                    "size_mismatch_info"
                ],
                "selected_object_info": "0:602f83fe:::foo:head(16'1 client.4110.0:1 dirty|data_digest|omap_digest s 968 uv 1 dd e978e67f od ffffffff alloc_hint [0 0 0])",
                "shards": [
                    {
                        "osd": 0,
                        "errors": [],
                        "size": 968,
                        "omap_digest": "0xffffffff",
                        "data_digest": "0xe978e67f"
                    },
                    {
                        "osd": 1,
                        "errors": [],
                        "size": 968,
                        "omap_digest": "0xffffffff",
                        "data_digest": "0xe978e67f"
                    },
                    {
                        "osd": 2,
                        "errors": [
                            "data_digest_mismatch_info",
                            "size_mismatch_info"
                        ],
                        "size": 0,
                        "omap_digest": "0xffffffff",
                        "data_digest": "0xffffffff"
                    }
                
            }
        ]
    }

此时，我们可以从输出里看到：

* 唯一不一致的对象名为 ``foo`` ，并且这就是\
  它不一致的 head 。
* 不一致分为两类：

  * ``errors``: 这些错误表明不一致性出现在分片之间，但是没说明\
    哪个（或哪些）分片有问题。如果 `shards` 阵列中有 ``errors``
    字段，且不为空，它会指出问题所在。

    * ``data_digest_mismatch``: OSD.2 内读取到的副本的数字摘要\
      与 OSD.0 和 OSD.1 的不一样。
    * ``size_mismatch``: OSD.2 内读取到的副本的尺寸是 0 ，而
      OSD.0 和 OSD.1 说是 968 。
  * ``union_shard_errors``: ``shards`` 阵列中、所有与分片相关\
    的错误 ``errors`` 的并集。 ``errors`` 是个错误原因集合，汇\
    集了相关分片的这类问题，如 ``read_error`` 。以 ``oi`` 结尾\
    的 ``errors`` 表明它是与 ``selected_object_info`` 的对照结\
    果。从 ``shards`` 阵列里可以查到哪个分片有什么样的错误。

    * ``data_digest_mismatch_info``: 存储在 object-info （对象信息）里的\
      数字签名不是 ``0xffffffff`` （这个是根据 OSD.2 上的分片计算出来的）。
    * ``size_mismatch_info``: object-info 内存储的尺寸与 OSD.2
      上的对象尺寸 0 不同。

你可以用下列命令修复不一致的归置组： ::

	ceph pg repair {placement-group-ID}

此命令会用\ `权威的`\ 副本覆盖\ `有问题的`\ 。根据既定规则，
多数情况下 Ceph 都能从若干副本中选择正确的，
但是也会有例外。比如，存储的数字签名可能正好丢了，
选择权威副本时又忽略了计算出的数字签名，
总之，用此命令时小心为好。

如果一个分片的 ``errors`` 里出现了 ``read_error`` ，
很可能是磁盘错误引起的不一致，
你最好先查验那个 OSD 所用的磁盘。

如果你时不时遇到时钟偏移引起的 ``active + clean + inconsistent`` 状态，
最好在监视器主机上配置 peer 角色的 `NTP`_ 服务。
配置细节可参考\ `网络时间协议`_\ 和 Ceph `时钟选项`_\ 。


纠删编码的归置组不是 active+clean
=================================
.. Erasure Coded PGs are not active+clean

CRUSH 找不到足够多的 OSD 映射到某个 PG 时，它会显示为
``2147483647`` ，意思是 ITEM_NONE 或 ``no OSD found`` ，例如： ::

	[2,1,6,0,5,8,2147483647,7,4]

OSD 不够多
----------
.. Not enough OSDs

如果 Ceph 集群仅有 8 个 OSD ，但是纠删码存储池需要 9 个，就会显示上面的错误。
这时候，你仍然可以另外创建需要较少 OSD 的纠删码存储池： ::

	ceph osd erasure-code-profile set myprofile k=5 m=3
	ceph osd pool create erasurepool erasure myprofile

或者新增一个 OSD ，这个 PG 会自动用上的。

CRUSH 条件不能满足
------------------
.. CRUSH constraints cannot be satisfied

即使集群拥有足够多的 OSD ， CRUSH 规则的强制要求仍有可能无法满足。
假如有 10 个 OSD 分布于两个主机上，且 CRUSH 规则要求\
相同归置组不得使用位于同一主机的两个 OSD ，这样映射就会失败，\
因为只能找到两个 OSD ，你可以从规则里查看必要条件： ::

    $ ceph osd crush rule ls
    [
        "replicated_rule",
        "erasurepool"]
    $ ceph osd crush rule dump erasurepool
    { "rule_id": 1,
      "rule_name": "erasurepool",
      "type": 3,
      "steps": [
            { "op": "take",
              "item": -1,
              "item_name": "default"},
            { "op": "chooseleaf_indep",
              "num": 0,
              "type": "host"},
            { "op": "emit"}]}

可以这样解决此问题，创建新存储池，其内的 PG 允许多个 OSD 位于\
同一主机，命令如下： ::

	ceph osd erasure-code-profile set myprofile crush-failure-domain=osd
	ceph osd pool create erasurepool erasure myprofile


CRUSH 过早中止
--------------
.. CRUSH gives up too soon

假设集群拥有的 OSD 足以映射到 PG
（比如有 9 个 OSD 和一个纠删码存储池的集群，
每个 PG 需要 9 个 OSD ）， CRUSH 仍然\
有可能在找到映射前就中止了。可以这样解决：

* 降低纠删存储池内 PG 的要求，让它使用较少的 OSD
  （需创建另一个存储池，
  因为纠删码配置不支持动态修改）。

* 向集群添加更多 OSD （无需修改纠删存储池，
  它会自动回到清洁状态）。

* 通过手工打造的 CRUSH 规则，让它多试几次以找到合适的映射。
  把 ``set_choose_tries`` 设置得\
  高于默认值即可。

你从集群中提取出 crushmap 之后，应该先用 ``crushtool`` 校验\
一下是否有问题，这样你的试验就无需触及 Ceph 集群，只要在一个\
本地文件上测试即可： ::

    $ ceph osd crush rule dump erasurepool
    { "rule_id": 1,
      "rule_name": "erasurepool",
      "type": 3,
      "steps": [
            { "op": "take",
              "item": -1,
              "item_name": "default"},
            { "op": "chooseleaf_indep",
              "num": 0,
              "type": "host"},
            { "op": "emit"}]}
    $ ceph osd getcrushmap > crush.map
    got crush map from osdmap epoch 13
    $ crushtool -i crush.map --test --show-bad-mappings \
       --rule 1 \
       --num-rep 9 \
       --min-x 1 --max-x $((1024 * 1024))
    bad mapping rule 8 x 43 num_rep 9 result [3,2,7,1,2147483647,8,5,6,0]
    bad mapping rule 8 x 79 num_rep 9 result [6,0,2,1,4,7,2147483647,5,8]
    bad mapping rule 8 x 173 num_rep 9 result [0,4,6,8,2,1,3,7,2147483647]

其中 ``--num-rep`` 是纠删码 CRUSH 规则所需的 OSD 数量，
``--rule`` 是 ``ceph osd crush rule dump`` 命令结果中
``rule_id`` 字段的值。此测试会尝试映射一百万个值
（即 ``[--min-x,--max-x]`` 所指定的范围），
且必须至少显示一个坏映射；如果它没有任何输出，
说明所有映射都成功了，你可以就此打住：
问题的根源不在这里。

反编译 crush 图后，你可以手动编辑其 CRUSH 规则： ::

	$ crushtool --decompile crush.map > crush.txt

并把下面这行加进规则： ::

	step set_choose_tries 100

然后 ``crush.txt`` 文件内的这部分大致如此： ::

     rule erasurepool {
             id 1
             type erasure
             step set_chooseleaf_tries 5
             step set_choose_tries 100
             step take default
             step chooseleaf indep 0 type host
             step emit
     }

然后编译、并再次测试： ::

	$ crushtool --compile crush.txt -o better-crush.map

所有映射都成功时，
用 ``crushtool`` 的 ``--show-choose-tries`` 选项\
能看到成功映射的尝试次数直方图： ::

	$ crushtool -i better-crush.map --test --show-bad-mappings \
	   --show-choose-tries \
	   --rule 1 \
	   --num-rep 9 \
	   --min-x 1 --max-x $((1024 * 1024))
	...
	11:        42
	12:        44
	13:        54
	14:        45
	15:        35
	16:        34
	17:        30
	18:        25
	19:        19
	20:        22
	21:        20
	22:        17
	23:        13
	24:        16
	25:        13
	26:        11
	27:        11
	28:        13
	29:        11
	30:        10
	31:         6
	32:         5
	33:        10
	34:         3
	35:         7
	36:         5
	37:         2
	38:         5
	39:         5
	40:         2
	41:         5
	42:         4
	43:         1
	44:         2
	45:         2
	46:         3
	47:         1
	48:         0
	...
	102:         0
	103:         1
	104:         0
	...

有 42 个归置组需 11 次重试、 44 个归置组需 12 次重试，\
以此类推。这样，重试的最高次数就是防止坏映射的最低值，也就是
``set_choose_tries`` 的取值（即上面输出中的 103 ，因为任意\
归置组成功映射的重试次数都没有超过 103 ）。

.. _检查: ../../operations/placement-groups#get-the-number-of-placement-groups
.. _这里: ../../configuration/pool-pg-config-ref
.. _归置组: ../../operations/placement-groups
.. _存储池、归置组和 CRUSH 配置参考: ../../configuration/pool-pg-config-ref
.. _NTP: https://en.wikipedia.org/wiki/Network_Time_Protocol
.. _网络时间协议: http://www.ntp.org/
.. _时钟选项: ../../configuration/mon-config-ref/#clock
