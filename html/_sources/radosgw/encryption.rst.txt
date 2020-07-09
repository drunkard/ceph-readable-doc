.. Encryption

======
 加密
======

.. versionadded:: Luminous

配置好 3 个加密密钥管理选项， Ceph 对象网关可支持在服务器端\
加密上传的对象。服务器端加密的含义是，通过 HTTP 发出的数据是\
未加密的，但是 Ceph 对象网关在 Ceph 存储集群中存储的却是\
加密数据。

.. note:: Requests for server-side encryption must be sent over a secure HTTPS
          connection to avoid sending secrets in plaintext. If a proxy is used
          for SSL termination, ``rgw trust forwarded https`` must be enabled
          before forwarded requests will be trusted as secure.

.. note:: Server-side encryption keys must be 256-bit long and base64 encoded.


.. Customer-Provided Keys

客户提供的密钥
==============

在此模式下，客户端的每个请求都需要传递加密密钥，用以读取或写入\
已加密数据。管理那些加密密钥是客户端的责任，而且得记住加密各对\
象时分别用了哪个密钥。

这是根据 `Amazon SSE-C`_ 标准在 S3 里实现的。

因为所有密钥管理事务都是客户端处理的，所以要支持这种加密模式不\
需要什么特殊的配置。


.. Key Management Service

密钥管理服务
============

此模式可以让密钥存储在一个安全的密钥管理服务中，并可以让 Ceph \
对象网关按需索取，然后用于加密或解密所要请求的数据。

这是根据 `Amazon SSE-KMS`_ 标准在 S3 里实现的。

原则上，这里可以用任何密钥管理服务，但现在只实现了与 `Barbican`_
和 `Vault`_ 的对接。

参见\ `与 OpenStack Barbican 对接`_\ 和\ `与 HashiCorp Vault 对接`_\ 。


.. Automatic Encryption (for testing only)

自动化加密（仅用于测试）
========================

在 ceph.conf 里配置 ``rgw crypt default encryption key`` 可以\
强制加密所有对象，包括那些没有指定加密模式的对象。

这个配置选项只接受 base64 编码的 256 位密钥，例如： ::

    rgw crypt default encryption key = 4YSmvJtBv0aZ7geVgAsdpRnLBEwWSWlMIGnRS8a9TSA=

.. important:: 此模式仅用于诊断目的！ Ceph 配置文件不是存储加\
   密密钥的安全之处，以此途径不小心泄露的密钥应该被当作被攻破的。


.. _Amazon SSE-C: https://docs.aws.amazon.com/AmazonS3/latest/dev/ServerSideEncryptionCustomerKeys.html
.. _Amazon SSE-KMS: http://docs.aws.amazon.com/AmazonS3/latest/dev/UsingKMSEncryption.html
.. _Barbican: https://wiki.openstack.org/wiki/Barbican
.. _Vault: https://www.vaultproject.io/docs/
.. _与 OpenStack Barbican 对接: ../barbican
.. _与 HashiCorp Vault 对接: ../vault
