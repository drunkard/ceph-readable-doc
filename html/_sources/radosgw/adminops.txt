==========
 管理操作
==========

An admin API request will be done on a URI that starts with the configurable 'admin'
resource entry point. Authorization for the admin API duplicates the S3 authorization
mechanism. Some operations require that the user holds special administrative capabilities.
The response entity type (XML or JSON) may be specified as the 'format' option in the
request and defaults to JSON if not specified.


.. _Get Usage:

查看使用率
==========

请求带宽利用率信息。

注意：此功能默认是禁用的，可以在 ceph.conf 里的适当段落加上
``rgw enable usage log = true`` 来启用，更改配置后需重启
radosgw 进程才能生效。

:caps: usage=read

语法
~~~~

::

	GET /{admin}/usage?format=json HTTP/1.1
	Host: {fqdn}


请求参数
~~~~~~~~

``uid``

:描述: The user for which the information is requested. If not specified will apply to all users.
:类型: String
:实例: ``foo_user``
:是否必需: No

``start``

:描述: Date and (optional) time that specifies the start time of the requested data.
:类型: String
:实例: ``2012-09-25 16:00:00``
:是否必需: No

``end``

:描述: Date and (optional) time that specifies the end time of the requested data (non-inclusive).
:类型: String
:实例: ``2012-09-25 16:00:00``
:是否必需: No


``show-entries``

:描述: 指示是否返回数据条目。
:类型: Boolean
:实例: True [True]
:是否必需: No


``show-summary``

:描述: 指示是否返回数据汇总信息。
:类型: Boolean
:实例: True [True]
:是否必需: No


响应内容解析
~~~~~~~~~~~~

如果成功，响应会包含请求的信息。

``usage``

:描述: A container for the usage information.
:类型: Container

``entries``

:描述: A container for the usage entries information.
:类型: Container

``user``

:描述: A container for the user data information.
:类型: Container

``owner``

:描述: The name of the user that owns the buckets.
:类型: String

``bucket``

:描述: The bucket name.
:类型: String

``time``

:描述: Time lower bound for which data is being specified (rounded to the beginning of the first relevant hour).
:类型: String

``epoch``

:描述: The time specified in seconds since 1/1/1970.
:类型: String

``categories``

:描述: A container for stats categories.
:类型: Container

``entry``

:描述: A container for stats entry.
:类型: Container

``category``

:描述: Name of request category for which the stats are provided.
:类型: String

``bytes_sent``

:描述: Number of bytes sent by the RADOS Gateway.
:类型: Integer

``bytes_received``

:描述: Number of bytes received by the RADOS Gateway.
:类型: Integer

``ops``

:描述: Number of operations.
:类型: Integer

``successful_ops``

:描述: Number of successful operations.
:类型: Integer

``summary``

:描述: A container for stats summary.
:类型: Container

``total``

:描述: A container for stats summary aggregated total.
:类型: Container


特殊错误响应
~~~~~~~~~~~~

TBD.


.. Trim Usage

裁剪使用率日志
==============

删除使用率信息。若未指定日期，会删除所有使用率信息。

注意：此功能默认是禁用的，可以在 ceph.conf 里的适当段落加上
``rgw enable usage log = true`` 来启用，更改配置后需重启
radosgw 进程才能生效。

:caps: usage=write

语法
~~~~

::

	DELETE /{admin}/usage?format=json HTTP/1.1
	Host: {fqdn}


请求参数
~~~~~~~~

``uid``

:描述: The user for which the information is requested. If not specified will apply to all users.
:类型: String
:实例: ``foo_user``
:是否必需: No

``start``

:描述: Date and (optional) time that specifies the start time of the requested data.
:类型: String
:实例: ``2012-09-25 16:00:00``
:是否必需: No

``end``

:描述: Date and (optional) time that specifies the end time of the requested data (none inclusive).
:类型: String
:实例: ``2012-09-25 16:00:00``
:是否必需: No


``remove-all``

:描述: 是否必需 when uid is not specified, in order to acknowledge multi user data removal.
:类型: Boolean
:实例: True [False]
:是否必需: No

特殊错误响应
~~~~~~~~~~~~

TBD.


.. Get User Info

查看用户信息
============

查看用户信息。

:caps: users=read


语法
~~~~

::

	GET /{admin}/user?format=json HTTP/1.1
	Host: {fqdn}


Request Parameters
~~~~~~~~~~~~~~~~~~

