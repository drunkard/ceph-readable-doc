.. _cephfs_metrics:

指标
====
.. Metrics

CephFS 使用 :ref:`Perf Counters`\ 跟踪指标。可以给这些计数器打标签（ :ref:`Labeled Perf Counters` ）。

客户端指标
----------
.. Client Metrics

CephFS 把客户端指标导出为 :ref:`Labeled Perf Counters`\ ，可用于监控性能。 CephFS 输出以下客户端指标。

.. list-table:: 客户端指标
   :widths: 25 25 75
   :header-rows: 1

   * - Name
     - Type
     - Description
   * - num_clients
     - Gauge
     - 客户端会话数量
   * - cap_hits
     - Gauge
     - 文件具有的能力占所有能力总数的百分比
   * - cap_miss
     - Gauge
     - 文件不具有的能力占所有能力总数的百分比
   * - avg_read_latency
     - Gauge
     - 读延时平均值
   * - avg_write_latency
     - Gauge
     - 写延时平均值
   * - avg_metadata_latency
     - Gauge
     - 元数据延时平均值
   * - dentry_lease_hits
     - Gauge
     - 发放的 dentry 租约申请占 dentry 租约请求总量的百分比
   * - dentry_lease_miss
     - Gauge
     - 发放的 dentry 租约失败占 dentry 租约请求总量的百分比
   * - opened_files
     - Gauge
     - 已打开文件的数量
   * - opened_inodes
     - Gauge
     - 已打开 inode 的数量
   * - pinned_icaps
     - Gauge
     - 锁定的 Inode Caps 数量
   * - total_inodes
     - Gauge
     - Inodes 总数
   * - total_read_ops
     - Gauge
     - 所有进程产生的读出操作总数
   * - total_read_size
     - Gauge
     - 所有进程产生的输入、输出操作中，读取的字节数
   * - total_write_ops
     - Gauge
     - 所有进程产生的写入操作总数
   * - total_write_size
     - Gauge
     - 所有进程产生的输入、输出操作中，写入的字节数

查看指标
========
.. Getting Metrics

这些指标可以从 MDS 管理套接字中获取，也可以用 tell 接口获取。 ``counter dump`` 命令输出中的 ``mds_client_metrics-<fsname>`` 部分显示了每个客户端的指标，如下所示： ::

    "mds_client_metrics": [
        {
            "labels": {
                "fs_name": "<fsname>",
                "id": "14213"
            },
            "counters": {
                "num_clients": 2
            }
        }
    ],
    "mds_client_metrics-<fsname>": [
        {
            "labels": {
                "client": "client.0",
                "rank": "0"
            },
            "counters": {
                "cap_hits": 5149,
                "cap_miss": 1,
                "avg_read_latency": 0.000000000,
                "avg_write_latency": 0.000000000,
                "avg_metadata_latency": 0.000000000,
                "dentry_lease_hits": 0,
                "dentry_lease_miss": 0,
                "opened_files": 1,
                "opened_inodes": 2,
                "pinned_icaps": 2,
                "total_inodes": 2,
                "total_read_ops": 0,
                "total_read_size": 0,
                "total_write_ops": 4836,
                "total_write_size": 633864192
            }
        },
        {
            "labels": {
                "client": "client.1",
                "rank": "0"
            },
            "counters": {
                "cap_hits": 3375,
                "cap_miss": 8,
                "avg_read_latency": 0.000000000,
                "avg_write_latency": 0.000000000,
                "avg_metadata_latency": 0.000000000,
                "dentry_lease_hits": 0,
                "dentry_lease_miss": 0,
                "opened_files": 1,
                "opened_inodes": 2,
                "pinned_icaps": 2,
                "total_inodes": 2,
                "total_read_ops": 0,
                "total_read_size": 0,
                "total_write_ops": 3169,
                "total_write_size": 415367168
            }
        }
    ]
