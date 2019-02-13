.. _Terminology:

术语
----

一个 Ceph 集群内可以没有、或者有多个 CephFS *文件系统*\ 。
CephFS 文件系统有一个人类可读的名字（用 ``fs new`` 设置）、和\
一个整数 ID ，这个 ID 称为文件系统集群 ID 或 *FSCID* 。

每个 CephFS 都有几个 *rank* ，默认是一个，从 0 起。一个 rank
可看作是一个元数据分片。文件系统 rank 数量的控制在
:doc:`/cephfs/multimds` 里详述。

CephFS 的每个 ceph-mds 进程（一个\ *守护进程*\ ）刚启动时都没\
有 rank ，它由监视器集群分配。一个守护进程一次只能占据一个
rank ，只有在 ceph-mds 守护进程停止的时候才会释放 rank 。

如果某个 rank 没有依附一个守护进程，那这个 rank 就\
*失效了（ failed ）*\ 。分配给守护进程的 rank 才被当作\
*正常的（ up ）*\ 。

首次配置守护进程时，管理员需分配静态的\ *名字*\ ，一般配置都\
会用守护进程所在的主机名作为其守护进程名字。

守护进程每次启动时，还会被分配一个 *GID* ，对于这个守护进程的\
特定进程的生命周期来说它是唯一的。 GID 是整数。

.. todo:: 译者注： rank 翻译为“席位”、“座席”？它们共同处理元数\
   据，且动态分配，类似客服中心的座席。


.. _Referring to MDS daemons:

MDS 守护进程的引用
------------------

针对 MDS 守护进程的大多数管理命令都接受一个灵活的参数格式，它\
可以包含 rank 、 GID 或名字。

使用 rank 时，它前面可以加文件系统的名字或 ID ，也可以不加。如\
果某个守护进程是灾备的（即当前还没给它分配 rank ），那就只能通\
过 GID 或名字引用。

例如，假设我们有一个 MDS 守护进程，名为 myhost ，其 GID 为
5446 ，分配的 rank 是 0 ，它位于名为 myfs 的文件系统内，文件系\
统的 FSCID 是 3 ，那么，下面的几种形式都适用于 fail 命令： ::

    ceph mds fail 5446     # GID
    ceph mds fail myhost   # Daemon name
    ceph mds fail 0        # Unqualified rank
    ceph mds fail 3:0      # FSCID and rank
    ceph mds fail myfs:0   # Filesystem name and rank


.. _Managing failover:

故障切换的管理
--------------

如果一个 MDS 守护进程不再与监视器通讯，监视器把它标记为 *laggy*
（滞后）状态前会等待 ``mds_beacon_grace`` 秒（默认是 15 秒）。

为安全起见，每个文件系统都要指定一些灾备守护进程，灾备数量包括\
处于灾备重放状态、等着接替失效 rank 的（记着，灾备重放守护进程\
不会被重分配去接管另一个 rank 、或者另一个 CephFS 文件系统内的\
失效守护进程）。不在重放状态的灾备守护进程会被任意文件系统当作\
自己的灾备。每个文件系统都可以单独配置期望的灾备守护进程数量，\
用这个： ::

    ceph fs set <fs name> standby_count_wanted <count>

``count`` 设置为 0 表示禁用健康检查。


.. _Configuring standby daemons:

灾备守护进程的配置
------------------

共有四个选项，用于控制守护进程处于灾备状态时如何工作： ::

    mds_standby_for_name
    mds_standby_for_rank
    mds_standby_for_fscid
    mds_standby_replay

这些配置可写入 MDS 守护进程所在主机（而非监视器上）的 ceph.conf
配置文件。守护进程会在启动时加载这些配置，并发送给监视器。

默认情况下，如果这些选项一个也没配置，那么没领到 rank 的所有
MDS 守护进程都会作为任意 rank 的灾备。

