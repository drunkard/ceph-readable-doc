.. _rados-cephx-config-ref:

================
 Cephx 配置参考
================

CephX 协议会默认开启。\
CephX 的加密认证要耗费一定计算资源，但通常很低。\
如果您的客户端和服务器网络环境相当安全，\
而且认证的负面效应更大，你可以关闭它。
**通常不建议您禁用认证**\ 。

.. note:: 如果禁用了认证，就会有篡改客户端/服务器消息\
   这样的中间人攻击风险，这会导致灾难性后果。

关于创建用户请参考\ `用户管理`_\ ；\
关于 Cephx 的体系结构请参考\
`体系结构——高可用性认证`_\ 。


部署场景
========
.. Deployment Scenarios

部署一套 Ceph 集群主要有两种情形，影响你首次配置 Cephx 的方式。
大多数 Ceph 新手都用 ``cephadm`` 创建集群（最简单）。
对于用其它部署工具（比如 Chef 、 Juju 、 Puppet 等等）的集群，
你就得执行手工步骤、或者配置部署工具，
让它们来自举引导你的监视器。


手动部署
--------
.. Manual Deployment

如果你手动部署集群，你就得手动自举引导监视器、并创建 ``client.admin`` 用户\
及其密钥环。要想自举引导监视器，按照 `监视器的自举引导`_ 里的步骤。
那些自举引导监视器的步骤是你必须执行的逻辑步骤，使用第三方部署工具
（如 Chef 、 Puppet 、 Juju 等等）时也是这些步骤。


启用和禁用 Cephx
================
.. Enabling/Disabling Cephx

启用 Cephx 需要你为监视器、 OSD 和元数据服务器部署密钥。
如果你只是简单地打开、关闭 Cephx ，那就没必要重复那些自举引导步骤。


启用 Cephx
----------
.. Enabling Cephx

启用 ``cephx`` 后， Ceph 将在默认搜索路径（包括
``/etc/ceph/ceph.$name.keyring`` ）里查找密钥环。你可以在
`Ceph 配置`_\ 文件的 ``[global]`` 段里添加 ``keyring`` 选项来\
修改，但不推荐。

在禁用了 CephX 的集群上执行下面的步骤来启用它，如果你
（或者部署工具）已经生成了密钥，你可以跳过相关步骤。

#. 创建 ``client.admin`` 密钥，并为客户端保存此密钥的副本：

   .. prompt:: bash $

      ceph auth get-or-create client.admin mon 'allow *' mds 'allow *' mgr 'allow *' osd 'allow *' -o /etc/ceph/ceph.client.admin.keyring

   **警告：** 此命令会覆盖任何存在的
   ``/etc/ceph/client.admin.keyring`` 文件，如果部署工具已经\
   完成此步骤，千万别再执行此命令。多加小心！

#. 创建监视器集群所需的密钥环、并给它们生成密钥。

   .. prompt:: bash $

      ceph-authtool --create-keyring /tmp/ceph.mon.keyring --gen-key -n mon. --cap mon 'allow *'

#. 把监视器密钥环复制到 ``ceph.mon.keyring`` 文件，
   再把此文件复制到各监视器的 ``mon data`` 目录下。
   比如要把它复制给名为 ``ceph`` 集群的 ``mon.a`` ，
   用此命令：

   .. prompt:: bash $

      cp /tmp/ceph.mon.keyring /var/lib/ceph/mon/ceph-a/keyring

#. 为每个 MGR 生成一个密钥， ``{$id}`` 是 MGR 的名字：

   .. prompt:: bash $

      ceph auth get-or-create mgr.{$id} mon 'allow profile mgr' mds 'allow *' osd 'allow *' -o /var/lib/ceph/mgr/ceph-{$id}/keyring

#. 为每个 OSD 生成密钥， ``{$id}`` 是 OSD 编号：

   .. prompt:: bash $

      ceph auth get-or-create osd.{$id} mon 'allow rwx' osd 'allow *' -o /var/lib/ceph/osd/ceph-{$id}/keyring

