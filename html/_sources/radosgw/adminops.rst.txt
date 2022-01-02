.. _radosgw admin ops:

==========
 管理操作
==========

管理 API 请求有专用的入口 URI ，是可配置的 'admin' 入口。
管理 API 的授权与 S3 的授权机制相同。
有些操作需要用户拥有特殊的管理权限。
响应实体类型（ XML 或 JSON ）可以在请求中的 'format' 选项指定，
如果没有指定，则默认为 JSON 。

Info
====

查看集群、终结点的信息。

:caps: info=read


语法
~~~~

::

	GET /{admin}/info?format=json HTTP/1.1
	Host: {fqdn}


请求参数
~~~~~~~~

None.


响应内容解析
~~~~~~~~~~~~

如果成功，响应会包含一个 ``info`` 段。

``info``

:描述: 所有返回信息的容器。
:类型: Container

``cluster_id``

:描述: RGW 集群的后端存储（一般是唯一的）标识符。
       在典型案例中，这是 librados::rados::cluster_fsid()
       返回的值。
:类型: String
:父节点: ``info``


特殊错误响应
~~~~~~~~~~~~

None.


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

:描述: 请求哪个用户的信息，如果不指定就是所有用户的。
:类型: String
:实例: ``foo_user``
:是否必需: No

``start``

:描述: 指定所请求数据的起始时间的日期和（可选的）时间。
:类型: String
:实例: ``2012-09-25 16:00:00``
:是否必需: No

``end``

:描述: 指定所请求数据的截止时间的日期和（可选的）时间（不包含）。
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

:描述: 使用率信息的容器。
:类型: Container

``entries``

:描述: 使用率条目信息的容器。
:类型: Container

``user``

:描述: 用户数据信息的容器。
:类型: Container

``owner``

:描述: 这个桶的所有者的名字。
:类型: String

``bucket``

:描述: 桶的名字。
:类型: String

``time``

:描述: 指定数据的时间下限（四舍五入至起始的第一个小时）。
:类型: String

``epoch``

:描述: 从 1/1/1970 起的秒数。
:类型: String

``categories``

:描述: 统计类别的容器。
:类型: Container

``entry``

:描述: 统计条目的容器。
:类型: Container

``category``

:描述: 这些统计信息指向的请求类别的名字。
:类型: String

``bytes_sent``

:描述: RADOS 网关发送的字节数。
:类型: Integer

``bytes_received``

:描述: RADOS 网关接收到的字节数。
:类型: Integer

``ops``

:描述: 操作数量。
:类型: Integer

``successful_ops``

:描述: 成功操作的数量。
:类型: Integer

``summary``

:描述: 统计概要的容器。
:类型: Container

``total``

:描述: 汇总起来的统计概要的容器。
:类型: Container

特殊错误响应
~~~~~~~~~~~~
TBD.


裁剪使用率日志
==============
.. Trim Usage

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

:描述: uid 没指定时必需，为了确认多个用户数据的删除。
:类型: Boolean
:实例: True [False]
:是否必需: No

特殊错误响应
~~~~~~~~~~~~
TBD.


查看用户信息
============
.. Get User Info

查看用户信息。

:caps: users=read


语法
~~~~

::

	GET /{admin}/user?format=json HTTP/1.1
	Host: {fqdn}


请求参数
~~~~~~~~

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

:描述: 这个用户最多可以拥有多少个桶。
:类型: Integer
:父节点: ``user``

``subusers``

:描述: 与此用户账户关联的子用户。
:类型: Container
:父节点: ``user``

``keys``

:描述: 与这个用户账户关联的 S3 密钥。
:类型: Container
:父节点: ``user``

``swift_keys``

:描述: 与这个用户账户关联的 Swift 密钥。
:类型: Container
:父节点: ``user``

``caps``

:描述: 用户能力。
:类型: Container
:父节点: ``user``

特殊错误响应
~~~~~~~~~~~~
None.


创建用户
========
.. Create User

新建一个用户。默认情况下，会自动创建一个 S3 密钥对、并在响应时返回。
如果只提供了一个 ``access-key`` 或 ``secret-key`` ，
缺失的那个密钥会自动生成。
默认情况下，生成的密钥会加进密钥环，而非替换已有的密钥对；
如果指定了 ``access-key`` 且引用的是此用户已有的密钥，此时会修改这个密钥。

