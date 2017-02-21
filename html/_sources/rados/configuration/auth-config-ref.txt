================
 Cephx 配置参考
================

（认证配置）

``cephx`` 协议已默认开启。加密认证要耗费一定计算资源，但通常很低。如果您的客户端和\
服务器网络环境相当安全，而且认证的负面效应更大，你可以关闭它，\ \
**通常不推荐您这么做**\ 。

.. note:: 如果禁用了认证，就会有篡改客户端/服务器消息这样的中间人攻击风险，这会导致\
   灾难性后果。

关于创建用户请参考\ `用户管理`_\ ；关于 Cephx 的体系结构请参考\ \
`体系结构——高可用性认证`_\ 。


部署场景
========

There are two main scenarios for deploying a Ceph cluster, which impact 
how you initially configure Cephx. Most first time Ceph users use 
``ceph-deploy`` to create a cluster (easiest). For clusters using
other deployment tools (e.g., Chef, Juju, Puppet, etc.), you will need
to use the manual procedures or configure your deployment tool to 
bootstrap your monitor(s).


ceph-deploy
-----------

When you deploy a cluster with ``ceph-deploy``, you do not have to bootstrap the
monitor manually or create the ``client.admin`` user or keyring. The steps you
execute in the `存储集群入门`_ will invoke ``ceph-deploy`` to do
that for you.

When you execute ``ceph-deploy new {initial-monitor(s)}``, Ceph will create a
monitor keyring for you (only used to bootstrap monitors), and it will generate
an  initial Ceph configuration file for you, which contains the following
authentication settings, indicating that Ceph enables authentication by
default::

	auth_cluster_required = cephx
	auth_service_required = cephx
	auth_client_required = cephx

When you execute ``ceph-deploy mon create-initial``, Ceph will bootstrap the
initial monitor(s), retrieve a ``ceph.client.admin.keyring`` file containing the
key for the  ``client.admin`` user. Additionally, it will also retrieve keyrings
that give ``ceph-deploy`` and ``ceph-disk`` utilities the ability to prepare and
activate OSDs and metadata servers.

When you execute ``ceph-deploy admin {node-name}`` (**note:** Ceph must be 
installed first), you are pushing a Ceph configuration file and the
``ceph.client.admin.keyring`` to the ``/etc/ceph``  directory of the node. You
will be able to execute Ceph administrative functions as ``root`` on the command 
line of that node.


手动部署
--------

When you deploy a cluster manually, you have to bootstrap the monitor manually
and create the ``client.admin`` user and keyring. To bootstrap monitors, follow
the steps in `监视器的自举引导`_. The steps for monitor bootstrapping are
the logical steps you must perform when using third party deployment tools like
Chef, Puppet,  Juju, etc.


启用和禁用 Cephx
================

Enabling Cephx requires that you have deployed keys for your monitors,
OSDs and metadata servers. If you are simply toggling Cephx on / off, 
you do not have to repeat the bootstrapping procedures.


启用 Cephx
----------

启用 ``cephx`` 后， Ceph 将在默认搜索路径（包括 ``/etc/ceph/ceph.$name.keyring`` \
）里查找密钥环。你可以在 `Ceph 配置`_\ 文件的 ``[global]`` 段里添加 ``keyring`` \
选项来修改，但不推荐。

在禁用了 ``cephx`` 的集群上执行下面的步骤来启用它，如果你（或者部署工具）已经生成了\
密钥，你可以跳过相关步骤。

#. 创建 ``client.admin`` 密钥，并为客户端保存此密钥的副本： ::

	ceph auth get-or-create client.admin mon 'allow *' mds 'allow *' osd 'allow *' -o /etc/ceph/ceph.client.admin.keyring

   **警告：** 此命令会覆盖任何存在的 ``/etc/ceph/client.admin.keyring`` 文件，如\
   果部署工具已经完成此步骤，千万别再执行此命令。多加小心！

#. 创建监视器集群所需的密钥环、并给它们生成密钥。 ::

	ceph-authtool --create-keyring /tmp/ceph.mon.keyring --gen-key -n mon. --cap mon 'allow *'

#. 把监视器密钥环复制到 ``ceph.mon.keyring`` 文件，再把此文件复制到各监视器的 
   ``mon data`` 目录下。比如要把它复制给名为 ``ceph`` 集群的 ``mon.a`` ，用此命令： ::

	cp /tmp/ceph.mon.keyring /var/lib/ceph/mon/ceph-a/keyring

#. 为每个 OSD 生成密钥， ``{$id}`` 是 OSD 编号： ::

	ceph auth get-or-create osd.{$id} mon 'allow rwx' osd 'allow *' -o /var/lib/ceph/osd/ceph-{$id}/keyring

#. 为每个 MDS 生成密钥， ``{$id}`` 是 MDS 的标识字母： ::

	ceph auth get-or-create mds.{$id} mon 'allow rwx' osd 'allow *' mds 'allow *' -o /var/lib/ceph/mds/ceph-{$id}/keyring

