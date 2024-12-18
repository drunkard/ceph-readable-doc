=======================
 Ceph 对象网关配置参考
=======================
.. Ceph Object Gateway Config Reference

下列的选项可加入 Ceph 配置文件（一般是 ``ceph.conf`` ）的
``[client.radosgw.{instance-name}]`` 段下，很多选项都有\
默认值，你若未指定，自然用默认。

rgw 或 radosgw-admin 命令执行时，如果指定了例程名，
``[client.radosgw.{instance-name}]`` 段下的配置变量才有效。
有些变量如果想要应用于所有 RGW 例程或所有的 radosgw-admin 选项，
可以把它们放到 ``[global]`` 或 ``[client]`` 段内，
这样就不用加 ``instance-name`` 了。

.. confval:: rgw_frontends
.. confval:: rgw_data
.. confval:: rgw_enable_apis
.. confval:: rgw_cache_enabled
.. confval:: rgw_cache_lru_size
.. confval:: rgw_dns_name
.. confval:: rgw_script_uri
.. confval:: rgw_request_uri
.. confval:: rgw_print_continue
.. confval:: rgw_remote_addr_param
.. confval:: rgw_op_thread_timeout
.. confval:: rgw_op_thread_suicide_timeout
.. confval:: rgw_thread_pool_size
.. confval:: rgw_num_control_oids
.. confval:: rgw_init_timeout
.. confval:: rgw_mime_types_file
.. confval:: rgw_s3_success_create_obj_status
.. confval:: rgw_resolve_cname
.. confval:: rgw_obj_stripe_size
.. confval:: rgw_extended_http_attrs
.. confval:: rgw_exit_timeout_secs
.. confval:: rgw_get_obj_window_size
.. confval:: rgw_get_obj_max_req_size
.. confval:: rgw_multipart_min_part_size
.. confval:: rgw_relaxed_s3_bucket_names
.. confval:: rgw_list_buckets_max_chunk
.. confval:: rgw_override_bucket_index_max_shards
.. confval:: rgw_curl_wait_timeout_ms
.. confval:: rgw_copy_obj_progress
.. confval:: rgw_copy_obj_progress_every_bytes
.. confval:: rgw_max_copy_obj_concurrent_io
.. confval:: rgw_admin_entry
.. confval:: rgw_content_length_compat
.. confval:: rgw_bucket_quota_ttl
.. confval:: rgw_user_quota_bucket_sync_interval
.. confval:: rgw_user_quota_sync_interval
.. confval:: rgw_bucket_default_quota_max_objects
.. confval:: rgw_bucket_default_quota_max_size
.. confval:: rgw_user_default_quota_max_objects
.. confval:: rgw_user_default_quota_max_size
.. confval:: rgw_account_default_quota_max_objects
.. confval:: rgw_account_default_quota_max_size
.. confval:: rgw_verify_ssl
.. confval:: rgw_max_chunk_size


生命周期配置
============
.. Lifecycle Settings

桶的生命周期（ Lifecycle ）配置可以用于管理对象，
使得它们在整个生命周期内得到有效存储。在以前的版本中，
生命周期的处理效率被单线程耽误了；到 Nautilus 版，这个问题得到了彻底解决，
Ceph 对象网关现在可以调用另外的 Ceph 对象网关例程、并用随机排序的序列取代了\
按顺序排列的索引分片枚举，以此实现了桶生命周期的并行多线程处理。

在寻求提高生命周期处理的激进性时，有两个选项需要特别注意：

.. confval:: rgw_lc_max_worker
.. confval:: rgw_lc_max_wp_worker

你可以根据自己的负载情况对这些选项进行调整，以得到更激进的生命周期处理效率。
对于有大量桶（数千）的情形，你应该试着增加 :confval:`rgw_lc_max_worker` 的值，
它的默认值是 3 ，而对于桶数量较少、每个桶内对象数却更高（数十万）的情形，
你应该试着降低 :confval:`rgw_lc_max_wp_worker` ，默认值是 3 。

.. note:: 试着调整这两个特定的取值前，请验证当前的集群性能、
   以及 Ceph 对象网关的利用率。


垃圾回收选项
============
.. Garbage Collection Settings