这些配置虽说可以把灾备守护进程关联到特定的名字或 rank ，然而并\
不能保证这个守护进程只用于那个 rank 。也就是说，有好几个灾备可\
用时，会用关联的守护进程。假如一个 rank 失效了，而且有一个可用\
的灾备，那么即使此灾备已关联了另外的 rank 或守护进程名，它仍然\
会被用掉。

mds_standby_replay
~~~~~~~~~~~~~~~~~~

如果此选项设置为 true ，灾备守护进程就会持续读取某个处于 up 状\
态的 rank 的元数据日志。这样它就有元数据的热缓存，在负责这个
rank 的守护进程失效时，可加速故障切换。

一个正常运行的 rank 只能有一个灾备重放守护进程（ standby replay
daemon ），如果两个守护进程都设置成了灾备重放状态，那么其中任\
意一个会取胜，另一个会变为普通的、非重放灾备状态。

一旦某个守护进程进入灾备重放状态，它就只能为它那个 rank 提供灾\
备。如果有另外一个 rank 失效了，即使没有灾备可用，这个灾备重放\
守护进程也不会去顶替那个失效的。

*历史问题：*\ 在 v10.2.1 版之前的 Ceph 中，只要设置了
``mds_standby_for_*`` ，这个选项（设置为 ``false`` 时）就始终为
true 。

mds_standby_for_name
~~~~~~~~~~~~~~~~~~~~

设置此选项可使灾备守护进程只接替与此名字相同的 rank 。

mds_standby_for_rank
~~~~~~~~~~~~~~~~~~~~

设置此选项可使灾备守护进程只接替指定的 rank ，如果有其它 rank
失效了，此守护进程不会去顶替它。

你有多个文件系统时，可联合使用 ``mds_standby_for_fscid`` 选项\
来指定想为哪个文件系统的 rank 提供灾备。

mds_standby_for_fscid
~~~~~~~~~~~~~~~~~~~~~

如果设置了 ``mds_standby_for_rank`` ，那么这里可把它限定为是哪\
个文件系统的 rank 。

如果没设置 ``mds_standby_for_rank`` ，那么设置 FSCID 后，这个\
守护进程可用于此 FSCID 所相关的任意 rank 。当你想让某一守护进\
程可作任意 rank 的灾备、但仅限于某个特定文件系统时，可用此选项\
实现。

mon_force_standby_active
~~~~~~~~~~~~~~~~~~~~~~~~

这些选项用于监视器主机，默认值为 true 。

如果设置为 false ，那么配置为 standby_replay=true 的灾备守护进\
程\ *只会*\ 顶替它们负责的 rank 或指定名字。反之，如果这里设置\
为 true ，那么配置为 standby_replay=true 的灾备守护进程有可能\
被分配其它 rank 。


.. _Examples:

实例
----

这些是 ceph.conf 配置实例。实践中，你可以让所有服务器上的所有\
守护进程都使用相同的配置文件，也可以让各服务器上的配置文件各不\
相同，其中只包含它的其守护进程相关的配置。

.. _Simple pair:

简单的一对
~~~~~~~~~~

两个 MDS 守护进程 a 和 b 作为一对，其中任意一个没分到 rank 的\
将作为另一个的灾备重放追随者。

::

    [mds.a]
    mds standby replay = true
    mds standby for rank = 0

    [mds.b]
    mds standby replay = true
    mds standby for rank = 0

.. _Floating standby:

浮动灾备
~~~~~~~~

某一文件系统有三个 MDS 守护进程： a 、 b 、 c ，其 ``max_mds``
为 2 。 ::

    # 无须过多配置，没有分到 rank 的守护进程会成为灾备，它会顶
    # 替任何一个失效的守护进程。

.. _Two MDS clusters:

两个 MDS 集群
~~~~~~~~~~~~~

有两个文件系统、四个 MDS 守护进程，我想让其中两个作为一对服务\
于一个文件系统，另外两个作为一对服务于另一个文件系统。

::

    [mds.a]
    mds standby for fscid = 1

    [mds.b]
    mds standby for fscid = 1

    [mds.c]
    mds standby for fscid = 2

    [mds.d]
    mds standby for fscid = 2

