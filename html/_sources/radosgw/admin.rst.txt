==========
 管理指南
==========

你配置好 Ceph 对象存储服务并运行正常之后，就可以管理服务了，
有用户管理、访问控制、配额管理、和使用情况跟踪等功能。


.. _radosgw-user-management:

用户管理
========
.. User Management:

Ceph 对象存储的用户管理指的是 Ceph 对象存储服务的用户（换句话说，
不是 Ceph 对象网关作为 Ceph 存储集群的一个用户）。你必须创建一个用户、
访问密钥和私钥，这样最终用户才能和 Ceph 对象网关服务交互。
为了便于管理，还可以选择让用户归属于 `Accounts`_ 。

有两种用户类型：

- **用户:** 'user' 这个术语反映的是 S3 接口的用户。

- **子用户:** 'subuser' 这个术语反映的是 Swift 接口的用户。子用户关联到了用户。

.. ditaa::

           +---------+
           | Account |
           +----+----+  
                |     
                |     +---------+
                +-----+  User   |
                      +----+----+
                           |
                           |     +-----------+
                           +-----+  Subuser  |
                                 +-----------+

Users and subusers can be created, modified, viewed, suspended and removed.
you may add a Display names and an email addresses can be added to user
profiles. Keys and secrets can either be specified or generated automatically.
When generating or specifying keys, remember that user IDs correspond to S3 key
types and subuser IDs correspond to Swift key types. 

Swift keys have access levels of ``read``, ``write``, ``readwrite`` and
``full``.


创建用户
--------
.. Create a User

To create a user (S3 interface), run a command of the following form:

.. prompt:: bash

   radosgw-admin user create --uid={username} --display-name="{display-name}" [--email={email}]

例如：

.. prompt:: bash
	
   radosgw-admin user create --uid=johndoe --display-name="John Doe" --email=john@example.com
  
.. code-block:: javascript
  
  { "user_id": "johndoe",
    "display_name": "John Doe",
    "email": "john@example.com",
    "suspended": 0,
    "max_buckets": 1000,
    "subusers": [],
    "keys": [
          { "user": "johndoe",
            "access_key": "11BS02LGFB6AL6H1ADMW",
            "secret_key": "vzCEkuryfn060dfee4fgQPqFrncKEIkh3ZcdOANY"}],
    "swift_keys": [],
    "caps": [],
    "op_mask": "read, write, delete",
    "default_placement": "",
    "placement_tags": [],
    "bucket_quota": { "enabled": false,
        "max_size_kb": -1,
        "max_objects": -1},
    "user_quota": { "enabled": false,
        "max_size_kb": -1,
        "max_objects": -1},
    "temp_url_keys": []}

The creation of a user entails the creation of an ``access_key`` and a
``secret_key`` entry, which can be used with any S3 API-compatible client.  

.. important:: Check the key output. Sometimes ``radosgw-admin`` generates a
   JSON escape (``\``) character, and some clients do not know how to handle
   JSON escape characters. Remedies include removing the JSON escape character
   (``\``), encapsulating the string in quotes, regenerating the key and
   ensuring that it does not have a JSON escape character, or specifying the
   key and secret manually.


创建子用户
----------
.. Create a Subuser

要创建用户的子用户（ Swift 接口），必须指定用户 ID （
``--uid={username}`` ）、子用户 ID 和这个子用户的访问级别。

.. prompt:: bash

   radosgw-admin subuser create --uid={uid} --subuser={uid} --access=[ read | write | readwrite | full ]

例如：

.. prompt:: bash

   radosgw-admin subuser create --uid=johndoe --subuser=johndoe:swift --access=full


.. note:: ``full`` 和 ``readwrite`` 不一样。 ``full`` 访问级别包括
   ``read`` 和 ``write`` ，而且还包括访问控制策略。

