=======================
 Ceph 对象网关配置参考
=======================

下列的选项可加入 Ceph 配置文件（一般是 ``ceph.conf`` ）的 \
``[client.radosgw.{instance-name}]`` 段下，很多选项都有默认值，你若未指定，自\
然用默认。


``rgw data``

:描述: 设置 Ceph 对象网关的数据文件位置。
:类型: String
:默认值: ``/var/lib/ceph/radosgw/$cluster-$id``


``rgw enable apis``

:描述: 启用指定的 API 。
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

:描述: 域套接字的路径， ``FastCgiExternalServer`` 要使用此套接字。若未指定， \
       Ceph 对象网关就不会以外部服务器运行。这里的路径必须与 ``rgw.conf`` 里\
       的路径相同。

:类型: String
:默认值: N/A


``rgw host``

:描述: Ceph 对象网关例程所在主机，可以是 IP 地址或者主机名。
:类型: String
:默认值: ``0.0.0.0``


``rgw port``

:描述: 例程接受请求的端口。若未指定， Ceph 对象网关将运行外部 FastCGI 。
:类型: String
:默认值: None


``rgw dns name``

:描述: 所服务域的 DNS 名称。请参考 region 配置里的 ``hostnames`` 选项。
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

:描述: 远端地址参数。例如， HTTP 字段或者 ``X-Forwarded-For`` 地址（如果用了\
       反向代理）。

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


``rgw num rados handles``

:描述: Ceph 对象网关的 `RADOS 集群处理器`_\ 数量。通过配置 RADOS \
       处理器数量可以使得各种类型的载荷都明显地提升性能，因为各个 \
       RGW 工作线程在其短暂的活跃期内都可以分别挂靠一个 RADOS 处理\
       器。

:类型: Integer
:默认值: ``1``


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


``rgw gc max objs``

:描述: 垃圾回收进程在一个处理周期内可处理的最大对象数。
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

:描述: 桶索引对象的分片数量， 0 表示没有分片。我们不建议把这个值\
       设置得太大（比如大于 1000 ），因为这样会增加罗列桶时的开销。

:类型: Integer
:默认值: ``0``


``rgw num zone opstate shards``

:描述: 用于保存 region 间复制进度的最大消息片数。
:类型: Integer
:默认值: ``128``


``rgw opstate ratelimit sec``

:描述: 各次上传后状态更新操作的最小间隔时间。 ``0`` 禁用此限速。
:类型: Integer
:默认值: ``30``


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

:描述: 允许兼容设置了 CONTENT_LENGTH 和 HTTP_CONTENT_LENGTH 的 FCGI 请求。
:类型: Boolean
:默认值: ``false``


region （域组）
===============

Ceph 从 v0.67 版开始，通过 region 概念支持 Ceph 对象网关联盟部署和统一的命\
名空间。 region 定义了位于一或多个域内的 Ceph 对象网关例程的地理位置。

region 的配置不同于一般配置过程，因为不是所有的配置都放在 Ceph 配置文件中。\
从 Ceph 0.67 版开始，你可以列举 region 、获取 region 配置或设置 region 配置。


罗列 region
-----------

Ceph 集群可包含一系列 region ，可用下列命令列举 region ： ::

	sudo radosgw-admin regions list

``radosgw-admin`` 命令会返回 JSON 格式的 region 列表。

.. code-block:: javascript

	{ "default_info": { "default_region": "default"},
	  "regions": [
	        "default"]}


获取 region-map
---------------

要获取各 region 的详细情况，可执行： ::

	sudo radosgw-admin region-map get


.. note:: 如果你的到了 ``failed to read region map`` 错误，先试试 \
   ``sudo radosgw-admin region-map update`` 。


获取单个 region
---------------

要查看某 region 的配置，执行： ::

	radosgw-admin region get [--rgw-region=<region>]

``default`` 这个 region 的配置大致如此：

.. code-block:: javascript

   {"name": "default",
    "api_name": "",
    "is_master": "true",
    "endpoints": [],
    "hostnames": [],
    "master_zone": "",
    "zones": [
      {"name": "default",
       "endpoints": [],
       "log_meta": "false",
       "log_data": "false"}
     ],
    "placement_targets": [
      {"name": "default-placement",
       "tags": [] }],
    "default_placement": "default-placement"}


设置一 region
-------------

定义 region 需创建一个 JSON 对象、并提供必需的选项：

#. ``name``: region 名字，必需。

#. ``api_name``: 此 region 的 API 名字，可选。

#. ``is_master``: 决定着此 region 是否为主 region ，必需。\ **注：**\ 只能\
   有一个主 region 。

#. ``endpoints``: region 内的所有终结点列表。例如，你可以用多个域名指向同\
   一 region 区，记得在斜杠前加反斜杠进行转义（ ``\/`` ）。也可以给终结点\
   指定端口号（ ``fqdn:port`` ），可选。

#. ``hostnames``: region 内所有主机名的列表。例如，这样你就可以在同一 \
   region 内使用多个域名了。可选配置。此列表会自动包含 ``rgw dns name`` \
   配置。更改此配置后需重启所有 ``radosgw`` 守护进程。

