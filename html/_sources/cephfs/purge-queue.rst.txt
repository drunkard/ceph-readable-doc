========
清理队列
========
.. Purge Queue

MDS 维护着一个名为“清理队列（ Purge Queue ）”的数据结构，
负责管理和执行文件的并行删除。每个 MDS rank 都有一个清理队列。
清理队列由清理条目组成，其中包含来自 inode 的标称信息，
如大小和布局（即其他所有不需要的元数据信息都会忽略，
使其独立于所有元数据结构）。


删除过程
========
.. Deletion process

当一个客户端请求删除一个目录时（假设用 ``rm -rf`` ）：

- MDS 把 pq （清理队列， purge queue ）日志里的那些文件和子目录
  （清理条目， purge items ）加进清理队列。
- 在后台以微小、且可管理的小块处理 inode 的删除。
- MDS 发指令让底层的 OSD 们清理掉数据存储池里的相关对象。
- 更新日志

.. note:: 如果用户删除文件的速度超过了清理队列的处理能力，
   那么随着时间的推移，数据存储池的使用量可能会增加。
   在极端情况下，清理队列积压得非常巨大，
   以至于减缓了容量回收，此时，
   CephFS 的 linux ``du`` 命令报告的数据\
   可能与 CephFS 数据存储池不一致。

有几个可调节的配置， MDS 内部用它们节制\
清理队列的处理：

.. confval:: filer_max_purge_ops
.. confval:: mds_max_purge_files
.. confval:: mds_max_purge_ops
.. confval:: mds_max_purge_ops_per_pg

一般来说，默认值足以满足大多数集群的需要。不过，
如果集群非常庞大，需要 ``pq_item_in_journal``
（待删除内容计数器）达到很大的数值，
那么可以试着把此配置调整为默认值的 4-5 倍，
进一步的增量取决于更大的需求。

从最微不足道的 ``filer_max_purge_ops`` 配置开始，
这应该有助于更快地回收空间： ::

    $ ceph config set mds filer_max_purge_ops 40

对大多数集群来说，增加 ``filer_max_purge_ops`` 应该就够了，
但如果不行，那就继续调整其他配置选项： ::

    $ ceph config set mds mds_max_purge_files 256
    $ ceph config set mds mds_max_purge_ops 32768
    $ ceph config set mds mds_max_purge_ops_per_pg 2

.. note:: 设置这些值不会立即破坏任何东西，
   因为它们只是控制着我们能向底层 RADOS 集群发出多少删除操作，
   如果设置的值高得惊人，
   可能会牺牲一些集群性能。

.. note:: 就作业限制而言，与正在进行的工作相比，
   清理队列并不是一个可以自动调整的系统。所以建议，在调整配置时，
   应该根据集群规模和载荷做明智的决定。


检查清理队列的 perf 计数器
==========================
.. Examining purge queue perf counters

分析 MDS 的 perf dump 时，清理队列的统计信息大致如下： ::

    "purge_queue": {
        "pq_executing_ops": 56655,
        "pq_executing_ops_high_water": 65350,
        "pq_executing": 1,
        "pq_executing_high_water": 3,
        "pq_executed": 25,
        "pq_item_in_journal": 6567004
    }

我们了解一下每一项的含义：

.. list-table::
   :widths: 50 50
   :header-rows: 1

   * - 名字
     - 描述
   * - pq_executing_ops
     - 正在进行的清理队列操作数
   * - pq_executing_ops_high_water
     - 记录到的正在执行的清理操作最大数量
   * - pq_executing 
     - 正在删除的清理队列文件
   * - pq_executing_high_water 
     - 正在执行的文件清理最大数量
   * - pq_executed 
     - 已删除的清理队列文件
   * - pq_item_in_journal
     - 日志里残留的清理条目（文件）

.. note:: ``pq_executing`` 和 ``pq_executing_ops`` 看起来很相似，
   但两者之间有细微差别。 ``pq_executing`` 追踪的是清理队列中的文件数量，
   而 ``pq_executing_ops`` 统计的是清理队列中所有文件的 RADOS 对象数量。
