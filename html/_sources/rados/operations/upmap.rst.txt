.. _upmap:

使用 pg-upmap
=============
.. Using the pg-upmap

从 Luminous v12.2.z 版起， OSDMap 里有一张新的 *pg-upmap* 例外\
表，以支持指定 PG 映射到指定的 OSD 。这样可以让集群可以更好地\
调整数据分布，在多数情况下，可以把 PG 完美地散布到各 OSD 上。

这个新机制的关键点是，它需要所有客户端都能理解 OSDMap 里新增的
*pg-upmap* 数据结构。

在线优化
========
.. Online Optimization

启用此功能
----------

In order to use ``pg-upmap``, the cluster cannot have any pre-Luminous clients.
By default, new clusters enable the *balancer module*, which makes use of
``pg-upmap``. If you want to use a different balancer or you want to make your
own custom ``pg-upmap`` entries, you might want to turn off the balancer in
order to avoid conflict: 

.. prompt:: bash $

   ceph balancer off

To allow use of the new feature on an existing cluster, you must restrict the
cluster to supporting only Luminous (and newer) clients.  To do so, run the
following command:

.. prompt:: bash $

   ceph osd set-require-min-compat-client luminous

This command will fail if any pre-Luminous clients or daemons are connected to
the monitors. To see which client versions are in use, run the following
command:

.. prompt:: bash $

   ceph features


均衡器模块
----------
.. Balancer Module

``ceph-mgr`` 的 `balancer` 模块会自动均衡各 OSD 的 PG 数量。见 :ref:`balancer` 。


离线优化
--------
.. Offline optimization

Upmap entries are updated with an offline optimizer that is built into the
:ref:`osdmaptool`.

#. Grab the latest copy of your osdmap:

   .. prompt:: bash $

      ceph osd getmap -o om

#. Run the optimizer:

   .. prompt:: bash $

      osdmaptool om --upmap out.txt [--upmap-pool <pool>] \ 
      [--upmap-max <max-optimizations>] \ 
      [--upmap-deviation <max-deviation>] \ 
      [--upmap-active]

   It is highly recommended that optimization be done for each pool
   individually, or for sets of similarly utilized pools. You can specify the
   ``--upmap-pool`` option multiple times. "Similarly utilized pools" means
   pools that are mapped to the same devices and that store the same kind of
   data (for example, RBD image pools are considered to be similarly utilized;
   an RGW index pool and an RGW data pool are not considered to be similarly
   utilized).

   The ``max-optimizations`` value determines the maximum number of upmap
   entries to identify. The default is `10` (as is the case with the
   ``ceph-mgr`` balancer module), but you should use a larger number if you are
   doing offline optimization.  If it cannot find any additional changes to
   make (that is, if the pool distribution is perfect), it will stop early.

   The ``max-deviation`` value defaults to `5`. If an OSD's PG count varies
   from the computed target number by no more than this amount it will be
   considered perfect.

   The ``--upmap-active`` option simulates the behavior of the active balancer
   in upmap mode. It keeps cycling until the OSDs are balanced and reports how
   many rounds have occurred and how long each round takes. The elapsed time
   for rounds indicates the CPU load that ``ceph-mgr`` consumes when it computes
   the next optimization plan.

#. Apply the changes:

   .. prompt:: bash $

      source out.txt

   In the above example, the proposed changes are written to the output file
   ``out.txt``. The commands in this procedure are normal Ceph CLI commands
   that can be run in order to apply the changes to the cluster.

The above steps can be repeated as many times as necessary to achieve a perfect
distribution of PGs for each set of pools.

To see some (gory) details about what the tool is doing, you can pass
``--debug-osd 10`` to ``osdmaptool``. To see even more details, pass
``--debug-crush 10`` to ``osdmaptool``.
