.. Using the pg-upmap

使用 pg-upmap
=============

从 Luminous v12.2.z 版起， OSDMap 里有一张新的 *pg-upmap* 例外\
表，以支持指定 PG 映射到指定的 OSD 。这样可以让集群可以更好地\
调整数据分布，在多数情况下，可以把 PG 完美地散布到各 OSD 上。

这个新机制的关键点是，它需要所有客户端都能理解 OSDMap 里新增的
*pg-upmap* 数据结构。

启用此功能
----------

To allow use of the feature, you must tell the cluster that it only
needs to support luminous (and newer) clients with::

  ceph osd set-require-min-compat-client luminous

This command will fail if any pre-luminous clients or daemons are
connected to the monitors.  You can see what client versions are in
use with::

  ceph features


一句忠告
--------
这是一个新功能，而且不太友好。写下此文档时，我们正致力于
ceph-mgr 的 `balancer` 新模块，它有可能让这一切实现自动化。

到那时，


.. Offline optimization

离线优化
--------

Upmap entries are updated with an offline optimizer built into ``osdmaptool``.

#. Grab the latest copy of your osdmap::

     ceph osd getmap -o om

#. Run the optimizer::

     osdmaptool om --upmap out.txt [--upmap-pool <pool>] [--upmap-max <max-count>] [--upmap-deviation <max-deviation>]

   It is highly recommended that optimization be done for each pool
   individually, or for sets of similarly-utilized pools.  You can
   specify the ``--upmap-pool`` option multiple times.  "Similar pools"
   means pools that are mapped to the same devices and store the same
   kind of data (e.g., RBD image pools, yes; RGW index pool and RGW
   data pool, no).

   The ``max-count`` value is the maximum number of upmap entries to
   identify in the run.  The default is 100, but you may want to make
   this a smaller number so that the tool completes more quickly (but
   does less work).  If it cannot find any additional changes to make
   it will stop early (i.e., when the pool distribution is perfect).

   The ``max-deviation`` value defaults to `.01` (i.e., 1%).  If an OSD
   utilization varies from the average by less than this amount it
   will be considered perfect.

#. The proposed changes are written to the output file ``out.txt`` in
   the example above.  These are normal ceph CLI commands that can be
   run to apply the changes to the cluster.  This can be done with::

     source out.txt

The above steps can be repeated as many times as necessary to achieve
a perfect distribution of PGs for each set of pools.

You can see some (gory) details about what the tool is doing by
passing ``--debug-osd 10`` to ``osdmaptool``.
