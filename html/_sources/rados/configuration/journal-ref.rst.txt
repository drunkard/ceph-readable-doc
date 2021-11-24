==============
 日志配置参考
==============

.. index:: journal; journal configuration

Ceph 的 OSD 使用日志的原因有二：速度和一致性。
Note that since Luminous, the BlueStore OSD back end has been preferred and default.
This information is provided for pre-existing OSDs and for rare situations where
Filestore is preferred for new deployments.

- **速度：** 日志使得 OSD 可以快速地提交小块数据的写入， Ceph 把小片、随机 IO 依次\
  写入日志，这样，后端文件系统就有可能归并写入动作，并最终提升并发承载力。因此，使\
  用 OSD 日志能展现出优秀的突发写性能，实际上数据还没有写入 OSD ，因为文件系统把它\
  们捕捉到了日志。

- **一致性：** Ceph 的 OSD 守护进程需要一个能保证原子操作的文件系统接口。 OSD 把一\
  个操作的描述写入日志，然后把操作应用到文件系统，这需要原子更新一个对象（例如归置组\
  元数据）。每隔一段  ``filestore max sync interval`` 和 `` filestore min sync \
  interval`` 之间的时间， OSD 停止写入、把日志同步到文件系统，这样允许 OSD 修整日\
  志里的操作并重用空间。若失败， OSD 从上个同步点开始重放日志。

OSD 守护进程支持下面的日志选项：

.. confval:: journal_dio
.. confval:: journal_aio
.. confval:: journal_block_align
.. confval:: journal_max_write_bytes
.. confval:: journal_max_write_entries
.. confval:: journal_align_min_size
.. confval:: journal_zero_on_create