.. code-block:: javascript

  { "user_id": "johndoe",
    "display_name": "John Doe",
    "email": "john@example.com",
    "suspended": 0,
    "max_buckets": 1000,
    "subusers": [
          { "id": "johndoe:swift",
            "permissions": "full-control"}],
    "keys": [
          { "user": "johndoe",
            "access_key": "11BS02LGFB6AL6H1ADMW",
            "secret_key": "vzCEkuryfn060dfee4fgQPqFrncKEIkh3ZcdOANY"}],
    "swift_keys": [],
    "caps": [],
    "op_mask": "read, write, delete",
    "default_placement": "",
    "placement_tags": [],
    "bucket_quota": { "enabled": false,
        "max_size_kb": -1,
        "max_objects": -1},
    "user_quota": { "enabled": false,
        "max_size_kb": -1,
        "max_objects": -1},
    "temp_url_keys": []}


获取用户信息
------------
.. Get User Info

要获取某一用户的信息，可指定 ``user info`` 和用户 ID （ ``--uid={username}`` ）。
执行下列命令：

.. prompt:: bash

   radosgw-admin user info --uid=johndoe


修改用户信息
------------
.. Modify User Info

To modify information about a user, specify the user ID (``--uid={username}``)
and the attributes that you want to modify. Typical modifications are made to
keys and secrets, email addresses, display names, and access levels. Use a
command of the following form: 

.. prompt:: bash

   radosgw-admin user modify --uid=johndoe --display-name="John E. Doe"

To modify subuser values, specify ``subuser modify``, user ID and the subuser
ID. Use a command of the following form:

.. prompt:: bash

   radosgw-admin subuser modify --uid=johndoe --subuser=johndoe:swift --access=full


User Suspend
------------

When a user is created, the user is enabled by default. However, it is possible
to suspend user privileges and to re-enable them at a later time. To suspend a
user, specify ``user suspend`` and the user ID in a command of the following
form:

.. prompt:: bash

   radosgw-admin user suspend --uid=johndoe

User Enable
-----------
To re-enable a suspended user, provide ``user enable`` and specify the user ID
in a command of the following form:

.. prompt:: bash

   radosgw-admin user enable --uid=johndoe

.. note:: Disabling the user also disables any subusers.


删除用户
--------
.. Remove a User

删除用户时，这个用户以及他的子用户都会被删除。

可以只删除子用户。
It is possible to remove a subuser without removing its associated user. This
is covered in the section called :ref:`Remove a Subuser <radosgw-admin-remove-a-subuser>`.

要删除用户（及其子用户），可指定 ``user rm`` 和用户 ID ：

.. prompt:: bash

   radosgw-admin user rm --uid=johndoe

选项有：

- **清除数据：** 加 ``--purge-data`` 选项可清除与此 UID 相关的所有\
  数据。

- **清除密钥：** 加 ``--purge-keys`` 选项可清除与此 UID 相关的所有\
  密钥。


.. _radosgw-admin-remove-a-subuser:

删除子用户
----------
.. Remove a Subuser

你删除子用户的同时，也失去了 Swift 接口的访问方式，但是这个用\
户还在系统中存在。

要删除子用户，可指定 ``subuser rm`` 及子用户 ID ：

.. prompt:: bash

   radosgw-admin subuser rm --subuser=johndoe:swift

选项有：

- **清除密钥：** 加 ``--purge-keys`` 选项可清除与此 UID 相关的\
  所有密钥。


增加、删除密钥
--------------
.. Add or  Remove a Key

用户和子用户都必须有密钥才能访问 S3 或 Swift 接口。用 S3 访问\
时，用户需要一个由访问密钥和私钥组成的密钥对；而用 Swift 访问\
时，通常只需要一个私钥（密码），并且要和相关的用户 ID 一起用\
才行。你可以创建密钥，并指定或生成访问密钥和/或私钥；也可以\
删除密钥。相关选项有：

- ``--key-type=<type>`` 指定密钥类型，选项有： s3 、 swift ；
- ``--access-key=<key>`` 手动指定 S3 的访问密钥；
- ``--secret-key=<key>`` 手动指定 S3 私钥或者 Swift 私钥；
- ``--gen-access-key`` 自动生成随机的 S3 访问密钥；
- ``--gen-secret`` 自动生成一个随机的 S3 私钥或随机的 Swift 私钥。
- ``--generate-key`` create user with or without credentials. If sets to false, then user cannot set ``gen-secret/gen-access-key/access-key/secret-key``

