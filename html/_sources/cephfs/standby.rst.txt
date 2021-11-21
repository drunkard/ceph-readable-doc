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

A ``ceph-mds`` daemon may be assigned to a specific file system by
setting its ``mds_join_fs`` configuration option to the file system's
``name``.

``ceph-mds`` 守护进程每次启动时，还会被分配一个整数 ``GID`` ，\
它对于当前这个守护进程的进程生命周期来说它是唯一的。换句话说，\
当一个 ``ceph-mds`` 守护进程重启后，它会作为新进程、\
而且会被分配一个 *新的* 、不同于之前进程的 ``GID`` 。

.. todo:: 译者注： rank 翻译为“席位”、“座席”？它们共同处理元数\
   据，且动态分配，类似客服中心的座席。

MDS 守护进程的引用
------------------
.. Referring to MDS daemons

与 ``ceph-mds`` 守护进程（MDS）相关的大多数管理命令都接受\
一个灵活的参数格式，可以指定 ``rank`` 、 ``GID`` 或 ``name`` 。

使用 ``rank`` 时，它前面可以加文件系统的 ``name`` 或 ``GID`` ，\
也可以不加。如果某个守护进程是灾备的\
（即当前还没给它分配 ``rank`` ），\
那就只能通过它的 ``GID`` 或 ``name`` 引用。

例如，假设我们有一个 MDS 守护进程，名为 myhost ，\
其 ``GID`` 为 5446 ，分配的 ``rank`` 是 0 ，\
它位于名为 myfs 的文件系统内，文件系统的 ``FSCID`` 是 3 ，\
那么，下面的几种形式都适用于 ``fail`` 命令：

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

热备的配置
----------
.. Configuring standby-replay

Each CephFS file system may be configured to add ``standby-replay`` daemons.
These standby daemons follow the active MDS's metadata journal in order to
reduce failover time in the event that the active MDS becomes unavailable. Each
active MDS may have only one ``standby-replay`` daemon following it.

Configuration of ``standby-replay`` on a file system is done using the below:

::

    ceph fs set <fs name> allow_standby_replay <bool>

Once set, the monitors will assign available standby daemons to follow the
active MDSs in that file system.

Once an MDS has entered the ``standby-replay`` state, it will only be used as a
standby for the ``rank`` that it is following. If another ``rank`` fails, this
``standby-replay`` daemon will not be used as a replacement, even if no other
standbys are available. For this reason, it is advised that if ``standby-replay``
is used then *every* active MDS should have a ``standby-replay`` daemon.

.. _mds-join-fs:

配置 MDS 与文件系统的亲和性
---------------------------
.. Configuring MDS file system affinity

You might elect to dedicate an MDS to a particular file system. Or, perhaps you
have MDSs that run on better hardware that should be preferred over a last-resort
standby on modest or over-provisioned systems. To configure this preference,
CephFS provides a configuration option for MDS called ``mds_join_fs`` which
enforces this affinity.

When failing over MDS daemons, a cluster's monitors will prefer standby daemons with
``mds_join_fs`` equal to the file system ``name`` with the failed ``rank``.  If no
standby exists with ``mds_join_fs`` equal to the file system ``name``, it will
choose an unqualified standby (no setting for ``mds_join_fs``) for the replacement,
or any other available standby, as a last resort. Note, this does not change the
behavior that ``standby-replay`` daemons are always selected before
other standbys.

Even further, the monitors will regularly examine the CephFS file systems even when
stable to check if a standby with stronger affinity is available to replace an
MDS with lower affinity. This process is also done for ``standby-replay`` daemons:
if a regular standby has stronger affinity than the ``standby-replay`` MDS, it will
replace the standby-replay MDS.

For example, given this stable and healthy file system:

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


You may set ``mds_join_fs`` on the standby to enforce your preference: ::

    $ ceph config set mds.b mds_join_fs cephfs

after automatic failover: ::

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

Note in the above example that ``mds.b`` now has ``join_fscid=27``. In this
output, the file system name from ``mds_join_fs`` is changed to the file system
identifier (27). If the file system is recreated with the same name, the
standby will follow the new file system as expected.

Finally, if the file system is degraded or undersized, no failover will occur
to enforce ``mds_join_fs``.
