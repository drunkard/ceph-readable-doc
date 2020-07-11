.. Ceph Object Gateway Config Reference

=======================
 Ceph 对象网关配置参考
=======================

下列的选项可加入 Ceph 配置文件（一般是 ``ceph.conf`` ）的
``[client.radosgw.{instance-name}]`` 段下，很多选项都有\
默认值，你若未指定，自然用默认。

Configuration variables set under the ``[client.radosgw.{instance-name}]``
section will not apply to rgw or radosgw-admin commands without an instance-name
specified in the command. Thus variables meant to be applied to all RGW
instances or all radosgw-admin commands can be put into the ``[global]`` or the
``[client]`` section to avoid specifying instance-name.


``rgw frontends``

:描述: 配置的是 HTTP 前端。多个前端的配置可以写成逗号分隔的\
       列表。各个前端的配置是空格分隔的一系列选项，各选项都得\
       遵照 key=value 或 key 格式。所支持的选项请参考
       `HTTP 前端`_\ 。
:类型: String
:默认值: ``beast port=7480``


``rgw data``

:描述: 设置 Ceph 对象网关的数据文件位置。
:类型: String
:默认值: ``/var/lib/ceph/radosgw/$cluster-$id``


``rgw enable apis``

:描述: 启用指定的 API 。

       .. note:: 在\ `多站 <../multisite>`_\ 配置中，任何要\
                 参与的 radosgw 例程都必须启用 ``s3`` API 。
:类型: String
:默认值: 所有 API ： ``s3, swift, swift_auth, admin`` 。


``rgw cache enabled``

:描述: 是否启用 Ceph 对象网关缓存。
:类型: Boolean
:默认值: ``true``


``rgw cache lru size``

:描述: Ceph 对象网关缓存的条目限制。
:类型: Integer
:默认值: ``10000``


``rgw socket path``

:描述: 域套接字的路径， ``FastCgiExternalServer`` 要使用此\
       套接字。若未指定， Ceph 对象网关就不会以外部服务器运行。\
       这里的路径必须与 ``rgw.conf`` 里的路径相同。
:类型: String
:默认值: N/A


``rgw fcgi socket backlog``

:描述: fcgi 的套接字积压容量。
:类型: Integer
:默认值: ``1024``


``rgw host``

:描述: Ceph 对象网关例程所在主机，可以是 IP 地址或者主机名。
:类型: String
:默认值: ``0.0.0.0``


``rgw port``

:描述: 例程接受请求的端口。若未指定， Ceph 对象网关将运行外部
       FastCGI 。
:类型: String
:默认值: None


``rgw dns name``

:描述: 所服务域的 DNS 名称。请参考 region 配置里的 ``hostnames``
       选项。
:类型: String
:默认值: None


``rgw script uri``

:描述: 如果请求中没带 ``SCRIPT_URI`` 变量，这里的设置可作为补救。
:类型: String
:默认值: None


``rgw request uri``

:描述: 如果请求中没带 ``REQUEST_URI`` 变量，这里的设置可作为补救。
:类型: String
:默认值: None


``rgw print continue``

:描述: 如果可能的话，启用 ``100-continue`` 。
:类型: Boolean
:默认值: ``true``


``rgw remote addr param``

:描述: 远端地址参数。例如， HTTP 字段或者 ``X-Forwarded-For``
       地址（如果用了反向代理）。

:类型: String
:默认值: ``REMOTE_ADDR``


``rgw op thread timeout``

:描述: 在运行线程的超时值。
:类型: Integer
:默认值: 600


``rgw op thread suicide timeout``

:描述: Ceph 对象网关进程的自杀超时值（ ``timeout`` ）。设置为 \
       ``0`` 时禁用。

:类型: Integer
:默认值: ``0``


``rgw thread pool size``

:描述: 线程池的尺寸。
:类型: Integer
:默认值: 100 threads.


``rgw num control oids``

:描述: 不同的 ``rgw`` 例程间用于缓存同步的通知对象数量。
:类型: Integer
:默认值: ``8``