``uid``

:描述: The user for which the information is requested.
:类型: String
:实例: ``foo_user``
:是否必需: Yes


响应内容解析
~~~~~~~~~~~~

如果成功了，这个响应会包含此用户的信息。

``user``

:描述: 用户数据信息的一个容器。
:类型: Container

``user_id``

:描述: 此用户的标识符。
:类型: String
:父节点: ``user``

``display_name``

:描述: 用户对外显示的名字。
:类型: String
:父节点: ``user``

``suspended``

:描述: 如果此用户被暂停，其值为 True 。
:类型: Boolean
:父节点: ``user``

``max_buckets``

:描述: The maximum number of buckets to be owned by the user.
:类型: Integer
:父节点: ``user``

``subusers``

:描述: Subusers associated with this user account.
:类型: Container
:父节点: ``user``

``keys``

:描述: S3 keys associated with this user account.
:类型: Container
:父节点: ``user``

``swift_keys``

:描述: Swift keys associated with this user account.
:类型: Container
:父节点: ``user``

``caps``

:描述: User capabilities.
:类型: Container
:父节点: ``user``

特殊错误响应
~~~~~~~~~~~~

None.


.. _Create User:

创建用户
========

Create a new user. By Default, a S3 key pair will be created automatically
and returned in the response. If only one of ``access-key`` or ``secret-key``
is provided, the omitted key will be automatically generated. By default, a
generated key is added to the keyring without replacing an existing key pair.
If ``access-key`` is specified and refers to an existing key owned by the user
then it will be modified.

:caps: users=write

语法
~~~~

::

	PUT /{admin}/user?format=json HTTP/1.1
	Host: {fqdn}

Request Parameters
~~~~~~~~~~~~~~~~~~

``uid``

:描述: The user ID to be created.
:类型: String
:实例: ``foo_user``
:是否必需: Yes

``display-name``

:描述: The display name of the user to be created.
:类型: String
:实例: ``foo user``
:是否必需: Yes


``email``

:描述: The email address associated with the user.
:类型: String
:实例: ``foo@bar.com``
:是否必需: No

``key-type``

:描述: Key type to be generated, options are: swift, s3 (default).
:类型: String
:实例: ``s3`` [``s3``]
:是否必需: No


``access-key``

:描述: Specify access key.
:类型: String
:实例: ``ABCD0EF12GHIJ2K34LMN``
:是否必需: No


``secret-key``

:描述: Specify secret key.
:类型: String
:实例: ``0AbCDEFg1h2i34JklM5nop6QrSTUV+WxyzaBC7D8``
:是否必需: No


``user-caps``

:描述: User capabilities.
:类型: String
:实例: ``usage=read, write; users=read``
:是否必需: No

``generate-key``

:描述: Generate a new key pair and add to the existing keyring.
:类型: Boolean
:实例: True [True]
:是否必需: No

``max-buckets``

:描述: Specify the maximum number of buckets the user can own.
:类型: Integer
:实例: 500 [1000]
:是否必需: No

``suspended``

:描述: Specify whether the user should be suspended.
:类型: Boolean
:实例: False [False]
:是否必需: No

Response Entities
~~~~~~~~~~~~~~~~~

If successful, the response contains the user information.

``user``

:描述: A container for the user data information.
:类型: Container

``user_id``

:描述: The user id.
:类型: String
:父节点: ``user``

``display_name``

:描述: Display name for the user.
:类型: String
:父节点: ``user``

``suspended``

:描述: True if the user is suspended.
:类型: Boolean
:父节点: ``user``

``max_buckets``

:描述: The maximum number of buckets to be owned by the user.
:类型: Integer
:父节点: ``user``

``subusers``

:描述: Subusers associated with this user account.
:类型: Container
:父节点: ``user``

``keys``

:描述: S3 keys associated with this user account.
:类型: Container
:父节点: ``user``

``swift_keys``

:描述: Swift keys associated with this user account.
:类型: Container
:父节点: ``user``

``caps``

:描述: User capabilities.
:类型: Container
:父节点: ``user``

特殊错误响应
~~~~~~~~~~~~

``UserExists``

:描述: Attempt to create existing user.
:状态码: 409 Conflict

``InvalidAccessKey``

:描述: Invalid access key specified.
:状态码: 400 Bad Request

``InvalidKey类型``

:描述: Invalid key type specified.
:状态码: 400 Bad Request

``InvalidSecretKey``