Ceph 对象网关会立即为新对象分配存储。

网关从桶索引中删除对象一段时间之后， Ceph 对象网关才会把\
已删除对象和已被覆盖对象在 Ceph 存储集群里占用的存储空间清理掉。
从 Ceph 存储集群清理已删除对象数据的过程叫做垃圾回收（ Garbage Collection ）或 GC 。

要查看等待垃圾回收的对象队列，执行以下命令

.. prompt:: bash $

   radosgw-admin gc list

.. note:: 加 ``--include-all`` 罗列所有条目，包括未到期的。

垃圾回收是后台活动，可以持续运行或在负载低的时候运行，
取决于管理员是如何配置 Ceph 对象网关的。
默认情况下， Ceph 会让 GC 操作持续运行。
GC 操作是 Ceph 对象网关各种操作的常规部分，
特别是有对象删除操作时，大多数时候都存在适合垃圾回收的对象。

有的工作负荷会暂时或者永久地超过垃圾回收活动的速率。
特别是有很多对象短暂存储然后删除的工作载荷，会有大量删除。
对于这样的工作载荷，管理员可以用下面的配置参数，
适当增加垃圾回收操作相对于其它操作的优先级。

.. confval:: rgw_gc_max_objs
.. confval:: rgw_gc_obj_min_wait
.. confval:: rgw_gc_processor_max_time
.. confval:: rgw_gc_processor_period
.. confval:: rgw_gc_max_concurrent_io

:为删除量大的工作载荷调整垃圾回收:

要把 Ceph 的垃圾回收调整得更激进，首先，建议在默认值的基础上增大下面的选项::

  rgw_gc_max_concurrent_io = 20
  rgw_gc_max_trim_chunk = 64

.. note:: 修改这些值需要重启 RGW 服务。

这些值调整得高于默认值后，需要在垃圾回收期间监控集群性能，
检验一下增加这些值没有对性能带来负面影响。


多站设置
========
.. Multisite Settings

.. versionadded:: Jewel

你可以在 Ceph 配置文件中的各例程 ``[client.radosgw.{instance-name}]``
段下设置下列选项。

.. confval:: rgw_zone
.. confval:: rgw_zonegroup
.. confval:: rgw_realm
.. confval:: rgw_run_sync_thread
.. confval:: rgw_data_log_window
.. confval:: rgw_data_log_changes_size
.. confval:: rgw_data_log_obj_prefix
.. confval:: rgw_data_log_num_shards
.. confval:: rgw_md_log_max_shards

.. important:: 开始同步后就不应该再更改 ``rgw data log num shards`` 和
   ``rgw md log max shards`` 的取值了。

S3 选项
=======
.. S3 Settings

.. confval:: rgw_s3_auth_use_ldap

Swift 选项
==========
.. Swift Settings

.. confval:: rgw_enforce_swift_acls
.. confval:: rgw_swift_tenant_name
.. confval:: rgw_swift_token_expiration
.. confval:: rgw_swift_url
.. confval:: rgw_swift_url_prefix
.. confval:: rgw_swift_auth_url
.. confval:: rgw_swift_auth_entry
.. confval:: rgw_swift_account_in_url
.. confval:: rgw_swift_versioning_enabled
.. confval:: rgw_trust_forwarded_https

日志记录选项
============
.. Logging Settings

.. confval:: rgw_log_nonexistent_bucket
.. confval:: rgw_log_object_name
.. confval:: rgw_log_object_name_utc
.. confval:: rgw_usage_max_shards
.. confval:: rgw_usage_max_user_shards
.. confval:: rgw_enable_ops_log
.. confval:: rgw_enable_usage_log
.. confval:: rgw_ops_log_rados
.. confval:: rgw_ops_log_socket_path
.. confval:: rgw_ops_log_data_backlog
.. confval:: rgw_usage_log_flush_threshold
.. confval:: rgw_usage_log_tick_interval
.. confval:: rgw_log_http_headers

Keystone 选项
=============
.. Keystone Settings

