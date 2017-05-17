.. _CephFS Administrative commands:

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


.. Daemons

守护进程管理
------------

以下命令适用于指定的 mds 守护进程或 rank 。

::

    mds fail <gid/name/role

下面的命令用于停用一个 MDS ，让它把所有日志刷回后端的 RADOS 对\
象、并关闭所有打开的客户端会话。 MDS 的停用功能主要是用于减少\
活跃 MDS （ max_mds ）后、拆除 rank 。

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