Adding S3 keys
~~~~~~~~~~~~~~

给用户人为指定 S3 密钥对的实例如下：

.. prompt:: bash

   radosgw-admin key create --uid=foo --key-type=s3 --access-key fooAccessKey --secret-key fooSecretKey

.. code-block:: javascript

  { "user_id": "foo",
    "rados_uid": 0,
    "display_name": "foo",
    "email": "foo@example.com",
    "suspended": 0,
    "keys": [
      { "user": "foo",
        "access_key": "fooAccessKey",
        "secret_key": "fooSecretKey"}],
  }

.. note:: 你可以给一个用户创建多个 S3 密钥对。

Adding Swift secret keys
~~~~~~~~~~~~~~~~~~~~~~~~

给一个子用户配置指定的 swift 私钥：

.. prompt:: bash

   radosgw-admin key create --subuser=foo:bar --key-type=swift --secret-key barSecret

.. code-block:: javascript

  { "user_id": "foo",
    "rados_uid": 0,
    "display_name": "foo",
    "email": "foo@example.com",
    "suspended": 0,
    "subusers": [
       { "id": "foo:bar",
         "permissions": "full-control"}],
    "swift_keys": [
      { "user": "foo:bar",
        "secret_key": "asfghjghghmgm"}]}

.. note:: 一个子用户只能有一个 swift 私钥。

Associating subusers with S3 key pairs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

如果将子用户与 S3 密钥对关联，那么这些子用户也能用于 S3 API ，执行下列命令：

.. prompt:: bash

   radosgw-admin key create --subuser=foo:bar --key-type=s3 --access-key barAccessKey --secret-key barSecretKey
	
.. code-block:: javascript

.. code-block:: javascript

  { "user_id": "foo",
    "rados_uid": 0,
    "display_name": "foo",
    "email": "foo@example.com",
    "suspended": 0,
    "subusers": [
       { "id": "foo:bar",
         "permissions": "full-control"}],
    "keys": [
      { "user": "foo:bar",
        "access_key": "barAccessKey",
        "secret_key": "barSecretKey"}],
  }

Removing S3 key pairs
~~~~~~~~~~~~~~~~~~~~~

要删除一个 S3 密钥对，需指定访问密钥。

.. prompt:: bash

   radosgw-admin key rm --uid=foo --key-type=s3 --access-key=fooAccessKey 

Removing Swift secret keys
~~~~~~~~~~~~~~~~~~~~~~~~~~

删除 swift 私钥。

.. prompt:: bash

   radosgw-admin key rm --subuser=foo:bar --key-type=swift


增加、删除管理能力
------------------
.. Add or Remove Admin Capabilities

Ceph 存储集群提供了一个管理 API ，用户可以通过 REST API 使用管\
理功能。默认情况下，用户\ **无权**\ 访问这个 API ，给用户分配\
管理能力后，他才能使用管理功能。

要给用户分配管理能力，执行下列命令：

.. prompt:: bash

	radosgw-admin caps add --uid={uid} --caps={caps}


你可以给 users 、 buckets 、 metadata 和 usage （利用率）分配
read 、 write 或 all 能力，例如：

.. prompt:: bash

   --caps="[users|buckets|metadata|usage|zone|amz-cache|info|bilog|mdlog|datalog|user-policy|oidc-provider|roles|ratelimit|user-info-without-keys]=[\*|read|write|read, write]"

例如：

.. prompt:: bash

	radosgw-admin caps add --uid=johndoe --caps="users=*;buckets=*"


要删除某用户的管理能力，可用下面的命令：

.. prompt:: bash

	radosgw-admin caps rm --uid=johndoe --caps={caps}


配额管理
========
.. Quota Management

Ceph 对象网关允许你给用户及其拥有的桶设置配额，可设置的配额有\
桶内的最大对象数、和桶可以存储的最大数据尺寸。

- **桶：** 加 ``--bucket`` 选项说明配额操作作用于用户拥有的桶。

- **最大对象数：** ``--max-objects`` 选项用于指定最大对象数，\
  负值表示禁用此配置。

