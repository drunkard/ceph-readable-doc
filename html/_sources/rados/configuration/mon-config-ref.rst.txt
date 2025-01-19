.. _monitor-config-reference:

================
 监视器配置参考
================
.. Monitor Config Reference

理解如何配置 :term:`Ceph 监视器`\ 是构建可靠的
:term:`Ceph 存储集群`\ 的重要方面，\
**任何 Ceph 集群都需要至少一个监视器**\ 。一个监视器通常相当\
一致，但是你可以增加、删除、或替换集群中的监视器，详情见\
`增加/删除监视器`_\ 。


.. index:: Ceph Monitor; Paxos

背景
====
.. Background

Ceph 监视器们维护着\ :term:`集群运行图`\ 的“主副本”。

.. TODO

:term:`Ceph 客户端`\ 只要连到一个 Ceph 监视器并获取一份当前的\
集群运行图就能确定所有监视器、 OSD 和元数据服务器的位置。
Ceph 客户端读写 OSD 或元数据服务器前，必须先连到一个监视器，\
用当前的集群运行图副本和 CRUSH 算法，客户端能计算出任何对象的\
位置，拥有计算对象位置的技能——故此客户端可以直接与 OSD 通讯，
这对 Ceph 的高伸缩性、高性能来说非常重要。更多信息见\
`伸缩性和高可用性`_\ 。

监视器的主要角色是维护集群运行图的主副本，它也提供认证和\
日志记录服务。 Ceph 监视器们把监视器服务的所有更改写入一个\
单独的 Paxos 例程，然后 Paxos 以键/值方式存储所有变更以实现\
高度一致性。同步期间， Ceph 监视器能查询集群运行图的近期版本，\
它们通过操作键/值存储快照和迭代器（用 leveldb ）来进行\
存储级同步。


.. ditaa::

    /-------------\               /-------------\
    |   Monitor   | Write Changes |    Paxos    |
    |   cCCC      +-------------->+   cCCC      |
    |             |               |             |
    +-------------+               \------+------/
    |    Auth     |                      |
    +-------------+                      | Write Changes
    |    Log      |                      |
    +-------------+                      v
    | Monitor Map |               /------+------\
    +-------------+               | Key / Value |
    |   OSD Map   |               |    Store    |
    +-------------+               |  cCCC       |
    |   PG Map    |               \------+------/
    +-------------+                      ^
    |   MDS Map   |                      | Read Changes
    +-------------+                      |
    |    cCCC     |*---------------------+
    \-------------/


.. index:: Ceph Monitor; cluster map

集群运行图
----------
.. Cluster Maps

集群运行图是多个图的组合，包括监视器图、 OSD 图、
归置组图和元数据服务器图。集群运行图追踪几个重要事件：
哪些进程在集群里（ ``in`` ）；
哪些进程在集群里（ ``in`` ）是 ``up`` 且在运行、
或 ``down`` ；归置组状态是 ``active`` 或 ``inactive`` 、 \
``clean`` 或其他状态；和其他反映当前集群状态的信息，
像总存储容量、和使用量。

当集群状态有明显变更时，如一 OSD 挂了、
一归置组降级了等等，集群运行图会被更新以反映集群当前状态。
另外，监视器也维护着集群的主要状态历史。
监视器图、 OSD 图、归置组图和元数据服务器图\
各自维护着它们的运行图版本。
我们把各图的版本称为一个 epoch 。

运营集群时，跟踪这些状态是系统管理任务的重要部分。
详情见\ `监控集群`_\ 和\ `监控 OSD 和归置组`_\ 。


.. index:: high availability; quorum

监视器法定人数
--------------
.. Monitor Quorum

本文配置部分提供了一个简陋的 `Ceph 配置文件`_\ ，
它提供了一个监视器用于测试。只用一个监视器集群\
可以良好地运行，然而\ **单监视器是一个单故障点**\ ，
生产集群要实现高可用性的话得配置多个监视器，
这样单个监视器的失效才\ **不会**\ 影响整个集群。

集群用多个监视器实现高可用性时，
多个监视器用 `Paxos`_ 算法对主集群运行图达成一致，
这里的一致要求大多数监视器都在运行且够成法定人数
（如 1 个、 3 之 2 在运行、 5 之 3 、 6 之 4 等等）。

