.. _rados_config_pool_pg_crush_ref:

=================================
 存储池、归置组和 CRUSH 配置参考
=================================
.. Pool, PG and CRUSH Config Reference

.. index:: pools; configuration

当你创建存储池并分别给它们设置归置组数量时，\
如果你没指定 Ceph 就用默认值。**我们建议**\ 覆盖一些默认值，\
特别是存储池的副本数和默认归置组数量，\
可以在运行 `pool`_ 命令的时候设置这些值。\
你也可以把配置写入 Ceph 配置文件的 ``[global]`` 段来覆盖默认值。


.. literalinclude:: pool-pg.conf
   :language: ini

.. confval:: mon_max_pool_pg_num
.. confval:: mon_pg_stuck_threshold
.. confval:: mon_pg_warn_min_per_osd
.. confval:: mon_pg_warn_min_objects
.. confval:: mon_pg_warn_min_pool_objects
.. confval:: mon_pg_check_down_all_threshold
.. confval:: mon_pg_warn_max_object_skew
.. confval:: mon_delta_reset_interval
.. confval:: osd_crush_chooseleaf_type
.. confval:: osd_crush_initial_weight
.. confval:: osd_pool_default_crush_rule
.. confval:: osd_pool_erasure_code_stripe_unit
.. confval:: osd_pool_default_size
.. confval:: osd_pool_default_min_size
.. confval:: osd_pool_default_pg_num
.. confval:: osd_pool_default_pgp_num
.. confval:: osd_pool_default_pg_autoscale_mode
.. confval:: osd_pool_default_flags
.. confval:: osd_max_pgls
.. confval:: osd_min_pg_log_entries
.. confval:: osd_max_pg_log_entries
.. confval:: osd_default_data_pool_replay_window
.. confval:: osd_max_pg_per_osd_hard_ratio


.. _pool: ../../operations/pools
.. _监控 OSD 和归置组: ../../operations/monitoring-osd-pg#peering
.. _调整桶条目的权重: ../../operations/crush-map#weightingbucketitems
