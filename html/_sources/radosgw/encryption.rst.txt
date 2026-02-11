.. _radosgw-encryption:

======
 加密
======

.. versionadded:: Luminous

配置好 3 个加密密钥管理选项， Ceph 对象网关可支持在服务器端加密上传的对象。
服务器端加密的含义是，通过 HTTP 发出的数据是未加密的，
但是 Ceph 对象网关在 Ceph 存储集群中存储的却是加密数据。

.. note:: 服务器端加密的请求必须通过\
   安全的 HTTPS 连接发送，以免用明文发送密钥信息。
   如果用代理作为 SSL 终结，要让转发的请求被认为是可信的，
   必须先启用 ``rgw trust forwarded https`` 。

.. note:: 服务器端加密密钥必须是 256 位长、且用 base64 编码过。

客户提供的密钥
==============
.. Customer-Provided Keys

在此模式下，客户端的每个请求都需要传递加密密钥，用以读取或写入已加密数据。
管理那些加密密钥是客户端的责任，而且得记住加密各对象时分别用了哪个密钥。

这是根据 `Amazon SSE-C`_ 标准在 S3 里实现的。

因为所有密钥管理事务都是客户端处理的，所以要支持这种加密模式不\
需要 Ceph 做什么特殊的配置。

密钥管理服务
============
.. Key Management Service

在此模式下，管理员把密钥存储在一个安全的密钥管理服务中，
并可以让 Ceph 对象网关按需索取密钥，
然后用于加密或解密所要请求的数据。

这是根据 `Amazon SSE-KMS`_ 标准在 S3 里实现的。

原则上，这里可以用任何密钥管理服务，但现在只实现了与 `Barbican`_
、 `Vault`_ 和 `KMIP`_ 的对接。

参见\ `与 OpenStack Barbican 对接`_ 、 `与 HashiCorp Vault 对接`_\
和\ `与 KMIP 对接`_\ 。

SSE-S3
======

This makes key management invisible to the user.  They are still stored
in Vault, but they are automatically created and deleted by Ceph and
retrieved as required to serve requests to encrypt
or decrypt data.

This is implemented in S3 according to the `Amazon SSE-S3`_ specification.

In principle, any key management service could be used here.  Currently
only integration with `Vault`_, is implemented.

见 `与 HashiCorp Vault 对接`_.

桶加密 API
==========
.. Bucket Encryption APIs

桶加密 API （Bucket Encryption API ）是为了支持用基于 Amazon S3 管理的密钥
（SSE-S3）或者 AWS KMS 客户主密钥（SSE-KMS）的服务器端加密。
通过 BucketEncryption API 的 SSE-KMS 实现还不支持。

见 `PutBucketEncryption`_, `GetBucketEncryption`_, `DeleteBucketEncryption`_ 。

自动化加密（仅用于测试）
========================
.. Automatic Encryption (for testing only)

在 ceph.conf 里配置 ``rgw crypt default encryption key`` 可以\
强制加密所有对象，包括那些没有指定加密模式的对象。

这个配置选项只接受 base64 编码的 256 位密钥，例如： ::

    rgw crypt default encryption key = 4YSmvJtBv0aZ7geVgAsdpRnLBEwWSWlMIGnRS8a9TSA=

.. important:: 此模式仅用于诊断目的！ Ceph 配置文件不是存储加密密钥的
   安全之处，以此途径不小心泄露的密钥应该被当作被攻破的。


.. _Amazon SSE-C: https://docs.aws.amazon.com/AmazonS3/latest/dev/ServerSideEncryptionCustomerKeys.html
.. _Amazon SSE-KMS: http://docs.aws.amazon.com/AmazonS3/latest/dev/UsingKMSEncryption.html
.. _Amazon SSE-S3: https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingServerSideEncryption.html
.. _Barbican: https://wiki.openstack.org/wiki/Barbican
.. _Vault: https://www.vaultproject.io/docs/
.. _KMIP: http://www.oasis-open.org/committees/kmip/
.. _PutBucketEncryption: https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutBucketEncryption.html
.. _GetBucketEncryption: https://docs.aws.amazon.com/AmazonS3/latest/API/API_GetBucketEncryption.html
.. _DeleteBucketEncryption: https://docs.aws.amazon.com/AmazonS3/latest/API/API_DeleteBucketEncryption.html
.. _与 OpenStack Barbican 对接: ../barbican
.. _与 HashiCorp Vault 对接: ../vault
.. _与 KMIP 对接: ../kmip
