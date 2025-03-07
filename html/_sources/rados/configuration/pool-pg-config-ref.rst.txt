.. _rados_config_pool_pg_crush_ref:

=================================
 存储池、归置组和 CRUSH 配置参考
=================================
.. Pool, PG and CRUSH Config Reference

.. index:: pools; configuration

CRUSH 算法分配给每个存储池的归置组数量，
由监视器集群的中央配置数据库中的变量值决定。

Ceph 的容器化部署（用 ``cephadm`` 或 Rook 做的部署）和 Ceph 的非容器化部署\
都依赖监视器集群内、中央配置数据库里的值为存储池分配归置组。

命令实例
--------
.. Example Commands

要查看指定存储池中决定归置组数量的变量值，执行下列命令：

.. prompt:: bash

   ceph config get osd osd_pool_default_pg_num

要设置指定存储池中决定归置组数量的变量值，执行下列命令：

.. prompt:: bash

   ceph config set osd osd_pool_default_pg_num

手动调整
--------
.. Manual Tuning

有些情况下，覆盖某些默认值更好。例如，您可能想设置存储池的副本数并\
覆盖存储池中归置组的默认数量。您可以用 `pool`_ 命令设置这些值。

参考
----
.. See Also

参见 :ref:`pg-autoscaler` 。


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