``rgw init timeout``

:描述: Ceph 对象网关放弃初始化前坚持的时间，秒。
:类型: Integer
:默认值: ``30``


``rgw mime types file``

:描述: MIME 类型数据库文件的路径，Swift 自动探测对象类型时要用到。
:类型: String
:默认值: ``/etc/mime.types``


``rgw s3 success create obj status``

:描述: ``create-obj`` 的另一种成功状态响应。
:类型: Integer
:默认值: ``0``


``rgw resolve cname``

:描述: 如果主机名与 ``rgw dns name`` 不同， ``rgw`` 是否应该用\
       请求的 hostname 字段的 DNS CNAME 记录。

:类型: Boolean
:默认值: ``false``


``rgw obj stripe size``

:描述: Ceph 对象网关的对象条带尺寸。关于条带化请参考\ \
       `体系结构`_\ 。

:类型: Integer
:默认值: ``4 << 20``


``rgw extended http attrs``

:描述: 为实体（用户、桶或对象）新增可设置的属性集。可以在上传实\
       体时把这些额外属性设置在 HTTP 头的字段里、或者用 POST 方\
       法修改；如果设置过，在此实体上执行 GET/HEAD 操作时这些属\
       性就会以 HTTP 头的字段返回。

:类型: String
:默认值: None
:实例: "content_foo, content_bar, x-foo-bar"


``rgw exit timeout secs``

:描述: 等待某一进程多长时间（秒）后无条件退出。
:类型: Integer
:默认值: ``120``


``rgw get obj window size``

:描述: 为单对象请求预留的窗口大小（字节）。
:类型: Integer
:默认值: ``16 << 20``


``rgw get obj max req size``

:描述: 向 Ceph 存储集群发起的一次 GET 请求的最大尺寸。
:类型: Integer
:默认值: ``4 << 20``


``rgw relaxed s3 bucket names``

:描述: 对 US region 的桶启用宽松的桶名规则。
:类型: Boolean
:默认值: ``false``


``rgw list buckets max chunk``

:描述: 列举用户桶时，每次检出的最大桶数。
:类型: Integer
:默认值: ``1000``


``rgw override bucket index max shards``

:描述: 桶索引对象的分片数量， 0 表示没有分片。我们不建议把这个\
       值设置得太大（比如大于 1000 ），因为这样会增加罗列桶时\
       的开销。本变量应该配置在 client 或 global 段下，这样它\
       就会自动应用到 radosgw-admin 命令。

:类型: Integer
:默认值: ``0``


``rgw curl wait timeout ms``

:描述: 某些特定 ``curl`` 调用的超时值，毫秒。
:类型: Integer
:默认值: ``1000``


``rgw copy obj progress``

:描述: 长时间复制操作时允许输出对象进度。
:类型: Boolean
:默认值: ``true``


``rgw copy obj progress every bytes``

:描述: 复制进度输出的粒度，字节数。
:类型: Integer
:默认值: ``1024 * 1024``


``rgw admin entry``

:描述: 管理 URL 请求的入口点。
:类型: String
:默认值: ``admin``


``rgw content length compat``

:描述: 允许兼容设置了 CONTENT_LENGTH 和 HTTP_CONTENT_LENGTH 的
       FCGI 请求。
:类型: Boolean
:默认值: ``false``


``rgw bucket quota ttl``

:描述: 在配置的这段时间（单位为秒）内，缓存的配额信息还是有效\
       的；超时后，配额信息需重新从集群读取。
:类型: Integer
:默认值: ``600``


``rgw user quota bucket sync interval``

:描述: 桶的配额信息同步到集群前暂存的时间，单位为秒。在此期\
       间，其它 RGW 例程看不到这个例程上的桶配额变更。

:类型: Integer
:默认值: ``180``


``rgw user quota sync interval``

:描述: 桶的配额信息同步到集群前暂存的时间，单位为秒。在此期\
       间，其它 RGW 例程看不到这个例程上的用户配额变更。