.. versionadded:: Luminous

指定租户 ``tenant`` 时，可以作为 uid 的一部分、或单独的请求参数。

:caps: users=write

语法
~~~~

::

	PUT /{admin}/user?format=json HTTP/1.1
	Host: {fqdn}


请求参数
~~~~~~~~
.. Request Parameters

``uid``

:描述: 要创建的用户 ID 。
:类型: String
:实例: ``foo_user``
:是否必需: Yes

``uid`` 可以带上租户名，遵守 ``tenant$user`` 语法就行，
详情请参考\ :ref:`多租户 <rgw-multitenancy>`\ 。

``display-name``

:描述: 要创建用户的显示名字。
:类型: String
:实例: ``foo user``
:是否必需: Yes


``email``

:描述: 与此用户关联的 email 地址。
:类型: String
:实例: ``foo@bar.com``
:是否必需: No

``key-type``

:描述: 要生成的密钥类型，可选的有 swift 、 s3 (默认的)。
:类型: String
:实例: ``s3`` [``s3``]
:是否必需: No

``access-key``

:描述: 指定访问密钥。
:类型: String
:实例: ``ABCD0EF12GHIJ2K34LMN``
:是否必需: No

``secret-key``

:描述: 指定私钥。
:类型: String
:实例: ``0AbCDEFg1h2i34JklM5nop6QrSTUV+WxyzaBC7D8``
:是否必需: No

``user-caps``

:描述: 用户能力。
:类型: String
:实例: ``usage=read, write; users=read``
:是否必需: No

``generate-key``

:描述: 生成一个新密钥对，并加进现有的密钥环。
:类型: Boolean
:实例: True [True]
:是否必需: No

``max-buckets``

:描述: 这个用户最多可以拥有多少个桶。
:类型: Integer
:实例: 500 [1000]
:是否必需: No

``suspended``

:描述: 指定是否挂起这个用户。
:类型: Boolean
:实例: False [False]
:是否必需: No

.. versionadded:: Jewel

``tenant``

:描述: 用户所属的租户。
:类型: string
:实例: tenant1
:是否必需: No


响应内容解析
~~~~~~~~~~~~
.. Response Entities

如果成功了，响应里包含用户信息。

``user``

:描述: 用户数据信息的容器。
:类型: Container

``tenant``

:描述: 用户所属的租户。
:类型: String
:父节点: ``user``

``user_id``

:描述: 此用户的 ID 。
:类型: String
:父节点: ``user``

``display_name``

:描述: 此用户的显示名字。
:类型: String
:父节点: ``user``

``suspended``

:描述: 用户被挂起时此值为 True 。
:类型: Boolean
:父节点: ``user``

``max_buckets``

:描述: 这个用户最多可以拥有多少个桶。
:类型: Integer
:父节点: ``user``

``subusers``

:描述: 与此用户账户关联的子用户。
:类型: Container
:父节点: ``user``

``keys``

:描述: 与此用户关联的 S3 密钥。
:类型: Container
:父节点: ``user``

``swift_keys``

:描述: 与此用户关联的 Swift 密钥。
:类型: Container
:父节点: ``user``

``caps``

:描述: 用户能力。
:类型: Container
:父节点: ``user``

特殊错误响应
~~~~~~~~~~~~

``UserExists``

:描述: 试图创建已存在的用户。
:状态码: 409 Conflict

``InvalidAccessKey``

:描述: 指定了无效的访问密钥。
:状态码: 400 Bad Request

``InvalidSecretKey``

:描述: 指定了无效的私钥。
:状态码: 400 Bad Request

``InvalidKeyType``

:描述: 指定了无效的密钥类型。
:状态码: 400 Bad Request

``KeyExists``

:描述: 提供的访问密钥已存在，但是它属于另外一个用户。
:状态码: 409 Conflict

``EmailExists``

:描述: 提供的邮件地址已存在。
:状态码: 409 Conflict

``InvalidCapability``

:描述: 试图赋予无效的管理员能力。
:状态码: 400 Bad Request


修改用户信息
============
.. Modify User

更改一个用户的信息。

:caps: users=write

语法
~~~~

::

	POST /{admin}/user?format=json HTTP/1.1
	Host: {fqdn}

请求参数
~~~~~~~~

``uid``