:描述: Invalid secret key specified.
:状态码: 400 Bad Request

``InvalidKey类型``

:描述: Invalid key type specified.
:状态码: 400 Bad Request

``KeyExists``

:描述: Provided access key exists and belongs to another user.
:状态码: 409 Conflict

``EmailExists``

:描述: Provided email address exists.
:状态码: 409 Conflict

``InvalidCapability``

:描述: Attempt to grant invalid admin capability.
:状态码: 400 Bad Request


.. _Modify User:

修改用户信息
============

Modify a user.

:caps: users=write

语法
~~~~

::

	POST /{admin}/user?format=json HTTP/1.1
	Host: {fqdn}


Request Parameters
~~~~~~~~~~~~~~~~~~

``uid``

:描述: The user ID to be modified.
:类型: String
:实例: ``foo_user``
:是否必需: Yes

``display-name``

:描述: The display name of the user to be modified.
:类型: String
:实例: ``foo user``
:是否必需: No

``email``

:描述: The email address to be associated with the user.
:类型: String
:实例: ``foo@bar.com``
:是否必需: No

``generate-key``

:描述: Generate a new key pair and add to the existing keyring.
:类型: Boolean
:实例: True [False]
:是否必需: No

``access-key``

:描述: Specify access key.
:类型: String
:实例: ``ABCD0EF12GHIJ2K34LMN``
:是否必需: No

``secret-key``

:描述: Specify secret key.
:类型: String
:实例: ``0AbCDEFg1h2i34JklM5nop6QrSTUV+WxyzaBC7D8``
:是否必需: No

``key-type``

:描述: Key type to be generated, options are: swift, s3 (default).
:类型: String
:实例: ``s3``
:是否必需: No

``user-caps``

:描述: User capabilities.
:类型: String
:实例: ``usage=read, write; users=read``
:是否必需: No

``max-buckets``

:描述: Specify the maximum number of buckets the user can own.
:类型: Integer
:实例: 500 [1000]
:是否必需: No

``suspended``

:描述: Specify whether the user should be suspended.
:类型: Boolean
:实例: False [False]
:是否必需: No

Response Entities
~~~~~~~~~~~~~~~~~

If successful, the response contains the user information.

``user``

:描述: A container for the user data information.
:类型: Container

``user_id``

:描述: The user id.
:类型: String
:父节点: ``user``

``display_name``

:描述: Display name for the user.
:类型: String
:父节点: ``user``


``suspended``

:描述: True if the user is suspended.
:类型: Boolean
:父节点: ``user``


``max_buckets``

:描述: The maximum number of buckets to be owned by the user.
:类型: Integer
:父节点: ``user``


``subusers``

:描述: Subusers associated with this user account.
:类型: Container
:父节点: ``user``


``keys``

:描述: S3 keys associated with this user account.
:类型: Container
:父节点: ``user``


``swift_keys``

:描述: Swift keys associated with this user account.
:类型: Container
:父节点: ``user``


``caps``

:描述: User capabilities.
:类型: Container
:父节点: ``user``


特殊错误响应
~~~~~~~~~~~~

``InvalidAccessKey``

:描述: Invalid access key specified.
:状态码: 400 Bad Request

``InvalidKey类型``

:描述: Invalid key type specified.
:状态码: 400 Bad Request

``InvalidSecretKey``

:描述: Invalid secret key specified.
:状态码: 400 Bad Request

``KeyExists``

:描述: Provided access key exists and belongs to another user.
:状态码: 409 Conflict

``EmailExists``

:描述: Provided email address exists.
:状态码: 409 Conflict

``InvalidCapability``

:描述: Attempt to grant invalid admin capability.
:状态码: 400 Bad Request


.. _Remove User:

删除用户
========

Remove an existing user.

:caps: users=write

语法
~~~~

::

	DELETE /{admin}/user?format=json HTTP/1.1
	Host: {fqdn}


Request Parameters
~~~~~~~~~~~~~~~~~~

``uid``

:描述: The user ID to be removed.
:类型: String
:实例: ``foo_user``
:是否必需: Yes.

``purge-data``

:描述: When specified the buckets and objects belonging
              to the user will also be removed.
:类型: Boolean
:实例: True
:是否必需: No

Response Entities
~~~~~~~~~~~~~~~~~

None

特殊错误响应
~~~~~~~~~~~~

None.


.. _Create Subuser:

创建子用户
==========

