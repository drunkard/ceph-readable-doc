本地存储池模块
==============
.. Local Pool Module

.. mgr_module:: localpool

The *localpool* module can automatically create RADOS pools that are
localized to a subset of the overall cluster.  For example, by default, it will
create a pool for each distinct ``rack`` in the cluster.  This can be useful for
deployments where it is desirable to distribute some data locally and other data
globally across the cluster.  One use-case is measuring performance and testing
behavior of specific drive, NIC, or chassis models in isolation.

启用
----
.. Enabling

*localpool* 模块用此命令启用： ::

  ceph mgr module enable localpool

配置
----
.. Configuring

*localpool* 模块支持以下选项：

.. confval:: subtree
.. confval:: failure_domain
.. confval:: pg_num
.. confval:: num_rep
.. confval:: min_size
.. confval:: prefix
   :default: by-$subtreetype-

These options are set via the config-key interface.  For example, to
change the replication level to 2x with only 64 PGs, ::

  ceph config set mgr mgr/localpool/num_rep 2
  ceph config set mgr mgr/localpool/pg_num 64

.. mgr_module:: None