#. 为每个 MDS 生成密钥， ``{$id}`` 是 MDS 的标识字母：

   .. prompt:: bash $

      ceph auth get-or-create mds.{$id} mon 'allow rwx' osd 'allow *' mds 'allow *' mgr 'allow profile mds' -o /var/lib/ceph/mds/ceph-{$id}/keyring

#. 把以下配置加入 `Ceph 配置`_\ 文件的
   ``[global]`` 段下以启用 CephX 认证：

   .. code-block:: ini

      auth_cluster_required = cephx
      auth_service_required = cephx
      auth_client_required = cephx

#. 启动或重启 Ceph 集群，详情见\ `操纵集群`_\ 。

要手动自启监视器，请参考\ `手动部署`_\ 。


禁用 Cephx
----------
.. Disabling Cephx

下述步骤描述了如何禁用 CephX 。
如果你的集群环境相对安全，可以减少\
认证耗费的计算资源，然而\ **我们不推荐**\ 。
但是临时禁用认证会使安装、和/或排障更简单，
可以稍后重新启用。

#. 把下列配置加入 `Ceph 配置`_\ 文件的 ``[global]`` 段下\
   即可禁用 CephX 认证：

   .. code-block:: ini

      auth_cluster_required = none
      auth_service_required = none
      auth_client_required = none

#. 启动或重启 Ceph 集群，具体参考\ `操纵集群`_\ 。


配置选项
========
.. Configuration Settings

启用事项
--------
.. Enablement


``auth_cluster_required``

:描述: 如果启用此配置选项， Ceph 存储集群守护进程
       （即 ``ceph-mon`` 、 ``ceph-osd`` 、 ``ceph-mds``
       和 ``ceph-mgr`` ）需要相互验证。
       有效配置有 ``cephx`` 或 ``none`` 。

:类型: String
:是否必需: No
:默认值: ``cephx``.


``auth_service_required``

:描述: 如果启用了此配置选项，那么只有当 Ceph 客户端与
       Ceph 存储集群通过身份验证时， Ceph 客户端才能访问 Ceph 服务。
       有效设置为 ``cephx`` 或 ``none`` 。

:类型: String
:是否必需: No
:默认值: ``cephx``.


``auth_client_required``

:描述: 如果启用了此配置选项，
       则只有在 Ceph 客户端通过了 Ceph 存储集群的身份验证时，
       Ceph 客户端和 Ceph 存储集群之间才能建立通信。
       有效配置有 ``cephx`` 或 ``none`` 。

:类型: String
:是否必需: No
:默认值: ``cephx``.


.. index:: keys; keyring

密钥
----
.. Keys

如果 Ceph 启用了认证， ``ceph`` 管理命令\
和客户端得有密钥才能访问集群。

给 ``ceph`` 管理命令和客户端提供密钥的最常用方法\
就是把密钥环放到 ``/etc/ceph`` 目录下。
Octopus 以及后续版本用 ``cephadm`` ，
其文件名通常是 ``ceph.client.admin.keyring`` 。
如果密钥环位于 ``/etc/ceph`` 目录下，
就不需要在 Ceph 配置文件里指定 ``keyring`` 选项了。

由于 Ceph 存储集群的密钥环包含 ``client.admin`` 密钥，
我们建议把这个密钥环复制到你执行管理命令的节点上。

手动执行这一步，执行下列命令：

.. prompt:: bash $

   sudo scp {user}@{ceph-cluster-host}:/etc/ceph/ceph.client.admin.keyring /etc/ceph/ceph.client.admin.keyring

.. tip:: 确保给客户端上的 ``ceph.keyring`` 设置合理的权限位
   （如 ``chmod 644`` ）。

你可以用 ``key`` 选项把密钥写在配置文件里
（建议别用此方法），
或者在 Ceph 配置文件里用 ``keyfile`` 选项指定密钥文件的路径。


``keyring``

:描述: 密钥环文件的路径。
:类型: String
:是否必需: No
:默认值: ``/etc/ceph/$cluster.$name.keyring,/etc/ceph/$cluster.keyring,/etc/ceph/keyring,/etc/ceph/keyring.bin``


``keyfile``

:描述: 密钥文件的路径（也就是，只包含密钥的文件）。
:类型: String
:是否必需: No
:默认值: None


``key``

