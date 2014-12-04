====================
 filestore 配置参考
====================


``filestore debug omap check``

:Description: 打开对同步检查过程的调试。代价很高，仅用于调试。
:Type: Boolean
:Required: No
:Default: ``0``


.. index:: filestore; extended attributes

扩展属性
========

扩展属性（ XATTR ）是配置里的重要部分。一些文件系统对 XATTR 字节数有限制，另外在某\
些情况下文件系统存储 XATTR 的速度不如其他方法。下面的选项让你用独立于文件系统的存储\
方法，或许能提升性能。

Ceph 扩展属性用底层文件系统的 XATTR （如果没有尺寸限制）存储为 ``inline xattr`` 。\
如果有限制，如 ext4 限制为 4KB ，达到 ``filestore max inline xattr size`` 或 \
``filestore max inline xattrs`` 阀值时一些 XATTR 将存储为键/值数据库（也叫 \
``omap`` ）。


``filestore xattr use omap``

:Description: 用 XATTR 存储对象图，采用 ``ext4`` 文件系统时要设置为 ``true`` 。
:Type: Boolean
:Required: No
:Default: ``false``


``filestore max inline xattr size``

:Description: 每个对象在文件系统（如 XFS 、 btrfs 、 ext4 等）里存储的 XATTR 最大\
              尺寸，应该小于文件系统支持的尺寸。

:Type: Unsigned 32-bit Integer
:Required: No
:Default: ``512``


``filestore max inline xattrs``

:Description: 每个对象存储在文件系统里的 XATTR 数量。
:Type: 32-bit Integer
:Required: No
:Default: ``2``


.. index:: filestore; synchronization

同步间隔
========

filestore 需要周期性地静默写入、同步文件系统，这创建了一个提交点，然后就能释放相应\
的日志条目了。较大的同步频率可减小执行同步的时间及保存在日志里的数据量；较小的频率使\
得后端的文件系统能优化归并较小的数据和元数据写入，因此可能使同步更有效。


``filestore max sync interval``

:Description: 同步 filestore 的最大间隔秒数。
:Type: Double
:Required: No
:Default: ``5``


``filestore min sync interval``

:Description: 同步 filestore 的最小间隔秒数。
:Type: Double
:Required: No
:Default: ``.01``


.. index:: filestore; flusher

回写器
======

文件存储回写器强制使用 ``sync file range`` 来写出大块数据，这样处理有望减小最终同\
步的代价。实践中，禁用“文件存储回写器”有时候能提升性能。


``filestore flusher``

:Description: 启用文件存储回写器。
:Type: Boolean
:Required: No
:Default: ``false``

.. deprecated:: v.65

``filestore flusher max fds``

:Description: 设置回写器的最大文件描述符数量。
:Type: Integer
:Required: No
:Default: ``512``

.. deprecated:: v.65

``filestore sync flush``

:Description: 启用同步回写器。
:Type: Boolean
:Required: No
:Default: ``false``

.. deprecated:: v.65

``filestore fsync flushes journal data``

:Description: 文件系统同步时也回写日志数据。
:Type: Boolean
:Required: No
:Default: ``false``


.. index:: filestore; queue

队列
====

下面的选项能限制文件存储队列的尺寸。


``filestore queue max ops``

:Description: 文件存储操作接受的最大并发数，超过此设置的请求会被拒绝。
:Type: Integer
:Required: 无。对性能影响最小。
:Default: ``500``


``filestore queue max bytes``

:Description: 一个操作的最大字节数。
:Type: Integer
:Required: No
:Default: ``100 << 20``


``filestore queue committing max ops``

:Description: 文件存储能提交的最大操作数。
:Type: Integer
:Required: No
:Default: ``500``


``filestore queue committing max bytes``

:Description: 文件存储器能提交的最大字节数。
:Type: Integer
:Required: No
:Default: ``100 << 20``


.. index:: filestore; timeouts

超时选项
========


``filestore op threads``

:Description: 允许并行操作文件系统的最大线程数。
:Type: Integer
:Required: No
:Default: ``2``


``filestore op thread timeout``

:Description: 文件系统操作线程超时值，单位为秒。
:Type: Integer
:Required: No
:Default: ``60``


``filestore op thread suicide timeout``

:Description: 提交操作超时值（秒），超时后会取消。
:Type: Integer
:Required: No
:Default: ``180``


.. index:: filestore; btrfs

B-Tree 文件系统
===============


``filestore btrfs snap``

:Description: 对 ``btrfs`` 文件存储器启用快照功能。
:Type: Boolean
:Required: 不。仅适用于 ``btrfs`` 。
:Default: ``true``


``filestore btrfs clone range``

:Description: 允许 ``btrfs`` 文件存储克隆动作排队。
:Type: Boolean
:Required: 不。仅适用于 ``btrfs`` 。
:Default: ``true``


.. index:: filestore; journal

日志
====


``filestore journal parallel``

:Description: 允许并行记日志，对 btrfs 默认开。
:Type: Boolean
:Required: No
:Default: ``false``


``filestore journal writeahead``

:Description: 允许预写日志，对 xfs 默认开。
:Type: Boolean
:Required: No
:Default: ``false``


``filestore journal trailing``

:Description: 过时了，从没用过。
:Type: Boolean
:Required: No
:Default: ``false``


杂项
====


``filestore merge threshold``

:Description: 并入父目录前，子目录内的最小文件数。注：负值表示禁用子目录合并功能。
:Type: Integer
:Required: No
:Default: ``10``


``filestore split multiple``

:Description:  ``filestore_split_multiple * abs(filestore_merge_threshold) * 16`` 
               是分割为子目录前某目录内的最大文件数。

:Type: Integer
:Required: No
:Default: ``2``


``filestore update to``

:Description: 限制文件存储自动更新到某个指定版本。
:Type: Integer
:Required: No
:Default: ``1000``


``filestore blackhole``

:Description: 丢弃任何讨论中的事务。
:Type: Boolean
:Required: No
:Default: ``false``


``filestore dump file``

:Description: 存储事务转储目的文件。
:Type: Boolean
:Required: No
:Default: ``false``


``filestore kill at``

:Description: 在第 N 次机会后注入一个失效。
:Type: String
:Required: No
:Default: ``false``


``filestore fail eio``

:Description: 在 IO 错误的时候失败或崩溃。
:Type: Boolean
:Required: No
:Default: ``true``

