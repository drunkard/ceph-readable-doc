.. _cephfs-multimds:

多活 MDS 守护进程的配置
-----------------------
.. Configuring multiple active MDS daemons

*也叫： multi-mds 、 active-active MDS*

每个 CephFS 文件系统默认情况下都只配置一个活跃 MDS 守护进程。\
在大型系统中，为了扩展元数据性能你可以配置多个活跃的 MDS 守护\
进程，它们会共同承担元数据负载。

什么情况下我需要多个活跃的 MDS 守护进程？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. When should I use multiple active MDS daemons?

当元数据默认的单个 MDS 成为瓶颈时，你应该配置多个活跃的 MDS 守\
护进程。

增加守护进程不一定都能提升性能，要看负载类型。典型地，
单个客户端上的单个应用程序就不会受益于 MDS 守护进程的增加，
除非这个应用程序是在并行地操作元数据。

通常，有很多客户端（操作着很多不同的目录时更好）时，
大量活跃的 MDS 守护进程有利于性能提升。

MDS 活跃集群的扩容
~~~~~~~~~~~~~~~~~~
.. Increasing the MDS active cluster size

每一个 CephFS 文件系统都有自己的 *max_mds* 配置，
它控制着会创建多少 rank 。有空闲守护进程可接管新 rank 时，
文件系统 rank 的实际数量才会增加，
比如只有一个 MDS 守护进程运行着、 max_mds 被设置成了 2 ，
此时不会创建第二个 rank 。（注意，这样的配置不具备高可用性（HA），
因为没有备机能够接管失败的 rank 。
配置成这样时，集群会通过健康告警发牢骚）

把 ``max_mds`` 设置为想要的 rank 数量。在下面的例子里，
``ceph status`` 输出的 fsmap 行是此命令可能输出的结果。

::

    # fsmap e5: 1/1/1 up {0=a=up:active}, 2 up:standby

    ceph fs set <fs_name> max_mds 2

    # fsmap e8: 2/2/2 up {0=a=up:active,1=c=up:creating}, 1 up:standby
    # fsmap e9: 2/2/2 up {0=a=up:active,1=c=up:active}, 1 up:standby

新创建的 rank (1) 会从 creating 状态过渡到
active 状态。

灾备守护进程
~~~~~~~~~~~~
.. Standby daemons

即使拥有多活 MDS 守护进程，一个高可用系统\
*仍然需要灾备守护进程*\ 来顶替失效的\
活跃守护进程。

因此，高可用系统的 ``max_mds`` 实际最大值比系统中 MDS 服务器的\
总数小一。

为了在多个服务器失效时仍能保持可用，
需增加系统中的灾备守护进程，
以弥补你能承受的服务器失效数量。

减少 rank 数量
~~~~~~~~~~~~~~
.. Decreasing the number of ranks

减少 rank 数量和减少 ``max_mds`` 一样简单：

::

    # fsmap e9: 2/2/2 up {0=a=up:active,1=c=up:active}, 1 up:standby
    ceph fs set <fs_name> max_mds 1
    # fsmap e10: 2/2/1 up {0=a=up:active,1=c=up:stopping}, 1 up:standby
    # fsmap e10: 2/2/1 up {0=a=up:active,1=c=up:stopping}, 1 up:standby
    ...
    # fsmap e10: 1/1/1 up {0=a=up:active}, 2 up:standby

集群将会自动逐步地停掉多余的 rank ，直到符合 ``max_mds`` 。

在 :doc:`/cephfs/administration` 文档里有关于 ``<role>`` 格式的\
详细描述。

注意：被停掉的 rank 会先进入 stopping 状态，并持续一段时间，
在此期间它要把它分享的那部分元数据转手给仍然活跃着的 MDS 守护进程，
这个过程可能要持续数秒到数分钟。
如果这个 MDS 卡在了 stopping 状态，
那可能是触发了软件缺陷。

如果一个 MDS 守护进程正处于 ``up:stopping`` 状态时崩溃了、或是\
被杀死了，就会有一个灾备顶替它，而且集群的监视器们也会阻止停止\
此守护进程的尝试。

守护进程完成 stopping 状态后，它会自己重生并成为灾备。


.. _cephfs-pinning:

手动将目录树插入特定的 rank
~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. Manually pinning directory trees to a particular rank

在多活元数据服务器配置中，均衡器负责在集群内\
均匀地散布元数据负荷。此设计对大多数用户来说都够用了，
但是，有时人们想要跳过动态均衡器，
手动把某些元数据映射到特定的 rank ；这样一来，
管理员或用户就可以均匀地散布应用负荷、或者限制用户的元数据请求，
以防他影响整个集群。