新建一个子用户（使用 Swift API 的客户端需要）。提醒一下，要创\
建可正常使用的子用户，必须用 ``access`` 授予权限；创建子用户\
时，如果没给 ``subuser`` 指定密钥 ``secret`` ，会自动生成一个。

:caps: users=write

语法
~~~~

::

	PUT /{admin}/user?subuser&format=json HTTP/1.1
	Host {fqdn}


Request Parameters
~~~~~~~~~~~~~~~~~~

``uid``

:描述: The user ID under which a subuser is to  be created.
:类型: String
:实例: ``foo_user``
:是否必需: Yes


``subuser``

:描述: 指定要创建的子用户 ID 。
:类型: String
:实例: ``sub_foo``
:是否必需: Yes


``secret-key``

:描述: 指定密钥。
:类型: String
:实例: ``0AbCDEFg1h2i34JklM5nop6QrSTUV+WxyzaBC7D8``
:是否必需: No


``key-type``

:描述: Key type to be generated, options are: swift (default), s3.
:类型: String
:实例: ``swift`` [``swift``]
:是否必需: No

``access``

:描述: Set access permissions for sub-user, should be one
              of ``read, write, readwrite, full``.
:类型: String
:实例: ``read``
:是否必需: No

``generate-secret``

:描述: Generate the secret key.
:类型: Boolean
:实例: True [False]
:是否必需: No

Response Entities
~~~~~~~~~~~~~~~~~

If successful, the response contains the subuser information.


``subusers``

:描述: Subusers associated with the user account.
:类型: Container

``id``

:描述: Subuser id.
:类型: String
:父节点: ``subusers``

``permissions``

:描述: Subuser access to user account.
:类型: String
:父节点: ``subusers``

特殊错误响应
~~~~~~~~~~~~

``SubuserExists``

:描述: Specified subuser exists.
:状态码: 409 Conflict

``InvalidKey类型``

:描述: Invalid key type specified.
:状态码: 400 Bad Request

``InvalidSecretKey``

:描述: Invalid secret key specified.
:状态码: 400 Bad Request

``InvalidAccess``

:描述: Invalid subuser access specified.
:状态码: 400 Bad Request


.. _Modify Subuser:

修改子用户信息
==============

Modify an existing subuser

:caps: users=write

语法
~~~~

::

	POST /{admin}/user?subuser&format=json HTTP/1.1
	Host {fqdn}


Request Parameters
~~~~~~~~~~~~~~~~~~

``uid``

:描述: The user ID under which the subuser is to be modified.
:类型: String
:实例: ``foo_user``
:是否必需: Yes

``subuser``

:描述: The subuser ID to be modified.
:类型: String
:实例: ``sub_foo``
:是否必需: Yes

``generate-secret``

:描述: Generate a new secret key for the subuser,
              replacing the existing key.
:类型: Boolean
:实例: True [False]
:是否必需: No

``secret``

:描述: Specify secret key.
:类型: String
:实例: ``0AbCDEFg1h2i34JklM5nop6QrSTUV+WxyzaBC7D8``
:是否必需: No

``key-type``

:描述: Key type to be generated, options are: swift (default), s3 .
:类型: String
:实例: ``swift`` [``swift``]
:是否必需: No

``access``

:描述: Set access permissions for sub-user, should be one
              of ``read, write, readwrite, full``.
:类型: String
:实例: ``read``
:是否必需: No


Response Entities
~~~~~~~~~~~~~~~~~

If successful, the response contains the subuser information.


``subusers``

:描述: Subusers associated with the user account.
:类型: Container

``id``

:描述: Subuser id.
:类型: String
:父节点: ``subusers``

``permissions``

:描述: Subuser access to user account.
:类型: String
:父节点: ``subusers``

特殊错误响应
~~~~~~~~~~~~

``InvalidKey类型``

:描述: Invalid key type specified.
:状态码: 400 Bad Request

``InvalidSecretKey``

:描述: Invalid secret key specified.
:状态码: 400 Bad Request

``InvalidAccess``

:描述: Invalid subuser access specified.
:状态码: 400 Bad Request


.. _Remove Subuser:

删除子用户
==========

Remove an existing subuser

:caps: users=write

语法
~~~~

::

	DELETE /{admin}/user?subuser&format=json HTTP/1.1
	Host {fqdn}


Request Parameters
~~~~~~~~~~~~~~~~~~

``uid``

:描述: The user ID under which the subuser is to be removed.
:类型: String
:实例: ``foo_user``
:是否必需: Yes