:描述: 要更改的用户 ID 。
:类型: String
:实例: ``foo_user``
:是否必需: Yes

``display-name``

:描述: 要更改的用户的显示名字。
:类型: String
:实例: ``foo user``
:是否必需: No

``email``

:描述: 要与此用户关联的 email 地址。
:类型: String
:实例: ``foo@bar.com``
:是否必需: No

``generate-key``

:描述: 生成一个新密钥对，并加进现有的密钥环。
:类型: Boolean
:实例: True [False]
:是否必需: No

``access-key``

:描述: 指定访问密钥。
:类型: String
:实例: ``ABCD0EF12GHIJ2K34LMN``
:是否必需: No

``secret-key``

:描述: 指定私钥。
:类型: String
:实例: ``0AbCDEFg1h2i34JklM5nop6QrSTUV+WxyzaBC7D8``
:是否必需: No

``key-type``

:描述: 要生成的密钥类型，可选的有 swift 、 s3 (默认值)。
:类型: String
:实例: ``s3``
:是否必需: No

``max-buckets``

:描述: 这个用户最多可以拥有多少个桶。
:类型: Integer
:实例: 500 [1000]
:是否必需: No

``suspended``

:描述: 是否挂起此用户。
:类型: Boolean
:实例: False [False]
:是否必需: No

``op-mask``

:描述: 要更改的用户的掩码（ op-mask ）。
:类型: String
:实例: ``read, write, delete, *``
:是否必需: No

响应内容解析
~~~~~~~~~~~~

如果成功了，响应会包含请求的信息。

``user``

:描述: 用户数据信息的容器。
:类型: Container

``user_id``

:描述: 用户的 ID 。
:类型: String
:父节点: ``user``

``display_name``

:描述: 此用户的显示名字。
:类型: String
:父节点: ``user``


``suspended``

:描述: 如果此用户被暂停，此值是 True。
:类型: Boolean
:父节点: ``user``


``max_buckets``

:描述: 这个用户最多可以拥有多少个桶。
:类型: Integer
:父节点: ``user``


``subusers``

:描述: 与此用户账户关联的子用户。
:类型: Container
:父节点: ``user``


``keys``

:描述: 与此用户关联的 S3 密钥。
:类型: Container
:父节点: ``user``

``swift_keys``

:描述: 与此用户关联的 Swift 密钥。
:类型: Container
:父节点: ``user``

``caps``

:描述: 用户能力。
:类型: Container
:父节点: ``user``


特殊错误响应
~~~~~~~~~~~~

``InvalidAccessKey``

:描述: 指定了无效的访问密钥。
:状态码: 400 Bad Request

``InvalidKeyType``

:描述: 指定了无效的密钥类型。
:状态码: 400 Bad Request

``InvalidSecretKey``

:描述: 指定了无效的私钥。
:状态码: 400 Bad Request

``KeyExists``

:描述: 提供的访问密钥已存在，但是它属于另外一个用户。
:状态码: 409 Conflict

``EmailExists``

:描述: 提供的邮件地址已存在。
:状态码: 409 Conflict

``InvalidCapability``

:描述: 试图赋予无效的管理员能力。
:状态码: 400 Bad Request


删除用户
========
.. Remove User

删除一个现有用户。

:caps: users=write

语法
~~~~

::

	DELETE /{admin}/user?format=json HTTP/1.1
	Host: {fqdn}


请求参数
~~~~~~~~

``uid``

:描述: 要删除用户的 ID 。
:类型: String
:实例: ``foo_user``
:是否必需: Yes.

``purge-data``

:描述: 指定后，属于此用户的桶和对象也会被删除。
:类型: Boolean
:实例: True
:是否必需: No

响应内容解析
~~~~~~~~~~~~
None

特殊错误响应
~~~~~~~~~~~~
None.


创建子用户
==========
.. Create Subuser

新建一个子用户（使用 Swift API 的客户端需要）。
提醒一下，要创建可正常使用的子用户，
必须用 ``access`` 授予权限；创建子用户时，
如果没给 ``subuser`` 指定密钥 ``secret`` ，会自动生成一个。

:caps: users=write

语法
~~~~

::

	PUT /{admin}/user?subuser&format=json HTTP/1.1
	Host: {fqdn}

请求参数
~~~~~~~~
.. Request Parameters

