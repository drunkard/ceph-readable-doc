.. _cephfs-administration:

CephFS 管理命令
===============

文件系统管理
------------
.. File Systems

.. note:: The names of the file systems, metadata pools, and data pools can
          only have characters in the set [a-zA-Z0-9\_-.].

这些命令适用于 Ceph 集群的 CephFS 文件系统。注意，默认情况下，\
只允许一个文件系统；执行 ``ceph fs flag set enable_multiple true``
后才允许创建多个文件系统。

::

    fs new <file system name> <metadata pool name> <data pool name>

This command creates a new file system. The file system name and metadata pool
name are self-explanatory. The specified data pool is the default data pool and
cannot be changed once set. Each file system has its own set of MDS daemons
assigned to ranks so ensure that you have sufficient standby daemons available
to accommodate the new file system.

::

    fs ls

List all file systems by name.

::

    fs lsflags <file system name>

List all the flags set on a file system.

::

    fs dump [epoch]

This dumps the FSMap at the given epoch (default: current) which includes all
file system settings, MDS daemons and the ranks they hold, and the list of
standby MDS daemons.

::

    fs rm <file system name> [--yes-i-really-mean-it]

Destroy a CephFS file system. This wipes information about the state of the
file system from the FSMap. The metadata pool and data pools are untouched and
must be destroyed separately.

::

    fs get <file system name>

Get information about the named file system, including settings and ranks. This
is a subset of the same information from the ``fs dump`` command.

::

    fs set <file system name> <var> <val>

Change a setting on a file system. These settings are specific to the named
file system and do not affect other file systems.

::

    fs add_data_pool <file system name> <pool name/id>

Add a data pool to the file system. This pool can be used for file layouts
as an alternate location to store file data.

::

    fs rm_data_pool <file system name> <pool name/id>

This command removes the specified pool from the list of data pools for the
file system.  If any files have layouts for the removed data pool, the file
data will become unavailable. The default data pool (when creating the file
system) cannot be removed.

::

    fs rename <file system name> <new file system name> [--yes-i-really-mean-it]

Rename a Ceph file system. This also changes the application tags on the data
pools and metadata pool of the file system to the new file system name.
The CephX IDs authorized to the old file system name need to be reauthorized
to the new name. Any on-going operations of the clients using these IDs may be
disrupted. Mirroring is expected to be disabled on the file system.


可配置选项
----------
.. Settings

::

    fs set <fs name> max_file_size <size in bytes>

CephFS 允许的最大文件尺寸是可以配置的，默认是 1TB 。如果你想在
CephFS 里存储大文件，也许得把这个限量设置得高些。它是个 64 位\
的字段。

把 ``max_file_size`` 设置为 0 并不意味着取消这个限量，而是阻止\
客户端创建空文件。


最大文件尺寸以及性能
--------------------
.. Maximum file sizes and performance

在追加到文件、或设置其尺寸时， CephFS 将确保不会超过最大文件\
尺寸限量；但不会影响（数据）是怎样存储的。

有用户创建了一个硕大的文件时（未必要写入什么数据），某些操作\
（像删除）会让 MDS 不得不做大量操作，去检查此文件的尺寸范围内\
是否有本应存在（据文件尺寸计算出的）、实际上却不存在的 RADOS \
对象。

``max_file_size`` 选项可防止用户创建巨型文件，如 EB 级的文件，\
导致在遇到类似查询状态或删除操作时产生无谓的 MDS 负载。


关闭集群
--------
.. Taking the cluster down

Taking a CephFS cluster down is done by setting the down flag:

::

    fs set <fs_name> down true

To bring the cluster back online:

::

    fs set <fs_name> down false

This will also restore the previous value of max_mds. MDS daemons are brought
down in a way such that journals are flushed to the metadata pool and all
client I/O is stopped.


快速关闭集群以进行反删除或灾难恢复
----------------------------------
.. Taking the cluster down rapidly for deletion or disaster recovery

To allow rapidly deleting a file system (for testing) or to quickly bring the
file system and MDS daemons down, use the ``fs fail`` command:

::

    fs fail <fs_name>

This command sets a file system flag to prevent standbys from
activating on the file system (the ``joinable`` flag).

This process can also be done manually by doing the following:

::

    fs set <fs_name> joinable false

Then the operator can fail all of the ranks which causes the MDS daemons to
respawn as standbys. The file system will be left in a degraded state.

::

    # For all ranks, 0-N:
    mds fail <fs_name>:<n>

Once all ranks are inactive, the file system may also be deleted or left in
this state for other purposes (perhaps disaster recovery).

To bring the cluster back up, simply set the joinable flag:

::

    fs set <fs_name> joinable true


守护进程管理
------------
.. Daemons

大多数可操纵 MDS 的命令都需要一个 ``<role>`` 参数，它必须是以\
下三种格式之一：

::

    <fs_name>:<rank>
    <fs_id>:<rank>
    <rank>

可操纵 MDS 守护进程的命令：

::

    mds fail <gid/name/role>

