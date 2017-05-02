.. _ceph-mgr administrator's guide:

ceph-mgr 管理员指南
===================

设置
----

给守护进程创建认证密钥： ::

    ceph auth get-or-create mgr.$name mon 'allow *'

把创建的密钥放入 ``mgr data`` 所指向的路径，对于名为 ceph 的集\
群、 mgr 的 $name 为 foo 的路径可能是 ``/var/lib/ceph/mgr/ceph-foo`` 。

启动 ceph-mgr 守护进程： ::

    ceph-mgr -i $name

通过 ``ceph status`` 的输出可检查 mgr 是否起来了，起来的话应该\
会包含一行 mgr 状态： ::

    mgr active: $name


高可用性
--------

通常来说，你应该在每台运行 ceph-mon 守护进程的主机上都配置一个
ceph-mgr ，以实现相同级别的可用性。

默认情况下，监视器会把任意一个最先启动的 ceph-mgr 例程当作活跃\
的，其它的作为备用。 ceph-mgr 守护进程无需形成法定人数。

如果活跃的守护进程在 ``mon mgr beacon grace`` （默认 30s ）这\
么长的时间内都没向监视器们发送信标，那它就会被备用顶替。

如果你想提前做故障切换，可以用 ``ceph mgr fail <mgr name>`` 把
ceph-mgr 明确地标记为已失效。


.. _Calling module commands:

调用模块命令
------------

Where a module implements command line hooks, using the Ceph CLI's
``tell`` command to call them like this::

    ceph tell mgr <command | help>

Note that it is not necessary to address a particular mgr instance,
simply ``mgr`` will pick the current active daemon.

Use the ``help`` command to get a list of available commands from all
modules.


配置选项
--------

OPTION(mgr_module_path, OPT_STR, CEPH_PKGLIBDIR "/mgr") // 从哪里载入 python 模块

``mgr module path``

:描述: 从这个路径载人模块
:类型: String
:默认值: ``"<library dir>/mgr"``

``mgr modules``

:描述: 要载入的 python 模块列表
:类型: String
:默认值: ``"rest"`` (Load the REST API module only)

``mgr data``

:描述: 从这个路径载人守护进程数据（如密钥环）
:类型: String
:默认值: ``"/var/lib/ceph/mgr/$cluster-$id"``

``mgr beacon period``

:描述: mgr 向监视器发送信标的时间间隔，单位为秒。
:类型: Integer
:默认值: ``5``

``mon mgr beacon grace``

:描述: 上一个信标收到后过多久没反应就当它失败了。
:类型: Integer
:默认值: ``30``

