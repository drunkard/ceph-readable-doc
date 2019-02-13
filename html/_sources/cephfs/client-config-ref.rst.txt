.. _Client Config Reference:

================
 客户端配置参考
================


``client acl type``

:描述: 设置 ACL 类型。现在还只能设置为 ``"posix_acl"`` 表示启用
       POSIX ACL ，或者可设置为空字符串。此选项只有在
       ``fuse_default_permissions`` 被设置为 ``false`` 时有效。
:类型: String
:默认值: ``""`` (no ACL enforcement)


``client cache mid``

:描述: 设置客户端缓存中点。此中点把最近用过的列表分割为热列表\
       和暖列表。
:类型: Float
:默认值: ``0.75``


``client_cache_size``

:描述: 设置客户端可保留在元数据缓存中的索引节点数量。
:类型: Integer
:默认值: ``16384``


``client_caps_release_delay``

:描述: 设置释放能力的延时，单位为秒。这个延时控制着客户端不再\
       需要能力时，再等多少秒就释放，以便其它需要这些能力的用\
       户使用。
:类型: Integer
:默认值: ``5`` (seconds)


``client_debug_force_sync_read``

:描述: 设置为 ``true`` 时，客户端会跳过本地页缓存、直接从 OSD
       读取数据。
:类型: Boolean
:默认值: ``false``


``client_dirsize_rbytes``

:描述: 设置为 ``true`` 时，就采用目录的递归尺寸（也就是其内所\
       有文件尺寸之和）。
:类型: Boolean
:默认值: ``true``


``client_max_inline_size``

:描述: 控制内联数据的最大尺寸，小于此尺寸就存储到文件的索引节\
       点内，超过则存到单独的 RADOS 对象内。本选项只有设置了
       MDS 图的 ``inline_data`` 标志时有效。
:类型: Integer
:默认值: ``4096``


``client_metadata``

:描述: 客户端向各 MDS 发送元数据时，除了自动生成的版本号、主机\
       名等信息，还可以附加逗号分隔的字符串作为附加元数据。
:类型: String
:默认值: ``""`` （没有附加元数据）


``client_mount_gid``

:描述: 设置 CephFS 挂载后的组 ID 。
:类型: Integer
:默认值: ``-1``


``client_mount_timeout``

:描述: 设置挂载 CephFS 时的超时时间，单位为秒。
:类型: Float
:默认值: ``300.0``


``client_mount_uid``

:描述: 设置 CephFS 挂载后的用户 ID 。
:类型: Integer
:默认值: ``-1``


``client_mountpoint``

:描述: 指定要挂载的 CephFS 文件系统目录。此选项的作用类似
       ``ceph-fuse`` 命令的 ``-r`` 选项。
:类型: String
:默认值: ``"/"``


``client_oc``

:描述: Enable object caching.
:类型: Boolean
:默认值: ``true``


``client_oc_max_dirty``

:描述: 设置对象缓存的脏数据上限，单位为字节。
:类型: Integer
:默认值: ``104857600`` (100MB)


``client_oc_max_dirty_age``

:描述: 设置脏数据在对象缓存中的最大存留时间，单位为秒。
:类型: Float
:默认值: ``5.0`` （秒）


``client_oc_max_objects``

:描述: 设置对象缓存允许的最大对象数。
:类型: Integer
:默认值: ``1000``


``client_oc_size``

:描述: 设置客户端可缓存的数据上限，单位为字节。
:类型: Integer
:默认值: ``209715200`` (200 MB)


``client_oc_target_dirty``

:描述: 设置认定为脏数据的目标尺寸。我们建议这个数字尽量小些。
:类型: Integer
:默认值: ``8388608`` (8MB)


``client_permissions``

:描述: 检查所有 I/O 操作的客户端权限。
:类型: Boolean
:默认值: ``true``


``client_quota``

:描述: 设置为 ``true`` 表示启用客户端配额。
:类型: Boolean
:默认值: ``true``


``client_quota_df``

:描述: 让 ``statfs`` 操作报告根目录的配额。
:类型: Boolean
:默认值: ``true``


``client_readahead_max_bytes``

:描述: 设置内核预读数据的最大尺寸，单位为字节。本选项可被
       ``client_readahead_max_periods`` 覆盖。
:类型: Integer
:默认值: ``0`` (unlimited)


``client_readahead_max_periods``

:描述: 设置内核预读的文件布局分片最大数量（对象尺寸 * 条带数\
       量）。本选项会覆盖 ``client_readahead_max_bytes`` 选项。
:类型: Integer
:默认值: ``4``


``client_readahead_min``

:描述: 设置内核预读的最小尺寸，单位为字节。
:类型: Integer
:默认值: ``131072`` (128KB)


``client_reconnect_stale``

:描述: 是否自动重连过期的会话。
:类型: Boolean
:默认值: ``false``


``client_snapdir``

:描述: 设置快照目录名。
:类型: String
:默认值: ``".snap"``


``client_tick_interval``

:描述: 设置更新能力及维持其它信息的间隔时长，单位为秒。
:类型: Float
:默认值: ``1.0`` （秒）


``client_use_random_mds``

:描述: 为各个请求随机选取 MDS 。
:类型: Boolean
:默认值: ``false``


``fuse_default_permissions``

:描述: 设置为 ``false`` 时， ``ceph-fuse`` 工具会用自己的权限\
       验证机制，而非依靠 FUSE 的强制权限。启用 POSIX ACL 需把\
       此选项设置为 ``false`` 、同时设置 ``client acl type=posix_acl`` 。
:类型: Boolean
:默认值: ``true``


.. _Developer Options:

开发者选项
----------

.. important:: 以下选项仅供内部测试，只是为了保持文档完整才罗\
   列在这里。


``client_debug_getattr_caps``

:描述: 检查 MDS 的响应是否有必要的能力。
:类型: Boolean
:默认值: ``false``


``client_debug_inject_tick_delay``

:描述: 在客户端动作之间人为地加入延时。
:类型: Integer
:默认值: ``0``


``client_inject_fixed_oldest_tid``

:描述:
:类型: Boolean
:默认值: ``false``


``client_inject_release_failure``

:描述:
:类型: Boolean
:默认值: ``false``


``client_trace``

:描述: 所有文件操作的追踪文件的路径。其输出可用于 Ceph 的\
       `人造客户端 <../../man/8/ceph-syn>`_\ 。
:类型: String
:默认值: ``""`` (disabled)