:类型: Integer
:默认值: ``180``


``rgw bucket default quota max objects``

:描述: 每个桶默认的最大对象数量。如果没其它配额操作，只给新用\
       户设置。对已有用户没影响。

:类型: Integer
:默认值: ``-1``


``rgw bucket default quota max size``

:描述: 每个桶默认的最大容量，单位为字节。如果没其它配额操作，\
       只给新用户设置。对已有用户没影响。

:类型: Integer
:默认值: ``-1``


``rgw user default quota max objects``

:描述: 用户默认的最大对象数，此用户的所有桶内的所有对象都计算\
       在内。如果没其它配额操作，只给新用户设置。对已有用户没\
       影响。

:类型: Integer
:默认值: ``-1``


``rgw user default quota max size``

:描述: 用户默认的最大容量，单位为字节。如果没其它配额操作，只\
       给新用户设置。对已有用户没影响。

:类型: Integer
:默认值: ``-1``


``rgw verify ssl``

:描述: 发出请求时验证 SSL 证书。
:类型: Boolean
:默认值: ``true``


.. Garbage Collection Settings

垃圾回收选项
============

The Ceph Object Gateway allocates storage for new objects immediately.

The Ceph Object Gateway purges the storage space used for deleted and overwritten 
objects in the Ceph Storage cluster some time after the gateway deletes the 
objects from the bucket index. The process of purging the deleted object data 
from the Ceph Storage cluster is known as Garbage Collection or GC.

To view the queue of objects awaiting garbage collection, execute the following::

  $ radosgw-admin gc list 

  Note: specify --include-all to list all entries, including unexpired
  
Garbage collection is a background activity that may
execute continuously or during times of low loads, depending upon how the
administrator configures the Ceph Object Gateway. By default, the Ceph Object
Gateway conducts GC operations continuously. Since GC operations are a normal
part of Ceph Object Gateway operations, especially with object delete
operations, objects eligible for garbage collection exist most of the time.

Some workloads may temporarily or permanently outpace the rate of garbage
collection activity. This is especially true of delete-heavy workloads, where
many objects get stored for a short period of time and then deleted. For these
types of workloads, administrators can increase the priority of garbage
collection operations relative to other operations with the following
configuration parameters.


``rgw gc max objs``

:描述: 垃圾回收进程在一个处理周期内可处理的最大对象数。首次\
       部署后请勿更改此值。
:类型: Integer
:默认值: ``32``


``rgw gc obj min wait``

:描述: 对象可被删除并由垃圾回收器处理前最少等待多长时间。
:类型: Integer
:默认值: ``2 * 3600``


``rgw gc processor max time``

:描述: 两个连续的垃圾回收周期起点的最大时间间隔。
:类型: Integer
:默认值: ``3600``


``rgw gc processor period``

:描述: 垃圾回收进程的运行周期。
:类型: Integer
:默认值: ``3600``


``rgw gc max concurrent io``

:描述: The maximum number of concurrent IO operations that the RGW garbage
              collection thread will use when purging old data.
:类型: Integer
:默认值: ``10``


.. Multisite Settings

多站设置
========

.. versionadded:: Jewel

你可以在 Ceph 配置文件中的各例程 ``[client.radosgw.{instance-name}]``
段下设置下列选项。


``rgw zone``

:描述: 网关例程所在的域名称。如果没配置过域，可以用命令
       ``radosgw-admin zone default`` 来配置集群范围的默认值。
:类型: String
:默认值: None


``rgw zonegroup``

:描述: 网关例程所在的域组名字。如果还没有域组，可以用
       ``radosgw-admin zonegroup default`` 命令配置集群范围的\
       默认值。
:类型: String
:默认值: None


``rgw realm``

:描述: 网关例程所在的 realm 。如果还没有 realm ，可以用
       ``radosgw-admin realm default`` 命令配置集群范围的默认\
       值。
:类型: String
:默认值: None


``rgw run sync thread``

