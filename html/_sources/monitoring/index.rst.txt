.. _monitoring:

==========
 监控概览
==========
.. Monitoring overview

这部分文档的目的是解释 Ceph 监控栈和主要 Ceph 指标的含义。

充分了解 Ceph 监控堆栈和指标后，用户就能创建定制的监控工具，
如 Prometheus 查询、 Grafana 面板或脚本。


Ceph 监控栈
===========
.. Ceph Monitoring stack

Ceph 提供了默认的监控堆栈，是由 cephadm 安装、并在 cephadm 文档的
:ref:`监控服务 <mgr-cephadm-monitoring>` 部分进行了说明。


Ceph 指标
=========
.. Ceph metrics

Ceph 指标的主要来源是每个 Ceph 守护进程暴露的性能计数器。
:doc:`../dev/perf_counters` 是原生的 Ceph 监控数据。

性能计数器由 Ceph exporter （导出器）守护进程转换为标准 Prometheus 指标。
这样的守护进程运行在每个 Ceph 集群主机上，并暴露一个指标端点（ end point ），
在该端点上，主机上运行的所有 Ceph 守护进程公开的\
所有性能计数器都以 Prometheus 指标的格式发布。

除了 Ceph exporter 外，还有另一个代理可以暴露 Ceph 指标。
它就是 Prometheus 管理器模块，用于公开与整个集群相关的指标，
基本上是单个 Ceph 守护进程不能生成的指标。

获取 Ceph 指标的主要来源是集群 Prometheus 服务器暴露的指标端点。
Ceph 给你提供了 Prometheus 端点，你可以从该端点获取完整的指标列表
（来自 Ceph exporter 守护进程和 Prometheus 管理器模块）并执行查询。

要获取集群中的 Prometheus 服务器端点，执行下列命令：

例如：

.. code-block:: bash

  # ceph orch ps --service_name prometheus
  NAME                         HOST                          PORTS   STATUS          REFRESHED  AGE  MEM USE  MEM LIM  VERSION  IMAGE ID      CONTAINER ID
  prometheus.cephtest-node-00  cephtest-node-00.cephlab.com  *:9095  running (103m)    50s ago   5w     142M        -  2.33.4   514e6a882f6e  efe3cbc2e521

有了这些信息，你就可以连接 ``http://cephtest-node-00.cephlab.com:9095`` ，
访问 Prometheus 服务器界面。

集群的完整指标列表（附带帮助）将可以在下列文件中看到：

``http://cephtest-node-00.cephlab.com:9095/api/v1/targets/metadata``

需要说明的是，用户观察和监控 Ceph 集群的主要工具是 **Ceph dashboard** 。它提供的图形界面能展示最重要的集群和服务指标。本文档中的大多数示例都是从仪表盘（ dashboard ）图例中提取或从 Ceph 仪表板展示的指标中推断出来的。


Ceph 守护进程健康指标
=====================
.. Ceph daemon health metrics

Ceph exporter 提供了一个名为 ``ceph_daemon_socket_up`` 的指标，用于报告暴露管理套接字的各个 Ceph 守护进程的存活状态。

``ceph_daemon_socket_up`` 指标反映着 Ceph 守护进程的健康状态，是基于它通过管理套接字做出响应的能力，值为 ``1`` 表示健康， ``0`` 表示不健康。虽然 Ceph 守护进程报告 ``ceph_daemon_socket_up=0`` 的时候可能仍然“活着（ alive ）”，但这种情况凸显了其功能中的重大问题。正因如此，这个指标成了探测所有主要 Ceph 守护进程问题的绝佳工具。

标签：
- **``ceph_daemon``**: Ceph 守护进程标识符，它会在主机上暴露管理套接字（ admin socket ）。
- **``hostname``**: Ceph 守护进程所在主机的名字。

实例：

.. code-block:: bash

   ceph_daemon_socket_up{ceph_daemon="mds.a",hostname="testhost"} 1
   ceph_daemon_socket_up{ceph_daemon="osd.1",hostname="testhost"} 0

要找出过去 12 小时内没有响应的所有 Ceph 守护进程，可用下列 PromQL 表达式：

.. code-block:: bash

   ceph_daemon_socket_up == 0 or min_over_time(ceph_daemon_socket_up[12h]) == 0


性能指标
========
.. Performance metrics

用于衡量集群 Ceph 性能的主要指标：

