.. _OpenStack Barbican Integration:

============================
 与 OpenStack Barbican 对接
============================

在\ `服务器端加密`_\ 中，可以用 OpenStack `Barbican`_ 作密钥管\
理服务。

.. image:: ../images/rgw-encryption-barbican.png

#. `配置 Keystone`_
#. `创建 Keystone 用户`_
#. `配置 Ceph 对象网关`_
#. `在 Barbican 里创建密钥`_


.. _Configure Keystone:

配置 Keystone
=============

Barbican 靠 Keystone 实现密钥的授权和访问控制。

参考 `与 OpenStack Keystone 对接`_\ 。


.. _Create a Keystone user:

创建 Keystone 用户
==================

创建个新用户， Ceph 对象网关索取密钥时要用到。

例如： ::

    user = rgwcrypt-user
    pass = rgwcrypt-password
    tenant = rgwcrypt

关于\ `管理项目、用户和角色`_\ 请参考 OpenStack 文档。


.. _Create a key in Barbican:

在 Barbican 里创建密钥
======================

想知道\ `如何创建密钥`_\ 请参考 Barbican 文档。向 Barbican 发\
起请求时， ``X-Auth-Token`` 头必须携带合法的 Keystone 令牌。

请求实例： ::

   POST /v1/secrets HTTP/1.1
   Host: barbican.example.com:9311
   Accept: */*
   Content-Type: application/json
   X-Auth-Token: 7f7d588dd29b44df983bc961a6b73a10
   Content-Length: 299
   
   {
       "name": "my-key",
       "expiration": "2016-12-28T19:14:44.180394",
       "algorithm": "aes",
       "bit_length": 256,
       "mode": "cbc",
       "payload": "6b+WOZ1T3cqZMxgThRcXAQBrS5mXKdDUphvpxptl9/4=",
       "payload_content_type": "application/octet-stream",
       "payload_content_encoding": "base64"
   }

响应： ::

   {"secret_ref": "http://barbican.example.com:9311/v1/secrets/d1e7ef3b-f841-4b7c-90b2-b7d90ca2d723"}

响应中的 ``d1e7ef3b-f841-4b7c-90b2-b7d90ca2d723`` 是密钥 id ，\
可以用于任何 `SSE-KMS`_ 请求。

``rgwcrypt-user`` 不能访问新创建的密钥，必须用 ACL 加上这个权\
限，请参考\ `如何设置、替换 ACL`_ 。

请求实例（假设 ``rgwcrypt-user`` 的 Keystone ID 是
``906aa90bd8a946c89cdff80d0869460f`` ）： ::

   PUT /v1/secrets/d1e7ef3b-f841-4b7c-90b2-b7d90ca2d723/acl HTTP/1.1
   Host: barbican.example.com:9311
   Accept: */*
   Content-Type: application/json
   X-Auth-Token: 7f7d588dd29b44df983bc961a6b73a10
   Content-Length: 101

   {
       "read":{
       "users":[ "906aa90bd8a946c89cdff80d0869460f" ],
       "project-access": true
       }
   }

响应： ::

   {"acl_ref": "http://barbican.example.com:9311/v1/secrets/d1e7ef3b-f841-4b7c-90b2-b7d90ca2d723/acl"}


.. _Configure the Ceph Object Gateway:

配置 Ceph 对象网关
==================

编辑 Ceph 配置文件，加上 Barbican 服务器和 Keystone 用户信息： ::

   rgw barbican url = http://barbican.example.com:9311
   rgw keystone barbican user = rgwcrypt-user
   rgw keystone barbican password = rgwcrypt-password

如果用的是 Keystone API v2::

   rgw keystone barbican tenant = rgwcrypt

如果用的是 API v3::

   rgw keystone barbican project
   rgw keystone barbican domain


.. _Barbican: https://wiki.openstack.org/wiki/Barbican
.. _服务器端加密: ../encryption
.. _与 OpenStack Keystone 对接: ../keystone
.. _管理项目、用户和角色: https://docs.openstack.org/admin-guide/cli-manage-projects-users-and-roles.html#create-a-user
.. _如何创建密钥: https://developer.openstack.org/api-guide/key-manager/secrets.html#how-to-create-a-secret
.. _SSE-KMS: http://docs.aws.amazon.com/AmazonS3/latest/dev/UsingKMSEncryption.html
.. _如何设置、替换 ACL: https://developer.openstack.org/api-guide/key-manager/acls.html#how-to-set-replace-acl