:描述: 如果 realm 里有要同步的其它域，就派生出线程来处理数据和\
       元数据的同步。
:类型: Boolean
:默认值: ``true``


``rgw data log window``

:描述: 数据日志条数窗口，秒。
:类型: Integer
:默认值: ``30``


``rgw data log changes size``

:描述: 内存中保留的数据变更日志条数。
:类型: Integer
:默认值: ``1000``


``rgw data log obj prefix``

:描述: 数据日志的对象名前缀。
:类型: String
:默认值: ``data_log``


``rgw data log num shards``

:描述: 用于保存数据变更日志的分片（对象）数量。
:类型: Integer
:默认值: ``128``


``rgw md log max shards``

:描述: 用于元数据日志的最大分片数量。
:类型: Integer
:默认值: ``64``

.. important:: 开始同步后就不应该再更改
   ``rgw data log num shards`` 和 ``rgw md log max shards`` 的\
   取值了。


.. S3 Settings

S3 选项
=======

``rgw s3 auth use ldap``

:描述: Should S3 authentication use LDAP.
:类型: Boolean
:默认值: ``false``


.. Swift Settings

Swift 选项
==========

``rgw enforce swift acls``

:描述: 强制使用 Swift 的访问控制列表（ ACL ）选项。
:类型: Boolean
:默认值: ``true``


``rgw swift token expiration``

:描述: Swift 令牌过期时间，秒。
:类型: Integer
:默认值: ``24 * 3600``


``rgw swift url``

:描述: Ceph 对象网关 Swift 接口的 URL 。
:类型: String
:默认值: None


``rgw swift url prefix``

:描述: Swift API 的 URL 前缀，为区别于 S3 API 的终结点。默认是
       ``swift`` ，这样 Swift API 就会以 URL
       ``http://host:port/swift/v1`` （或者，启用
       ``rgw swift account in url`` 时将是
       ``http://host:port/swift/v1/AUTH_%(tenant_id)s`` ）暴露\
       出来。

       为兼容起见，此选项为空字符串时，将使用默认值 ``swift`` ；\
       如果你想要前缀为空，可以把此选项设置为 ``/`` 。

       .. warning:: If you set this option to ``/``, you must
                           disable the S3 API by modifying ``rgw
                           enable apis`` to exclude ``s3``. It is not
                           possible to operate radosgw with ``rgw
                           swift url prefix = /`` and simultaneously
                           support both the S3 and Swift APIs. If you
                           do need to support both APIs without
                           prefixes, deploy multiple radosgw instances
                           to listen on different hosts (or ports)
                           instead, enabling some for S3 and some for
                           Swift.
:默认值: ``swift``
:实例: ``/swift-testing``


``rgw swift auth url``

:描述: 验证 v1 版令牌的默认 URL （如果没用 Swift 内建认证）。
:类型: String
:默认值: None


``rgw swift auth entry``

:描述: Swift 认证 URL 的入口点。
:类型: String
:默认值: ``auth``


``rgw swift account in url``