所有指标都有下列标签：
``ceph_daemon`` ：产生指标的 OSD 守护进程的标识符
``instance`` ：暴露度量指标的 ceph exporter 实例的 IP 地址。
``job`` ： prometheus 扫描作业

实例：

.. code-block:: bash

   ceph_osd_op_r{ceph_daemon="osd.0", instance="192.168.122.7:9283", job="ceph"} = 73981

*集群 I/O （吞吐量）：*
用 ``ceph_osd_op_r_out_bytes`` 和 ``ceph_osd_op_w_in_bytes`` 获取客户端产生的集群吞吐量

实例：

.. code-block:: bash

  Writes (B/s):
  sum(irate(ceph_osd_op_w_in_bytes[1m]))

  Reads (B/s):
  sum(irate(ceph_osd_op_r_out_bytes[1m]))


*集群 I/O （操作）：*
用 ``ceph_osd_op_r`` 、 ``ceph_osd_op_w`` 获取客户端产生的操作数量

实例：

.. code-block:: bash

  Writes (ops/s):
  sum(irate(ceph_osd_op_w[1m]))

  Reads (ops/s):
  sum(irate(ceph_osd_op_r[1m]))

*延迟：*
用 ``ceph_osd_op_latency_sum`` ，它表示在客户端发出数据传输指令后、 OSD 开始传输数据前的延迟时间。

实例：

.. code-block:: bash

  sum(irate(ceph_osd_op_latency_sum[1m]))


OSD 性能
========
.. OSD performance

前面介绍的集群性能指标是基于 OSD 指标的，选择正确的标签，我们就可以为单个 OSD 获取到与集群性能指标相当的信息：

实例：

.. code-block:: bash

  OSD 0 read latency
  irate(ceph_osd_op_r_latency_sum{ceph_daemon=~"osd.0"}[1m]) / on (ceph_daemon) irate(ceph_osd_op_r_latency_count[1m])

  OSD 0 write IOPS
  irate(ceph_osd_op_w{ceph_daemon=~"osd.0"}[1m])

  OSD 0 write thughtput (bytes)
  irate(ceph_osd_op_w_in_bytes{ceph_daemon=~"osd.0"}[1m])

  OSD.0 total raw capacity available
  ceph_osd_stat_bytes{ceph_daemon="osd.0", instance="cephtest-node-00.cephlab.com:9283", job="ceph"} = 536451481


物理磁盘性能
============
.. Physical disk performance:

将 Prometheus ``node_exporter`` 指标与 Ceph 指标相结合，
我们就能获得 OSD 所用物理磁盘提供的性能信息。

实例：

.. code-block:: bash

  OSD 0 所用设备的读延时:
  label_replace(irate(node_disk_read_time_seconds_total[1m]) / irate(node_disk_reads_completed_total[1m]), "instance", "$1", "instance", "([^:.]*).*") and on (instance, device) label_replace(label_replace(ceph_disk_occupation_human{ceph_daemon=~"osd.0"}, "device", "$1", "device", "/dev/(.*)"), "instance", "$1", "instance", "([^:.]*).*")

  OSD 0 所用设备的写延时：
  label_replace(irate(node_disk_write_time_seconds_total[1m]) / irate(node_disk_writes_completed_total[1m]), "instance", "$1", "instance", "([^:.]*).*") and on (instance, device) label_replace(label_replace(ceph_disk_occupation_human{ceph_daemon=~"osd.0"}, "device", "$1", "device", "/dev/(.*)"), "instance", "$1", "instance", "([^:.]*).*")

  IOPS (OSD.0 所用的设备)
  reads:
  label_replace(irate(node_disk_reads_completed_total[1m]), "instance", "$1", "instance", "([^:.]*).*") and on (instance, device) label_replace(label_replace(ceph_disk_occupation_human{ceph_daemon=~"osd.0"}, "device", "$1", "device", "/dev/(.*)"), "instance", "$1", "instance", "([^:.]*).*")

  writes:
  label_replace(irate(node_disk_writes_completed_total[1m]), "instance", "$1", "instance", "([^:.]*).*") and on (instance, device) label_replace(label_replace(ceph_disk_occupation_human{ceph_daemon=~"osd.0"}, "device", "$1", "device", "/dev/(.*)"), "instance", "$1", "instance", "([^:.]*).*")

  吞吐量 (OSD.0 所用的设备)
  reads:
  label_replace(irate(node_disk_read_bytes_total[1m]), "instance", "$1", "instance", "([^:.]*).*") and on (instance, device) label_replace(label_replace(ceph_disk_occupation_human{ceph_daemon=~"osd.0"}, "device", "$1", "device", "/dev/(.*)"), "instance", "$1", "instance", "([^:.]*).*")

  writes:
  label_replace(irate(node_disk_written_bytes_total[1m]), "instance", "$1", "instance", "([^:.]*).*") and on (instance, device) label_replace(label_replace(ceph_disk_occupation_human{ceph_daemon=~"osd.0"}, "device", "$1", "device", "/dev/(.*)"), "instance", "$1", "instance", "([^:.]*).*")

  过去 5 分钟， OSD.0 的物理设备利用率 (%)
  label_replace(irate(node_disk_io_time_seconds_total[5m]), "instance", "$1", "instance", "([^:.]*).*") and on (instance, device) label_replace(label_replace(ceph_disk_occupation_human{ceph_daemon=~"osd.0"}, "device", "$1", "device", "/dev/(.*)"), "instance", "$1", "instance", "([^:.]*).*")