.. confval:: mon_force_quorum_join


.. index:: Ceph Monitor; consistency

一致性
------
.. Consistency

你把监视器加进 Ceph 配置文件时，
得注意一些架构问题， Ceph 发现集群内的\
其他监视器时对其有着\ **严格的一致性要求**\ 。
尽管如此， Ceph 客户端和其他 Ceph 守护进程\
用配置文件发现监视器，监视器却用监视器图（ monmap ）
相互发现而非配置文件。

一个监视器发现集群内的其他监视器时总是参考 monmap 的本地副本，\
用 monmap 而非 Ceph 配置文件避免了可能损坏集群的错误
（如 ``ceph.conf`` 中指定地址或端口的拼写错误）。
正因为监视器把 monmap 用于发现、
并共享于客户端和其他 Ceph 守护进程间，
**monmap可严格地保证监视器的一致性是可靠的**\ 。

严格的一致性也适用于 monmap 的更新，
因为关于监视器的任何更新、关于 monmap 的变更\
都是通过称为 `Paxos`_ 的分布式一致性算法传递的。
监视器们必须就 monmap 的每次更新达成一致，
以确保法定人数里的每个监视器 monmap 版本相同，
如增加、删除一个监视器。 monmap 的更新是增量的，
所以监视器们都有最新的一致版本，以及一系列之前版本。
历史版本的存在允许一个落后的监视器跟上集群当前状态。

如果监视器通过配置文件而非 monmap 相互发现，
这会引进其他风险，因为 Ceph 配置文件不是自动更新并分发的，
监视器有可能不小心用了较老的配置文件，
以致于不认识某监视器、放弃法定人数、或者\
产生一种 `Paxos`_ 不能确定当前系统状态的情形。


.. index:: Ceph Monitor; bootstrapping monitors

初始化监视器
------------
.. Bootstrapping Monitors

在大多数配置和部署案例中，部署 Ceph 的工具可以\
帮你生成一个监视器图来初始化监视器（如 ``cephadm`` 等），
一个监视器需要 4 个选项：

- **文件系统标识符：** ``fsid`` 是对象存储的唯一标识符。
  因为你可以在一套硬件上运行多个集群，
  所以在初始化监视器时必须指定对象存储的唯一标识符。
  部署工具通常可替你完成（如 ``cephadm`` 会调用\
  类似 ``uuidgen`` 的程序），但是你也可以手动指定 ``fsid`` 。

- **监视器标识符：** 监视器标识符是分配给\
  集群内各监视器的唯一 ID ，它是一个字母数字组合，
  为方便起见，标识符通常以字母顺序结尾
  （如 ``a`` 、 ``b`` 等等），可以设置于 Ceph 配置文件
  （如 ``[mon.a]`` 、 ``[mon.b]`` 等等）、部署工具、
  或 ``ceph`` 命令行工具。

- **密钥：** 监视器必须有密钥。像 ``cephadm``
  这样的部署工具通常会自动生成，也可以手动完成。
  见\ `监视器密钥环`_\ 。

关于初始化的具体信息见\ `初始化监视器`_\ 。


.. index:: Ceph Monitor; configuring monitors

监视器的配置
============
.. Configuring Monitors

要把配置应用到整个集群，把它们放到 ``[global]`` 下；
要用于所有监视器，置于 ``[mon]`` 下；
要用于某监视器，指定监视器例程，\
如 ``[mon.a]`` ）。按惯例，监视器例程用字母命名。

.. code-block:: ini

	[global]

	[mon]

	[mon.a]

	[mon.b]

	[mon.c]


最小配置
--------
.. Minimum Configuration

Ceph 监视器的最简配置必须包括一主机名及其监视器地址，
这些配置可置于 ``[mon]`` 下或某个监视器下。

.. code-block:: ini

	[mon]
		mon host = hostname1,hostname2,hostname3
		mon addr = 10.0.0.10:6789,10.0.0.11:6789,10.0.0.12:6789


.. code-block:: ini

	[mon.a]
		host = hostname1
		mon addr = 10.0.0.10:6789

详情见\ `网络配置参考`_\ 。

.. note:: 这里的监视器最简配置假设部署工具会自动给你生成
   ``fsid`` 和 ``mon.`` 密钥。