:描述: Whether or not the Swift account name should be included
              in the Swift API URL.

              If set to ``false`` (the default), then the Swift API
              will listen on a URL formed like
              ``http://host:port/<rgw_swift_url_prefix>/v1``, and the
              account name (commonly a Keystone project UUID if
              radosgw is configured with `Keystone integration
              <../keystone>`_) will be inferred from request
              headers.

              If set to ``true``, the Swift API URL will be
              ``http://host:port/<rgw_swift_url_prefix>/v1/AUTH_<account_name>``
              (or
              ``http://host:port/<rgw_swift_url_prefix>/v1/AUTH_<keystone_project_id>``)
              instead, and the Keystone ``object-store`` endpoint must
              accordingly be configured to include the
              ``AUTH_%(tenant_id)s`` suffix.

              You **must** set this option to ``true`` (and update the
              Keystone service catalog) if you want radosgw to support
              publicly-readable containers and `temporary URLs
              <../swift/tempurl>`_.
:类型: Boolean
:默认值: ``false``


``rgw swift versioning enabled``

:描述: 启用 OpenStack 对象存储 API 的对象版本控制功能，这样\
       客户端就能在想要做版本控制的容器上设置
       ``X-Versions-Location`` 属性了，该属性用于指定存储着\
       存档版本的容器。做版本控制的容器必须是同一个用户拥有的，\
       因为要通过访问控制验证—— ACL **不会**\ 被纳入版本控制。\
       容器版本控制与 S3 对象版本控制机制不兼容。

       还有个稍有不同的属性， ``X-History-Location`` ，
       `OpenStack Swift <https://docs.openstack.org/swift/latest/api/object_versioning.html>`_
       也支持，是用于处理 ``DELETE`` 操作的。现在还不支持。
:类型: Boolean
:默认值: ``false``


``rgw trust forwarded https``

:描述: When a proxy in front of radosgw is used for ssl termination, radosgw
              does not know whether incoming http connections are secure. Enable
              this option to trust the ``Forwarded`` and ``X-Forwarded-Proto`` headers
              sent by the proxy when determining whether the connection is secure.
              This is required for some features, such as server side encryption.
              (Never enable this setting if you do not have a trusted proxy in front of
              radosgw, or else malicious users will be able to set these headers in
              any request.)
:类型: Boolean
:默认值: ``false``


.. Logging Settings

日志记录选项
============


``rgw log nonexistent bucket``

:描述: 让 Ceph 对象网关记录访问不存在的桶的请求。
:类型: Boolean
:默认值: ``false``


``rgw log object name``

:描述: 对象名的记录格式。关于格式说明见 :manpage:`date` 。
:类型: Date
:默认值: ``%Y-%m-%d-%H-%i-%n``


``rgw log object name utc``

:描述: 记录的对象名是否需包含 UTC 时间，设置为 ``false`` 时将\
       使用本地时间。
:类型: Boolean
:默认值: ``false``


``rgw usage max shards``

:描述: 使用率日志的最大数量。
:类型: Integer
:默认值: ``32``


``rgw usage max user shards``

:描述: 单个用户使用率日志的最大数量。
:类型: Integer
:默认值: ``1``


``rgw enable ops log``

:描述: 允许记录各次成功的 Ceph 对象网关操作。
:类型: Boolean
:默认值: ``false``


``rgw enable usage log``

:描述: 允许记录使用率日志。
:类型: Boolean
:默认值: ``false``


``rgw ops log rados``

:描述: 操作日志是否应该写入 Ceph 存储集群后端。
:类型: Boolean
:默认值: ``true``


``rgw ops log socket path``

:描述: 用于写入操作日志的 Unix 域套接字。
:类型: String
:默认值: None


``rgw ops log data backlog``

:描述: 最多积攒多少操作日志数据才写入 Unix 域套接字。
:类型: Integer
:默认值: ``5 << 20``


``rgw usage log flush threshold``

:描述: 使用率日志合并过多少条目才刷回。
:类型: Integer
:默认值: 1024


``rgw usage log tick interval``

:描述: 每 ``n`` 秒执行一次使用率日志刷回。
:类型: Integer
:默认值: ``30``


``rgw log http headers``

:描述: 操作日志里要记录的 HTTP 头，挨个罗列，以逗号分隔。头名\
       字必须是全名，对大小写不敏感，单词用下划线分隔。
:类型: String
:默认值: None
:示例: "http_x_forwarded_for, http_x_special_k"


``rgw intent log object name``

:描述: 意图日志对象名的记录格式。格式的详细说明见
       :manpage:`date` 。
:类型: Date
:默认值: ``%Y-%m-%d-%i-%n``


``rgw intent log object name utc``

:描述: 意图日志对象名是否应包含 UTC 时间，设置为 ``false`` 时\
       使用本地时间。
:类型: Boolean
:默认值: ``false``



.. Keystone Settings

Keystone 选项
=============


``rgw keystone url``

:描述: Keystone 服务器的 URL 。
:类型: String
:默认值: None


``rgw keystone api version``

:描述: 与 Keystone 服务器通讯时，使用哪个版本（ 2 或 3 ）的
       OpenStack Identity API 。
:类型: Integer
:默认值: ``2``


``rgw keystone admin domain``

:描述: 使用 v3 版本的 OpenStack Identity API 时，需在这里设置\
       具备管理权限的 OpenStack 域名。
:类型: String
:默认值: None


``rgw keystone admin project``

:描述: 使用 v3 版本的 OpenStack Identity API 时，需在这里设置\
       具备管理权限的 OpenStack 项目名。未设置时，将采用
       ``rgw keystone admin tenant`` 的值。
:类型: String
:默认值: None


``rgw keystone admin token``

:描述: Keystone 的管理令牌（共享密钥）。在 Ceph RadosGW 认证\
       时，会优先用管理令牌，然后才是管理凭证（
       ``rgw keystone admin user`` 、 ``rgw keystone admin password`` 、
       ``rgw keystone admin tenant`` 、 ``rgw keystone admin project`` 、
       ``rgw keystone admin domain`` ）。管理令牌功能正在被废弃。
:类型: String
:默认值: None


``rgw keystone admin token path``

:描述: Path to a file containing the Keystone admin token
	      (shared secret).  In Ceph RadosGW authentication with
	      the admin token has priority over authentication with
	      the admin credentials
              (``rgw keystone admin user``, ``rgw keystone admin password``,
              ``rgw keystone admin tenant``, ``rgw keystone admin project``,
              ``rgw keystone admin domain``).
              The Keystone admin token has been deprecated, but can be
              used to integrate with older environments.
:类型: String
:默认值: None


``rgw keystone admin tenant``

:描述: 使用 v2 版本的 OpenStack Identity API 时，在这里配置具\
       备管理权限的 OpenStack 租户（服务租户， Service Tenant
       ）的名字。
:类型: String
:默认值: None


``rgw keystone admin user``

:描述: 使用 v2 版本的 OpenStack Identity API 时，在这里配置具\
       备管理权限的 OpenStack 用户的名字，用来与 Keystone 认证\
       （服务用户， Service User ）。
:类型: String
:默认值: None


``rgw keystone admin password``

:描述: 使用 v2 版本的 OpenStack Identity API 时，在这里配置
       OpenStack 管理用户的密码。
:类型: String
:默认值: None


``rgw keystone admin password path``

:描述: Path to a file containing the password for OpenStack
              admin user when using OpenStack Identity API v2.
:类型: String
:默认值: None


``rgw keystone accepted roles``

:描述: 要接受请求所需的角色。
:类型: String
:默认值: ``Member, admin``


``rgw keystone token cache size``

:描述: 各 Keystone 令牌缓存的最大条数。
:类型: Integer
:默认值: ``10000``


``rgw keystone revocation interval``

:描述: 令牌有效期查验的周期，秒。
:类型: Integer
:默认值: ``15 * 60``


``rgw keystone verify ssl``

:描述: 向 keystone 发出令牌请求时要核对 SSL 证书。
:类型: Boolean
:默认值: ``true``


.. Server-side encryption Settings

服务端加密选项
==============

``rgw crypt s3 kms backend``

:描述: Where the SSE-KMS encryption keys are stored. Supported KMS
              systems are OpenStack Barbican (``barbican``, the default) and
              HashiCorp Vault (``vault``).
:类型: String
:默认值: None


.. Barbican Settings

Barbican 选项
=============

``rgw barbican url``

:描述: 访问 Barbican 服务器的 URL 。
:类型: String
:默认值: None


``rgw keystone barbican user``

:描述: 有权访问 `Barbican`_ 内\ `加密`_\ 密钥的 OpenStack 用户\
       名。
:类型: String
:默认值: None


``rgw keystone barbican password``

:描述: `Barbican`_ 用户的密码。
:类型: String
:默认值: None


``rgw keystone barbican tenant``

:描述: 使用 OpenStack Identity API v2 时，在这里配置与
       `Barbican`_ 用户相关联的 OpenStack 租户名字。
:类型: String
:默认值: None


``rgw keystone barbican project``

:描述: 使用 OpenStack Identity API v3 时，在这里配置与
       `Barbican`_ 用户相关联的 OpenStack 项目名字。
:类型: String
:默认值: None


``rgw keystone barbican domain``

:描述: 使用 OpenStack Identity API v3 时，在这里配置与
       `Barbican`_ 用户相关联的 OpenStack 域名。
:类型: String
:默认值: None


.. HashiCorp Vault Settings

HashiCorp Vault 选项
====================

``rgw crypt vault auth``

:描述: Type of authentication method to be used. The only method
              currently supported is ``token``.
:类型: String
:默认值: ``token``

``rgw crypt vault token file``

:描述: If authentication method is ``token``, provide a path to the token
              file, which should be readable only by Rados Gateway.
:类型: String
:默认值: None

``rgw crypt vault addr``

:描述: Vault server base address, e.g. ``http://vaultserver:8200``.
:类型: String
:默认值: None

``rgw crypt vault prefix``

:描述: The Vault secret URL prefix, which can be used to restrict access
              to a particular subset of the secret space, e.g. ``/v1/secret/data``.
:类型: String
:默认值: None

``rgw crypt vault secret engine``

:描述: Vault Secret Engine to be used to retrieve encryption keys: choose
              between kv-v2, transit.
:类型: String
:默认值: None

``rgw crypt vault namespace``

:描述: If set, Vault Namespace provides tenant isolation for teams and individuals
              on the same Vault Enterprise instance, e.g. ``acme/tenant1``
:类型: String
:默认值: None


.. QoS settings

QoS 选项
--------

.. versionadded:: Nautilus

The ``civetweb`` frontend has a threading model that uses a thread per
connection and hence automatically throttled by ``rgw thread pool size``
configurable when it comes to accepting connections. The ``beast`` frontend is
not restricted by the thread pool size when it comes to accepting new
connections, so a scheduler abstraction is introduced in Nautilus release which
for supporting ways for scheduling requests in the future.

Currently the scheduler defaults to a throttler which throttles the active
connections to a configured limit. QoS based on mClock is currently in an
*experimental* phase and not recommended for production yet. Current
implementation of *dmclock_client* op queue divides RGW Ops on admin, auth
(swift auth, sts) metadata & data requests.


``rgw max concurrent requests``

:描述: Maximum number of concurrent HTTP requests that the beast frontend
              will process. Tuning this can help to limit memory usage under
              heavy load.
:类型: Integer
:默认值: 1024


``rgw scheduler type``

:描述: The type of RGW Scheduler to use. Valid values are throttler,
              dmclock. Currently defaults to throttler which throttles beast
              frontend requests. dmclock is *experimental* and will need the
              experimental flag set


The options below are to tune the experimental dmclock scheduler. For some
further reading on dmclock, see :ref:`dmclock-qos`. `op_class` for the flags below is
one of admin, auth, metadata or data.

``rgw_dmclock_<op_class>_res``

:描述: The mclock reservation for `op_class` requests
:类型: float
:默认值: 100.0

``rgw_dmclock_<op_class>_wgt``

:描述: The mclock weight for `op_class` requests
:类型: float
:默认值: 1.0

``rgw_dmclock_<op_class>_lim``

:描述: The mclock limit for `op_class` requests
:类型: float
:默认值: 0.0



.. _体系结构: ../../architecture#data-striping
.. _存储池配置: ../../rados/configuration/pool-pg-config-ref/
.. _集群存储池: ../../rados/operations/pools
.. _RADOS 集群句柄: ../../rados/api/librados-intro/#step-2-configuring-a-cluster-handle
.. _Barbican: ../barbican
.. _加密: ../encryption
.. _HTTP 前端: ../frontends