- **最大尺寸：** ``--max-size`` 选项用于指定配额尺寸，单位是 \
  B/K/M/G/T ，默认值为 B 。负值表示禁用此配置。

- **配额作用域：** ``--quota-scope`` 参数可指定配额的作用域，\
  可选的有 ``bucket`` 和 ``user`` 。桶配额作用于用户拥有的桶；\
  用户配额作用于用户。


设置用户配额
------------
.. Set User Quota

启用配额前，必须先配置配额参数。例如：

.. prompt:: bash

   radosgw-admin quota set --quota-scope=user --uid=<uid> [--max-objects=<num objects>] [--max-size=<max size>]

例如：

.. prompt:: bash

   radosgw-admin quota set --quota-scope=user --uid=johndoe --max-objects=1024 --max-size=1024B

``--max-objects`` 或 ``--max-size`` 的参数为负值时，表示禁用这种配额属性。


启用或禁用用户配额
------------------
.. Enable/Disable User Quota

设置好用户配额后就可以启用了。例如：

.. prompt:: bash

	radosgw-admin quota enable --quota-scope=user --uid=<uid>

你也可以关闭已启用的用户配额功能。例如：

.. prompt:: bash

	radosgw-admin quota disable --quota-scope=user --uid=<uid>


设置桶配额
----------
.. Set Bucket Quota

Bucket quotas apply to the buckets owned by the specified ``uid``. They are
independent of the user. To set a bucket quota, run a command of the following
form:

.. prompt:: bash

   radosgw-admin quota set --uid=<uid> --quota-scope=bucket [--max-objects=<num objects>] [--max-size=<max size]

A negative value for ``--max-objects`` or ``--max-size`` means that the
specific quota attribute is disabled.


启用、禁用桶配额
----------------
.. Enable and Disabling Bucket Quota

设置好桶配额后，必须启用才能生效。启用桶配额用下面的命令：

.. prompt:: bash

	radosgw-admin quota enable --quota-scope=bucket --uid=<uid>

要禁用一个已经启用的桶配额，按下列格式运行命令：

.. prompt:: bash

	radosgw-admin quota disable --quota-scope=bucket --uid=<uid>


查看配额配置信息
----------------
.. Get Quota Settings

You may access each user's quota settings via the user information
API. To read user quota setting information with the CLI interface,
execute the following:

.. prompt:: bash

	radosgw-admin user info --uid=<uid>


更新配额统计信息
----------------
.. Update Quota Stats

Quota stats are updated asynchronously. You can update quota statistics for all
users and all buckets manually to force an update of the latest quota stats. To
update quota statistics for all users and all buckets in order to retrieve the
latest quota statistics, run a command of the following form:

.. prompt:: bash

	radosgw-admin user stats --uid=<uid> --sync-stats


.. _rgw_user_usage_stats:

查看用户使用情况的统计信息
--------------------------
.. Get User Usage Stats

查看用户已经消耗了多少配额可以用下列命令：

.. prompt:: bash

	radosgw-admin user stats --uid=<uid>

.. note:: 你可以用 ``radosgw-admin user stats`` 命令，加上
   ``--sync-stats`` 选项来获取最新数据。


默认配额
--------
.. Default Quotas

你可以在配置文件里设置默认配额，新增用户会采用这些默认值，而已\
经存在的用户不受影响。如果相关的默认配额是写在配置文件里的，那\
么这些配额会分配给新用户，并对其启用配额管理功能。请参考
`Ceph 对象网关配置参考`_\ 里的 ``rgw bucket default quota max objects`` 、
``rgw bucket default quota max size`` 、 ``rgw user default quota max objects``
和 ``rgw user default quota max size`` 。


配额缓存
--------
.. Quota Cache

配额统计信息缓存在各个 RGW 例程内。如果有多个例程，\
这些缓存就会妨碍配额的完整施行，因为各例程可能持有不同的配额信息。

控制此行为的选项有：

:confval:`rgw_bucket_quota_ttl`
:confval:`rgw_user_quota_bucket_sync_interval`
:confval:`rgw_user_quota_sync_interval`