``uid``

:描述: 子用户在哪个用户 ID 下创建。
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

:描述: 要生成的密钥类型，可选的有 swift (默认值)、 s3 。
:类型: String
:实例: ``swift`` [``swift``]
:是否必需: No

``access``

:描述: 设置子用户的访问权限，应该是 ``read, write, readwrite, full`` 其中之一。
:类型: String
:实例: ``read``
:是否必需: No

``generate-secret``

:描述: 生成密钥。
:类型: Boolean
:实例: True [False]
:是否必需: No

响应内容解析
~~~~~~~~~~~~

如果成功了，响应里包含子用户信息。

``subusers``

:描述: 与此用户账户关联的子用户。
:类型: Container

``id``

:描述: 子用户的 ID 。
:类型: String
:父节点: ``subusers``

``permissions``

:描述: 子用户访问用户账户的权限。
:类型: String
:父节点: ``subusers``

特殊错误响应
~~~~~~~~~~~~

``SubuserExists``

:描述: 指定的子用户已存在。
:状态码: 409 Conflict

``InvalidKeyType``

:描述: 指定的密钥类型无效。
:状态码: 400 Bad Request

``InvalidSecretKey``

:描述: 指定的私钥无效。
:状态码: 400 Bad Request

``InvalidAccess``

:描述: 指定的子用户权限无效。
:状态码: 400 Bad Request


修改子用户信息
==============
.. Modify Subuser

更改现有的子用户。

:caps: users=write

语法
~~~~

::

	POST /{admin}/user?subuser&format=json HTTP/1.1
	Host: {fqdn}

请求参数
~~~~~~~~

``uid``

:描述: 要修改的子用户所属的用户 ID 。
:类型: String
:实例: ``foo_user``
:是否必需: Yes

``subuser``

:描述: 要更改的子用户的 ID 。
:类型: String
:实例: ``sub_foo``
:是否必需: Yes

``generate-secret``

:描述: 给这个子用户生成一个新的私钥，来替换现有密钥。
:类型: Boolean
:实例: True [False]
:是否必需: No

``secret``

:描述: 指定私钥。
:类型: String
:实例: ``0AbCDEFg1h2i34JklM5nop6QrSTUV+WxyzaBC7D8``
:是否必需: No

``key-type``

:描述: 要生成的密钥类型，可选的有 swift （默认值）、 s3 。
:类型: String
:实例: ``swift`` [``swift``]
:是否必需: No

``access``

:描述: 设置子用户的访问权限，应该是 ``read, write, readwrite, full`` 里面的。
:类型: String
:实例: ``read``
:是否必需: No

响应内容解析
~~~~~~~~~~~~

如果成功了，响应里包含子用户信息。


``subusers``

:描述: 与此用户账户关联的子用户。
:类型: Container

``id``

:描述: 子用户的 ID 。
:类型: String
:父节点: ``subusers``

``permissions``

:描述: 子用户对用户账户的访问权限。
:类型: String
:父节点: ``subusers``

特殊错误响应
~~~~~~~~~~~~

``InvalidKeyType``

:描述: 指定的密钥类型无效。
:状态码: 400 Bad Request

``InvalidSecretKey``

:描述: 指定的私钥无效。
:状态码: 400 Bad Request

``InvalidAccess``

:描述: 指定的子用户权限无效。
:状态码: 400 Bad Request


删除子用户
==========
.. Remove Subuser

删除一个现有子用户。

:caps: users=write

语法
~~~~

::

	DELETE /{admin}/user?subuser&format=json HTTP/1.1
	Host: {fqdn}


请求参数
~~~~~~~~

``uid``

:描述: 要删除子用户所属的用户 ID 。
:类型: String
:实例: ``foo_user``
:是否必需: Yes


``subuser``

:描述: 要删除的子用户 ID 。
:类型: String
:实例: ``sub_foo``
:是否必需: Yes

``purge-keys``

:描述: 删除子用户的密钥。
:类型: Boolean
:实例: True [True]
:是否必需: No

响应内容解析
~~~~~~~~~~~~

None.

特殊错误响应
~~~~~~~~~~~~

None.


创建密钥
========
.. Create Key

