.. _rados-cephx-config-ref:

================
 Cephx 配置参考
================

``cephx`` 协议已默认开启。\
加密认证要耗费一定计算资源，但通常很低。\
如果您的客户端和服务器网络环境相当安全，\
而且认证的负面效应更大，你可以关闭它，\ **通常不推荐您这么做**\ 。

.. note:: 如果禁用了认证，就会有篡改客户端/服务器消息\
   这样的中间人攻击风险，这会导致灾难性后果。

关于创建用户请参考\ `用户管理`_\ ；\
关于 Cephx 的体系结构请参考\ `体系结构——高可用性认证`_\ 。


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

在禁用了 ``cephx`` 的集群上执行下面的步骤来启用它，如果你（\
或者部署工具）已经生成了密钥，你可以跳过相关步骤。

#. 创建 ``client.admin`` 密钥，并为客户端保存此密钥的副本：

   .. prompt:: bash $

      ceph auth get-or-create client.admin mon 'allow *' mds 'allow *' mgr 'allow *' osd 'allow *' -o /etc/ceph/ceph.client.admin.keyring

   **警告：** 此命令会覆盖任何存在的
   ``/etc/ceph/client.admin.keyring`` 文件，如果部署工具已经\
   完成此步骤，千万别再执行此命令。多加小心！

#. 创建监视器集群所需的密钥环、并给它们生成密钥。

   .. prompt:: bash $

      ceph-authtool --create-keyring /tmp/ceph.mon.keyring --gen-key -n mon. --cap mon 'allow *'

#. 把监视器密钥环复制到 ``ceph.mon.keyring`` 文件，再把此文件\
   复制到各监视器的 ``mon data`` 目录下。比如要把它复制给名为
   ``ceph`` 集群的 ``mon.a`` ，用此命令：

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

#. 把以下配置加入 `Ceph 配置`_\ 文件的 ``[global]`` 段下以启用
   ``cephx`` 认证：

   .. code-block:: ini

      auth_cluster_required = cephx
      auth_service_required = cephx
      auth_client_required = cephx

#. 启动或重启 Ceph 集群，详情见\ `操纵集群`_\ 。

要手动自启监视器，请参考\ `手动部署`_\ 。


禁用 Cephx
----------
.. Disabling Cephx

下述步骤描述了如何禁用 Cephx 。如果你的集群环境相对安全，
你可以减免认证耗费的计算资源，然而\ **我们不推荐**\ 。
但是临时禁用认证会使安装、和/或排障更简单。

#. 把下列配置加入 `Ceph 配置`_\ 文件的 ``[global]`` 段下\
   即可禁用 ``cephx`` 认证：

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

.. confval:: auth_cluster_required
.. confval:: auth_service_required
.. confval:: auth_client_required


.. index:: keys; keyring

密钥
----
.. Keys

如果你的集群启用了认证， ``ceph`` 管理命令和客户端得有密钥\
才能访问集群。

给 ``ceph`` 管理命令和客户端提供密钥的最常用方法就是把密钥环\
放到 ``/etc/ceph`` 目录下。在 Octopus 和后续版本用
``cephadm`` ，其文件名通常是 ``ceph.client.admin.keyring``
（或 ``$cluster.client.admin.keyring`` ）。如果你的密钥环位于
``/etc/ceph`` 目录下，就不需要在 Ceph 配置文件里指定
``keyring`` 选项了。

我们建议把集群的密钥环复制到你执行管理命令的节点，它包含
``client.admin`` 密钥。

手动复制可执行此命令： ::

	sudo scp {user}@{ceph-cluster-host}:/etc/ceph/ceph.client.admin.keyring /etc/ceph/ceph.client.admin.keyring

.. tip:: 确保给客户端上的 ``ceph.keyring`` 设置合理的权限位（如
   ``chmod 644`` ）。

你可以用 ``key`` 选项把密钥写在配置文件里（别这样），或者用
``keyfile`` 选项指定个路径。

.. confval:: keyring
   :default: /etc/ceph/$cluster.$name.keyring,/etc/ceph/$cluster.keyring,/etc/ceph/keyring,/etc/ceph/keyring.bin
.. confval:: keyfile
.. confval:: key


.. index:: signatures

签名
----
.. Signatures

Ceph 施行的签名检查可以为消息提供一些有限的保护，
以防消息被在线篡改（比如被“中间人”攻击篡改）。

像 Ceph 认证的其他部分一样，客户端和集群间的\
消息签名也能做到细粒度控制；而且\
能启用或禁用 Ceph 守护进程间的签名。

注意，即便启用了签名，数据也没在线加密。

.. confval:: cephx_require_signatures
.. confval:: cephx_cluster_require_signatures
.. confval:: cephx_service_require_signatures
.. confval:: cephx_sign_messages


生存期
------
.. Time to Live

.. confval:: auth_service_ticket_ttl


.. _监视器的自举引导: ../../../install/manual-deployment#monitor-bootstrapping
.. _操纵集群: ../../operations/operating
.. _手动部署: ../../../install/manual-deployment
.. _Ceph 配置: ../ceph-conf
.. _体系结构——高可用性认证: ../../../architecture#high-availability-authentication
.. _用户管理: ../../operations/user-management