一旦部署完 Ceph 集群，监视器 IP 地址就\ **不应该**\ 更改了。\
然而，如果你决意要改，必须严格遵循特定的步骤，
详情见\ :ref:`更改监视器的 IP 地址`\ 。

也可以让客户端通过 DNS 的 SRV 记录发现监视器，
详情见\ `通过 DNS 查询监视器`_\ 。


集群 ID
-------
.. Cluster ID

每个 Ceph 存储集群都有一个唯一标识符（ ``fsid`` ）。如果\
指定了，它应该出现在配置文件的 ``[global]`` 段下。部署工具\
通常会生成 ``fsid`` 并存于监视器图，所以不一定会写入配置文件，\
``fsid`` 使得在一套硬件上运行多个集群成为可能。

.. confval:: fsid


.. index:: Ceph Monitor; initial members

初始成员
--------
.. Initial Members

我们建议在生产环境下最少部署 3 个监视器，以确保高可用性。运行\
多个监视器时，你可以指定为形成法定人数成员所需的初始监视器，\
这能减小集群上线时间。

.. code-block:: ini

	[mon]
		mon_initial_members = a,b,c

.. confval:: mon_initial_members


.. index:: Ceph Monitor; data path

数据
----
.. Data

Ceph 监视器有存储数据的默认路径。为优化性能，在生产集群上，\
我们建议在独立主机上运行 Ceph 监视器，不要与运行 Ceph OSD
守护进程的主机混用。因为 leveldb 靠 ``mmap()`` 写数据， Ceph
监视器会频繁地把数据从内存刷回磁盘，如果其数据与 OSD
守护进程共用存储器，就会与 Ceph OSD 守护进程的载荷冲突。

在 Ceph 0.58 及更早版本中，监视器数据以文件保存，这样人们可以\
用 ``ls`` 和 ``cat`` 这些普通工具检查监视器数据，然而它不能\
提供健壮的一致性。

在 Ceph 0.59 及后续版本中，监视器以键/值对存储数据。
监视器需要 `ACID`_ 事务，数据存储的使用可防止监视器\
用损坏的版本进行恢复，除此之外，
它允许在一个原子批量操作中进行多个修改操作。

一般来说我们不建议更改默认数据位置，如果要改，我们建议所有\
监视器统一配置，加到配置文件的 ``[mon]`` 下。

.. confval:: mon_data
.. confval:: mon_data_size_warn
.. confval:: mon_data_avail_warn
.. confval:: mon_data_avail_crit
.. confval:: mon_warn_on_cache_pools_without_hit_sets
.. confval:: mon_warn_on_crush_straw_calc_version_zero
.. confval:: mon_warn_on_legacy_crush_tunables
.. confval:: mon_crush_min_required_version
.. confval:: mon_warn_on_osd_down_out_interval_zero
.. confval:: mon_warn_on_slow_ping_ratio
.. confval:: mon_warn_on_slow_ping_time
.. confval:: mon_warn_on_pool_no_redundancy
.. confval:: mon_cache_target_full_warn_ratio
.. confval:: mon_health_to_clog
.. confval:: mon_health_to_clog_tick_interval
.. confval:: mon_health_to_clog_interval


.. index:: Ceph Storage Cluster; capacity planning, Ceph Monitor; capacity planning

.. _storage-capacity:

存储容量
--------
.. Storage Capacity

Ceph 存储集群利用率接近最大容量时（即 ``mon osd full ratio`` ），\
作为防止数据丢失的安全措施，它会阻止你读写 OSD 。
因此，让生产集群用满可不是好事，因为牺牲了高可用性。
full ratio 默认值是 ``.95`` 或容量的 95% 。
对小型测试集群来说这是非常激进的设置。

.. tip:: 监控集群时，要警惕和 ``nearfull`` 相关的警告。
   这意味着一些 OSD 的失败会导致临时服务中断，
   应该增加一些 OSD 来扩展存储容量。

在测试集群时，一个常见场景是：
系统管理员从集群删除一个 OSD 、\
接着观察重均衡；然后继续删除其他 OSD ，
直到集群达到占满率并锁死。我们建议，
即使在测试集群里也要规划一点空闲容量用于保证高可用性。
理想情况下，要做好这样的预案：一系列 OSD 失败后，\
短时间内不更换它们仍能恢复到 ``active + clean`` 状态。
你也可以在 ``active + degraded`` 状态运行集群，
但对正常使用来说并不好。