存储池指标
==========
.. Pool metrics

这些指标的标签如下：
``instance`` ：生成度量指标的 Ceph 输出守护进程的 IP 地址。
``pool_id`` ：池的标识符
``job`` ：prometheus scrape （扫描）作业。

- ``ceph_pool_metadata``: 这个存储池的信息。它可与其他指标一起使用，就能在\
  查询和图表中提供更多上下文信息。除这三个常用标签外，该指标还提供下列额外标签：

  - ``compression_mode``: 存储池中使用的压缩方式（ lz4、snappy、zlib、zstd、none）。
    示例： compression_mode="none"

  - ``description``: 存储池类型的简要说明（replica: 副本数或纠删码： ec profile ）。
    例如： description="replica:3"
  - ``name``: 存储池名字，实例： name=".mgr"
  - ``type``: 存储池类型 (replicated/erasure code) 。实例： type="replicated"

- ``ceph_pool_bytes_used``: 用户数据消耗的原始总容量以及相关的存储池开销（元数据 + 冗余）：

- ``ceph_pool_stored``: 存储池内的\ **客户端**\ 数据总数

- ``ceph_pool_compress_under_bytes``: 存储池内符合压缩条件的数据

- ``ceph_pool_compress_bytes_used``:  存储池内已压缩的数据

- ``ceph_pool_rd``: 每个存储池的\ **客户端**\ 读操作（每秒读取数量）

- ``ceph_pool_rd_bytes``: 每个存储池的\ **客户端**\ 读操作字节数

- ``ceph_pool_wr``: 每个存储池的\ **客户端**\ 写操作（每秒写入数量）

- ``ceph_pool_wr_bytes``: 每个存储池的\ **客户端**\ 写操作字节数


**有用的查询**:

.. code-block:: bash

  整个集群的可用原始容量：
  sum(ceph_osd_stat_bytes)

  整个集群用掉的原始容量（包括元数据、冗余）：
  sum(ceph_pool_bytes_used)

  集群内存储的客户端数据总量：
  sum(ceph_pool_stored)

  压缩节约的：
  sum(ceph_pool_compress_under_bytes - ceph_pool_compress_bytes_used)

  一个存储池（ testrbdpool ）的客户端 IOPS
  reads: irate(ceph_pool_rd[1m]) * on(pool_id) group_left(instance,name) ceph_pool_metadata{name=~"testrbdpool"}
  writes: irate(ceph_pool_wr[1m]) * on(pool_id) group_left(instance,name) ceph_pool_metadata{name=~"testrbdpool"}

  一个存储池的客户端吞吐量：
  reads: irate(ceph_pool_rd_bytes[1m]) * on(pool_id) group_left(instance,name) ceph_pool_metadata{name=~"testrbdpool"}
  writes: irate(ceph_pool_wr_bytes[1m]) * on(pool_id) group_left(instance,name) ceph_pool_metadata{name=~"testrbdpool"}

对象指标
========
.. Object metrics

这些指标的标签如下：
``instance``: 提供指标的 ceph exporter 守护进程的 IP 地址
``instance_id``: rgw 守护进程的标识符
``job``: prometheus scrape （扫描）作业

实例：

.. code-block:: bash

  ceph_rgw_req{instance="192.168.122.7:9283", instance_id="154247", job="ceph"} = 12345


通用指标
--------
.. Generic metrics

