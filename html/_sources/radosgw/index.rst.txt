.. Ceph Object Gateway
.. _object-gateway:

===============
 Ceph 对象网关
===============

:term:`Ceph 对象网关`\ 是个对象存储接口，在 ``librados`` 之上\
为应用程序构建了一个 RESTful 风格的 Ceph 存储集群网关。
:term:`Ceph 对象存储`\ 支持 2 种接口：

#. **S3-compatible:** 提供了对象存储接口，与亚马逊的 S3 RESTful
   风格的接口兼容。

#. **Swift-compatible:** 提供了对象存储接口，与 OpenStack 的
   Swift 接口兼容。

Ceph 对象存储使用 Ceph 对象网关守护进程（ ``radosgw`` ），它是\
个与 Ceph 存储集群交互的 FastCGI 模块。因为它提供了与 OpenStack
Swift 和 Amazon S3 兼容的接口， RADOS 要有它自己的用户管理。
Ceph 对象网关可与 CephFS 客户端或 Ceph 块设备客户端共用一个存\
储集群。 S3 和 Swift API 共用一个通用命名空间，所以你可以用一个
API 写、然后用另一个检出。

.. ditaa::  +------------------------+ +------------------------+
            |   S3 compatible API    | |  Swift compatible API  |
            +------------------------+-+------------------------+
            |                      radosgw                      |
            +---------------------------------------------------+
            |                      librados                     |
            +------------------------+-+------------------------+
            |          OSDs          | |        Monitors        |
            +------------------------+ +------------------------+

.. note:: Ceph 对象存储\ **不**\ 使用 Ceph 元数据服务器。


.. toctree::
   :maxdepth: 1

   基于 Civetweb 手动安装 <../../install/install-ceph-gateway>
   HTTP 前端 <frontends>
   Pool Placement and Storage Classes <placement>
   基于 Apache/FastCGI 的简单配置 <config-fcgi>
   多站配置 <multisite>
   多站同步策略配置 <multisite-sync-policy>
   存储池的配置 <pools>
   配置参考 <config-ref>
   管理指南 <admin>
   S3 API <s3>
   Data caching and CDN <rgw-cache.rst>
   Swift API <swift>
   管理操作 API <adminops>
   Python 接口 <api>
   通过 NFS 导出 <nfs>
   与 OpenStack Keystone 对接 <keystone>
   与 OpenStack Barbican 对接 <barbican>
   与 HashiCorp Vault 对接 <vault>
   与 Open Policy Agent 对接 <opa>
   多租户 <multitenancy>
   压缩 <compression>
   LDAP 认证 <ldap-auth>
   服务器端加密 <encryption>
   桶策略 <bucketpolicy>
   动态的桶索引重分片 <dynamicresharding>
   多因子认证 <mfa>
   同步模块 <sync-modules>
   Bucket Notifications <notifications>
   RADOS 中的数据布局 <layout>
   STS <STS>
   STS Lite <STSLite>
   Keycloak <keycloak>
   Role <role>
   Orphan List and Associated Tooliing <orphans>
   OpenID Connect Provider <oidc>
   troubleshooting
   radosgw 手册页 <../../man/8/radosgw>
   radosgw-admin 手册页 <../../man/8/radosgw-admin>
   QAT Acceleration for Encryption and Compression <qat-accel>
   S3-select <s3select>