#. ``master_zone``: region 的主域，可选。若未指定，则选择默认域。\
   **注：**\ 每个 region 只能有一个主域。

#. ``zones``: region 内所有域的列表。各个域都有名字（必需的）、一系列终结\
   点（可选的）、以及网关是否要记录元数据和数据操作（默认不记录）。

#. ``placement_targets``: 放置目标列表（可选）。每个放置目标都包含此放置目标\
   的名字（必需）、还有一个标签列表（可选），这样只有带这些标签的用户可以使用\
   此放置目标（即用户信息中的 ``placement_tags`` 字段）。

#. ``default_placement``: 对象索引及数据的默认放置目标，默认为 \
   ``default-placement`` 。你可以在用户信息里给各用户设置一个用户级的默认放置\
   目标。

要配置起一个 region ，需创建一个包含必需字段的 JSON 对象，把它存入文件（如 \
``region.json`` ），然后执行下列命令： ::

	sudo radosgw-admin region set --infile region.json

其中 ``region.json`` 是你创建的 JSON 文件。


.. important:: 默认 region ``default`` 的 ``is_master`` 字段值默认为 \
   ``true`` 。如果你想新建一 region 并让它作为主 region ，那你必须把 \
   ``default`` region 的 ``is_master`` 设置为 ``false`` ，或者干脆删除 \
   ``default`` region 。


最后，更新 region 图。 ::

	sudo radosgw-admin region-map update


配置 region 图
--------------

配置 region 图的过程包括创建含一或多个 region 的 JSON 对象，还有设置集群的\
主 region ``master_region`` 。 region 图内的各 region 都由键/值对组成，其\
中 ``key`` 选项等价于单独配置 region 时的 ``name`` 选项， ``val`` 是包含单\
个 region 完整配置的 JSON 对象。

你可以只有一个 region ，其 ``is_master`` 设置为 ``true`` ，而且必须在 \
region 图末尾设置为 ``master_region`` 。下面的 JSON 对象是默认 region 图的\
实例。


.. code-block:: javascript

     { "regions": [
          { "key": "default",
            "val": { "name": "default",
            "api_name": "",
            "is_master": "true",
            "endpoints": [],
            "hostnames": [],
            "master_zone": "",
            "zones": [
              { "name": "default",
                "endpoints": [],
                "log_meta": "false",
                 "log_data": "false"}],
                 "placement_targets": [
                   { "name": "default-placement",
                     "tags": []}],
                     "default_placement": "default-placement"
                   }
               }
            ],
        "master_region": "default"
     }

要配置一个 region 图，执行此命令： ::

	sudo radosgw-admin region-map set --infile regionmap.json

其中 ``regionmap.json`` 是创建的 JSON 文件。确保你创建了 region 图里所指\
的那些域。最后，更新此图。 ::

	sudo radosgw-admin regionmap update


域
==

从 Ceph v0.67 版起， Ceph 对象网关支持域概念，它是一或多个 Ceph 对象网关例程\
组成的逻辑组。

域的配置不同于典型配置过程，因为并非所有配置都位于 Ceph 配置文件内。从 0.67 \
版起，你可以列举域、获取域配置、设置域配置。


列举域
------

要列举某集群内的域，执行： ::

	sudo radosgw-admin zone list


获取单个域
----------

要获取某一域的配置，执行： ::

	sudo radosgw-admin zone get [--rgw-zone=<zone>]

``default`` 这个默认域的配置大致如此：

.. code-block:: javascript

   { "domain_root": ".rgw",
     "control_pool": ".rgw.control",
     "gc_pool": ".rgw.gc",
     "log_pool": ".log",
     "intent_log_pool": ".intent-log",
     "usage_log_pool": ".usage",
     "user_keys_pool": ".users",
     "user_email_pool": ".users.email",
     "user_swift_pool": ".users.swift",
     "user_uid_pool": ".users.uid",
     "system_key": { "access_key": "", "secret_key": ""},
     "placement_pools": [
         {  "key": "default-placement",
            "val": { "index_pool": ".rgw.buckets.index",
                     "data_pool": ".rgw.buckets"}
         }
       ]
     }


配置域
------

配置域时需指定一系列的 Ceph 对象网关存储池。为保持一致性，我们建议用区域名\
作为存储池名字的前缀。存储池配置见\ `存储池`_\ 。

要配置起一个域，需创建包含存储池的 JSON 对象、并存入文件（如 \
``zone.json`` ）；然后执行下列命令，把 ``{zone-name}`` 替换为域名称： ::

	sudo radosgw-admin zone set --rgw-zone={zone-name} --infile zone.json

其中， ``zone.json`` 是你创建的 JSON 文件。


region 和域选项
===============

你可以在 Ceph 配置文件中的各例程 ``[client.radosgw.{instance-name}]`` 段下设\
置下列选项。


.. versionadded:: v.67

``rgw zone``

:描述: 网关例程所在的域名称。
:类型: String
:默认值: None


.. versionadded:: v.67

``rgw region``

:描述: 网关例程所在的 region 名。
:类型: String
:默认值: None


.. versionadded:: v.67