创建一个新密钥。如果指定了 ``subuser`` ，默认会创建 swift 类型的密钥。
如果只指定了 ``access-key`` 或 ``secret-key`` 其中之一，
另一个密钥也会自动生成，也就是说，如果只指定了 ``secret-key`` ，
那么 ``access-key`` 会自动生成。默认情况下，
生成的密钥会被加进密钥环，而不是替换已经存在的密钥对。
如果指定的是 ``access-key`` ，且引用的是这个用户已经存在的密钥，
就会修改它。响应结果是一个容器，罗列了和刚创建密钥同一类型的所有密钥。
注意，创建 swift 密钥时，指定 ``access-key`` 选项无效；
另外，每个用户或子用户只能持有一个 swift 密钥。

:caps: users=write

语法
~~~~

::

	PUT /{admin}/user?key&format=json HTTP/1.1
	Host: {fqdn}


请求参数
~~~~~~~~

``uid``

:描述: 接收新密钥的用户 ID 。
:类型: String
:实例: ``foo_user``
:是否必需: Yes

``subuser``

:描述: 接收新密钥的子用户 ID 。
:类型: String
:实例: ``sub_foo``
:是否必需: No

``key-type``

:描述: 要生成的密钥类型，可选的有 swift 、 s3 （默认值）。
:类型: String
:实例: ``s3`` [``s3``]
:是否必需: No

``access-key``

:描述: 指定访问密钥。
:类型: String
:实例: ``AB01C2D3EF45G6H7IJ8K``
:是否必需: No

``secret-key``

:描述: 指定私钥。
:类型: String
:实例: ``0ab/CdeFGhij1klmnopqRSTUv1WxyZabcDEFgHij``
:是否必需: No

``generate-key``

:描述: 生成一个新的密钥对，并加进现有的密钥环。
:类型: Boolean
:实例: True [``True``]
:是否必需: No


响应内容解析
~~~~~~~~~~~~

``keys``

:描述: 与此用户账户关联的、创建的密钥类型。
:类型: Container

``user``

:描述: 和密钥关联的用户账户。
:类型: String
:父节点: ``keys``

``access-key``

:描述: 访问密钥。
:类型: String
:父节点: ``keys``

``secret-key``

:描述: 密钥。
:类型: String
:父节点: ``keys``


特殊错误响应
~~~~~~~~~~~~

``InvalidAccessKey``

:描述: 指定的访问密钥无效。
:状态码: 400 Bad Request

``InvalidSecretKey``

:描述: 指定的私钥无效。
:状态码: 400 Bad Request

``InvalidKeyType``

:描述: 指定的密钥类型无效。
:状态码: 400 Bad Request

``KeyExists``

:描述: 提供的访问密钥存在，但是它属于另外一个用户。
:状态码: 409 Conflict


删除密钥
========
.. Remove Key

删除一个存在的密钥。

:caps: users=write

语法
~~~~

::

	DELETE /{admin}/user?key&format=json HTTP/1.1
	Host: {fqdn}

请求参数
~~~~~~~~

``access-key``

:描述: 要删除的 S3 密钥对里的访问密钥。
:类型: String
:实例: ``AB01C2D3EF45G6H7IJ8K``
:是否必需: Yes

``uid``

:描述: 要删除密钥的用户。
:类型: String
:实例: ``foo_user``
:是否必需: No

``subuser``

:描述: 要删除密钥的子用户。
:类型: String
:实例: ``sub_foo``
:是否必需: No

``key-type``

:描述: 要删除的密钥类型，可选的有 swift 、 s3 。
       注意：删除 swift 密钥时必须提供。
:类型: String
:实例: ``swift``
:是否必需: No

特殊错误响应
~~~~~~~~~~~~

None.

响应内容解析
~~~~~~~~~~~~~~~~~

None.


查看桶信息
==========
.. Get Bucket Info

获取一部分已有桶的相关信息。如果指定了 ``uid`` 却没有 ``bucket`` ，
就会得到属于此用户的所有桶；如果还指定了 ``bucket`` ，
就只去检索那一个桶的信息。

:caps: buckets=read

语法
~~~~

::

	GET /{admin}/bucket?format=json HTTP/1.1
	Host: {fqdn}

请求参数
~~~~~~~~

``bucket``

:描述: 返回信息的桶。
:类型: String
:实例: ``foo_bucket``
:是否必需: No

``uid``

:描述: 为哪个用户检索桶信息。
:类型: String
:实例: ``foo_user``
:是否必需: No

