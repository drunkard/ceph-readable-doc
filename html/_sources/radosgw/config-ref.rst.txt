=======================
 Ceph 对象网关配置参考
=======================
.. Ceph Object Gateway Config Reference

下列的选项可加入 Ceph 配置文件（一般是 ``ceph.conf`` ）的
``[client.radosgw.{instance-name}]`` 段下，很多选项都有\
默认值，你若未指定，自然用默认。

Configuration variables set under the ``[client.radosgw.{instance-name}]``
section will not apply to rgw or radosgw-admin commands without an instance-name
specified in the command. Thus variables meant to be applied to all RGW
instances or all radosgw-admin options can be put into the ``[global]`` or the
``[client]`` section to avoid specifying ``instance-name``.

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
.. confval:: rgw_admin_entry
.. confval:: rgw_content_length_compat
.. confval:: rgw_bucket_quota_ttl
.. confval:: rgw_user_quota_bucket_sync_interval
.. confval:: rgw_user_quota_sync_interval
.. confval:: rgw_bucket_default_quota_max_objects
.. confval:: rgw_bucket_default_quota_max_size
.. confval:: rgw_user_default_quota_max_objects
.. confval:: rgw_user_default_quota_max_size
.. confval:: rgw_verify_ssl
.. confval:: rgw_max_chunk_size


Lifecycle Settings
==================

Bucket Lifecycle configuration can be used to manage your objects so they are stored
effectively throughout their lifetime. In past releases Lifecycle processing was rate-limited
by single threaded processing. With the Nautilus release this has been addressed and the
Ceph Object Gateway now allows for parallel thread processing of bucket lifecycles across
additional Ceph Object Gateway instances and replaces the in-order
index shard enumeration with a random ordered sequence.

There are two options in particular to look at when looking to increase the
aggressiveness of lifecycle processing:

.. confval:: rgw_lc_max_worker
.. confval:: rgw_lc_max_wp_worker

These values can be tuned based upon your specific workload to further increase the
aggressiveness of lifecycle processing. For a workload with a larger number of buckets (thousands)
you would look at increasing the :confval:`rgw_lc_max_worker` value from the default value of 3 whereas for a
workload with a smaller number of buckets but higher number of objects (hundreds of thousands)
per bucket you would consider decreasing :confval:`rgw_lc_max_wp_worker` from the default value of 3.

.. note:: When looking to tune either of these specific values please validate the
       current Cluster performance and Ceph Object Gateway utilization before increasing.


垃圾回收选项
============
.. Garbage Collection Settings

The Ceph Object Gateway allocates storage for new objects immediately.

The Ceph Object Gateway purges the storage space used for deleted and overwritten 
objects in the Ceph Storage cluster some time after the gateway deletes the 
objects from the bucket index. The process of purging the deleted object data 
from the Ceph Storage cluster is known as Garbage Collection or GC.

To view the queue of objects awaiting garbage collection, execute the following

.. prompt:: bash $

   radosgw-admin gc list

.. note:: specify ``--include-all`` to list all entries, including unexpired
  
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

.. confval:: rgw_gc_max_objs
.. confval:: rgw_gc_obj_min_wait
.. confval:: rgw_gc_processor_max_time
.. confval:: rgw_gc_processor_period
.. confval:: rgw_gc_max_concurrent_io

:Tuning Garbage Collection for Delete Heavy Workloads:

As an initial step towards tuning Ceph Garbage Collection to be more aggressive the following options are suggested to be increased from their default configuration values::

  rgw_gc_max_concurrent_io = 20
  rgw_gc_max_trim_chunk = 64

.. note:: Modifying these values requires a restart of the RGW service.

Once these values have been increased from default please monitor for performance of the cluster during Garbage Collection to verify no adverse performance issues due to the increased values.


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

.. important:: 开始同步后就不应该再更改
   ``rgw data log num shards`` 和 ``rgw md log max shards`` 的\
   取值了。

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

QoS 选项
--------
.. QoS settings

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