.. confval:: rgw_keystone_url
.. confval:: rgw_keystone_api_version
.. confval:: rgw_keystone_admin_domain
.. confval:: rgw_keystone_admin_project
.. confval:: rgw_keystone_admin_token
.. confval:: rgw_keystone_admin_token_path
.. confval:: rgw_keystone_admin_tenant
.. confval:: rgw_keystone_admin_user
.. confval:: rgw_keystone_admin_password
.. confval:: rgw_keystone_admin_password_path
.. confval:: rgw_keystone_accepted_roles
.. confval:: rgw_keystone_token_cache_size
.. confval:: rgw_keystone_verify_ssl

服务端加密选项
==============
.. Server-side encryption Settings

.. confval:: rgw_crypt_s3_kms_backend

Barbican 选项
=============
.. Barbican Settings

.. confval:: rgw_barbican_url
.. confval:: rgw_keystone_barbican_user
.. confval:: rgw_keystone_barbican_password
.. confval:: rgw_keystone_barbican_tenant
.. confval:: rgw_keystone_barbican_project
.. confval:: rgw_keystone_barbican_domain

HashiCorp Vault 选项
====================
.. HashiCorp Vault Settings

.. confval:: rgw_crypt_vault_auth
.. confval:: rgw_crypt_vault_token_file
.. confval:: rgw_crypt_vault_addr
.. confval:: rgw_crypt_vault_prefix
.. confval:: rgw_crypt_vault_secret_engine
.. confval:: rgw_crypt_vault_namespace

SSE-S3 选项
===========
.. SSE-S3 Settings

.. confval:: rgw_crypt_sse_s3_backend
.. confval:: rgw_crypt_sse_s3_vault_secret_engine
.. confval:: rgw_crypt_sse_s3_key_template
.. confval:: rgw_crypt_sse_s3_vault_auth
.. confval:: rgw_crypt_sse_s3_vault_token_file
.. confval:: rgw_crypt_sse_s3_vault_addr
.. confval:: rgw_crypt_sse_s3_vault_prefix
.. confval:: rgw_crypt_sse_s3_vault_namespace
.. confval:: rgw_crypt_sse_s3_vault_verify_ssl
.. confval:: rgw_crypt_sse_s3_vault_ssl_cacert
.. confval:: rgw_crypt_sse_s3_vault_ssl_clientcert
.. confval:: rgw_crypt_sse_s3_vault_ssl_clientkey

QoS 选项
--------
.. QoS settings

.. versionadded:: Nautilus

``civetweb`` 前端有一个线程池模型，它给每个连接使用一个线程，因此它在接受请求时\
自动接受 ``rgw thread pool size`` 配置的约束。 ``beast`` 前端在接受新连接时\
不受线程池大小的限制，所以 Nautilus 版引进了调度器抽象层，
以此支持未来调度请求的多种方法。

当前，这个调度器默认就是个减速器，它可以把活跃连接数压制到配置的限值之下。
基于 mClock 的 QoS 现在还处于 *实验* 阶段，而且也不建议用于生产环境。
当前实现的 *dmclock_client* 操作队列会把 RGW 的各种操作分割成管理的、认证的
（ swift 认证、 sts ）、元数据和数据请求。


.. confval:: rgw_max_concurrent_requests
.. confval:: rgw_scheduler_type
.. confval:: rgw_dmclock_auth_res
.. confval:: rgw_dmclock_auth_wgt
.. confval:: rgw_dmclock_auth_lim
.. confval:: rgw_dmclock_admin_res
.. confval:: rgw_dmclock_admin_wgt
.. confval:: rgw_dmclock_admin_lim
.. confval:: rgw_dmclock_data_res
.. confval:: rgw_dmclock_data_wgt
.. confval:: rgw_dmclock_data_lim
.. confval:: rgw_dmclock_metadata_res
.. confval:: rgw_dmclock_metadata_wgt
.. confval:: rgw_dmclock_metadata_lim


.. _体系结构: ../../architecture#data-striping
.. _存储池配置: ../../rados/configuration/pool-pg-config-ref/
.. _集群存储池: ../../rados/operations/pools
.. _RADOS 集群句柄: ../../rados/api/librados-intro/#step-2-configuring-a-cluster-handle
.. _Barbican: ../barbican
.. _加密: ../encryption
.. _HTTP 前端: ../frontends
