.. _mgr-administrator-guide:

ceph-mgr 管理员指南
===================


.. Manual setup

手动设置
--------

平时，你可以用 ceph-ansible 之类的工具配置 ceph-mgr 守护进程。\
下面说明了如何手动配置好一个 ceph-mgr 守护进程。

首先，给守护进程创建认证密钥： ::

    ceph auth get-or-create mgr.$name mon 'allow profile mgr' osd 'allow *' mds 'allow *'

把创建的密钥放入 ``mgr data`` 所指向的路径，对于名为 ceph 的集\
群、 mgr 的 $name 为 foo 的路径可能是
``/var/lib/ceph/mgr/ceph-foo`` 。

启动 ceph-mgr 守护进程： ::

    ceph-mgr -i $name

通过 ``ceph status`` 的输出可检查 mgr 是否起来了，起来的话应该\
会包含一行 mgr 状态： ::

    mgr active: $name


.. Client authentication

客户端认证
----------

管理器是一种新的守护进程，需要新的 CephX 能力。如果你是从旧版
Ceph 升级集群、或者使用默认的安装/部署工具，那么管理客户端应该\
可以自动获取到这个能力。如果你用了非标准工具，在调用某些 ceph
集群命令时可能会遇到 EACCES 错误。要修正此问题，需\
`更改用户能力`_\ ，在客户端的 cephx 能力里加上 ``mgr allow \*``
声明。


.. High availability

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


.. Calling module commands

调用模块命令
------------

对于实现了命令行钩子的模块，其实现的命令可以像一般的 Ceph 命令\
那样调用： ::

    ceph <command | help>

如果你想查看管理器可以处理的命令列表（通常 ``ceph help`` 会显\
示所有 mon 和 mgr 命令），你可以直接向管理器守护进程发命令： ::

    ceph tell mgr help

注意，没必要盯住具体的 mgr 例程，用 ``mgr`` 会自动选当前活跃的\
守护进程。


.. Configuration

配置选项
--------

``mgr module path``

:描述: 从这个路径载人模块
:类型: String
:默认值: ``"<library dir>/mgr"``


``mgr data``

:描述: 从这个路径载人守护进程数据（如密钥环）
:类型: String
:默认值: ``"/var/lib/ceph/mgr/$cluster-$id"``


``mgr tick period``

:描述: mgr 向监视器发送信标、以及其它周期性检查的时间间隔，单\
       位为秒。
:类型: Integer
:默认值: ``5``


``mon mgr beacon grace``

:描述: 上一个信标收到后过多久没反应就当它失败了。
:类型: Integer
:默认值: ``30``


.. _更改用户能力: ../../rados/operations/user-management/#modify-user-capabilities
