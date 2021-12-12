客户端配置
==========

更新客户端配置
--------------
.. Updating Client Configuration

有些客户端配置可以在运行时应用生效。要看一个配置选项能否在运行时生效
（客户端接受并生效），用 `config help` 命令： ::

   ceph config help debug_client
    debug_client - Debug level for client
    (str, advanced)                                                                                                                      Default: 0/5
    Can update at runtime: true

    The value takes the form 'N' or 'N/M' where N and M are values between 0 and 99.  N is the debug level to log (all values below this are included), and M is the level to gather and buffer in memory.  In the event of a crash, the most recent items <= M are dumped to the log file.

`config help` 会告诉你指定的配置能否在运行时生效，
还有它的默认值、这个配置选项的描述。

要在运行时更新一个配置选项，用 `config set` 命令： ::

   ceph config set client debug_client 20/20

注意，这个配置变更会应用于所有客户端。

要检查配置的选项，用 `config get` 命令： ::

   ceph config get client
    WHO    MASK LEVEL    OPTION                    VALUE     RO 
    client      advanced debug_client              20/20          
    global      advanced osd_pool_default_min_size 1            
    global      advanced osd_pool_default_size     3            

客户端配置参考
--------------
.. Client Config Reference

.. confval:: client_acl_type
.. confval:: client_cache_mid
.. confval:: client_cache_size
.. confval:: client_caps_release_delay
.. confval:: client_debug_force_sync_read
.. confval:: client_dirsize_rbytes
.. confval:: client_max_inline_size
.. confval:: client_metadata
.. confval:: client_mount_gid
.. confval:: client_mount_timeout
.. confval:: client_mount_uid
.. confval:: client_mountpoint
.. confval:: client_oc
.. confval:: client_oc_max_dirty
.. confval:: client_oc_max_dirty_age
.. confval:: client_oc_max_objects
.. confval:: client_oc_size
.. confval:: client_oc_target_dirty
.. confval:: client_permissions
.. confval:: client_quota_df
.. confval:: client_readahead_max_bytes
.. confval:: client_readahead_max_periods
.. confval:: client_readahead_min
.. confval:: client_reconnect_stale
.. confval:: client_snapdir
.. confval:: client_tick_interval
.. confval:: client_use_random_mds
.. confval:: fuse_default_permissions
.. confval:: fuse_max_write
.. confval:: fuse_disable_pagecache

开发者选项
##########
.. Developer Options

.. important:: 以下选项仅供内部测试，只是为了保持文档完整才罗列在这里。

.. confval:: client_debug_getattr_caps
.. confval:: client_debug_inject_tick_delay
.. confval:: client_inject_fixed_oldest_tid
.. confval:: client_inject_release_failure
.. confval:: client_trace