``subuser``

:描述: The subuser ID to be removed.
:类型: String
:实例: ``sub_foo``
:是否必需: Yes

``purge-keys``

:描述: Remove keys belonging to the subuser.
:类型: Boolean
:实例: True [True]
:是否必需: No

Response Entities
~~~~~~~~~~~~~~~~~

None.

特殊错误响应
~~~~~~~~~~~~

None.


.. _Create Key:

创建密钥
========

Create a new key. If a ``subuser`` is specified then by default created keys
will be swift type. If only one of ``access-key`` or ``secret-key`` is provided the
committed key will be automatically generated, that is if only ``secret-key`` is
specified then ``access-key`` will be automatically generated. By default, a
generated key is added to the keyring without replacing an existing key pair.
If ``access-key`` is specified and refers to an existing key owned by the user
then it will be modified. The response is a container listing all keys of the same
type as the key created. Note that when creating a swift key, specifying the option
``access-key`` will have no effect. Additionally, only one swift key may be held by
each user or subuser.

:caps: users=write

语法
~~~~

::

	PUT /{admin}/user?key&format=json HTTP/1.1
	Host {fqdn}


Request Parameters
~~~~~~~~~~~~~~~~~~

``uid``

:描述: The user ID to receive the new key.
:类型: String
:实例: ``foo_user``
:是否必需: Yes

``subuser``

:描述: The subuser ID to receive the new key.
:类型: String
:实例: ``sub_foo``
:是否必需: No

``key-type``

:描述: Key type to be generated, options are: swift, s3 (default).
:类型: String
:实例: ``s3`` [``s3``]
:是否必需: No

``access-key``

:描述: Specify the access key.
:类型: String
:实例: ``AB01C2D3EF45G6H7IJ8K``
:是否必需: No

``secret-key``

:描述: Specify the secret key.
:类型: String
:实例: ``0ab/CdeFGhij1klmnopqRSTUv1WxyZabcDEFgHij``
:是否必需: No

``generate-key``

:描述: Generate a new key pair and add to the existing keyring.
:类型: Boolean
:实例: True [``True``]
:是否必需: No


Response Entities
~~~~~~~~~~~~~~~~~

``keys``

:描述: Keys of type created associated with this user account.
:类型: Container

``user``

:描述: The user account associated with the key.
:类型: String
:父节点: ``keys``

``access-key``

:描述: The access key.
:类型: String
:父节点: ``keys``

``secret-key``

:描述: The secret key
:类型: String
:父节点: ``keys``


特殊错误响应
~~~~~~~~~~~~

``InvalidAccessKey``

:描述: Invalid access key specified.
:状态码: 400 Bad Request

``InvalidKey类型``

:描述: Invalid key type specified.
:状态码: 400 Bad Request

``InvalidSecretKey``

:描述: Invalid secret key specified.
:状态码: 400 Bad Request

``InvalidKey类型``

:描述: Invalid key type specified.
:状态码: 400 Bad Request

``KeyExists``

:描述: Provided access key exists and belongs to another user.
:状态码: 409 Conflict


.. _Remove Key:

删除密钥
========

Remove an existing key.

:caps: users=write

语法
~~~~

::

	DELETE /{admin}/user?key&format=json HTTP/1.1
	Host {fqdn}


Request Parameters
~~~~~~~~~~~~~~~~~~

``access-key``

:描述: The S3 access key belonging to the S3 key pair to remove.
:类型: String
:实例: ``AB01C2D3EF45G6H7IJ8K``
:是否必需: Yes

``uid``

:描述: The user to remove the key from.
:类型: String
:实例: ``foo_user``
:是否必需: No

``subuser``

:描述: The subuser to remove the key from.
:类型: String
:实例: ``sub_foo``
:是否必需: No

``key-type``

:描述: Key type to be removed, options are: swift, s3.
              NOTE: 是否必需 to remove swift key.
:类型: String
:实例: ``swift``
:是否必需: No

特殊错误响应
~~~~~~~~~~~~

None.

Response Entities
~~~~~~~~~~~~~~~~~

None.


.. _Get Bucket Info:

查看桶信息
==========

Get information about a subset of the existing buckets. If ``uid`` is specified
without ``bucket`` then all buckets beloning to the user will be returned. If
``bucket`` alone is specified, information for that particular bucket will be
retrieved.

:caps: buckets=read

语法
~~~~