:描述: 密钥（就是密钥本身的文本字符串）。
       我们不建议您使用此选项，
       除非您清楚自己在干什么。
:类型: String
:是否必需: No
:默认值: None


守护进程密钥环
--------------
.. Daemon Keyrings

管理员或部署工具（如 ``cephadm`` ）
生成守护进程密钥的方式和生成用户密钥的方式相同。
默认情况下， Ceph 会把守护进程的密钥存储在它自己的数据目录中。
默认的密钥环位置和守护进程运行必需的能力见下文。


``ceph-mon``

:位置: ``$mon_data/keyring``
:能力: ``mon 'allow *'``

``ceph-osd``

:位置: ``$osd_data/keyring``
:能力: ``mgr 'allow profile osd' mon 'allow profile osd' osd 'allow *'``

``ceph-mds``

:位置: ``$mds_data/keyring``
:能力: ``mds 'allow' mgr 'allow profile mds' mon 'allow profile mds' osd 'allow rwx'``

``ceph-mgr``

:位置: ``$mgr_data/keyring``
:能力: ``mon 'allow profile mgr' mds 'allow *' osd 'allow *'``

``radosgw``

:位置: ``$rgw_data/keyring``
:能力: ``mon 'allow rwx' osd 'allow rwx'``


.. note:: 监视器密钥环（即 ``mon.`` ）包含一个密钥，但不包含能力，
   而且这个密钥环不是集群 ``auth`` （认证）数据库的一部分。

守护进程的数据目录位置遵循如下格式： ::

  /var/lib/ceph/$type/$cluster-$id

例如， ``osd.12`` 的数据目录如下： ::

  /var/lib/ceph/osd/ceph-12

这些位置可以覆盖，但不建议那样做。


.. index:: signatures

签名
----
.. Signatures

Ceph 施行的签名检查可以为消息提供一些有限的保护，
以防消息被在线篡改（比如被“中间人”攻击篡改）。

像 Ceph 认证的其他部分一样，客户端和集群间的\
消息签名也能做到细粒度控制；而且\
能启用或禁用 Ceph 守护进程间的签名。

注意，即便启用了签名，线路中的数据也没被加密。


``cephx_require_signatures``

:描述: 如果把这个选项配置为 ``true`` ，
       那么 Ceph 会要求对 Ceph 客户端与 Ceph 存储集群之间、
       以及 Ceph 存储集群内守护进程之间的所有消息流量进行签名。

.. note::
          **陈年笔记：**

          Ceph Argonaut 和版本号小于 3.19 的 Linux 内核都不支持签名；
          如果用了这些版本的客户端，可以禁用 ``cephx_require_signatures`` ，
          让客户端连接进来。

:类型: Boolean
:是否必需: No
:默认值: ``false``


``cephx_cluster_require_signatures``

:描述: 如果把这个选项设置为 ``true`` ，
       那么 Ceph 要求对存储集群内
       Ceph 守护进程之间的所有消息流量进行签名。

:类型: Boolean
:是否必需: No
:默认值: ``false``


``cephx_service_require_signatures``

:描述: 如果把这个选项设置为 ``true`` ，
       那么 Ceph 要求对客户端和 Ceph 存储集群\
       之间的所有消息流量进行签名。

:类型: Boolean
:是否必需: No
:默认值: ``false``


``cephx_sign_messages``

:描述: 如果把这个选项设置为 ``true`` ，
       并且这个 Ceph 版本支持消息签名，
       那么 Ceph 将对所有消息进行签名，使其更难被欺骗。

:类型: Boolean
:默认值: ``true``


生存期
------
.. Time to Live

``auth_service_ticket_ttl``

:描述: 当 Ceph 存储集群向 Ceph 客户端发送用于身份验证的票据时，
       Ceph 存储集群会为该票据分配一个有效时间
       (Time To Live, TTL) 。

:类型: Double
:默认值: ``60*60``


.. _监视器的自举引导: ../../../install/manual-deployment#monitor-bootstrapping
.. _操纵集群: ../../operations/operating
.. _手动部署: ../../../install/manual-deployment
.. _Ceph 配置: ../ceph-conf
.. _体系结构——高可用性认证: ../../../architecture#high-availability-authentication
.. _用户管理: ../../operations/user-management