为实现此目的，引入了一个机制，名为 ``export pin`` （导出销），\
是目录的一个扩展属性，名为 ``ceph.dir.pin`` 。用户可以用\
标准命令配置此属性：

::

    setfattr -n ceph.dir.pin -v 2 path/to/dir

这个扩展属性的值是给这个目录树分配的 rank ，默认值 ``-1`` 表示\
此目录没有销进（某个 rank ）。

一个目录的导出销是从最近的、配置了导出销的父目录继承的；
同理，在一个目录上配置导出销会影响它的所有子目录。
然而，设置子目录的导出销可以覆盖从父目录继承来的销子，
例如：

::

    mkdir -p a/b
    # "a" and "a/b" both start without an export pin set
    setfattr -n ceph.dir.pin -v 1 a/
    # a and b are now pinned to rank 1
    setfattr -n ceph.dir.pin -v 0 a/b
    # a/b is now pinned to rank 0 and a/ and the rest of its children are still pinned to rank 1


.. _cephfs-ephemeral-pinning:

设置子树分区策略
~~~~~~~~~~~~~~~~
.. Setting subtree partitioning policies

通过一系列 **策略** 配置起 **自动化的** 子树静态分区也是可以的。
在 CephFS 里，这种自动化的静态分区被称为
**（临时挂单） ephemeral pinning** 。
任何临时挂单的目录（ inode ）都会根据其
inode 号的一致性哈希值被自动分配到一个特定的 rank 。
所有这些临时挂单的目录都会统一地分布到所有 rank 上。

给临时挂单的目录这样命名是因为，一旦这个目录 inode
被缓存丢弃 pin 是不会持久化的。然而，
一个 MDS 的故障不会影响挂单目录们“临时”的天性。
MDS 在日志中记录了哪些子树是临时挂单的，
这样 MDS 的故障就不会丢失这个信息。

一个目录可以是临时挂单的，也可以不是。
挂单到哪个 rank 是通过它的 inode 号和一致性哈希决定的。
这意味着临时挂单的目录们大致均匀地散布在 MDS 集群中。
**一致性哈希** 也最小化了在 MDS 集群扩容或缩小时的重分布。
所以，扩容 MDS 集群后，无需管理介入就可以自动地增加\
你的元数据吞吐量。

目前，有两种类型的临时挂单：

**分布式的临时挂单**: 这个策略会引起目录分片
（即使远低于常规分片阈值）、
并把它的分片作为临时挂单子树散布出去。
借助这个特性正好可以把直系子目录散布到一系列 MDS rank 上。
经典的应用案例就是 ``/home`` 目录：我们希望每个用户的家目录\
都被散布到整个 MDS 集群上。这样设置：

::

    setfattr -n ceph.dir.pin.distributed -v 1 /cephfs/home


**随机的临时挂单**: 这个策略表面它下面的\
任意子目录分支都可以被临时挂单出去。
这是通过扩展属性 ``ceph.dir.pin.random`` 设置的，
值是允许挂单的目录百分比。例如：

::

    setfattr -n ceph.dir.pin.random -v 0.5 /cephfs/tmp

这将使得任何被载入缓存的、或者在 ``/tmp`` 下创建的目录\
在 50% 的时间里都可以临时挂单。

建议你只设置很小的数值，像 ``.001`` 或 ``0.1%`` 。
子树太多的话会降低性能。正因为如此，
配置选项 ``mds_export_ephemeral_random_max`` 对\
这一比例的上限进行了规定（默认为 ``.01`` ）。
试图设置得高于此配置时 MDS 会返回 ``EINVAL`` 。

在 Octopus 版里，随机的和分布式的临时挂单策略默认都是关闭的。
此功能可以通过 
``mds_export_ephemeral_random`` 和 ``mds_export_ephemeral_distributed``
配置选项来启用。

临时挂单可以覆盖父级导出的 pin ，反之亦然。
施行哪条策略遵循的规则是最近的父级：
如果一个比较近的父级目录有一条策略冲突，就用这一条。例如：

::

    mkdir -p foo/bar1/baz foo/bar2
    setfattr -n ceph.dir.pin -v 0 foo
    setfattr -n ceph.dir.pin.distributed -v 1 foo/bar1

``foo/bar1/baz`` 这一目录就会被临时挂单，
因为 ``foo/bar1`` 策略覆盖了 ``foo`` 上的 export pin 。
``foo/bar2`` 会正常地遵守 ``foo`` 上的 pin 。

对于相反的情况：

::

    mkdir -p home/{patrick,john}
    setfattr -n ceph.dir.pin.distributed -v 1 home
    setfattr -n ceph.dir.pin -v 2 home/patrick

``home/patrick`` 目录及其子目录会被插入 rank 2,
因为它的 export pin 覆盖了 ``home`` 上的策略。