::

	GET /{admin}/bucket?format=json HTTP/1.1
	Host {fqdn}


Request Parameters
~~~~~~~~~~~~~~~~~~

``bucket``

:描述: The bucket to return info on.
:类型: String
:实例: ``foo_bucket``
:是否必需: No

``uid``

:描述: The user to retrieve bucket information for.
:类型: String
:实例: ``foo_user``
:是否必需: No

``stats``

:描述: Return bucket statistics.
:类型: Boolean
:实例: True [False]
:是否必需: No

Response Entities
~~~~~~~~~~~~~~~~~

If successful the request returns a buckets container containing
the desired bucket information.

``stats``

:描述: Per bucket information.
:类型: Container

``buckets``

:描述: Contains a list of one or more bucket containers.
:类型: Container

``bucket``

:描述: Container for single bucket information.
:类型: Container
:父节点: ``buckets``

``name``

:描述: The name of the bucket.
:类型: String
:父节点: ``bucket``

``pool``

:描述: The pool the bucket is stored in.
:类型: String
:父节点: ``bucket``

``id``

:描述: The unique bucket id.
:类型: String
:父节点: ``bucket``

``marker``

:描述: Internal bucket tag.
:类型: String
:父节点: ``bucket``

``owner``

:描述: The user id of the bucket owner.
:类型: String
:父节点: ``bucket``

``usage``

:描述: Storage usage information.
:类型: Container
:父节点: ``bucket``

``index``

:描述: Status of bucket index.
:类型: String
:父节点: ``bucket``

特殊错误响应
~~~~~~~~~~~~

``IndexRepairFailed``

:描述: Bucket index repair failed.
:状态码: 409 Conflict


.. _Check Bucket Index:

检查桶索引
==========

Check the index of an existing bucket. NOTE: to check multipart object
accounting with ``check-objects``, ``fix`` must be set to True.

:caps: buckets=write

语法
~~~~

::

	GET /{admin}/bucket?index&format=json HTTP/1.1
	Host {fqdn}


Request Parameters
~~~~~~~~~~~~~~~~~~

``bucket``

:描述: The bucket to return info on.
:类型: String
:实例: ``foo_bucket``
:是否必需: Yes

``check-objects``

:描述: Check multipart object accounting.
:类型: Boolean
:实例: True [False]
:是否必需: No

``fix``

:描述: Also fix the bucket index when checking.
:类型: Boolean
:实例: False [False]
:是否必需: No

Response Entities
~~~~~~~~~~~~~~~~~

``index``

:描述: Status of bucket index.
:类型: String

特殊错误响应
~~~~~~~~~~~~

``IndexRepairFailed``

:描述: Bucket index repair failed.
:状态码: 409 Conflict


.. _Remove Bucket:

删除桶
======

Delete an existing bucket.

:caps: buckets=write

语法
~~~~

::

	DELETE /{admin}/bucket?format=json HTTP/1.1
	Host {fqdn}


Request Parameters
~~~~~~~~~~~~~~~~~~

``bucket``

:描述: The bucket to remove.
:类型: String
:实例: ``foo_bucket``
:是否必需: Yes

``purge-objects``

:描述: Remove a buckets objects before deletion.
:类型: Boolean
:实例: True [False]
:是否必需: No

Response Entities
~~~~~~~~~~~~~~~~~

None.

特殊错误响应
~~~~~~~~~~~~

``BucketNotEmpty``

:描述: Attempted to delete non-empty bucket.
:状态码: 409 Conflict

``ObjectRemovalFailed``

:描述: Unable to remove objects.
:状态码: 409 Conflict


.. _Unlink Bucket:

解绑桶
======

解绑桶和用户，主要用于更改桶的所有者。

:caps: buckets=write

语法
~~~~

::

	POST /{admin}/bucket?format=json HTTP/1.1
	Host {fqdn}


Request Parameters
~~~~~~~~~~~~~~~~~~

``bucket``

:描述: The bucket to unlink.
:类型: String
:实例: ``foo_bucket``
:是否必需: Yes

``uid``

:描述: The user ID to unlink the bucket from.
:类型: String
:实例: ``foo_user``
:是否必需: Yes

Response Entities
~~~~~~~~~~~~~~~~~

None.

特殊错误响应
~~~~~~~~~~~~

``BucketUnlinkFailed``

:描述: Unable to unlink bucket from specified user.
:状态码: 409 Conflict