``stats``

:描述: 返回桶的统计信息。
:类型: Boolean
:实例: True [False]
:是否必需: No

响应内容解析
~~~~~~~~~~~~

如果成功，这个请求会返回一个桶容器，
包含着想要的桶信息。

``stats``

:描述: 单个桶的信息 。
:类型: Container

``buckets``

:描述: 包含一个或多个桶容器的列表。
:类型: Container

``bucket``

:描述: 单个桶的信息容器。
:类型: Container
:父节点: ``buckets``

``name``

:描述: 桶的名字。
:类型: String
:父节点: ``bucket``

``pool``

:描述: 桶所在的存储池。
:类型: String
:父节点: ``bucket``

``id``

:描述: 唯一的桶 ID 。
:类型: String
:父节点: ``bucket``

``marker``

:描述: 内部的桶标签。
:类型: String
:父节点: ``bucket``

``owner``

:描述: 桶所有者的用户 ID 。
:类型: String
:父节点: ``bucket``

``usage``

:描述: 存储使用率信息。
:类型: Container
:父节点: ``bucket``

``index``

:描述: 桶索引的状态。
:类型: String
:父节点: ``bucket``

特殊错误响应
~~~~~~~~~~~~

``IndexRepairFailed``

:描述: 桶索引修复失败了。
:状态码: 409 Conflict


检查桶索引
==========
.. Check Bucket Index

检查一个现有桶的索引。注意，要检查多块对象的记帐信息\
需要加 ``check-objects`` ， ``fix`` 必须设置为 True 。

:caps: buckets=write

语法
~~~~

::

	GET /{admin}/bucket?index&format=json HTTP/1.1
	Host: {fqdn}

请求参数
~~~~~~~~

``bucket``

:描述: 返回信息的桶。
:类型: String
:实例: ``foo_bucket``
:是否必需: Yes

``check-objects``

:描述: 检查分块对象的记账信息。
:实例: True [False]
:是否必需: No

``fix``

:描述: 检查时顺便修复桶索引。
:类型: Boolean
:实例: False [False]
:是否必需: No

响应内容解析
~~~~~~~~~~~~

``index``

:描述: 桶索引的状态。
:类型: String

特殊错误响应
~~~~~~~~~~~~

``IndexRepairFailed``

:描述: 桶索引修复失败了。
:状态码: 409 Conflict


删除桶
======
.. Remove Bucket

删除一个存在的桶。

:caps: buckets=write

语法
~~~~

::

	DELETE /{admin}/bucket?format=json HTTP/1.1
	Host: {fqdn}

请求参数
~~~~~~~~

``bucket``

:描述: 要删除的桶。
:类型: String
:实例: ``foo_bucket``
:是否必需: Yes

``purge-objects``

:描述: 删除桶前先删光里面的对象。
:类型: Boolean
:实例: True [False]
:是否必需: No

响应内容解析
~~~~~~~~~~~~~~~~~

None.

特殊错误响应
~~~~~~~~~~~~

``BucketNotEmpty``

:描述: 试图删除非空的桶。
:状态码: 409 Conflict

``ObjectRemovalFailed``

:描述: 无法删除对象。
:状态码: 409 Conflict


解绑桶
======
.. Unlink Bucket

解绑桶和用户，主要用于更改桶的所有者。

:caps: buckets=write

语法
~~~~

::

	POST /{admin}/bucket?format=json HTTP/1.1
	Host: {fqdn}


请求参数
~~~~~~~~

``bucket``

:描述: 要解除连接的桶。
:类型: String
:实例: ``foo_bucket``
:是否必需: Yes

``uid``

:描述: 要断开桶与哪个用户 ID 的连接。
:类型: String
:实例: ``foo_user``
:是否必需: Yes

响应内容解析
~~~~~~~~~~~~

None.

特殊错误响应
~~~~~~~~~~~~

``BucketUnlinkFailed``

:描述: 不能断开桶与指定用户的链接。
:状态码: 409 Conflict


链接桶
======
.. Link Bucket

把桶链接到指定用户，同时断开与之前用户的链接。

:caps: buckets=write

语法
~~~~

::

	PUT /{admin}/bucket?format=json HTTP/1.1
	Host: {fqdn}

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

响应内容解析
~~~~~~~~~~~~

