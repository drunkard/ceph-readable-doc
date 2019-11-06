.. _cephfs-administration:

CephFS 管理命令
===============


.. Filesystems

文件系统管理
------------

这些命令适用于 Ceph 集群的 CephFS 文件系统。注意，默认情况下，\
只允许一个文件系统；执行 ``ceph fs flag set enable_multiple true``
后才允许创建多个文件系统。

::

    fs new <filesystem name> <metadata pool name> <data pool name>

This command creates a new file system. The file system name and metadata pool
name are self-explanatory. The specified data pool is the default data pool and
cannot be changed once set. Each file system has its own set of MDS daemons
assigned to ranks so ensure that you have sufficient standby daemons available
to accommodate the new file system.

::

    fs ls

List all file systems by name.

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


.. Settings

可配置选项
----------

::

    fs set <fs name> max_file_size <size in bytes>

CephFS 允许的最大文件尺寸是可以配置的，默认是 1TB 。如果你想在
CephFS 里存储大文件，也许得把这个限量设置得高些。它是个 64 位\
的字段。

把 ``max_file_size`` 设置为 0 并不意味着取消这个限量，而是阻止\
客户端创建空文件。


.. Maximum file sizes and performance

最大文件尺寸以及性能
--------------------

在追加到文件、或设置其尺寸时， CephFS 将确保不会超过最大文件尺\
寸限量；但不会影响（数据）是怎样存储的。

有用户创建了一个硕大的文件时（未必要写入什么数据），某些操作\
（像删除）会让 MDS 不得不做大量操作，去检查此文件的尺寸范围内\
是否有本应存在（据文件尺寸计算出的）、实际上却不存在的 RADOS \
对象。

``max_file_size`` 选项可防止用户创建巨型文件，如 EB 级的文件，\
导致在遇到类似查询状态或删除操作时产生无谓的 MDS 负载。


.. Taking the cluster down

关闭集群
--------

Taking a CephFS cluster down is done by setting the down flag:
 
:: 
 
    fs set <fs_name> down true
 
To bring the cluster back online:
 
:: 

    fs set <fs_name> down false

This will also restore the previous value of max_mds. MDS daemons are brought
down in a way such that journals are flushed to the metadata pool and all
client I/O is stopped.


.. Taking the cluster down rapidly for deletion or disaster recovery

快速关闭集群以进行反删除或灾难恢复
----------------------------------

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


.. Daemons

守护进程管理
------------

大多数可操纵 MDS 的命令都需要一个 ``<role>`` 参数，它必须是以\
下三种格式之一： ::

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


.. Minimum Client Version

客户端最低版本限制
------------------

It is sometimes desirable to set the minimum version of Ceph that a client must be
running to connect to a CephFS cluster. Older clients may sometimes still be
running with bugs that can cause locking issues between clients (due to
capability release). CephFS provides a mechanism to set the minimum
client version:

::

    fs set <fs name> min_compat_client <release>

For example, to only allow Nautilus clients, use:

::

    fs set cephfs min_compat_client nautilus

Clients running an older version will be automatically evicted.


.. Global settings

全局配置选项
------------

::

    fs flag set <flag name> <flag val> [<confirmation string>]

设置全局的 CephFS 标记（即不是特定于某个文件系统的）。当前，\
仅有的标记是 enable_multiple ，启用它就可以支持多个 CephFS
文件系统。

有些标志会强迫你用 ``--yes-i-really-mean-it`` 或者类似的语句（\
执行时会提示）来确认你的意图。运行这类命令时要三思而后行，它们\
通常用于提示非常危险的动作。


.. Advanced

高级选项
--------

以下这些命令在常规操作中用不到，在遇到异常时才需要。这些命令若\
使用不当会产生严重问题，甚至会导致文件系统无法访问。

::

    mds compat rm_compat

删除一个兼容性功能标记。

::

    mds compat rm_incompat

删除一个不兼容性功能标记。

::

    mds compat show

展示 MDS 的兼容性标记。

::

    mds rmfailed

从失效集合中删除一个 rank 。

::

    fs reset <file system name>

此命令可把文件系统状态（除名字和存储池以外的）重置为默认值。\
所有非 0 rank 都会保存在停止集里面。