下图描述了一个简化的 Ceph 集群，它包含 33 个节点、
每主机一个 OSD 、每 OSD 3TB 容量，
所以这个小白鼠集群有 99TB 的实际容量，
其 ``mon osd full ratio`` 为 ``.95`` 。
如果它只剩余 5TB 容量，\
集群就不允许客户端再读写数据，
所以它的运行容量是 95TB ，而非 99TB 。

.. ditaa::

 +--------+  +--------+  +--------+  +--------+  +--------+  +--------+
 | Rack 1 |  | Rack 2 |  | Rack 3 |  | Rack 4 |  | Rack 5 |  | Rack 6 |
 | cCCC   |  | cF00   |  | cCCC   |  | cCCC   |  | cCCC   |  | cCCC   |
 +--------+  +--------+  +--------+  +--------+  +--------+  +--------+
 | OSD 1  |  | OSD 7  |  | OSD 13 |  | OSD 19 |  | OSD 25 |  | OSD 31 |
 +--------+  +--------+  +--------+  +--------+  +--------+  +--------+
 | OSD 2  |  | OSD 8  |  | OSD 14 |  | OSD 20 |  | OSD 26 |  | OSD 32 |
 +--------+  +--------+  +--------+  +--------+  +--------+  +--------+
 | OSD 3  |  | OSD 9  |  | OSD 15 |  | OSD 21 |  | OSD 27 |  | OSD 33 |
 +--------+  +--------+  +--------+  +--------+  +--------+  +--------+
 | OSD 4  |  | OSD 10 |  | OSD 16 |  | OSD 22 |  | OSD 28 |  | Spare  |
 +--------+  +--------+  +--------+  +--------+  +--------+  +--------+
 | OSD 5  |  | OSD 11 |  | OSD 17 |  | OSD 23 |  | OSD 29 |  | Spare  |
 +--------+  +--------+  +--------+  +--------+  +--------+  +--------+
 | OSD 6  |  | OSD 12 |  | OSD 18 |  | OSD 24 |  | OSD 30 |  | Spare  |
 +--------+  +--------+  +--------+  +--------+  +--------+  +--------+

在这样的集群里，坏一或两个 OSD 很平常；
一种罕见但可能发生的情形是一个机架的路由器或电源挂了，
这会导致多个 OSD 同时离线（如 OSD 7-12 ），
在这种情况下，你仍要力争保持集群可运行\
并达到 ``active + clean`` 状态，
即使这意味着你得在短期内额外增加一些 OSD 及主机。
如果集群利用率太高，在解决故障域期间也许不会丢数据，
但很可能牺牲数据可用性，因为利用率超过了 full ratio 。\
故此，我们建议至少要粗略地规划下容量。

找出你集群的两个数量：

#. OSD 数量。
#. 集群总容量。

用集群里 OSD 总数除以集群总容量，
就能得到 OSD 平均容量；
如果按预计的 OSD 数乘以这个值所得的结果计算（偏小），
实际应用时将出错；
最后再用集群容量乘以占满率能得到最大运行容量，
然后，扣除预估的 OSD 失败率；
用较高的失败率（如整机架的 OSD ）
重复前述过程看是否接近占满率。

下列配置仅在创建集群时有效，之后就存储在 OSDMap 里。
要说明的是，在日常操作中，
OSD 们使用的数值是 OSDMap 里的，
不是配置文件或中央配置库里的。

.. code-block:: ini

	[global]

		mon_osd_full_ratio = .80
		mon_osd_backfillfull_ratio = .75
		mon_osd_nearfull_ratio = .70


``mon_osd_full_ratio``

:描述: OSD 硬盘使用率达到多少就认为它 ``full`` 。
:类型: Float
:默认值: ``.95``


``mon_osd_backfillfull_ratio``

:描述: OSD 磁盘空间利用率达到多少就认为它太满了，
       不能再接受回填。
:类型: Float
:默认值: ``.90``


``mon_osd_nearfull_ratio``

:描述: OSD 硬盘使用率达到多少就认为它 ``nearfull`` 。
:类型: Float
:默认值: ``.85``


.. tip:: 如果一些 OSD 快满了，但其他的仍有足够空间，
   你可能配错 CRUSH 权重了。

