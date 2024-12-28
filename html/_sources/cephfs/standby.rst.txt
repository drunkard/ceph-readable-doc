.. _mds-standby:

术语
----

一个 Ceph 集群内可以没有、或者有多个 CephFS *文件系统* 。\
每一个 CephFS 文件系统都有一个人类可读的名字（用 ``fs new`` 创建时设置的）、\
和一个整数 ID ，这个 ID 称为文件系统集群 ID 或 *FSCID* 。

每个 CephFS 都有几个 *rank* ，编号从 0 起。\
默认情况下，每个文件系统有一个 rank 。一个 rank 可看作是一个元数据分片。
rank 的管理在 :doc:`/cephfs/multimds` 里详述。

CephFS 的每个 ``ceph-mds`` 进程刚启动时都没有 rank ，\
它可以由集群的监视器们分配。一个守护进程一次只能占据一个 rank ，\
只有在 ``ceph-mds`` 守护进程停止的时候才会释放 rank 。

如果某个 rank 没有依附任何一个守护进程，那这个 rank 就*失效了\
（ ``failed`` ）*\ 。分配给守护进程的 rank 才被当作 *正常的（ ``up`` ）* 。

首次配置 ``ceph-mds`` 守护进程时，管理员需分配固定的\ *名字*\ ，\
一般都会用守护进程所在的主机名作为其守护进程名字。

把一个 MDS 的 ``mds_join_fs`` 配置选项设置为指定文件系统的名字，
可以将 ``ceph-mds`` 守护进程分配给这个文件系统。

``ceph-mds`` 守护进程每次启动时，还会被分配一个整数 ``GID`` ，\
它对于当前这个守护进程的进程生命周期来说它是唯一的。换句话说，\
当一个 ``ceph-mds`` 守护进程重启后，它会作为新进程、\
而且会被分配一个 *新的* 、不同于之前进程的 ``GID`` 。

.. todo:: 译者注： rank 翻译为“席位”、“座席”？它们共同处理元数据，
   且动态分配，类似客服中心的座席。


MDS 守护进程的引用
------------------
.. Referring to MDS daemons

与 ``ceph-mds`` 守护进程（MDS）相关的大多数管理命令都接受\
一个灵活的参数格式，可以指定 ``rank`` 、 ``GID`` 或 ``name`` 。

使用 ``rank`` 时，它前面可以加文件系统的 ``name`` 或 ``GID`` ，\
也可以不加。如果某个守护进程是灾备的（即当前还没给它分配 ``rank`` ），\
那就只能通过它的 ``GID`` 或 ``name`` 引用。

例如，假设我们有一个 MDS 守护进程，名为 myhost ，其 ``GID`` 为 5446 ，
分配的 ``rank`` 是 0 ，它位于名为 myfs 的文件系统内，
文件系统的 ``FSCID`` 是 3 ，那么，下面的几种形式都适用于 ``fail`` 命令：

::

    ceph mds fail 5446     # GID
    ceph mds fail myhost   # Daemon name
    ceph mds fail 0        # Unqualified rank
    ceph mds fail 3:0      # FSCID and rank
    ceph mds fail myfs:0   # Filesystem name and rank


故障切换的管理
--------------
.. Managing failover

如果一个 MDS 守护进程不再与监视器通讯，监视器把它标记为 *laggy*
（滞后）状态前会等待 ``mds_beacon_grace`` 秒（默认是 15 秒）。\
如果有可用灾备，监视器会立即替换处于 laggy 状态的守护进程。

为安全起见，每个文件系统都要指定一些灾备守护进程，灾备数量包括\
处于 ``standby-replay`` 热备状态、等着接替失效 ``rank`` 的。\
记着， ``standby-replay`` 热备守护进程不会被重分配去接管另一个 ``rank`` 、\
或者另一个 CephFS 文件系统内的失效守护进程）。\
不在 ``replay`` 重放状态的灾备守护进程们会被所有文件系统算作自己的灾备。\
每个文件系统都可以单独配置期望的灾备守护进程数量，用这个：

::

    ceph fs set <fs name> standby_count_wanted <count>

``count`` 设置为 0 表示禁用健康检查。


.. _mds-standby-replay:

热备（ standby-replay ）的配置
------------------------------
.. Configuring standby-replay

