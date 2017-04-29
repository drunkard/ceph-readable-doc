:orphan:

=================================
 rbd-replay -- 重放 RBD 工作负荷
=================================

.. program:: rbd-replay

提纲
====

| **rbd-replay** [ *options* ] *replay_file*


描述
====

**rbd-replay** 工具用于重放 RBD 载荷。


选项
====

.. option:: -c ceph.conf, --conf ceph.conf

   使用 ceph.conf 配置文件而非默认的 /etc/ceph/ceph.conf 来确定启动期间所需\
   的监视器地址。

.. option:: -p pool, --pool pool

   与指定存储池交互，默认为 'rbd' 。

.. option:: --latency-multiplier

   请求间延时加倍，默认为 1 。

.. option:: --read-only

   只重放非破坏性的请求。

.. option:: --map-image rule

   增加一条规则把跟踪文件中的映像名映射为重放集群中的映像名。此规则 \
   image1@snap1=image2@snap2 将把 image1 的快照 snap1 映射为 image2 的快照 \
   snap2 。

.. option:: --dump-perf-counters

   **实验功能**
   关闭映像前先把性能计数器转储到标准输出。如果关闭了多个映像或者同一映像被\
   打开、关闭多次，那么性能计数器就可能转储多次。性能计数器及其含义可能因版\
   本而不同。


实例
====

尽可能快地重放 workload1::

       rbd-replay --latency-multiplier=0 workload1

重放 workload1 ，并用 test_image 取代 prod_image::

       rbd-replay --map-image=prod_image=test_image workload1


使用范围
========

**rbd-replay** 是 Ceph 的一部分，这是个伸缩力强、开源、分布式的存储系统，\
更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`rbd-replay-prep <rbd-replay-prep>`\(8),
:doc:`rbd <rbd>`\(8)