这些值设置得越高，配额操作越高效，
但是多个例程也会变得更不同步；
这些值设置得越低，多个例程就越接近完整地施行配额。

如果三者都是 ``0`` ，那就意味着配额缓存被禁用了，
这样多个例程就会完整地施行配额。
请参考\ `Ceph 对象网关配置参考`_\ 。


读取、写入全局配额
------------------
.. Reading / Writing Global Quotas

你可以在 period 配置中读取或写入全局配额设置，查看全局配额配置\
可以用：

.. prompt:: bash

   radosgw-admin global quota get

全局配额选项可以用 ``global quota`` 系列命令修改，如
``quota set`` 、 ``quota enable`` 和 ``quota disable`` 命令。

.. prompt:: bash

   radosgw-admin global quota set --quota-scope bucket --max-objects 1024
   radosgw-admin global quota enable --quota-scope bucket

.. note:: 多站配置方案中有 realm 和 period ，改变全局配额后，\
   必须用 ``period update --commit`` 提交变更。如果压根没有
   period ，必须重启网关，以使变更生效。


Rate Limit Management
=====================

Quotas can be set for The Ceph Object Gateway on users and buckets. The "rate
limit" includes the maximum number of read operations (read ops) and write
operations (write ops) per minute as well as the number of bytes per minute
that can be written or read per user or per bucket.

Read Requests and Write Requests
--------------------------------
Operations that use the ``GET`` method or the ``HEAD`` method in their REST
requests are "read requests". All other requests are "write requests".  

How Metrics Work
----------------
Each object gateway tracks per-user metrics separately from bucket metrics.
These metrics are not shared with other gateways. The configured limits should
be divided by the number of active object gateways. For example, if "user A" is
to be be limited to 10 ops per minute and there are two object gateways in the
cluster, then the limit on "user A" should be ``5`` (10 ops per minute / 2
RGWs). If the requests are **not** balanced between RGWs, the rate limit might
be underutilized. For example: if the ops limit is ``5`` and there are two
RGWs, **but** the Load Balancer sends load to only one of those RGWs, the
effective limit is 5 ops, because this limit is enforced per RGW. If the rate
limit that has been set for the bucket has been reached but the rate limit that
has been set for the user has not been reached, then the request is cancelled.
The contrary holds as well: if the rate limit that has been set for the user
has been reached but the rate limit that has been set for the bucket has not
been reached, then the request is cancelled.

The accounting of bandwidth happens only after a request has been accepted.
This means that requests will proceed even if the bucket rate limit or user
rate limit is reached during the execution of the request. The RGW keeps track
of a "debt" consisting of bytes used in excess of the configured value; users
or buckets that incur this kind of debt are prevented  from sending more
requests until the "debt" has been repaid. The maximum size of the "debt" is
twice the max-read/write-bytes per minute. If "user A" is subject to a 1-byte
read limit per minute and they attempt to GET an object that is 1 GB in size,
then the ``GET`` action will fail. After "user A" has completed this 1 GB
operation, RGW blocks the user's requests for up to two minutes. After this
time has elapsed, "user A" will be able to send ``GET`` requests again.


- **Bucket:** The ``--bucket`` option allows you to specify a rate limit for a
  bucket.

- **User:** The ``--uid`` option allows you to specify a rate limit for a
  user.

- **Maximum Read Ops:** The ``--max-read-ops`` setting allows you to limit read
  bytes per minute per RGW instance. A ``0`` value disables throttling. 
  
- **Maximum Read Bytes:** The ``--max-read-bytes`` setting allows you to limit
  read bytes per minute per RGW instance. A ``0`` value disables throttling. 

- **Maximum Write Ops:** The ``--max-write-ops`` setting allows you to specify
  the maximum number of write ops per minute per RGW instance. A ``0`` value
  disables throttling.
  
- **Maximum Write Bytes:** The ``--max-write-bytes`` setting allows you to
  specify the maximum number of write bytes per minute per RGW instance. A
  ``0`` value disables throttling.
 
- **Rate Limit Scope:** The ``--ratelimit-scope`` option sets the scope for the
  rate limit.  The options are ``bucket`` , ``user`` and ``anonymous``. Bucket
  rate limit apply to buckets.  The user rate limit applies to a user.  The
  ``anonymous`` option applies to an unauthenticated user. Anonymous scope is
  available only for global rate limit.


