.. _cephfs-top:

===============
CephFS Top 工具
===============
.. CephFS Top Utility

CephFS 提供了一个 `top(1)` 风格的工具，用于实时显示各种 Ceph 文件系统指标。
`cephfs-top` 是一个基于 curses 的 Python 脚本，
它利用 Ceph 管理器中的 `stats` 插件来获取（并显示）指标。

管理器插件
==========
.. Manager Plugin

Ceph 文件系统客户端会周期性地把各种指标转发给 Ceph 元数据服务器 (MDS) ，
然后再由 MDS 的 rank 0 转发给 Ceph 管理器。每个活跃 MDS 都会把各自的指标集\
转发给 MDS 的 rank 0 。指标汇总后转发给 Ceph 管理器。

指标分为两类 - 全局指标和 MDS 个体指标。全局指标代表整个文件系统的指标集（例如，
客户端读延时），而 mds 个体指标则是一个特定 MDS rank 的（例如，这个 MDS 处理的子树数量）。

.. note:: 目前，只追踪全局指标。

`stats` 插件默认是禁用的，可以用下列命令启用： ::

  $ ceph mgr module enable stats

启用后， Ceph 文件系统指标可以这样获取： ::

  $ ceph fs perf stats

输出格式是 JSON ，包含以下字段：

- `version`: stats 输出的版本
- `global_counters`: 全局性能指标列表
- `counters`: mds 个体性能指标列表
- `client_metadata`: Ceph 文件系统客户端元数据
- `global_metrics`: 全局性能计数器
- `metrics`: MDS 个体性能计数器（目前还是空的）、以及延迟的 rank

.. note:: `delayed_ranks` 是正在报告过期指标的一些活跃 MDS rank 。
   这种情况可能是由于 MDS rank 0 和其他活跃 MDS 之间的（临时）网络问题。

指标可以从指定客户端和/或多个活跃 MDS 提取。
提取指定客户端的指标（比如 client-id 为 1234 的）： ::

  $ ceph fs perf stats --client_id=1234

只提取活跃 MDS 一个子集（如 MDS rank 1 和 2）的指标： ::

  $ ceph fs perf stats --mds_rank=1,2

`cephfs-top`
============

`cephfs-top` 工具要靠 `stats` 插件提取性能指标，并按 `top(1)` 风格展示。
`cephfs-top` 工具在 `cephfs-top` 软件包里。

默认情况下， `cephfs-top` 以 `client.fstop` 用户的名义连接 Ceph 集群： ::

  $ ceph auth get-or-create client.fstop mon 'allow r' mds 'allow r' osd 'allow r' mgr 'allow r'
  $ cephfs-top

字段描述
--------
.. Description of Fields

1. chit     : Cap hit
             用到的能力占能力总数的百分比

2. dlease   : Dentry lease （dentry 租约）
             发放出去的 dentry 租约占 dentry 租约请求总数的百分比

3. ofiles   : Opened files （打开的文件）
             打开的文件数

4. oicaps   : Pinned caps （锁定的能力）
             被锁定的能力数量

5. oinodes  : Opened inodes （打开的 inode ）
             打开的 inode 数量

6. rtio     : Total size of read IOs （读 IO 总尺寸）
             所有进程产生的 IO 操作中，读出的总字节数。

7. wtio     : Total size of write IOs （写 IO 总尺寸）
             所有进程产生的 IO 操作中，写入的总字节数。

8. raio     : Average size of read IOs （读 IO 平均尺寸）
             所有进程产生的所有已完成的读出 IO 操作，它们的平均字节数。

9. waio     : Average size of write IOs （写 IO 平均尺寸）
             所有进程产生的所有已完成的写入 IO 操作，它们的平均字节数。

10. rsp     : Read speed （读速度）
             从上次刷新客户端以来的读取 IO 速度。

11. wsp     : Write speed （写速度）
             从上次刷新客户端以来的写入 IO 速度。

12. rlatavg : Average read latency （读延时均值）
             读延时的平均值

13. rlatsd  : Standard deviation (variance) for read latency （读取延时的标准差［方差］）
             读延时指标相对于平均值的离散程度

14. wlatavg : Average write latency （写延时均值）
             写延时平均值

15. wlatsd  : Standard deviation (variance) for write latency （写入延时的标准差［方差］）
             写延时指标相对于平均值的离散程度

16. mlatavg : Average metadata latency （元数据延时均值）
             元数据延时的平均值

17. mlatsd  : Standard deviation (variance) for metadata latency （元数据延时的标准差［方差］）
             元数据延时指标相对于平均值的离散程度

命令行选项
----------
.. Command-Line Options

要用非默认用户（ `client.fstop` 以外的），像这样： ::

  $ cephfs-top --id <name>

默认情况下， `cephfs-top` 连接的是集群名 `ceph` ，用非默认集群名的话： ::

  $ cephfs-top --cluster <cluster>

`cephfs-top` 默认每秒刷新统计信息。可以选用不同的刷新间隔： ::

  $ cephfs-top -d <seconds>

刷新间隔应该是正整数。

只转储各个指标到标准输出，不创建 curses 显示屏，这样操作： ::

  $ cephfs-top --dump

要转储指定文件系统的指标到标准输出，不创建 curses 显示屏： ::

  $ cephfs-top --dumpfs <fs_name>

交互命令
--------
.. Interactive Commands

1. m : Filesystem selection （文件系统选择）
      显示一个文件系统菜单，供选择。

2. s : Sort field selection （选择排序字段）
      指定排序字段，默认是 cap_hit 。

3. l : Client limit （客户端限量）
      限制要显示的客户端数量。

4. r : Reset （重置）
      把排序字段和限量数值重置成默认值。

5. q : Quit （退出）
      如果当前位于主屏幕（所有文件系统信息），就退出此工具；要不是就退回到主屏幕。

显示的指标可以滚动，用方向键、 PgUp/PgDn 、 Home/End 和鼠标。

运行 `cephfs-top` 查看两个文件系统的示例截图：

.. image:: cephfs-top.png

.. note:: cephfs-top 兼容的 python 版本最低是 3.6.0 。
   cephfs-top 支持的发行版有 RHEL 8 、 Ubuntu 18.04 和 CentOS 8 及以上版本。