- ``ceph_rgw_metadata``: 提供 RGW 守护进程的通用信息。它可与其他指标一起使用，
  以便在查询和图表中提供更多上下文信息。除这三个通用标签外，该指标还提供下列额外标签：

  - ``ceph_daemon``: Ceph 守护进程名字。实例：
    ceph_daemon="rgw.rgwtest.cephtest-node-00.sxizyq",
  - ``ceph_version``: Ceph 守护进程版本。实例： ceph_version="ceph
    version 17.2.6 (d7ff0d10654d2280e08f1ab989c7cdf3064446a5) quincy (stable)" 。
  - ``hostname``: 此守护进程所在主机的名字。实例：
    hostname:"cephtest-node-00.cephlab.com" 。

- ``ceph_rgw_req``: 此守护进程处理的请求总数（ GET+PUT+DELETE ）
    探测瓶颈和优化负载分布时有用。

- ``ceph_rgw_qlen``: 此守护进程的 RGW 操作队列长度。
    探测瓶颈和优化负载分布时有用。

- ``ceph_rgw_failed_req``: 中止的请求。
    探测守护进程错误时有用。


GET 操作：相关的指标
--------------------
.. GET operations: related metrics

- ``ceph_rgw_get_initial_lat_count``: GET 操作数量

- ``ceph_rgw_get_initial_lat_sum``: GET 操作的总延时

- ``ceph_rgw_get``: GET 请求总数

- ``ceph_rgw_get_b``: GET 操作传输的总字节数


PUT 操作：相关的指标
--------------------
.. Put operations: related metrics

- ``ceph_rgw_put_initial_lat_count``: PUT 操作数量

- ``ceph_rgw_put_initial_lat_sum``: PUT 操作的总延时

- ``ceph_rgw_put``: PUT 操作的总数

- ``ceph_rgw_get_b``: PUT 操作传输的总字节数


有用的查询
----------
.. Useful queries

.. code-block:: bash

  GET 的平均延时：
  rate(ceph_rgw_get_initial_lat_sum[30s]) / rate(ceph_rgw_get_initial_lat_count[30s]) * on (instance_id) group_left (ceph_daemon) ceph_rgw_metadata

  PUT 的平均延时：
  rate(ceph_rgw_put_initial_lat_sum[30s]) / rate(ceph_rgw_put_initial_lat_count[30s]) * on (instance_id) group_left (ceph_daemon) ceph_rgw_metadata

  每秒请求总数：
  rate(ceph_rgw_req[30s]) * on (instance_id) group_left (ceph_daemon) ceph_rgw_metadata

  “其他”操作（ LIST 、 DELETE ）的总数
  rate(ceph_rgw_req[30s]) -  (rate(ceph_rgw_get[30s]) + rate(ceph_rgw_put[30s]))

  GET 延时
  rate(ceph_rgw_get_initial_lat_sum[30s]) /  rate(ceph_rgw_get_initial_lat_count[30s]) * on (instance_id) group_left (ceph_daemon) ceph_rgw_metadata

  PUT 延时
  rate(ceph_rgw_put_initial_lat_sum[30s]) /  rate(ceph_rgw_put_initial_lat_count[30s]) * on (instance_id) group_left (ceph_daemon) ceph_rgw_metadata

  GET 操作消耗的带宽
  sum(rate(ceph_rgw_get_b[30s]))

  PUT 操作消耗的带宽
  sum(rate(ceph_rgw_put_b[30s]))

  RGW 例程消耗的带宽（ PUT 、 GET 操作）
  sum by (instance_id) (rate(ceph_rgw_get_b[30s]) + rate(ceph_rgw_put_b[30s])) * on (instance_id) group_left (ceph_daemon) ceph_rgw_metadata

  Http 错误:
  rate(ceph_rgw_failed_req[30s])


文件系统指标
============
.. Filesystem Metrics

这些指标的标签如下：
``ceph_daemon``: MDS 守护进程的名字。
``instance``: 暴露指标的 ceph exporter 守护进程的 IP 地址（和端口）。
``job``: prometheus scrape （扫描）作业

实例：

.. code-block:: bash

  ceph_mds_request{ceph_daemon="mds.test.cephtest-node-00.hmhsoh", instance="192.168.122.7:9283", job="ceph"} = 1452


主要指标
--------
.. Main metrics

