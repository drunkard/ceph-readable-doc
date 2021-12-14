.. _cephfs-nfs:

===
NFS
===

CephFS 命名空间可以通过 NFS 协议导出，用 `NFS-Ganesha NFS 服务器`_ 。本文档说的是
NFS-Ganesha 集群的手动配置方法。最简单的、首选的方法是通过 ``ceph nfs ...`` 系列命令\
来管理 NFS-Ganesha 集群和 CephFS 的导出。详情见 :doc:`/mgr/nfs` 。
假设 Ceph 是用 cephadm 或 rook 部署的。


必要条件
========
.. Requirements

-  Ceph 文件系统
-  在 NFS 服务器所在主机上有 ``libcephfs2`` 、 ``nfs-ganesha``
   和 ``nfs-ganesha-ceph`` 软件包。
-  连接到 Ceph 公共网的 NFS-Ganesha 服务器主机。

.. note::
   建议用 3.5 或更高稳定版的 NFS-Ganesha 软件包，
   对接 pacific (16.2.x) 或更高版本的 Ceph 。


配置 NFS-Ganesha 来导出 CephFS
==============================
.. Configuring NFS-Ganesha to export CephFS

NFS-Ganesha 有一个文件系统抽象层（ File System Abstraction Layer, FSAL ）来适配\
不同的存储后端。 FSAL_CEPH_ 是适配 CephFS 的 FSAL 插件。对于每一个 NFS-Ganesha 导出，
FSAL_CEPH_ 都会用一个 libcephfs 客户端来挂载 NFS-Ganesha 要导出的 CephFS 路径。

对接 NFS-Ganesha 和 CephFS 的配置，涉及 NFS-Ganesha 的配置、 Ceph 配置文件、
NFS-Ganesha 创建的、用于访问 CephFS 的客户端所需的 CephX 访问证书。


NFS-Ganesha 配置
----------------
.. NFS-Ganesha configuration

这里有个用 FSAL_CEPH_ 配置的 `ganesha.conf 样本`_ 。它适合独立的 NFS-Ganesha 服务器，
或者主从式的 NFS-Ganesha 服务器，这样适合让某些集群化软件管理（如 Pacemaker ）。
关于选项的重要信息在样板配置的注释里。其中有些选项是：

- 尽可能最小化 Ganesha 的缓存，因为 FSAL_CEPH_ 的 libcephfs 客户端也会激进地缓存；

- 从 Ganesha 配置文件里读出，它存储在 RADOS 对象里；

- 用 RADOS OMAP 的键值接口存储客户端的恢复数据；

- 指明访问协议为 NFSv4.1+

- 启用读取授权（需要最低 v13.0.1 版的 ``libcephfs2`` 软件包和
  v2.6.0 稳定版的 ``nfs-ganesha`` 和 ``nfs-ganesha-ceph`` 软件包）


给 libcephfs 客户端的配置
-------------------------
.. Configuration for libcephfs clients

libcephfs 客户端的 ``ceph.conf`` 包含一个 ``[client]`` 段，要设置其下的 ``mon_host`` 选项，
客户端才能连接到集群的监视器，通常是用 ``ceph config generate-minimal-conf`` 命令生成的。
例如： ::

    [client]
            mon host = [v2:192.168.1.7:3300,v1:192.168.1.7:6789], [v2:192.168.1.8:3300,v1:192.168.1.8:6789], [v2:192.168.1.9:3300,v1:192.168.1.9:6789]


用 NFSv4 客户端挂载
===================
.. Mount using NFSv4 clients

最好用 NFSv4.1+ 协议挂载 NFS-Ganesha 的出口，这样才能获得会话上的好处。

挂载 NFS 资源的惯例因平台而变，下面的惯例适合 Linux 和某些 Unix 平台：

.. code:: bash

    mount -t nfs -o nfsvers=4.1,proto=tcp <ganesha-host-name>:<ganesha-pseudo-path> <mount-point>


.. _FSAL_CEPH: https://github.com/nfs-ganesha/nfs-ganesha/tree/next/src/FSAL/FSAL_CEPH
.. _NFS-Ganesha NFS 服务器: https://github.com/nfs-ganesha/nfs-ganesha/wiki
.. _ganesha.conf 样本: https://github.com/nfs-ganesha/nfs-ganesha/blob/next/src/config_samples/ceph.conf