.. _Link Bucket:

链接桶
======

把桶链接到指定用户，同时断开与之前用户的链接。

:caps: buckets=write

语法
~~~~

::

	PUT /{admin}/bucket?format=json HTTP/1.1
	Host {fqdn}


请求参数
~~~~~~~~

``bucket``

:描述: 要切断链接的桶。
:类型: String
:实例: ``foo_bucket``
:是否必需: Yes


``bucket-id``

:描述: 要切断链接的桶 id 。
:类型: String
:示例: ``dev.6607669.420``
:是否必需: Yes


``uid``

:描述: 用户的 ID ，桶会被链接到此用户。
:类型: String
:实例: ``foo_user``
:是否必需: Yes

Response Entities
~~~~~~~~~~~~~~~~~

``bucket``

:描述: Container for single bucket information.
:类型: Container

``name``

:描述: The name of the bucket.
:类型: String
:父节点: ``bucket``

``pool``

:描述: The pool the bucket is stored in.
:类型: String
:父节点: ``bucket``

``id``

:描述: The unique bucket id.
:类型: String
:父节点: ``bucket``

``marker``

:描述: Internal bucket tag.
:类型: String
:父节点: ``bucket``

``owner``

:描述: The user id of the bucket owner.
:类型: String
:父节点: ``bucket``

``usage``

:描述: Storage usage information.
:类型: Container
:父节点: ``bucket``

``index``

:描述: Status of bucket index.
:类型: String
:父节点: ``bucket``

特殊错误响应
~~~~~~~~~~~~

``BucketUnlinkFailed``

:描述: Unable to unlink bucket from specified user.
:状态码: 409 Conflict

``BucketLinkFailed``

:描述: Unable to link bucket to specified user.
:状态码: 409 Conflict


.. _Remove Object:

删除对象
========

Remove an existing object. NOTE: Does not require owner to be non-suspended.

:caps: buckets=write

语法
~~~~

::

	DELETE /{admin}/bucket?object&format=json HTTP/1.1
	Host {fqdn}

Request Parameters
~~~~~~~~~~~~~~~~~~

``bucket``

:描述: The bucket containing the object to be removed.
:类型: String
:实例: ``foo_bucket``
:是否必需: Yes

``object``

:描述: The object to remove.
:类型: String
:实例: ``foo.txt``
:是否必需: Yes

Response Entities
~~~~~~~~~~~~~~~~~

None.

特殊错误响应
~~~~~~~~~~~~

``NoSuchObject``

:描述: Specified object does not exist.
:状态码: 404 Not Found

``ObjectRemovalFailed``

:描述: Unable to remove objects.
:状态码: 409 Conflict


.. _Get Bucket or Object Policy:

查看桶或对象的策略
==================

Read the policy of an object or bucket.

:caps: buckets=read

语法
~~~~

::

	GET /{admin}/bucket?policy&format=json HTTP/1.1
	Host {fqdn}


Request Parameters
~~~~~~~~~~~~~~~~~~

``bucket``

:描述: The bucket to read the policy from.
:类型: String
:实例: ``foo_bucket``
:是否必需: Yes

``object``

:描述: The object to read the policy from.
:类型: String
:实例: ``foo.txt``
:是否必需: No

Response Entities
~~~~~~~~~~~~~~~~~

If successful, returns the object or bucket policy

``policy``

:描述: Access control policy.
:类型: Container

特殊错误响应
~~~~~~~~~~~~

``IncompleteBody``

:描述: Either bucket was not specified for a bucket policy request or bucket
              and object were not specified for an object policy request.
:状态码: 400 Bad Request


.. _Add A User Capability:

增加用户能力
============

给指定用户增加管理能力。

:caps: users=write

语法
~~~~

::

	PUT /{admin}/user?caps&format=json HTTP/1.1
	Host {fqdn}

请求参数
~~~~~~~~

``uid``

:描述: 要给增加管理能力的用户 ID 。
:类型: String
:实例: ``foo_user``
:是否必需: Yes

``user-caps``

:描述: 给用户增加的管理能力。
:类型: String
:实例: ``usage=read,write;user=write``
:是否必需: Yes

Response Entities
~~~~~~~~~~~~~~~~~

If successful, the response contains the user's capabilities.

``user``

:描述: A container for the user data information.
:类型: Container
:父节点: ``user``

``user_id``

:描述: The user id.
:类型: String
:父节点: ``user``

``caps``

