==============
 OSD 配置参考
==============

.. index:: OSD; configuration

你可以通过配置文件调整 OSD ，但靠默认值和极少的配置 OSD 守护进程就能运行。最简 OSD \
配置需设置 ``osd journal size`` 和 ``osd host`` ，其他几乎都能用默认值。

OSD 用递增数字标识，按惯例以 ``0`` 开始，如下：
::

	osd.0
	osd.1
	osd.2

在配置文件里， ``[osd]`` 段下的配置适用于所有 OSD ；要添加针对特定 OSD 的选项（如 \
``osd host`` ），把它放到那个 OSD 段下即可，如：

.. code-block:: ini

	[osd]
		osd journal size = 1024

	[osd.0]
		osd host = osd-host-a

	[osd.1]
		osd host = osd-host-b


.. index:: OSD; config settings

常规配置
========

下列选项可配置一 OSD 的唯一标识符、以及数据和日志的路径。 Ceph 部署脚本通常会自动生\
成 UUID 。我们\ **不建议**\ 更改数据和日志的默认路径，因为这样会增加后续的排障难度。

日志尺寸应该大于期望的驱动器速度和 ``filestore max sync interval`` 之乘积的两倍；\
最常见的方法是为日志驱动器（通常是 SSD ）分区并挂载好，这样 Ceph 就可以用整个分区做\
日志。


``osd uuid``

:Description: OSD 的全局唯一标识符（ UUID ）。
:Type: UUID
:Default: The UUID.
:Note: ``osd uuid`` 适用于单个 OSD ， ``fsid`` 适用于整个集群。


``osd data`` 

:Description: OSD 数据存储位置，你得创建并把数据盘挂载到其下。我们不推荐更改默认值。
:Type: String
:Default: ``/var/lib/ceph/osd/$cluster-$id``


``osd max write size`` 

:Description: 一次写入的最大尺寸，MB。
:Type: 32-bit Integer
:Default: ``90``


``osd client message size cap`` 

:Description: 内存里允许的最大客户端数据消息。
:Type: 64-bit Integer Unsigned
:Default: 默认为 500MB 。 ``500*1024L*1024L`` 


``osd class dir`` 

:Description: RADOS 类插件的路径。
:Type: String
:Default: ``$libdir/rados-classes``


.. index:: OSD; file system

文件系统选项
============

Ceph 可自动创建并挂载所需的文件系统。


``osd mkfs options {fs-type}`` 

:Description: 为 OSD 新建 {fs-type} 类型的文件系统时使用的选项。
:Type: String
:xfs 默认值: ``-f -i 2048``
:其余文件系统默认值: {empty string}
:实例: ``osd mkfs options xfs = -f -d agcount=24``


``osd mount options {fs-type}`` 

:Description: 挂载 {fs-type} 类型的文件系统作为 OSD 数据目录时所用的选项。
:Type: String
:xfs 默认值: ``rw,noatime,inode64``
:其余文件系统默认值: ``rw, noatime``
:实例: ``osd mount options xfs = rw, noatime, inode64, nobarrier, logbufs=8``


.. index:: OSD; journal settings

日志选项
========

默认情况下， Ceph 觉得你会把 OSD 日志存储于下列路径：
::

	/var/lib/ceph/osd/$cluster-$id/journal

未做性能优化时， Ceph 会把日志存储在与 OSD 数据相同的硬盘上。追求高性能的 OSD 可用\
单独的硬盘存储日志数据，如固态硬盘能提供高性能日志。

``osd journal size`` 默认值是 0 ，所以你得在 ``ceph.conf`` 里设置。日志尺寸应该\
是 ``filestore max sync interval`` 与期望吞吐量的乘积再乘以 2 。
::

	osd journal size = {2 * (expected throughput * filestore max sync interval)}

期望吞吐量应考虑期望的硬盘吞吐量（即持续数据传输速率）、和网络吞吐量，例如一个 \
7200 转硬盘的速度大致是 100MB/s 。硬盘和网络吞吐量中较小的（ ``min()`` ）一个是相\
对合理的吞吐量，有的用户则以 10GB 日志尺寸起步，例如：
::

	osd journal size = 10000


``osd journal`` 