``bucket``

:描述: 单个桶的信息容器。
:类型: Container

``name``

:描述: 桶的名字。
:类型: String
:父节点: ``bucket``

``pool``

:描述: 桶所在的存储池。
:类型: String
:父节点: ``bucket``

``id``

:描述: 唯一的桶 ID 。
:类型: String
:父节点: ``bucket``

``marker``

:描述: 内部的桶标签。
:类型: String
:父节点: ``bucket``

``owner``

:描述: 桶所有者的用户 ID 。
:类型: String
:父节点: ``bucket``

``usage``

:描述: 存储使用率信息。
:类型: Container
:父节点: ``bucket``

``index``

:描述: 桶索引的状态。
:类型: String
:父节点: ``bucket``

特殊错误响应
~~~~~~~~~~~~

``BucketUnlinkFailed``

:描述: 不能断开桶到用户的链接，
:状态码: 409 Conflict

``BucketLinkFailed``

:描述: 不能把桶链接到指定的用户。
:状态码: 409 Conflict


删除对象
========
.. Remove Object

删除一个存在的对象。注意：不要求所有者是没被暂停的。

:caps: buckets=write

语法
~~~~

::

	DELETE /{admin}/bucket?object&format=json HTTP/1.1
	Host: {fqdn}

请求参数
~~~~~~~~

``bucket``

:描述: 要删除的对象所在的桶。
:类型: String
:实例: ``foo_bucket``
:是否必需: Yes

``object``

:描述: 要删除的对象。
:类型: String
:实例: ``foo.txt``
:是否必需: Yes

响应内容解析
~~~~~~~~~~~~
None.

特殊错误响应
~~~~~~~~~~~~

``NoSuchObject``

:描述: 指定的对象不存在。
:状态码: 404 Not Found

``ObjectRemovalFailed``

:描述: 无法删除对象。
:状态码: 409 Conflict


查看桶或对象的策略
==================
.. Get Bucket or Object Policy

读取一个对象或桶的策略。

:caps: buckets=read

语法
~~~~

::

	GET /{admin}/bucket?policy&format=json HTTP/1.1
	Host: {fqdn}

请求参数
~~~~~~~~

``bucket``

:描述: 从哪个桶读取策略。
:类型: String
:实例: ``foo_bucket``
:是否必需: Yes

``object``

:描述: 从哪个对象读取策略。
:实例: ``foo.txt``
:是否必需: No

响应内容解析
~~~~~~~~~~~~

如果成功了，返回此对象或桶的策略。

``policy``

:描述: 访问控制策略。
:类型: Container

特殊错误响应
~~~~~~~~~~~~

``IncompleteBody``

:描述: 桶策略请求中没有指定桶，或者对象策略请求中没有指定桶和对象。
:状态码: 400 Bad Request


增加用户能力
============
.. Add A User Capability

给指定用户增加管理能力。

:caps: users=write

语法
~~~~

::

	PUT /{admin}/user?caps&format=json HTTP/1.1
	Host: {fqdn}

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

响应内容解析
~~~~~~~~~~~~

如果成功了，响应里包含用户的能力。

``user``

:描述: 用户数据信息的容器。
:类型: Container
:父节点: ``user``

``user_id``

:描述: 此用户的 ID 。
:类型: String
:父节点: ``user``

``caps``

:描述: 用户的能力。
:类型: Container
:父节点: ``user``

特殊错误响应
~~~~~~~~~~~~

``InvalidCapability``

:描述: 试图授予无效的管理能力。
:状态码: 400 Bad Request

请求实例
~~~~~~~~

::

	PUT /{admin}/user?caps&user-caps=usage=read,write;user=write&format=json HTTP/1.1
	Host: {fqdn}
	Content-类型: text/plain
	Authorization: {your-authorization-token}


删除用户能力
============
.. Remove A User Capability

删除指定用户的管理能力。

:caps: users=write

语法
~~~~

::

	DELETE /{admin}/user?caps&format=json HTTP/1.1
	Host: {fqdn}

请求参数
~~~~~~~~

``uid``

:描述: 要删除这个用户 ID 的管理能力。
:类型: String
:实例: ``foo_user``
:是否必需: Yes

``user-caps``

:描述: 要为此用户删除的管理能力。
:类型: String
:实例: ``usage=read, write``
:是否必需: Yes

