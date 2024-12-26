MDS 日志记录
============
.. MDS Journaling

CephFS 元数据存储池
-------------------
.. CephFS Metadata Pool

CephFS 用一个单独的（元数据）存储池来管理 Ceph 文件系统中的文件元数据
（ inodes 和 dentry ）。元数据存储池拥有 Ceph 文件系统中文件的所有信息，
包括文件系统层次结构。此外， CephFS 还维护与文件系统中其他实体相关的元信息，
如文件系统日志、打开文件表、会话映射等。

本文档介绍 Ceph 元数据服务器如何使用和依赖日志。


CephFS MDS 日志记录
-------------------
.. CephFS MDS Journaling

在执行文件系统操作之前， CephFS 元数据服务器会把元数据事件汇集成日志流，
放入元数据存储池中的 RADOS 。活跃 MDS 守护进程管理 CephFS 中文件和目录的元数据。

CephFS 使用日志有几个原因：

#. 一致性：在 MDS 故障切换时，通过重放日志事件可以达到一致的文件系统状态。
   此外，需要对后端存储进行多次更新的元数据操作也需要日志来实现崩溃一致性
   （以及其他一致性机制，如加锁等）。

#. 性能：日志更新（大部分）是顺序操作，因此日志的更新速度很快。
   此外，可以一次写入一批更新，从而节省了更新文件不同部分所需的磁盘寻道时间。
   拥有大容量日志还有助于热备 MDS 预热缓存，
   从而间接地帮助 MDS 故障切换。

每个活跃的元数据服务器都在元数据存储池中维护着自己的日志。
日志条带花到了多个对象上。元数据服务器会剔除不需要的日志条目（视为旧日志）。


日志事件
--------
.. Journal Events

除了记录文件系统元数据更新日志外， CephFS 还记录各种其他事件，
如客户端会话信息、和目录导入/导出状态等。
这些事件在元数据服务器重建正确的状态时是必需的，比如，
Ceph MDS 在重启后试图重连客户端，重放日志事件时，
日志里的一种事件类型表明：有个客户端实体类型和这个 MDS 重启前有过一个会话。

要检查日志中记录的此类事件， CephFS 有一个命令行工具
`cephfs-journal-tool` ，使用方法如下：

::

   cephfs-journal-tool --rank=<fs>:<rank> event get list

`cephfs-journal-tool` 还能用于发现和修复损坏的 Ceph 文件系统。
（详情见 :doc:`/cephfs/cephfs-journal-tool` ）


日志事件类型
------------
.. Journal Event Types

下面是 MDS 记入日志的各种事件类型。

#. `EVENT_COMMITTED`: 把一个请求（ id ）标记为已提交。

#. `EVENT_EXPORT`: 把目录映射到一个 MDS rank 。

#. `EVENT_FRAGMENT`: 跟踪目录分段的各种阶段（分割、合并）。

#. `EVENT_IMPORTSTART`: 一个 MDS rank 开始导入目录分段时做个记录。

#. `EVENT_IMPORTFINISH`: 一个 MDS rank 完成目录分段导入后做个记录。

#. `EVENT_NOOP`: 无操作事件类型用于跳过一段日志区间。

#. `EVENT_OPEN`: 跟踪那些 inode 有打开的文件句柄。

#. `EVENT_RESETJOURNAL`: 用于把日志标记为截断后重置（ `reset` ）。

#. `EVENT_SESSION`: 跟踪打开的客户端会话。

#. `EVENT_SLAVEUPDATE`: 记录转发到其他（辅助的） mds 的那个操作的各种阶段。

#. `EVENT_SUBTREEMAP`: 目录 inode 到目录内容（子树分区）的映射关系。

#. `EVENT_TABLECLIENT`: 记录 MDS 视角下的客户端表（快照 snap 、锚点 anchor ）的转换状态。

#. `EVENT_TABLESERVER`: 记录 MDS 视角下的服务器表（快照、锚点）的转换状态。

#. `EVENT_UPDATE`: 记录一个 inode 上的文件操作。

#. `EVENT_SEGMENT`: 记录一个新日志分段的边界。

#. `EVENT_LID`: 标记没有逻辑子树映射的日志起始点。


日志分段
--------
.. Journal Segments

MDS 日志由逻辑段组成，代码中称为 LogSegments 。
这些分段用于把多个事件的元数据更新收集到一个逻辑单元中，以方便修剪。
每当日志尝试提交元数据操作（例如把文件创建作为 omap 更新刷到 dirfrag 对象）时，
日志都会在 LogSegment 中把它们汇集成可重放的一批更新。
这些更新必须是可重放的，以防 MDS 在对各种元数据对象进行一系列更新时出现故障。
之所以要批量更新，是因为要把对同一元数据对象（一个 dirfrag ）的更新进行分组，
因为多个 omap 条目可能是在同一时间段内更新的。

一旦某个分段被修剪掉，就会被视为“过期了”。过期的分段可由日志程序删除，
因为它的所有更新都已经刷回到后端的 RADOS 对象里了。
具体做法是更新日志程序的“过期位置 (expire position)”，使其超过过期分段的末尾。
某些过期分段可能会保留在日志中，以便在 MDS 重启时有更多缓存位于本地。

在 CephFS 的大部分历史中（直到 2023 年），日志分段都是由\
子树映射（ ``ESubtreeMap`` 事件）划分的。这样做的主要原因是，
日志恢复必须从子树映射的副本开始，然后才能重放其他事件。

现在，日志段可以通过 ``SegmentBoundary`` 事件来划分。
这些事件包括 ``ESubtreeMap`` 、 ``EResetJournal`` 、
``ESegment`` (2023) 或 ``ELid`` (2023)。
对于 ``ESegment`` ，这种轻量级分段边界允许 MDS 减少记录子树映射日志的频率，
还能保持较小的日志段，同时修剪事件也简短。
为了维持约束条件，也就是日志重放看到的第一个事件是 ``ESubtreeMap`` ，
以该事件开头的分段被当作“主要段 (major segments)”，
并在删除过期段时增加了新的约束条件：
日志的第一个段必须始终是主要段。

``ELid`` 事件用于把 MDS 日志标记为“新的”，这个时候，
对其他操作来说逻辑 ``LogSegment`` 和日志序列号是必需的，
特别是 MDSTable 操作。
MDS 在创建 rank 或关闭 rank 时使用该事件。
从这个初始状态重放 rank 时，不需要子树映射。


配置
----
.. Configurations

以事件数量为单位的日志分段目标大小由以下因素控制：

.. confval:: mds_log_events_per_segment

上一个主要日志段之后的次要 mds 日志段数量由以下参数控制：

.. confval:: mds_log_minor_segments_per_major_segment

此选项控制 MDS 修剪过期日志段的频率（值越高，
MDS 更新日志过期位置以便修剪的频率越低）。

分段的目标最大数由以下选项控制：

.. confval:: mds_log_max_segments

由于非主要分段等着修剪成下一个主要分段，因此 MDS 通常会略高于这个数字。