:Description: OSD 日志路径，可以是一个文件或块设备（ SSD 的一个分区）的路径。如果\
              是文件，要先创建相应目录。我们建议用 ``osd data`` 以外的独立驱动器。

:Type: String
:Default: ``/var/lib/ceph/osd/$cluster-$id/journal``


``osd journal size`` 

:Description: 日志尺寸（ MB ）。如果是 0 且日志文件是块设备，它会使用整个块设备。\
              从 v0.54 起，如果日志文件是块设备，这个选项会被忽略，且使用整个块设备。

:Type: 32-bit Integer
:Default: ``5120``
:Recommended: 最少 1G ，应该是期望的驱动器速度和 ``filestore max sync interval`` \
              的乘积。


详情见\ `日志配置参考`_\ 。


监视器和 OSD 的交互
===================

OSD 周期性地相互检查心跳并报告给监视器。 Ceph 默认配置可满足多数情况，但是如果你的\
网络延时大，就得用较长间隔。关于心跳的讨论参见\ `监视器与 OSD 交互的配置`_\ 。


数据归置
========

详情见\ `存储池和归置组配置参考`_\ 。


.. index:: OSD; scrubbing

洗刷
====

除了为对象复制多个副本外， Ceph 还要洗刷归置组以确保数据完整性。这种洗刷类似对象存\
储层的 ``fsck`` ，对每个归置组， Ceph 生成一个所有对象的目录，并比对每个主对象及其\
副本以确保没有对象丢失或错配。轻微洗刷（每天）检查对象尺寸和属性，深层洗刷（每周）会\
读出数据并用校验和方法确认数据完整性。

洗刷对维护数据完整性很重要，但会影响性能；你可以用下列选项来增加或减少洗刷操作。


``osd max scrubs`` 

:Description: 一 OSD 的最大并发洗刷操作数。
:Type: 32-bit Int
:Default: ``1`` 


``osd scrub thread timeout`` 

:Description: 洗刷线程最大死亡时值。
:Type: 32-bit Integer
:Default: ``60`` 


``osd scrub finalize thread timeout`` 

:Description: 洗刷终结线程最大超时值。
:Type: 32-bit Integer
:Default: ``60*10``


``osd scrub load threshold`` 

:Description: 最大负载，当前系统负载（ ``getloadavg()`` 所定义的）高于此值时 \
              Ceph 不会洗刷。默认 ``0.5`` 。

:Type: Float
:Default: ``0.5`` 


``osd scrub min interval`` 

:Description: 集群负载低的时候，洗刷的最大间隔时间，秒。
:Type: Float
:Default: 每天一次。 ``60*60*24``


``osd scrub max interval`` 

:Description: 不论集群负载如何，都要进行洗刷的时间间隔。

:Type: Float
:Default: 每周一次。 ``7*60*60*24``


``osd deep scrub interval``

:Description: 深层洗刷的间隔（完整地读所有数据）。 ``osd scrub load threshold`` \
              不会影响此选项。

:Type: Float
:Default: 每周一次。 ``60*60*24*7``


``osd deep scrub stride``

:Description: 深层洗刷时的读取尺寸。
:Type: 32-bit Integer
:Default: 512 KB. ``524288``


.. index:: OSD; operations settings

操作数
======

操作数选项允许你设置用于服务的线程数，如果把 ``osd op threads`` 设置为 ``0`` 就禁\
用了多线程。默认情况下， Ceph 用 30 秒超时和 30 秒抗议时间来把握 2 个线程的运行情\
况。你可以调整客户端操作和恢复操作的优先程度来优化恢复期间的性能。


``osd op threads`` 

:Description: OSD 操作线程数， ``0`` 为禁用。增大数量可以增加请求处理速度。
:Type: 32-bit Integer
:Default: ``2`` 


``osd client op priority``

:Description: 设置客户端操作优先级，它相对于 ``osd recovery op priority`` 。
:Type: 32-bit Integer
:Default: ``63`` 
:Valid Range: 1-63


``osd recovery op priority``

:Description: 设置恢复优先级，其值相对于 ``osd client op priority`` 。
:Type: 32-bit Integer
:Default: ``10`` 
:Valid Range: 1-63