.. tip:: 这些配置仅在创建集群时有效。
   之后要改它们就在 OSDMap 里了，
   可以用 ``ceph osd set-nearfull-ratio`` 和
   ``ceph osd set-full-ratio`` 。


.. index:: heartbeat

心跳
----
.. Heartbeat

Ceph 监视器要求各 OSD 向它报告、
并接收 OSD 们的邻居状态报告，\
以此来掌握集群。 Ceph 提供了监视器与 OSD 交互的合理默认值，\
然而你可以按需修改，详情见\ `监视器与 OSD 的交互`_\ 。


.. index:: Ceph Monitor; leader, Ceph Monitor; provider, Ceph Monitor; requester, Ceph Monitor; synchronization

监视器存储同步
--------------
.. Monitor Store Synchronization

当你用多个监视器（建议的）支撑一个生产集群时，
各监视器都要检查邻居是否有集群运行图的最新版本
（如，邻居监视器的图有一或多个 epoch 版本\
高于当前监视器的最高版 epoch ），
过一段时间，集群里的某个监视器可能\
落后于其它监视器太多而不得不离开法定人数，
然后同步到集群当前状态，并重回法定人数。
为了同步，监视器可能承担三种中的一种角色：

#. **Leader**: `Leader` 是实现最新 Paxos 版本的\
   第一个监视器。

#. **Provider**: `Provider` 有最新集群运行图的监视器，
   但不是第一个实现最新版。

#. **Requester:** `Requester` 落后于 leader ，
   重回法定人数前，\
   必须同步以获取关于集群的最新信息。

有了这些角色区分， leader 就可以给 provider 委派同步任务，\
这会避免同步请求压垮 leader 、影响性能。
在下面的图示中， requester 已经知道它\
落后于其它监视器，然后向 leader 请求同步，
leader 让它去和 provider 同步。


.. ditaa::

           +-----------+          +---------+          +----------+
           | Requester |          | Leader  |          | Provider |
           +-----------+          +---------+          +----------+
                  |                    |                     |
                  |                    |                     |
                  | Ask to Synchronize |                     |
                  |------------------->|                     |
                  |                    |                     |
                  |<-------------------|                     |
                  | Tell Requester to  |                     |
                  | Sync with Provider |                     |
                  |                    |                     |
                  |               Synchronize                |
                  |--------------------+-------------------->|
                  |                    |                     |
                  |<-------------------+---------------------|
                  |        Send Chunk to Requester           |
                  |         (repeat as necessary)            |
                  |    Requester Acks Chuck to Provider      |
                  |--------------------+-------------------->|
                  |                    |
                  |   Sync Complete    |
                  |    Notification    |
                  |------------------->|
                  |                    |
                  |<-------------------|
                  |        Ack         |
                  |                    |


新监视器加入集群时有必要进行同步。在运行中，监视器会不定时收到\
集群运行图的更新，这就意味着 leader 和 provider 角色可能在监视器间变幻。
如果这事发生在同步期间（如 provider 落后于 leader ），
provider 能终结和 requester 间的同步。

一旦同步完成， Ceph 需要修复整个集群，
使归置组回到 ``active + clean`` 状态。

.. confval:: mon_sync_timeout
.. confval:: mon_sync_max_payload_size
.. confval:: paxos_max_join_drift
.. confval:: paxos_stash_full_interval
.. confval:: paxos_propose_interval
.. confval:: paxos_min
.. confval:: paxos_min_wait
.. confval:: paxos_trim_min
.. confval:: paxos_trim_max
.. confval:: paxos_service_trim_min
.. confval:: paxos_service_trim_max
.. confval:: paxos_service_trim_max_multiplier
.. confval:: mon_mds_force_trim_to
.. confval:: mon_osd_force_trim_to
.. confval:: mon_osd_cache_size
.. confval:: mon_election_timeout
.. confval:: mon_lease
.. confval:: mon_lease_renew_interval_factor
.. confval:: mon_lease_ack_timeout_factor
.. confval:: mon_accept_timeout_factor
.. confval:: mon_min_osdmap_epochs
.. confval:: mon_max_log_epochs


.. index:: Ceph Monitor; clock

.. _mon-config-ref-clock:

时钟
----
.. Clock

