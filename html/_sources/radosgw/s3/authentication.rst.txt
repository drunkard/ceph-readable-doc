====================
 认证和访问控制列表
====================

Requests to the RADOS Gateway (RGW) can be either authenticated or
unauthenticated. RGW assumes unauthenticated requests are sent by an anonymous
user. RGW supports canned ACLs.

认证
----

Authenticating a request requires including an access key and a Hash-based
Message Authentication Code (HMAC) in the request before it is sent to the
RGW server. RGW uses an S3-compatible authentication approach.

::

	HTTP/1.1
	PUT /buckets/bucket/object.mpeg
	Host: cname.domain.com
	Date: Mon, 2 Jan 2012 00:01:01 +0000
	Content-Encoding: mpeg
	Content-Length: 9999999

	Authorization: AWS {access-key}:{hash-of-header-and-secret}

In the foregoing example, replace ``{access-key}`` with the value for your access
key ID followed by a colon (``:``). Replace ``{hash-of-header-and-secret}`` with
a hash of the header string and the secret corresponding to the access key ID.

要生成头部字符串和密钥的哈希值，必须：

#. 获取头部字符串的值。
#. Normalize the request header string into canonical form.
#. 用 SHA-1 哈希算法生成一个 HMAC 。
   详情见 `RFC 2104`_ 和 `HMAC`_ 。
#. 把 ``hmac`` 结果用 base-64 编码。

To normalize the header into canonical form:

#. 提取所有以 ``x-amz-`` 打头的字段。
#. Ensure that the fields are all lowercase.
#. Sort the fields lexicographically.
#. Combine multiple instances of the same field name into a
   single field and separate the field values with a comma.
#. Replace white space and line breaks in field values with a single space.
#. Remove white space before and after colons.
#. Append a new line after each field.
#. Merge the fields back into the header.

把 ``{hash-of-header-and-secret}`` 替换成 base-64 编码的 HMAC 字符串。


向 OpenStack Keystone 发起认证
------------------------------
.. Authentication against OpenStack Keystone

In a radosgw instance that is configured with authentication against
OpenStack Keystone, it is possible to use Keystone as an authoritative
source for S3 API authentication. To do so, you must set:

* ``rgw keystone`` 配置选项在 :doc:`../keystone` 里解释了，
* ``rgw s3 auth use keystone = true`` 。

另外，想要使用 S3 API 的用户必须取得 AWS 风格的访问密钥和私钥。
可以用 ``openstack ec2 credentials create`` 命令获取::

  $ openstack --os-interface public ec2 credentials create
  +------------+---------------------------------------------------------------------------------------------------------------------------------------------+
  | Field      | Value                                                                                                                                       |
  +------------+---------------------------------------------------------------------------------------------------------------------------------------------+
  | access     | c921676aaabbccdeadbeef7e8b0eeb2c                                                                                                            |
  | links      | {u'self': u'https://auth.example.com:5000/v3/users/7ecbebaffeabbddeadbeefa23267ccbb24/credentials/OS-EC2/c921676aaabbccdeadbeef7e8b0eeb2c'} |
  | project_id | 5ed51981aab4679851adeadbeef6ebf7                                                                                                            |
  | secret     | ********************************                                                                                                            |
  | trust_id   | None                                                                                                                                        |
  | user_id    | 7ecbebaffeabbddeadbeefa23267cc24                                                                                                            |
  +------------+---------------------------------------------------------------------------------------------------------------------------------------------+

那样生成的访问密钥和私钥稍后可以用来访问 S3 API 的 radosgw 。

.. note:: Consider that most production radosgw deployments
          authenticating against OpenStack Keystone are also set up
          for :doc:`../multitenancy`, for which special
          considerations apply with respect to S3 signed URLs and
          public read ACLs.


访问控制列表（ ACL ）
---------------------
.. Access Control Lists (ACLs)

RGW supports S3-compatible ACL functionality. An ACL is a list of access grants
that specify which operations a user can perform on a bucket or on an object.
Each grant has a different meaning when applied to a bucket versus applied to
an object:

+------------------+--------------------------------------------------------+----------------------------------------------+
| Permission       | Bucket                                                 | Object                                       |
+==================+========================================================+==============================================+
| ``READ``         | Grantee can list the objects in the bucket.            | Grantee can read the object.                 |
+------------------+--------------------------------------------------------+----------------------------------------------+
| ``WRITE``        | Grantee can write or delete objects in the bucket.     | N/A                                          |
+------------------+--------------------------------------------------------+----------------------------------------------+
| ``READ_ACP``     | Grantee can read bucket ACL.                           | Grantee can read the object ACL.             |
+------------------+--------------------------------------------------------+----------------------------------------------+
| ``WRITE_ACP``    | Grantee can write bucket ACL.                          | Grantee can write to the object ACL.         |
+------------------+--------------------------------------------------------+----------------------------------------------+
| ``FULL_CONTROL`` | Grantee has full permissions for object in the bucket. | Grantee can read or write to the object ACL. |
+------------------+--------------------------------------------------------+----------------------------------------------+

在内部， S3 操作是这样映射到 ACL 权限的：

+---------------------------------------+---------------+
| 操作                                  | 权限          |
+=======================================+===============+
| ``s3:GetObject``                      | ``READ``      |
+---------------------------------------+---------------+
| ``s3:GetObjectTorrent``               | ``READ``      |
+---------------------------------------+---------------+
| ``s3:GetObjectVersion``               | ``READ``      |
+---------------------------------------+---------------+
| ``s3:GetObjectVersionTorrent``        | ``READ``      |
+---------------------------------------+---------------+
| ``s3:GetObjectTagging``               | ``READ``      |
+---------------------------------------+---------------+
| ``s3:GetObjectVersionTagging``        | ``READ``      |
+---------------------------------------+---------------+
| ``s3:ListAllMyBuckets``               | ``READ``      |
+---------------------------------------+---------------+
| ``s3:ListBucket``                     | ``READ``      |
+---------------------------------------+---------------+
| ``s3:ListBucketMultipartUploads``     | ``READ``      |
+---------------------------------------+---------------+
| ``s3:ListBucketVersions``             | ``READ``      |
+---------------------------------------+---------------+
| ``s3:ListMultipartUploadParts``       | ``READ``      |
+---------------------------------------+---------------+
| ``s3:AbortMultipartUpload``           | ``WRITE``     |
+---------------------------------------+---------------+
| ``s3:CreateBucket``                   | ``WRITE``     |
+---------------------------------------+---------------+
| ``s3:DeleteBucket``                   | ``WRITE``     |
+---------------------------------------+---------------+
| ``s3:DeleteObject``                   | ``WRITE``     |
+---------------------------------------+---------------+
| ``s3:s3DeleteObjectVersion``          | ``WRITE``     |
+---------------------------------------+---------------+
| ``s3:PutObject``                      | ``WRITE``     |
+---------------------------------------+---------------+
| ``s3:PutObjectTagging``               | ``WRITE``     |
+---------------------------------------+---------------+
| ``s3:PutObjectVersionTagging``        | ``WRITE``     |
+---------------------------------------+---------------+
| ``s3:DeleteObjectTagging``            | ``WRITE``     |
+---------------------------------------+---------------+
| ``s3:DeleteObjectVersionTagging``     | ``WRITE``     |
+---------------------------------------+---------------+
| ``s3:RestoreObject``                  | ``WRITE``     |
+---------------------------------------+---------------+
| ``s3:GetAccelerateConfiguration``     | ``READ_ACP``  |
+---------------------------------------+---------------+
| ``s3:GetBucketAcl``                   | ``READ_ACP``  |
+---------------------------------------+---------------+
| ``s3:GetBucketCORS``                  | ``READ_ACP``  |
+---------------------------------------+---------------+
| ``s3:GetBucketLocation``              | ``READ_ACP``  |
+---------------------------------------+---------------+
| ``s3:GetBucketLogging``               | ``READ_ACP``  |
+---------------------------------------+---------------+
| ``s3:GetBucketNotification``          | ``READ_ACP``  |
+---------------------------------------+---------------+
| ``s3:GetBucketPolicy``                | ``READ_ACP``  |
+---------------------------------------+---------------+
| ``s3:GetBucketRequestPayment``        | ``READ_ACP``  |
+---------------------------------------+---------------+
| ``s3:GetBucketTagging``               | ``READ_ACP``  |
+---------------------------------------+---------------+
| ``s3:GetBucketVersioning``            | ``READ_ACP``  |
+---------------------------------------+---------------+
| ``s3:GetBucketWebsite``               | ``READ_ACP``  |
+---------------------------------------+---------------+
| ``s3:GetLifecycleConfiguration``      | ``READ_ACP``  |
+---------------------------------------+---------------+
| ``s3:GetObjectAcl``                   | ``READ_ACP``  |
+---------------------------------------+---------------+
| ``s3:GetObjectVersionAcl``            | ``READ_ACP``  |
+---------------------------------------+---------------+
| ``s3:GetReplicationConfiguration``    | ``READ_ACP``  |
+---------------------------------------+---------------+
| ``s3:GetBucketEncryption``            | ``READ_ACP``  |
+---------------------------------------+---------------+
| ``s3:DeleteBucketPolicy``             | ``WRITE_ACP`` |
+---------------------------------------+---------------+
| ``s3:DeleteBucketWebsite``            | ``WRITE_ACP`` |
+---------------------------------------+---------------+
| ``s3:DeleteReplicationConfiguration`` | ``WRITE_ACP`` |
+---------------------------------------+---------------+
| ``s3:PutAccelerateConfiguration``     | ``WRITE_ACP`` |
+---------------------------------------+---------------+
| ``s3:PutBucketAcl``                   | ``WRITE_ACP`` |
+---------------------------------------+---------------+
| ``s3:PutBucketCORS``                  | ``WRITE_ACP`` |
+---------------------------------------+---------------+
| ``s3:PutBucketLogging``               | ``WRITE_ACP`` |
+---------------------------------------+---------------+
| ``s3:PutBucketNotification``          | ``WRITE_ACP`` |
+---------------------------------------+---------------+
| ``s3:PutBucketPolicy``                | ``WRITE_ACP`` |
+---------------------------------------+---------------+
| ``s3:PutBucketRequestPayment``        | ``WRITE_ACP`` |
+---------------------------------------+---------------+
| ``s3:PutBucketTagging``               | ``WRITE_ACP`` |
+---------------------------------------+---------------+
| ``s3:PutPutBucketVersioning``         | ``WRITE_ACP`` |
+---------------------------------------+---------------+
| ``s3:PutBucketWebsite``               | ``WRITE_ACP`` |
+---------------------------------------+---------------+
| ``s3:PutLifecycleConfiguration``      | ``WRITE_ACP`` |
+---------------------------------------+---------------+
| ``s3:PutObjectAcl``                   | ``WRITE_ACP`` |
+---------------------------------------+---------------+
| ``s3:PutObjectVersionAcl``            | ``WRITE_ACP`` |
+---------------------------------------+---------------+
| ``s3:PutReplicationConfiguration``    | ``WRITE_ACP`` |
+---------------------------------------+---------------+
| ``s3:PutBucketEncryption``            | ``WRITE_ACP`` |
+---------------------------------------+---------------+

Some mappings, (e.g. ``s3:CreateBucket`` to ``WRITE``) are not
applicable to S3 operation, but are required to allow Swift and S3 to
access the same resources when things like Swift user ACLs are in
play. This is one of the many reasons that you should use S3 bucket
policies rather than S3 ACLs when possible.


.. _RFC 2104: http://www.ietf.org/rfc/rfc2104.txt
.. _HMAC: https://en.wikipedia.org/wiki/HMAC