``rgw default region info oid``

:描述: 用于保存默认 region 的 OID 。我们不建议更改此选项。
:类型: String
:默认值: ``default.region``


存储池
======

Ceph 域会映射到一系列 Ceph 存储集群的存储池。

.. topic:: 手动创建存储池与自动生成的存储池对比

   如果你给 Ceph 对象网关的用户密钥分配了写权限，此网关就有能力自动创建存储\
   池。这样虽便捷，但 Ceph 对象存储集群的归置组数量会是默认值（此值也许不太\
   理想）或者 Ceph 配置文件中的自定义配置。如果你想让 Ceph 对象网关自动创建\
   存储池，确保归置组数量的默认值要合理。详情见\ `存储池配置`_\ ，关于创建存\
   储池见\ `集群存储池`_\ 。

Ceph 对象网关的默认域的默认存储池有：

- ``.rgw``
- ``.rgw.control``
- ``.rgw.gc``
- ``.log``
- ``.intent-log``
- ``.usage``
- ``.users``
- ``.users.email``
- ``.users.swift``
- ``.users.uid``

你应该能够清晰地判断某个域会怎样访问各存储池。你可以为每个域创建一系列存储\
池，或者让多个域共用同一系列的存储池。作为最佳实践，我们建议分别位于各 \
region 中的主域和二级域都要有各自的存储池系列。为某个域创建存储池时，建议\
默认存储池名以 region 名和域名作为前缀，例如：

- ``.region1-zone1.domain.rgw``
- ``.region1-zone1.rgw.control``
- ``.region1-zone1.rgw.gc``
- ``.region1-zone1.log``
- ``.region1-zone1.intent-log``
- ``.region1-zone1.usage``
- ``.region1-zone1.users``
- ``.region1-zone1.users.email``
- ``.region1-zone1.users.swift``
- ``.region1-zone1.users.uid``

Ceph 对象网关会把桶索引（ ``index_pool`` ）和桶数据（ ``data_pool`` ）存储到\
归置存储池，这些可以重叠——也就是你可以把索引和数据存入同一存储池。索引存储池\
的默认归置地是 ``.rgw.buckets.index`` ，数据存储池的默认归置地是 \
``.rgw.buckets`` ，给域指定存储池的方法见\ `域`_\ 。


.. deprecated:: v.67

``rgw cluster root pool``

:描述: 为此例程存储 ``radosgw`` 元数据的存储池。从 v0.67 之后不再支持，可改\
       用 ``rgw zone root pool`` 。

:类型: String
:是否必需: No
:默认值: ``.rgw.root``
:替代选项: ``rgw zone root pool``


.. versionadded:: v.67

``rgw region root pool``

:描述: 用于存储此 region 所有相关信息的存储池。
:类型: String
:默认值: ``.rgw.root``



.. versionadded:: v.67

``rgw zone root pool``

:描述: 用于存储此域所有相关信息的存储池。
:类型: String
:默认值: ``.rgw.root``


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

:描述: Swift API 的 URL 前缀。
:默认值: ``swift``
:实例: http://fqdn.com/swift


``rgw swift auth url``

:描述: 验证 v1 版令牌的默认 URL （如果没用 Swift 内建认证）。
:类型: String
:默认值: None


``rgw swift auth entry``

:描述: Swift 认证 URL 的入口点。
:类型: String
:默认值: ``auth``



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

:描述: 记录的对象名是否需包含 UTC 时间，设置为 ``false`` 时将使用本地时间。
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


``rgw intent log object name``

:描述: 意图日志对象名的记录格式。格式的详细说明见 :manpage:`date` 。
:类型: Date
:默认值: ``%Y-%m-%d-%i-%n``


``rgw intent log object name utc``

:描述: 意图日志对象名是否应包含 UTC 时间，设置为 ``false`` 时使用本地时间。
:类型: Boolean
:默认值: ``false``


``rgw data log window``

:描述: 数据日志窗口，秒。
:类型: Integer
:默认值: ``30``


``rgw data log changes size``

:描述: 内存中保留的数据变更日志条数。
:类型: Integer
:默认值: ``1000``


``rgw data log num shards``

:描述: 用于保存数据变更日志的碎片（对象）数量。
:类型: Integer
:默认值: ``128``


``rgw data log obj prefix``

:描述: 数据日志的对象名前缀。
:类型: String
:默认值: ``data_log``


``rgw replica log obj prefix``

:描述: 复制日志的对象名前缀。
:类型: String
:默认值: ``replica log``


``rgw md log max shards``

:描述: 用于元数据日志的最大碎片数。
:类型: Integer
:默认值: ``64``



Keystone 选项
=============


``rgw keystone url``

:描述: Keystone 服务器的 URL 。
:类型: String
:默认值: None


``rgw keystone admin token``

:描述: Keystone 的管理令牌（共享密钥）。
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


.. _体系结构: ../../architecture#data-striping
.. _存储池配置: ../../rados/configuration/pool-pg-config-ref/
.. _集群存储池: ../../rados/operations/pools
.. _RADOS 集群处理器: ../../rados/api/librados-intro/#step-2-configuring-a-cluster-handle