#. 把以下配置加入 `Ceph 配置`_\ 文件的 ``[global]`` 段下以启用 ``cephx`` 认证： ::

	auth cluster required = cephx
	auth service required = cephx
	auth client required = cephx

#. 启动或重启 Ceph 集群，详情见\ `操纵集群`_\ 。

要手动自启监视器，请参考\ `手动部署`_\ 。


禁用 Cephx
----------

下述步骤描述了如何禁用 Cephx 。如果你的集群环境相对安全，你可以减免认证耗费的计算资\
源，然而\ **我们不推荐**\ 。但是临时禁用认证会使安装、和/或排障更简单。

#. 把下列配置加入 `Ceph 配置`_\ 文件的 ``[global]`` 段下即可禁用 ``cephx`` 认证： ::

	auth cluster required = none
	auth service required = none
	auth client required = none

#. 启动或重启 Ceph 集群，具体参考\ `操纵集群`_\ 。


配置选项
========

启用事项
--------


``auth cluster required``

:描述: 如果启用了，集群守护进程（如 ``ceph-mon`` 、 ``ceph-osd`` 和 \
       ``ceph-mds`` ）间必须相互认证。可用选项有 ``cephx`` 或 ``none`` 。

:类型: String
:是否必需: No
:默认值: ``cephx``.


``auth service required``

:描述: 如果启用，客户端要访问 Ceph 服务的话，集群守护进程会要求它和集群认\
       证。可用选项为 ``cephx`` 或 ``none`` 。

:类型: String
:是否必需: No
:默认值: ``cephx``.


``auth client required``

:描述: 如果启用，客户端会要求 Ceph 集群和它认证。可用选项为 ``cephx`` 或 \
       ``none`` 。

:类型: String
:是否必需: No
:默认值: ``cephx``.


.. index:: keys; keyring

密钥
----

如果你的集群启用了认证， ``ceph`` 管理命令和客户端得有密钥才能访问集群。

给 ``ceph`` 管理命令和客户端提供密钥的最常用方法就是把密钥环放到 ``/etc/ceph`` ，\
通过 ``ceph-deploy`` 部署的 Cuttlefish 及更高版本，其文件名通常是 \
``ceph.client.admin.keyring`` （或 ``$cluster.client.admin.keyring`` ）。如果你\
的密钥环位于 ``/etc/ceph`` 下，就不需要在 Ceph 配置文件里指定 ``keyring`` 选项了。

我们建议把集群的密钥环复制到你执行管理命令的节点，它包含 ``client.admin`` 密钥。

你可以用 ``ceph-deploy admin`` 命令做此事，详情见\ '部署管理主机'_\ ，手动复制可执\
行此命令： ::

	sudo scp {user}@{ceph-cluster-host}:/etc/ceph/ceph.client.admin.keyring /etc/ceph/ceph.client.admin.keyring

.. tip:: 确保给客户端上的 ``ceph.keyring`` 设置合理的权限位（如 ``chmod 644`` ）。

你可以用 ``key`` 选项把密钥写在配置文件里（别这样），或者用 ``keyfile`` 选项指定个\
路径。


``keyring``

:描述: 密钥环文件的路径。
:类型: String
:是否必需: No
:默认值: ``/etc/ceph/$cluster.$name.keyring,/etc/ceph/$cluster.keyring,/etc/ceph/keyring,/etc/ceph/keyring.bin``


``keyfile``

:描述: 到密钥文件的路径，如一个只包含密钥的文件。
:类型: String
:是否必需: No
:默认值: None


``key``

:描述: 密钥（密钥文本），最好别这样做。
:类型: String
:是否必需: No
:默认值: None


守护进程密钥环
--------------

管理员用户们或部署工具（如 ``ceph-deploy`` ）生成守护进程密钥环与生成用户密钥环的\
方法一样。默认情况下，守护进程把密钥环保存在各自的数据目录下，默认密钥环位置、和守护\
进程发挥作用必需的能力展示如下：

``ceph-mon``

:位置: ``$mon_data/keyring``
:能力: ``mon 'allow *'``

``ceph-osd``

:位置: ``$osd_data/keyring``
:能力: ``mon 'allow profile osd' osd 'allow *'``

``ceph-mds``

:位置: ``$mds_data/keyring``
:能力: ``mds 'allow' mon 'allow profile mds' osd 'allow rwx'``

``radosgw``

:位置: ``$rgw_data/keyring``
:能力: ``mon 'allow rwx' osd 'allow rwx'``


.. note:: 监视器密钥环（即 ``mon.`` ）包含一个密钥，但没有能力，且不是集群 \
   ``auth`` 数据库的一部分。

守护进程数据目录位置默认格式如下： ::

	/var/lib/ceph/$type/$cluster-$id