- ``ceph_mds_metadata``: 提供 MDS 守护进程的一般信息。它可与其他指标一起使用，
  以便在查询和图表中提供更多上下文信息。它提供下列额外标签：

  - ``ceph_version``: MDS 守护进程的 Ceph 版本
  - ``fs_id``: 文件系统集群 id （ fscid ）
  - ``hostname``: MDS 守护进程所在主机的名字
  - ``public_addr``: MDS 守护进程所用的公网地址（ public address ）
  - ``rank``: 这个 MDS 守护进程的 rank

实例：

.. code-block:: bash

 ceph_mds_metadata{ceph_daemon="mds.test.cephtest-node-00.hmhsoh", ceph_version="ceph version 17.2.6 (d7ff0d10654d2280e08f1ab989c7cdf3064446a5) quincy (stable)", fs_id="-1", hostname="cephtest-node-00.cephlab.com", instance="cephtest-node-00.cephlab.com:9283", job="ceph", public_addr="192.168.122.145:6801/118896446", rank="-1"}


- ``ceph_mds_request``: 这个 MDS 守护进程处理的请求总数

- ``ceph_mds_reply_latency_sum``: 响应延时总计

- ``ceph_mds_reply_latency_count``: 响应延时计数

- ``ceph_mds_server_handle_client_request``: 客户端请求数量

- ``ceph_mds_sessions_session_count``: 会话计数

- ``ceph_mds_sessions_total_load``: 总负载

- ``ceph_mds_sessions_sessions_open``: 当前打开着的会话

- ``ceph_mds_sessions_sessions_stale``: 当前过期的会话

- ``ceph_objecter_op_r``: 读操作数量

- ``ceph_objecter_op_w``: 写操作数量

- ``ceph_mds_root_rbytes``: 此守护进程管理的总字节数

- ``ceph_mds_root_rfiles``: 此守护进程管理的文件总数


有用的查询
----------
.. Useful queries:

.. code-block:: bash

  MDS 守护进程们的读出载荷总计：
  sum(rate(ceph_objecter_op_r[1m]))

  MDS 守护进程们的写入载荷总计：
  sum(rate(ceph_objecter_op_w[1m]))

  MDS 守护进程读载荷：（守护进程名为 mdstest ）
  sum(rate(ceph_objecter_op_r{ceph_daemon=~"mdstest"}[1m]))

  MDS 守护进程写入载荷：（守护进程名为 mdstest ）
  sum(rate(ceph_objecter_op_r{ceph_daemon=~"mdstest"}[1m]))

  响应的平均延时：
  rate(ceph_mds_reply_latency_sum[30s]) / rate(ceph_mds_reply_latency_count[30s])

  每秒的总请求数：
  rate(ceph_mds_request[30s]) * on (instance) group_right (ceph_daemon) ceph_mds_metadata


块设备指标
==========
.. Block metrics

默认不提供映像的 RBD 指标，是为了 prometheus 管理器模块能达到最佳性能。

要生成 RBD 映像的指标，需要正确配置管理器选项 ``mgr/prometheus/rbd_stats_pools`` 。
更多信息参阅 :ref:`prometheus-rbd-io-statistics` 。

这些指标的标签如下：
``image``: 产生指标数值的映像的名字。
``instance``: 产生这个 RBD 指标的节点（也就是 Ceph exporter 守护进程）
``job``: Prometheus scrape （扫描）作业的名字。
``pool``: 映像存储池名字。

实例：

.. code-block:: bash

  ceph_rbd_read_bytes{image="test2", instance="cephtest-node-00.cephlab.com:9283", job="ceph", pool="testrbdpool"}


主要指标
--------
.. Main metrics

- ``ceph_rbd_read_bytes``: RBD 映像的读字节数

- ``ceph_rbd_read_latency_count``: RBD 映像的读延时计数

- ``ceph_rbd_read_latency_sum``: RBD 映像的读延时总计

- ``ceph_rbd_read_ops``: RBD 映像的读取计数

- ``ceph_rbd_write_bytes``: RBD 映像的写入字节数

- ``ceph_rbd_write_latency_count``: RBD 映像的写延时计数

- ``ceph_rbd_write_latency_sum``: RBD 映像的写延时总计

- ``ceph_rbd_write_ops``: RBD 映像的写入计数


有用的查询
----------
.. Useful queries

.. code-block:: bash

  读延时平均值：
  rate(ceph_rbd_read_latency_sum[30s]) / rate(ceph_rbd_read_latency_count[30s]) * on (instance) group_left (ceph_daemon) ceph_rgw_metadata


硬件监控
========
.. Hardware monitoring

见 :ref:`hardware-monitoring` 。