``osd op thread timeout`` 

:Description: OSD 线程超时秒数。
:Type: 32-bit Integer
:Default: ``30`` 


``osd op complaint time`` 

:Description: 一个操作进行多久后开始抱怨。
:Type: Float
:Default: ``30`` 


``osd disk threads`` 

:Description: 硬盘线程数，用于在后台执行磁盘密集型操作，像数据洗刷和快照修复。
:Type: 32-bit Integer
:Default: ``1``


``osd disk thread ioprio class``

:Description: Warning: it will only be used if both ``osd disk thread
	      ioprio class`` and ``osd disk thread ioprio priority`` are
	      set to a non default value.  Sets the ioprio_set(2) I/O
	      scheduling ``class`` for the disk thread. Acceptable
	      values are ``idle``, ``be`` or ``rt``. The ``idle``
	      class means the disk thread will have lower priority
	      than any other thread in the OSD. This is useful to slow
	      down scrubbing on an OSD that is busy handling client
	      operations. ``be`` is the default and is the same
	      priority as all other threads in the OSD. ``rt`` means
	      the disk thread will have precendence over all other
	      threads in the OSD. This is useful if scrubbing is much
	      needed and must make progress at the expense of client
	      operations. Note: Only works with the Linux Kernel CFQ
	      scheduler.
:Type: String
:Default: the empty string


``osd disk thread ioprio priority``

:Description: Warning: it will only be used if both ``osd disk thread
	      ioprio class`` and ``osd disk thread ioprio priority`` are
	      set to a non default value. It sets the ioprio_set(2)
	      I/O scheduling ``priority`` of the disk thread ranging
	      from 0 (highest) to 7 (lowest). If all OSDs on a given
	      host were in class ``idle`` and compete for I/O
	      (i.e. due to controller congestion), it can be used to
	      lower the disk thread priority of one OSD to 7 so that
	      another OSD with priority 0 can potentially scrub
	      faster. Note: Only works with the Linux Kernel CFQ
	      scheduler.
:Type: Integer in the range of 0 to 7 or -1 if not to be used.
:Default: ``-1``


``osd op history size``

:Description: 要跟踪的最大已完成操作数量。
:Type: 32-bit Unsigned Integer
:Default: ``20``


``osd op history duration``

:Description: 要跟踪的最老已完成操作。
:Type: 32-bit Unsigned Integer
:Default: ``600``


``osd op log threshold``

:Description: 一次显示多少操作日志。
:Type: 32-bit Integer
:Default: ``5``

.. index:: OSD; backfilling

回填
====

当集群新增或移除 OSD 时，按照 CRUSH 算法应该重新均衡集群，它会把一些归置组移出或移\
入多个 OSD 以回到均衡状态。归置组和对象的迁移会导致集群运营性能显著降低，为维持运营\
性能， Ceph 用 backfilling 来执行此迁移，它可以使得 Ceph 的回填操作优先级低于用户\
读写请求。


``osd max backfills``

:Description: 单个 OSD 允许的最大回填操作数。
:Type: 64-bit Unsigned Integer
:Default: ``10``


``osd backfill scan min`` 

:Description: 集群负载低时，回填操作时扫描间隔。
:Type: 32-bit Integer
:Default: ``64`` 


``osd backfill scan max`` 

:Description: 回填操作时最大扫描间隔。
:Type: 32-bit Integer
:Default: ``512`` 


``osd backfill full ratio``

:Description: OSD 的占满率达到多少时拒绝接受回填请求。
:Type: Float
:Default: ``0.85``


``osd backfill retry interval``

:Description: 重试回填请求前等待秒数。
:Type: Double
:Default: ``10.0``

.. index:: OSD; osdmap

OSD 运行图
==========

OSD 运行图反映集群中运行的 OSD 守护进程，斗转星移，图元增加。 Ceph 用一些选项来确\
保 OSD 运行图增大时仍运行良好。


``osd map dedup``

:Description: 允许删除 OSD 图里的重复项。
:Type: Boolean
:Default: ``true``


``osd map cache size`` 

:Description: OSD 图缓存尺寸， MB 。
:Type: 32-bit Integer
:Default: ``500``


``osd map cache bl size``

