==============
 日志配置参考
==============

.. index:: journal; journal configuration

Ceph 的 OSD 使用日志的原因有二：速度和一致性。

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


``journal dio``

:描述: 对日志启用径直 IO ，需要  ``journal block align`` 设置为 ``true`` 。
:类型: Boolean
:是否必需: 用 ``aio`` 时自动启用。
:默认值: ``true``


``journal aio``

.. versionchanged:: 0.61 Cuttlefish

:描述: 异步写入日志时用 ``libaio`` 库，需要  ``journal dio`` 设为 ``true`` 。
:类型: Boolean
:是否必需: No.
:默认值: 0.61 版之后为 ``true`` ， 0.60 及之前为 ``false`` 。


``journal block align``

:描述: 块对齐写， ``dio`` 和 ``aio`` 需要。
:类型: Boolean
:是否必需: 用 ``dio`` 和 ``aio`` 时自动启用。
:默认值: ``true``


``journal max write bytes``

:描述: 一次写入日志的最大尺寸。
:类型: Integer
:是否必需: No
:默认值: ``10 << 20``


``journal max write entries``

:描述: 一次写入日志的最大数量。
:类型: Integer
:是否必需: No
:默认值: ``100``


``journal queue max ops``

:描述: 队列里一次允许的最大操作数量。
:类型: Integer
:是否必需: No
:默认值: ``500``


``journal queue max bytes``

:描述: 队列里一次允许的最大字节数。
:类型: Integer
:是否必需: No
:默认值: ``10 << 20``


``journal align min size``

:描述: 对齐大于指定最小值的数据有效载荷。
:类型: Integer
:是否必需: No
:默认值: ``64 << 10``


``journal zero on create``

:描述: 在创建文件系统（ ``mkfs`` ）期间用 ``0`` 填充整个日志。
:类型: Boolean
:是否必需: No
:默认值: ``false``