:描述: User capabilities.
:类型: Container
:父节点: ``user``


特殊错误响应
~~~~~~~~~~~~

``InvalidCapability``

:描述: Attempt to grant invalid admin capability.
:状态码: 400 Bad Request

实例 Request
~~~~~~~~~~~~~~~

::

	PUT /{admin}/user?caps&user-caps=usage=read,write;user=write&format=json HTTP/1.1
	Host: {fqdn}
	Content-类型: text/plain
	Authorization: {your-authorization-token}


.. _Remove A User Capability:

删除用户能力
============

删除指定用户的管理能力。

:caps: users=write

语法
~~~~

::

	DELETE /{admin}/user?caps&format=json HTTP/1.1
	Host {fqdn}

请求参数
~~~~~~~~

``uid``

:描述: The user ID to remove an administrative capability from.
:类型: String
:实例: ``foo_user``
:是否必需: Yes

``user-caps``

:描述: The administrative capabilities to remove from the user.
:类型: String
:实例: ``usage=read, write``
:是否必需: Yes

Response Entities
~~~~~~~~~~~~~~~~~

If successful, the response contains the user's capabilities.

``user``

:描述: A container for the user data information.
:类型: Container
:父节点: ``user``

``user_id``

:描述: The user id.
:类型: String
:父节点: ``user``

``caps``

:描述: User capabilities.
:类型: Container
:父节点: ``user``


特殊错误响应
~~~~~~~~~~~~

``InvalidCapability``

:描述: Attempt to remove an invalid admin capability.
:状态码: 400 Bad Request

``NoSuchCap``

:描述: User does not possess specified capability.
:状态码: 404 Not Found

特殊错误响应
~~~~~~~~~~~~

None.


.. _Quotas:

配额管理
========

你可以用管理操作 API 给用户和用户拥有的桶设置配额，设置细节见\
`配额管理`_\ 。可设置的配额包括桶内对象的最大数量、和最大尺寸\
（单位为 MB ）。

要查看配额信息，用户必须有 ``users=read`` 能力；要设置、修改或\
禁用配额，用户必须有 ``users=write`` 能力。详情见\ `管理指南`_\ 。

管理配额的可用参数有：

- **桶：** ``bucket`` 选项指定了配额针对的是用户拥有的桶。

- **最大对象数：** ``max-objects`` 选项用于指定最大对象数，负\
  数表示禁用此选项。

- **最大尺寸** ``max-size`` 选项用于指定最大字节数，负数表示禁\
  用此选项。

- **配额类型：** ``quota-type`` 选项用于指定配额的适用范围，可\
  以是 ``bucket`` 和 ``user`` 。

- **配额开关：** ``enabled`` 选项用于配置是否开启配额，取值可\
  以是 'True' 或 'False' 。


.. _Get User Quota:

查看用户配额
~~~~~~~~~~~~

To get a quota, the user must have ``users`` capability set with ``read`` 
permission. ::

	GET /admin/user?quota&uid=<uid>&quota-type=user


.. Set User Quota

设置用户配额
~~~~~~~~~~~~

To set a quota, the user must have ``users`` capability set with ``write`` 
permission. ::

	PUT /admin/user?quota&uid=<uid>&quota-type=user

The content must include a JSON representation of the quota settings
as encoded in the corresponding read operation.


.. _Get Bucket Quota:

查看桶配额
~~~~~~~~~~

To get a quota, the user must have ``users`` capability set with ``read`` 
permission. ::

	GET /admin/user?quota&uid=<uid>&quota-type=bucket


.. Set Bucket Quota

设置桶配额
~~~~~~~~~~

To set a quota, the user must have ``users`` capability set with ``write`` 
permission. ::

	PUT /admin/user?quota&uid=<uid>&quota-type=bucket

The content must include a JSON representation of the quota settings
as encoded in the corresponding read operation.


.. _Standard Error Responses:

标准错误响应
============

``AccessDenied``

:描述: 访问被拒绝。
:状态码: 403 Forbidden

``InternalError``

:描述: 服务器内部错误。
:状态码: 500 Internal Server Error

``NoSuchUser``

:描述: 用户不存在。
:状态码: 404 Not Found

``NoSuchBucket``

:描述: 桶不存在。
:状态码: 404 Not Found

``NoSuchKey``

:描述: 此访问密钥不存在。
:状态码: 404 Not Found


.. _管理指南: ../admin
.. _配额管理: ../admin#quota-management