:Description: OSD 进程中，驻留内存的 OSD 图缓存尺寸。
:Type: 32-bit Integer
:Default: ``50``


``osd map cache bl inc size``

:Description: OSD 进程中，驻留内存的 OSD 图缓存增量尺寸。
:Type: 32-bit Integer
:Default: ``100``


``osd map message max`` 

:Description: 每个  MOSDMap 图消息允许的最大条目数量。
:Type: 32-bit Integer
:Default: ``100``



.. index:: OSD; recovery

恢复
====

当集群启动、或某 OSD 守护进程崩溃后重启时，此 OSD 开始与其它 OSD 们建立连接，这样才\
能正常工作。详情见\ `监控 OSD 和归置组`_\ 。

如果某 OSD 崩溃并重生，通常会落后于其他 OSD ，也就是没有同归置组内最新版本的对象。\
这时， OSD 守护进程进入恢复模式并检索最新数据副本，并更新运行图。根据 OSD 挂的时间\
长短， OSD 的对象和归置组可能落后得厉害，另外，如果挂的是一个失效域（如一个机柜），\
多个 OSD 会同时重生，这样恢复时间更长、更耗资源。

为保持运营性能， Ceph 进行恢复时会限制恢复请求数、线程数、对象块尺寸，这样在降级状\
态下也能保持良好的性能。


``osd recovery delay start`` 

:Description: 对等关系建立完毕后， Ceph 开始对象恢复前等待的时间（秒）。
:Type: Float
:Default: ``0`` 


``osd recovery max active`` 

:Description: 每个 OSD 一次处理的活跃恢复请求数量，增大此值能加速恢复，但它们会增\
              加集群负载。

:Type: 32-bit Integer
:Default: ``15``


``osd recovery max chunk`` 

:Description: 一次推送的数据块的最大尺寸。
:Type: 64-bit Integer Unsigned
:Default: ``8 << 20`` 


``osd recovery threads`` 

:Description: 数据恢复时的线程数。
:Type: 32-bit Integer
:Default: ``1``


``osd recovery thread timeout`` 

:Description: 恢复线程最大死亡时值。
:Type: 32-bit Integer
:Default: ``30``


``osd recover clone overlap``

:Description: 在数据恢复期间保留重叠副本。应该总是 ``true`` 。
:Type: Boolean
:Default: ``true``



杂项
====


``osd snap trim thread timeout`` 

:Description: 快照修复线程最大死亡时值。
:Type: 32-bit Integer
:Default: ``60*60*1`` 


``osd backlog thread timeout`` 

:Description: 积压线程最大死亡时值。
:Type: 32-bit Integer
:Default: ``60*60*1`` 


``osd default notify timeout`` 

:Description: OSD 默认通告超时，秒。
:Type: 32-bit Integer Unsigned
:Default: ``30`` 


``osd check for log corruption`` 

:Description: 根据日志文件查找数据损坏，会耗费大量计算时间。
:Type: Boolean
:Default: ``false`` 


``osd remove thread timeout`` 

:Description: OSD 删除线程的最大死亡时值。
:Type: 32-bit Integer
:Default: ``60*60``


``osd command thread timeout`` 

:Description: 命令线程最大超时值。
:Type: 32-bit Integer
:Default: ``10*60`` 


``osd command max records`` 

:Description: 限制返回的丢失对象数量。
:Type: 32-bit Integer
:Default: ``256`` 


``osd auto upgrade tmap`` 

:Description: 在旧对象上给 ``omap`` 使用 ``tmap`` 。
:Type: Boolean
:Default: ``true``
 

``osd tmapput sets users tmap`` 

:Description: 只在调试时使用 ``tmap`` 。
:Type: Boolean
:Default: ``false`` 


``osd preserve trimmed log``

:Description: 保留本该修剪掉的日志文件，但是会占用更多磁盘空间。
:Type: Boolean
:Default: ``false``



.. _pool: ../../operations/pools
.. _监视器与 OSD 交互的配置: ../mon-osd-interaction
.. _监控 OSD 和归置组: ../../operations/monitoring-osd-pg#peering
.. _存储池和归置组配置参考: ../pool-pg-config-ref
.. _日志配置参考: ../journal-ref