例如， ``osd.12`` 的目录会是： ::

	/var/lib/ceph/osd/ceph-12

你可以覆盖这些位置，但不推荐。


.. index:: signatures

签名
----

在 Bobtail 及后续版本中， Ceph 会用开始认证时生成的会话密钥认证所有在线实体。然而 \
Argonaut 及之前版本不知道如何认证在线消息，为保持向后兼容性（如在同一个集群里运行 \
Bobtail 和 Argonaut ），消息签名默认是\ **关闭**\ 的。如果你只运行 Bobtail 和后续\
版本，可以让 Ceph 要求签名。

像 Ceph 认证的其他部分一样，客户端和集群间的消息签名也能做到细粒度控制；而且能启用\
或禁用 Ceph 守护进程间的签名。


``cephx require signatures``

:描述: 若设置为 ``true`` ， Ceph 集群会要求客户端签名所有消息，包括集群内\
       其他守护进程间的。

:类型: Boolean
:是否必需: No
:默认值: ``false``


``cephx cluster require signatures``

:描述: 若设置为 ``true`` ， Ceph 要求集群内所有守护进程签名相互之间的消息。
:类型: Boolean
:是否必需: No
:默认值: ``false``


``cephx service require signatures``

:描述: 若设置为 ``true`` ， Ceph 要求签名所有客户端和集群间的消息。
:类型: Boolean
:是否必需: No
:默认值: ``false``


``cephx sign messages``

:描述: 如果 Ceph 版本支持消息签名， Ceph 会签名所有消息以防欺骗。
:类型: Boolean
:默认值: ``true``


生存期
------

``auth service ticket ttl``

:描述: Ceph 存储集群发给客户端一个用于认证的票据时分配给这个票据的生存期。
:类型: Double
:默认值: ``60*60``


向后兼容性
==========

关于 Cuttlefish 及更低版本，参考 `Cephx`_ 。

在 Ceph 0.48 及更早版本，启用 ``cephx`` 认证后， Ceph 仅认证客户端和守护进程间的\
最初通讯，不会认证后续相互发送的消息，这导致了安全隐患； Bobtail 及后续版本会用认证\
后生成的会话密钥来认证所有消息。

我们确定了一个向后兼容性问题，在 Argonaut v0.48 （及之前版本）和 Bobtail（ 及后续\
版本）之间。测试发现，如果你混用 Argonaut （及更小版）和 Bobtail 的守护进程， \
Argonaut 的守护进程将对正在进行的消息不知所措，因为 Bobtail 进程坚持要求认证最初请\
求/响应之后的消息，导致二者无法交互。

我们已经提供了一种方法，解决了 Argonaut （及之前）和 Bobtail （及后续）系统间交互\
的潜在的问题。是这样解决的，默认情况下，较新系统不会再坚持要求较老系统的签名，只是简\
单地接收这些消息而不对其认证。这个默认行为使得两个不同版本可以交互，但\ \
**我们不推荐作为长期方案**\ 。允许较新进程不认证正在进行的消息会导致安全问题，因为\
攻击者如果能控制你的机器、或访问你的网络，就可以宣称不能签署消息，从而禁用会话安全。

.. note:: 即使你没有使用旧版的 Ceph ，在默认配置下，攻击者也可以强制一些未签署消息\
   被接受；虽然初始通讯认证通过了，但你失去了会话安全。

如果你确定不会使用旧版 Ceph 、或者新旧服务器不能交互无所谓，那就可以排除这个安全风\
险；如果你这样做了，任何支持会话认证、启用了 Cephx 的 Ceph 系统都会拒绝未签名的消\
息。要防止新服务器和旧服务器交互，在 `Ceph 配置`_\ 文件的 ``[global]`` 下添加下列\
这行，要加到启用 Cephx 之后。 ::

	cephx require signatures = true    ; everywhere possible

你也可以选择只对集群内部通讯要求签名，它和面向客户端的服务是分离的： ::

	cephx cluster require signatures = true    ; for cluster-internal communication
	cephx service require signatures = true    ; for client-facing service

客户端向集群要求签名的选项还没实现。

**我们推荐把所有进程迁移到较新版本，并启用前述选项选项**\ ，以增强认证安全性。

.. note:: Ceph 内核模块还不支持签名。


.. _存储集群入门: ../../../start/quick-ceph-deploy/
.. _监视器的自举引导: ../../../install/manual-deployment#monitor-bootstrapping
.. _操纵集群: ../../operations/operating
.. _手动部署: ../../../install/manual-deployment
.. _Cephx: http://ceph.com/docs/cuttlefish/rados/configuration/auth-config-ref/
.. _Ceph 配置: ../ceph-conf
.. _部署管理主机: ../../deployment/ceph-deploy-admin
.. _体系结构——高可用性认证: ../../../architecture#high-availability-authentication
.. _用户管理: ../../operations/user-management
