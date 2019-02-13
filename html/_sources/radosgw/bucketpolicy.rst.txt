.. _Bucket Policies:

========
 桶策略
========

.. versionadded:: Luminous

Ceph 对象网关支持部分 Amazon S3 桶策略语义。


.. Creation and Removal

创建和删除
==========

桶策略可通过标准的 S3 操作管理，而不是用 radosgw-admin 。

比如，用 s3cmd 可以这样设置或删除策略： ::

    $ cat > examplepol
    {
        "Version": "2012-10-17",
        "Statement": [{
            "Effect": "Allow",
            "Principal": {"AWS": ["arn:aws:iam::usfolks:user/fred"]},
            "Action": "s3PutObjectAcl",
            "Resource": [
                "arn:aws:s3:::happybucket/*"
            ]
        }]
    }

    $ s3cmd setpolicy examplepol s3://happybucket
    $ s3cmd delpolicy s3://happybucket


.. Limitations

限制
====

现在，我们只支持如下操作：

- s3:AbortMultipartUpload
- s3:CreateBucket
- s3:DeleteBucketPolicy
- s3:DeleteBucket
- s3:DeleteBucketWebsite
- s3:DeleteObject
- s3:DeleteObjectVersion
- s3:DeleteReplicationConfiguration
- s3:GetAccelerateConfiguration
- s3:GetBucketAcl
- s3:GetBucketCORS
- s3:GetBucketLocation
- s3:GetBucketLogging
- s3:GetBucketNotification
- s3:GetBucketPolicy
- s3:GetBucketRequestPayment
- s3:GetBucketTagging
- s3:GetBucketVersioning
- s3:GetBucketWebsite
- s3:GetLifecycleConfiguration
- s3:GetObjectAcl
- s3:GetObject
- s3:GetObjectTorrent
- s3:GetObjectVersionAcl
- s3:GetObjectVersion
- s3:GetObjectVersionTorrent
- s3:GetReplicationConfiguration
- s3:ListAllMyBuckets
- s3:ListBucketMultiPartUploads
- s3:ListBucket
- s3:ListBucketVersions
- s3:ListMultipartUploadParts
- s3:PutAccelerateConfiguration
- s3:PutBucketAcl
- s3:PutBucketCORS
- s3:PutBucketLogging
- s3:PutBucketNotification
- s3:PutBucketPolicy
- s3:PutBucketRequestPayment
- s3:PutBucketTagging
- s3:PutBucketVersioning
- s3:PutBucketWebsite
- s3:PutLifecycleConfiguration
- s3:PutObjectAcl
- s3:PutObject
- s3:PutObjectVersionAcl
- s3:PutReplicationConfiguration
- s3:RestoreObject

还不支持对用户、组或角色设置策略。

我们用 RGW “租户”标识符代替 Amazon 的 12 位帐户 ID 。未来，我\
们会允许你给租户分配帐户 ID ，但是目前，如果你想使用跨 AWS S3
和 RGW S3 的策略，在创建用户时还只能把 Amazon 帐户 ID 当作租户
ID 用。

在 AWS 下，所有租户共享同一个命名空间，而 RGW 会给每个租户分配\
它自己的桶命名空间。未来版本可能会增加一个选项来启用像 AWS 一\
样的“扁平”桶命名空间。在现有版本中，通过 S3 接口访问另一个租户\
的桶可以按 "tenant:bucket" 格式指定。

在 AWS 中，桶策略可以授权让另一个帐户访问，然后那个帐户的所有\
者又可以转手授权给他的用户。正因为我们现在还不支持用户、角色和\
组权限，所以帐户所有者现在还只能直接授权给独立用户，而且给一个\
帐户授予访问权限的同时也授权给了这个帐户内的所有用户们。

桶变量现在还不支持字符串插值。

当前仅支持下面的条件关键字：

- aws:CurrentTime
- aws:EpochTime
- aws:PrincipalType
- aws:Referer
- aws:SecureTransport
- aws:SourceIp
- aws:UserAgent
- aws:username

很快会支持更多，因为我们正在与重写过的认证、授权子系统对接。


Swift
=====

在 Swift 下还不能设置策略，但是通过 S3 设置的桶策略一样会影响 \
Swift 。

Swift 凭证与策略中定义的 Principal 匹配时，所用的方法因正在使\
用的后端而异。