Ceph 的守护进程会相互传递关键消息，
这些消息必须在达到超时阀值前处理掉。
如果 Ceph 监视器时钟不同步，就可能出现多种异常情况。例如：

- 守护进程忽略了收到的消息（如时间戳过时了）
- 消息未及时收到时，超时触发得太快或太晚。

详情见\ `监视器存储同步`_\ 。

.. tip:: 你必须在所有监视器主机上安装 NTP 或 PTP 守护进程\
   以确保监视器集群在时钟同步的前提下运行。可以让各个监视器之间相互同步、
   也可以和几个高品质的上游时间源同步。

时钟漂移即使尚未造成损坏也能被 NTP 感知， Ceph 的时钟漂移或时\
钟偏差警告即使在 NTP 同步水平合理时也会被触发。提高时钟漂移值\
有时候尚可容忍，然而很多因素（像载荷、网络延时、覆盖默认超时值\
和\ `监视器存储同步`_\ 选项）都能在不降低 Paxos 保证级别的情况\
下影响可接受的时钟漂移水平。

Ceph 提供了下列这些可调选项，让你自己琢磨可接受的值。

.. confval:: mon_tick_interval
.. confval:: mon_clock_drift_allowed
.. confval:: mon_clock_drift_warn_backoff
.. confval:: mon_timecheck_interval
.. confval:: mon_timecheck_skew_interval


客户端
------
.. Client

.. confval:: mon_client_hunt_interval
.. confval:: mon_client_ping_interval
.. confval:: mon_client_max_log_entries_per_message
.. confval:: mon_client_bytes


.. _pool-settings:

存储池选项
==========
.. Pool settings

从 v0.94 版起，存储池可通过标记来表明这个存储池允许或禁止更改。\
如果配置得当，监视器也可以禁止存储池的删除。\
这种防护方式虽有不便，和它防止的存储池（还有数据）误删比起来就差远了。

.. confval:: mon_allow_pool_delete
.. confval:: osd_pool_default_ec_fast_read
.. confval:: osd_pool_default_flag_hashpspool
.. confval:: osd_pool_default_flag_nodelete
.. confval:: osd_pool_default_flag_nopgchange
.. confval:: osd_pool_default_flag_nosizechange

关于存储池标记详情请看\ :ref:`存储池标记值 <setpoolvalues>`\ 。


杂项
====
.. Miscellaneous

.. confval:: mon_max_osd
.. confval:: mon_globalid_prealloc
.. confval:: mon_subscribe_interval
.. confval:: mon_stat_smooth_intervals
.. confval:: mon_probe_timeout
.. confval:: mon_daemon_bytes
.. confval:: mon_max_log_entries_per_event
.. confval:: mon_osd_prime_pg_temp
.. confval:: mon_osd_prime_pg_temp_max_time
.. confval:: mon_osd_prime_pg_temp_max_estimate
.. confval:: mon_mds_skip_sanity
.. confval:: mon_max_mdsmap_epochs
.. confval:: mon_config_key_max_entry_size
.. confval:: mon_scrub_interval
.. confval:: mon_scrub_max_keys
.. confval:: mon_compact_on_start
.. confval:: mon_compact_on_bootstrap
.. confval:: mon_compact_on_trim
.. confval:: mon_cpu_threads
.. confval:: mon_osd_mapping_pgs_per_chunk
.. confval:: mon_session_timeout
.. confval:: mon_osd_cache_size_min
.. confval:: mon_memory_target
.. confval:: mon_memory_autotune


.. _Paxos: https://en.wikipedia.org/wiki/Paxos_(computer_science)
.. _监视器密钥环: ../../../dev/mon-bootstrap#secret-keys
.. _Ceph 配置文件: ../ceph-conf/#monitors
.. _网络配置参考: ../network-config-ref
.. _通过 DNS 查询监视器: ../mon-lookup-dns
.. _ACID: https://en.wikipedia.org/wiki/ACID
.. _增加/删除监视器: ../../operations/add-or-rm-mons
.. _监控集群: ../../operations/monitoring
.. _监控 OSD 和归置组: ../../operations/monitoring-osd-pg
.. _初始化监视器: ../../../dev/mon-bootstrap
.. _监视器与 OSD 的交互: ../mon-osd-interaction
.. _伸缩性和高可用性: ../../../architecture#scalability-and-high-availability