Set User Rate Limit
-------------------

Before you can enable a rate limit, you must first set the rate limit
parameters. The following is the general form of commands that set rate limit
parameters: 

.. prompt:: bash

   radosgw-admin ratelimit set --ratelimit-scope=user --uid=<uid>
   <[--max-read-ops=<num ops>] [--max-read-bytes=<num bytes>]
   [--max-write-ops=<num ops>] [--max-write-bytes=<num bytes>]>

An example of using ``radosgw-admin ratelimit set`` to set a rate limit might
look like this: 

.. prompt:: bash

   radosgw-admin ratelimit set --ratelimit-scope=user --uid=johndoe --max-read-ops=1024 --max-write-bytes=10240


A value of ``0`` assigned to ``--max-read-ops``, ``--max-read-bytes``,
``--max-write-ops``, or ``--max-write-bytes`` disables the specified rate
limit.  

Get User Rate Limit
-------------------

The ``radosgw-admin ratelimit get`` command returns the currently configured
rate limit parameters.

The following is the general form of the command that returns the current
configured limit parameters:  

.. prompt:: bash

   radosgw-admin ratelimit get --ratelimit-scope=user --uid=<uid>

An example of using ``radosgw-admin ratelimit get`` to return the rate limit
parameters might look like this: 

.. prompt:: bash

   radosgw-admin ratelimit get --ratelimit-scope=user --uid=johndoe

A value of ``0`` assigned to ``--max-read-ops``, ``--max-read-bytes``,
``--max-write-ops``, or ``--max-write-bytes`` disables the specified rate
limit.  


Enable/Disable User Rate Limit
------------------------------

After you have set a user rate limit, you must enable it in order for it to
take effect. Run a command of the following form to enable a user rate limit: 

.. prompt:: bash

   radosgw-admin ratelimit enable --ratelimit-scope=user --uid=<uid>

To disable an enabled user rate limit, run a command of the following form: 

.. prompt:: bash

   radosgw-admin ratelimit disable --ratelimit-scope=user --uid=johndoe


Set Bucket Rate Limit
---------------------

Before you enable a rate limit, you must first set the rate limit parameters.
The following is the general form of commands that set rate limit parameters:

.. prompt:: bash

   radosgw-admin ratelimit set --ratelimit-scope=bucket --bucket=<bucket> <[--max-read-ops=<num ops>] [--max-read-bytes=<num bytes>]
  [--max-write-ops=<num ops>] [--max-write-bytes=<num bytes>]>

An example of using ``radosgw-admin ratelimit set`` to set a rate limit for a
bucket might look like this: 

.. prompt:: bash

   radosgw-admin ratelimit set --ratelimit-scope=bucket --bucket=mybucket --max-read-ops=1024 --max-write-bytes=10240


A value of ``0`` assigned to ``--max-read-ops``, ``--max-read-bytes``,
``--max-write-ops``, or ``-max-write-bytes`` disables the specified bucket rate
limit. 

Get Bucket Rate Limit
---------------------

The ``radosgw-admin ratelimit get`` command returns the current configured rate
limit parameters.

The following is the general form of the command that returns the current
configured limit parameters:

.. prompt:: bash

   radosgw-admin ratelimit get --ratelimit-scope=bucket --bucket=<bucket>

An example of using ``radosgw-admin ratelimit get`` to return the rate limit
parameters for a bucket might look like this:

.. prompt:: bash

   radosgw-admin ratelimit get --ratelimit-scope=bucket --bucket=mybucket

A value of ``0`` assigned to ``--max-read-ops``, ``--max-read-bytes``,
``--max-write-ops``, or ``--max-write-bytes`` disables the specified rate
limit.


Enable and Disable Bucket Rate Limit
------------------------------------

After you set a bucket rate limit, you can enable it. The following is the
general form of the ``radosgw-admin ratelimit enable`` command that enables
bucket rate limits: 

.. prompt:: bash

   radosgw-admin ratelimit enable --ratelimit-scope=bucket --bucket=<bucket>