把一个 MDS 守护进程标记为已失效。假如一个 MDS 守护进程在
``mds_beacon_grace`` 秒内都没向监视器发送一条消息，这个操作就\
等价于集群自己的操作。如果此守护进程之前是活跃的，而且有可用的\
备机，用命令 ``mds fail`` 将迫使业务转移到备机。

如果此 MDS 守护进程事实上仍在运行，那么执行 ``mds fail`` 将\
使之重启；如果它之前是活跃的、并且还有可用的备机，那么这个\
“已失效”的守护进程回来后将作为备机。


::

    tell mds.<daemon name> command ...

向 MDS 守护进程发出一个命令，指定 ``mds.*`` 可向所有守护进程\
发送命令。用 ``ceph tell mds.* help`` 命令获取所有可用命令。


::

    mds metadata <gid/name/role>

获取指定 MDS （监视器知道它）的元数据。


::

    mds repaired <role>

把文件系统 rank 标记为已修复。这里不像名字说明的那样，这个命令\
不会更改 MDS ，它操纵的是先前被标记为已损坏的文件系统 rank 。


Required Client Features
------------------------

It is sometimes desirable to set features that clients must support to talk to
CephFS. Clients without those features may disrupt other clients or behave in
surprising ways. Or, you may want to require newer features to prevent older
and possibly buggy clients from connecting.

Commands to manipulate required client features of a file system:

::

    fs required_client_features <fs name> add reply_encoding
    fs required_client_features <fs name> rm reply_encoding

To list all CephFS features

::

    fs feature ls

Clients that are missing newly added features will be evicted automatically.

Here are the current CephFS features and first release they came out:

+------------------+--------------+-----------------+
| Feature          | Ceph release | Upstream Kernel |
+==================+==============+=================+
| jewel            | jewel        | 4.5             |
+------------------+--------------+-----------------+
| kraken           | kraken       | 4.13            |
+------------------+--------------+-----------------+
| luminous         | luminous     | 4.13            |
+------------------+--------------+-----------------+
| mimic            | mimic        | 4.19            |
+------------------+--------------+-----------------+
| reply_encoding   | nautilus     | 5.1             |
+------------------+--------------+-----------------+
| reclaim_client   | nautilus     | N/A             |
+------------------+--------------+-----------------+
| lazy_caps_wanted | nautilus     | 5.1             |
+------------------+--------------+-----------------+
| multi_reconnect  | nautilus     | 5.1             |
+------------------+--------------+-----------------+
| deleg_ino        | octopus      | 5.6             |
+------------------+--------------+-----------------+
| metric_collect   | pacific      | N/A             |
+------------------+--------------+-----------------+
| alternate_name   | pacific      | PLANNED         |
+------------------+--------------+-----------------+

CephFS Feature Descriptions


::

    reply_encoding

MDS encodes request reply in extensible format if client supports this feature.


::

    reclaim_client

MDS allows new client to reclaim another (dead) client's states. This feature
is used by NFS-Ganesha.


::

    lazy_caps_wanted

When a stale client resumes, if the client supports this feature, mds only needs
to re-issue caps that are explictly wanted.


::

    multi_reconnect

When mds failover, client sends reconnect messages to mds, to reestablish cache
states. If MDS supports this feature, client can split large reconnect message
into multiple ones.


::

    deleg_ino

MDS delegate inode numbers to client if client supports this feature. Having
delegated inode numbers is a prerequisite for client to do async file creation.


::

    metric_collect

Clients can send performance metric to MDS if MDS support this feature.

::

    alternate_name

Clients can set and understand "alternate names" for directory entries. This is
to be used for encrypted file name support.


全局配置选项
------------
.. Global settings

::

    fs flag set <flag name> <flag val> [<confirmation string>]

设置全局的 CephFS 标记（即不是特定于某个文件系统的）。当前，\
仅有的标记是 enable_multiple ，启用它就可以支持多个 CephFS
文件系统。

有些标志会强迫你用 ``--yes-i-really-mean-it`` 或者类似的语句（\
执行时会提示）来确认你的意图。运行这类命令时要三思而后行，它们\
通常用于提示非常危险的动作。


.. _advanced-cephfs-admin-settings:

高级选项
--------
.. Advanced

以下这些命令在常规操作中用不到，在遇到异常时才需要。这些命令若\
使用不当会产生严重问题，甚至会导致文件系统无法访问。

::

    mds rmfailed

从失效集合中删除一个 rank 。

::

    fs reset <file system name>

此命令可把文件系统状态（除名字和存储池以外的）重置为默认值。\
所有非 0 rank 都会保存在停止集里面。

::

    fs new <file system name> <metadata pool name> <data pool name> --fscid <fscid> --force

This command creates a file system with a specific **fscid** (file system cluster ID).
You may want to do this when an application expects the file system's ID to be
stable after it has been recovered, e.g., after monitor databases are lost and
rebuilt. Consequently, file system IDs don't always keep increasing with newer
file systems.
