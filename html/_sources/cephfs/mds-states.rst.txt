MDS 的各种状态
==============
.. MDS States

在 CephFS 正常运行期间，元数据服务器 (MDS) 会经历几种状态。
例如，某些状态表示 MDS 正在从 MDS 先前实例的故障转移中恢复。
在此，我们将记录所有这些状态，并提供一个状态图来直观展示相互之间的转换。


状态描述
--------
.. State Descriptions

常见状态
~~~~~~~~
.. Common states

::

    up:active

这是 MDS 的正常运行状态。
它表明文件系统中的 MDS 及其 rank 是可用的。


::

    up:standby

MDS 可用于接管发生故障的 rank （另请参阅 :ref:`mds-standby` ）。
只要有可用的，监视器会自动将处于此状态的 MDS 分配给\
故障的 rank 。


::

    up:standby_replay

MDS 正在跟随另一个状态是 ``up:active`` 的 MDS 的日志。
如果活跃 MDS 出现故障，最好有一个处于重放模式的备用 MDS ，
因为这个 MDS 一直在重放实时日志，可以更快地接管。
备用重放（ standby replay ） MDS 的一个缺点是，
如果其他 MDS 出现故障，它们无法接管，只能接管它们跟随的 MDS 。


不太常见的或过渡状态
~~~~~~~~~~~~~~~~~~~~
.. Less common or transitory states

::

    up:boot

这个状态是在启动期间向 Ceph 监视器们广播。
这个状态永远不可见，因为监视器会立即将 MDS 分配给可用 rank
或命令 MDS 作为热备运行。
在此记录该状态只是为了完整性。


::

    up:creating

MDS 正在创建一个新 rank （也许是 rank 0 ），
需要构建一些元数据（比如日志）、然后加入 MDS 集群。


::

    up:starting

MDS 正在重启一个停掉的 rank 。
它会打开相关的 per-rank 元数据，并加入 MDS 集群。


::

    up:stopping

当 rank 停止时，监视器会命令活跃的 MDS 进入 ``up:stopping`` 状态。
在此状态下，这个 MDS 不再接受新的客户端连接，
将所有子树迁移到文件系统中的其他 rank ，
刷回它的元数据日志，并且，如果这是最后一个 rank (0)，
那就驱逐所有客户端、并关停（另请参阅 :ref:`cephfs-administration` ）。


::

    up:replay

MDS 接管发生故障的 rank 。
这种状态表示 MDS 正在恢复日志和其他元数据。


::

    up:resolve

如果 Ceph 文件系统有多个 rank （包括这一个），就是说，
它不是只有单个活跃 MDS 的集群，那么 MDS 会从 ``up:replay`` 进入此状态。
这个 MDS 正在解决所有尚未提交的 MDS 之间的操作。
文件系统中的所有 rank 都必须处于此状态或更靠后的状态才能取得进展，
即任何 rank 都不能出现故障/损坏或 ``up:replay`` 。


::

    up:reconnect

MDS 从 ``up:replay`` 或 ``up:resolve`` 进入此状态。
此状态用于邀请客户端重新连接过来。
所有与本 rank 有过会话的客户端都必须在这段时间内重新连接，
可通过 ``mds_reconnect_timeout`` 进行配置。


::

    up:rejoin

MDS 从 ``up:reconnect`` 进入此状态。在此状态下，MDS 将重新加入 MDS 集群缓存。
特别是，有关元数据的所有 MDS 间的锁都会重新建立。

如果没有已知的客户端请求需要重放，MDS 将直接从该状态进入 ``up:active`` 。


::

    up:clientreplay

MDS 可从 ``up:rejoin`` 进入此状态。
MDS 正在重放所有已经回复但尚未持久化（未记入日志）的客户端请求。
客户端们在 ``up:reconnect`` 期间重新发送这些请求，
这些请求将被再次重放。重放完成后， MDS 进入 ``up:active`` 。


失败状态
~~~~~~~~
.. Failed states

::

    down:failed

实际上，没有任何 MDS 拥有这种状态。相反，它被应用于文件系统中的 rank 。例如：

::

    $ ceph fs dump
    ...
    max_mds 1
    in      0
    up      {}
    failed  0
    ...

rank 0 是故障集的一部分，正等待备用 MDS 接管。如果这种状态持续存在，则表明没有\
找到合适的 MDS 守护进程分配给该 rank 。其中的原因可能是没有足够的备用守护进程，
或者与所有备用守护进程的 compat 不兼容（另请参阅 :ref:`upgrade-mds-cluster` ）。


::

    down:damaged

没有任何 MDS 事实上拥有这种状态。相反，它被应用于文件系统中的 rank 。例如：


::

    $ ceph fs dump
    ...
    max_mds 1
    in      0
    up      {}
    failed  
    damaged 0
    ...

rank 0 已损坏（另请参阅 :ref:`cephfs-disaster-recovery` ），并被归入损坏集。
以 rank 0 运行的 MDS 发现元数据损坏，无法自动恢复。需要操作员干预。


::

    down:stopped

没有任何 MDS 事实上拥有此状态。相反，它被应用于文件系统中的 rank 。例如：


::

    $ ceph fs dump
    ...
    max_mds 1
    in      0
    up      {}
    failed  
    damaged 
    stopped 1
    ...

通过减少 ``max_mds`` ，此 rank 已被停止（另请参阅 :ref:`cephfs-multimds` ）。

状态图
------
.. State Diagram

这张状态图展示了 MDS/rank 可能的状态转变，图例如下：

颜色
~~~~

- 绿色: MDS 是活跃的。
- 橙色: MDS 处于尝试活跃状态的瞬间状态。
- 红色: MDS 在指出此状态导致 rank 被标记为失败。
- 紫色: MDS 和 rank 正在停止。
- 黑色: MDS 在指出此状态导致 rank 被标记为损坏。

形状
~~~~

- 圆圈: MDS 持有这个状态。
- 六边形: 没有 MDS 能持有此状态（它是用于 rank 的）。

线
~~

- 双线形状表示 rank 处于 "in" 状态。

.. graphviz:: mds-state-diagram.dot