An enabled bucket rate limit can be disabled by running a command of the following form:

.. prompt:: bash

   radosgw-admin ratelimit disable --ratelimit-scope=bucket --uid=mybucket

Reading and Writing Global Rate Limit Configuration
---------------------------------------------------

You can read and write global rate limit settings in the period's configuration.
To view the global rate limit settings, run the following command:

.. prompt:: bash

   radosgw-admin global ratelimit get

The global rate limit settings can be manipulated with the ``global ratelimit``
counterparts of the ``ratelimit set``, ``ratelimit enable``, and ``ratelimit
disable`` commands. Per-user and per-bucket ratelimit configurations override
the global configuration:

.. prompt:: bash

   radosgw-admin global ratelimit set --ratelimit-scope bucket --max-read-ops=1024
   radosgw-admin global ratelimit enable --ratelimit-scope bucket

The global rate limit can be used to configure the scope of the rate limit for
all authenticated users:

.. prompt:: bash

   radosgw-admin global ratelimit set --ratelimit-scope user --max-read-ops=1024
   radosgw-admin global ratelimit enable --ratelimit-scope user

The global rate limit can be used to configure the scope of the rate limit for
all unauthenticated users:

.. prompt:: bash
  
   radosgw-admin global ratelimit set --ratelimit-scope=anonymous --max-read-ops=1024
   radosgw-admin global ratelimit enable --ratelimit-scope=anonymous

.. note:: In a multisite configuration where a realm and a period are present,
   any changes to the global rate limit must be committed using ``period update
   --commit``. If no period is present, the rados gateway(s) must be restarted
   for the changes to take effect.


使用情况
========
.. Usage

Ceph 对象网关会记录每个用户的使用情况，
你可以跟踪查看某段时间内每个用户的使用情况。

- 需要在 ``ceph.conf`` 的 ``[client.rgw]`` 段下加
  ``rgw enable usage log = true`` 配置，然后重启 ``radosgw`` 服务。

  .. note:: Until Ceph has a linkable macro that handles all the many ways that options can be set, we advise that you set ``rgw_enable_usage_log = true`` in central config or in ``ceph.conf`` and restart all RGWs.

选项有：

- **Start Date:** The ``--start-date`` option allows you to filter usage
  stats from a specified start date and an optional start time
  (**format:** ``yyyy-mm-dd [HH:MM:SS]``).

- **End Date:** The ``--end-date`` option allows you to filter usage up
  to a particular end date and an optional end time
  (**format:** ``yyyy-mm-dd [HH:MM:SS]``). 

- **Log Entries:** The ``--show-log-entries`` option allows you to specify
  whether to include log entries with the usage stats 
  (options: ``true`` | ``false``).

.. note:: You can specify time to a precision of minutes and seconds, but the
   specified time is stored only with a one-hour resolution.


查看使用情况
------------
.. Show Usage

To show usage statistics, use the ``radosgw-admin usage show`` command. To show
usage for a particular user, you must specify a user ID. You can also specify a
start date, end date, and whether to show log entries. The following is an example
of such a command:

.. prompt:: bash $

	radosgw-admin usage show --uid=johndoe --start-date=2012-03-01 --end-date=2012-04-01

You can show a summary of usage information for all users by omitting the user
ID, as in the following example command:

.. prompt:: bash $

   radosgw-admin usage show --show-log-entries=false


清理统计日志
------------
.. Trim Usage

Usage logs can consume significant storage space, especially over time and with
heavy use. You can trim the usage logs for all users and for specific users.
You can also specify date ranges for trim operations, as in the following
example commands:

.. prompt:: bash $

   radosgw-admin usage trim --start-date=2010-01-01 --end-date=2010-12-31
   radosgw-admin usage trim --uid=johndoe	
   radosgw-admin usage trim --uid=johndoe --end-date=2013-12-31


.. _radosgw-admin: ../../man/8/radosgw-admin/
.. _Pool Configuration: ../../rados/configuration/pool-pg-config-ref/
.. _Ceph 对象网关配置参考: ../config-ref/
.. _Accounts: ../account/