响应内容解析
~~~~~~~~~~~~

如果成功了，响应里包含用户的能力。

``user``

:描述: 用户数据信息的容器。
:类型: Container
:父节点: ``user``

``user_id``

:描述: 此用户的 ID 。
:类型: String
:父节点: ``user``

``caps``

:描述: 用户的能力。
:类型: Container
:父节点: ``user``


特殊错误响应
~~~~~~~~~~~~

``InvalidCapability``

:描述: 试图删除一个无效的管理员能力。
:状态码: 400 Bad Request

``NoSuchCap``

:描述: 用户没有指定的能力。
:状态码: 404 Not Found


配额管理
========
.. Quotas

你可以用管理操作 API 给用户和用户拥有的桶设置配额，设置细节见\
`配额管理`_\ 。可设置的配额包括桶内对象的最大数量、和最大尺寸\
（单位为 MB ）。

要查看配额信息，用户必须有 ``users=read`` 能力；
要设置、修改或禁用配额，用户必须有 ``users=write`` 能力。
详情见\ `管理指南`_\ 。

管理配额的可用参数有：

- **桶：** ``bucket`` 选项指定了配额针对的是用户拥有的桶。

- **最大对象数：** ``max-objects`` 选项用于指定最大对象数，
  负数表示禁用此选项。

- **最大尺寸** ``max-size`` 选项用于指定最大字节数，
  ``max-size-kb`` 选项指定的以 KiB 为单位。
  负数表示禁用此选项。

- **配额类型：** ``quota-type`` 选项用于指定配额的适用范围，
  可以是 ``bucket`` 和 ``user`` 。

- **配额开关：** ``enabled`` 选项用于配置是否开启配额，
  取值可以是 'True' 或 'False' 。

查看用户配额
~~~~~~~~~~~~
.. Get User Quota

要查看配额信息，此用户必须有 ``users`` 能力的 ``read`` 权限。 ::

	GET /admin/user?quota&uid=<uid>&quota-type=user


设置用户配额
~~~~~~~~~~~~
.. Set User Quota

要设置配额，此用户必须有 ``users`` 能力的 ``write`` 权限。 ::

	PUT /admin/user?quota&uid=<uid>&quota-type=user

内容必须包含一个 JSON 格式的配额设置信息，
编码应该和读操作对应。


查看桶配额
~~~~~~~~~~
.. Get Bucket Quota

要查看配额信息，此用户必须有 ``users`` 能力的 ``read`` 权限。 ::

	GET /admin/user?quota&uid=<uid>&quota-type=bucket


设置桶配额
~~~~~~~~~~
.. Set Bucket Quota

要设置配额，此用户必须有 ``users`` 能力的 ``write`` 权限。 ::

	PUT /admin/user?quota&uid=<uid>&quota-type=bucket

内容必须包含一个 JSON 格式的配额设置信息，
编码应该和读操作对应。


设置个人桶的配额
~~~~~~~~~~~~~~~~
.. Set Quota for an Individual Bucket

要设置配额，此用户必须有设置了 ``write`` 权限的
``buckets`` 能力。 ::

	PUT /admin/bucket?quota&uid=<uid>&bucket=<bucket-name>

其内容必须包含一个以 JSON 格式表达的配额配置，如前面\
`设置桶配额`_\ 一节所述。


标准错误响应
============
.. Standard Error Responses

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




绑定库
======
.. Binding libraries

``Golang``

 - `IrekFasikhov/go-rgwadmin`_
 - `QuentinPerez/go-radosgw`_

``Java``

 - `twonote/radosgw-admin4j`_

``Python``

 - `UMIACS/rgwadmin`_
 - `valerytschopp/python-radosgw-admin`_



.. _管理指南: ../admin
.. _配额管理: ../admin#quota-management
.. _IrekFasikhov/go-rgwadmin: https://github.com/IrekFasikhov/go-rgwadmin
.. _QuentinPerez/go-radosgw: https://github.com/QuentinPerez/go-radosgw
.. _twonote/radosgw-admin4j: https://github.com/twonote/radosgw-admin4j
.. _UMIACS/rgwadmin: https://github.com/UMIACS/rgwadmin
.. _valerytschopp/python-radosgw-admin: https://github.com/valerytschopp/python-radosgw-admin

