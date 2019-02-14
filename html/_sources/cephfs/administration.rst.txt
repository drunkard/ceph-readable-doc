.. _CephFS Administrative commands:

CephFS 管理命令
===============



.. Filesystems

文件系统管理
------------

这些命令适用于 Ceph 集群的 CephFS 文件系统。注意，默认情况下，\
只允许一个文件系统；执行
``ceph fs flag set enable_multiple true`` 后才允许创建多个文件\
系统。

::

    fs new <filesystem name> <metadata pool name> <data pool name>

::

    fs ls

::

    fs rm <filesystem name> [--yes-i-really-mean-it]

::

    fs reset <filesystem name>

::

    fs get <filesystem name>

::

    fs set <filesystem name> <var> <val>

::

    fs add_data_pool <filesystem name> <pool name/id>

::

    fs rm_data_pool <filesystem name> <pool name/id>



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



.. Daemons

守护进程管理
------------

以下命令适用于指定的 mds 守护进程或 rank 。

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

    mds deactivate <role>

::

    tell mds.<daemon name>

::

    mds metadata <gid/name/role>

::

    mds repaired <role>


.. Global settings

全局配置选项
------------

::

    fs dump

::

    fs flag set <flag name> <flag val> [<confirmation string>]

<flag name> 必须是 ['enable_multiple'] 之一

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

::

    mds compat rm_incompat

::

    mds compat show

::

    mds getmap

::

    mds set_state

::

    mds rmfailed


旧命令
------

::

    mds stat
    mds dump  # replaced by "fs get"
    mds stop  # replaced by "mds deactivate"
    mds set_max_mds  # replaced by "fs set max_mds"
    mds set  # replaced by "fs set"
    mds cluster_down  # replaced by "fs set cluster_down"
    mds cluster_up  # replaced by "fs set cluster_up"
    mds newfs  # replaced by "fs new"
    mds add_data_pool  # replaced by "fs add_data_pool"
    mds remove_data_pool # replaced by "fs remove_data_pool"