每个 CephFS 文件系统都能做配置，增加几个热备（ ``standby-replay`` ）守护进程。
这些备用守护进程会跟踪活动 MDS 的元数据日志，
以便在活跃 MDS 不可用时减少故障切换时间。
每个活跃 MDS 只能有一个 ``standby-replay`` 守护进程跟着它。

在文件系统上配置 ``standby-replay`` 的方法如下：

::

    ceph fs set <fs name> allow_standby_replay <bool>

设置以后，监视器将分配可用的备用守护进程，让它们跟随此文件系统中的活跃 MDS 。

一旦 MDS 进入 ``standby-replay`` 状态，它就只能用作跟随着的 ``rank`` 的备机。
如果另一个 ``rank`` 发生故障，即使没有其他备机，也不会用这个
``standby-replay`` 守护进程作替代品。因此，如果启用 ``standby-replay`` ，
建议\ *每个*\ 活跃 MDS 都应该配备一个 ``standby-replay`` 守护进程。


.. _mds-join-fs:

配置 MDS 与文件系统的亲和性
---------------------------
.. Configuring MDS file system affinity

您可能会选择将 MDS 焊死到指定的文件系统上。或者，让 MDS 在较好的硬件上运行，
而不是在普通的或还有空余资源的系统上运行。要配置出这种优先级，
CephFS 为 MDS 提供了一个名为 ``mds_join_fs`` 的配置选项，
用于实现这种亲和性。

当 MDS 守护进程做故障切换时，集群的监视器会优先选择 ``mds_join_fs`` 值\
等于故障 ``rank`` 对应文件系统名字 ``name`` 的备用守护进程。
如果不存在 ``mds_join_fs`` 等于文件系统名的备机，
它将选择一个不合格的备用系统（没有设置 ``mds_join_fs`` ）来替代。
在万不得已的情况下，系统会选择另一个文件系统的备机，
但这种行为可以禁用：

::

    ceph fs set <fs name> refuse_standby_for_another_fs true

注意，配置 MDS 的文件系统亲和性不会改变备机选择行为，
总是先选择 ``standby-replay`` 守护进程、再选其他备机。

此外，即使在稳定状态下，监视器也会定期检查 CephFS 文件系统，
以检查是否有亲和性更强的备机来替换亲和性较低的 MDS 。
这个过程也适用于 ``standby-replay`` 守护进程：
如果常规备机的亲和性比 ``standby-replay`` MDS 更强，
它就会取代 ``standby-replay`` MDS 。

例如，这是个稳定、健康的文件系统：

::

    $ ceph fs dump
    dumped fsmap epoch 399
    ...
    Filesystem 'cephfs' (27)
    ...
    e399
    max_mds 1
    in      0
    up      {0=20384}
    failed
    damaged
    stopped
    ...
    [mds.a{0:20384} state up:active seq 239 addr [v2:127.0.0.1:6854/966242805,v1:127.0.0.1:6855/966242805]]

    Standby daemons:

    [mds.b{-1:10420} state up:standby seq 2 addr [v2:127.0.0.1:6856/2745199145,v1:127.0.0.1:6857/2745199145]]


给备机配置 ``mds_join_fs`` ，展现自己的偏好： ::

    $ ceph config set mds.b mds_join_fs cephfs

发生故障，自动切换后： ::

    $ ceph fs dump
    dumped fsmap epoch 405
    e405
    ...
    Filesystem 'cephfs' (27)
    ...
    max_mds 1
    in      0
    up      {0=10420}
    failed
    damaged
    stopped
    ...
    [mds.b{0:10420} state up:active seq 274 join_fscid=27 addr [v2:127.0.0.1:6856/2745199145,v1:127.0.0.1:6857/2745199145]]

    Standby daemons:

    [mds.a{-1:10720} state up:standby seq 2 addr [v2:127.0.0.1:6854/1340357658,v1:127.0.0.1:6855/1340357658]]

注意上例中 ``mds.b`` 现在是 ``join_fscid=27`` 。
在此输出中， ``mds_join_fs`` 中的文件系统名称已经改成了文件系统标识符 (27)。
如果以相同的名字重新创建文件系统，
备机将按照预期跟随新文件系统。

最后，如果文件系统降级或副本数降低，
故障转移不会影响 ``mds_join_fs`` 的施行。
