.. OSD Config Reference

==============
 OSD 配置参考
==============

.. index:: OSD; configuration

你可以通过配置文件调整 OSD ，但靠默认值和极少的配置 OSD 守护进程\
就能运行。最简 OSD 配置需设置 ``osd journal size`` 和 ``host`` ，\
其他几乎都能用默认值。

Ceph 的 OSD 守护进程用递增的数字作标识，按惯例以 ``0`` 开始，\
如下： ::

	osd.0
	osd.1
	osd.2

在配置文件里， ``[osd]`` 段下的配置适用于所有 OSD ；要添加针对\
特定 OSD 的选项（如 ``host`` ），把它放到那个 OSD 段下即可，如：

.. code-block:: ini

	[osd]
		osd journal size = 5120

	[osd.0]
		host = osd-host-a

	[osd.1]
		host = osd-host-b


.. index:: OSD; config settings
.. General Settings

常规配置
========

下列选项可配置一 OSD 的唯一标识符、以及数据和日志的路径。 Ceph
部署脚本通常会自动生成 UUID 。

.. warning:: 我们\ **不建议**\ 更改数据和日志的默认路径，因为\
   这样会增加后续的排障难度。

日志尺寸应该大于期望的驱动器速度和 ``filestore max sync interval``
之乘积的两倍；最常见的方法是为日志驱动器（通常是 SSD ）分区并\
挂载好，这样 Ceph 就可以用整个分区做日志。


``osd uuid``

:描述: OSD 的全局唯一标识符（ UUID ）。
:类型: UUID
:默认值: The UUID.
:备注: ``osd uuid`` 适用于单个 OSD ， ``fsid`` 适用于整个集群。


``osd data``

:描述: OSD 数据存储位置，你得创建并把数据盘挂载到其下。我们\
       不推荐更改默认值。
:类型: String
:默认值: ``/var/lib/ceph/osd/$cluster-$id``


``osd max write size``

:描述: 一次写入的最大尺寸，MB。
:类型: 32-bit Integer
:默认值: ``90``


``osd max object size``

:描述: 单个 RADOS 对象的最大尺寸，单位为字节。
:类型: 32-bit Unsigned Integer
:默认值: 128MB


``osd client message size cap``

:描述: 内存里允许的最大客户端数据消息。
:类型: 64-bit Unsigned Integer
:默认值: 默认为 500MB 。 ``500*1024L*1024L``


``osd class dir``

:描述: RADOS 类插件的路径。
:类型: String
:默认值: ``$libdir/rados-classes``


.. index:: OSD; file system
.. File System Settings

文件系统选项
============

Ceph 可自动创建并挂载所需的文件系统。


``osd mkfs options {fs-type}``

:描述: 为 OSD 新建 {fs-type} 类型的文件系统时使用的选项。
:类型: String
:xfs 默认值: ``-f -i 2048``
:其余文件系统默认值: {empty string}
:实例: ``osd mkfs options xfs = -f -d agcount=24``


``osd mount options {fs-type}``

:描述: 挂载 {fs-type} 类型的文件系统作为 OSD 数据目录时所用的选项。
:类型: String
:xfs 默认值: ``rw,noatime,inode64``
:其余文件系统默认值: ``rw, noatime``
:实例: ``osd mount options xfs = rw, noatime, inode64, logbufs=8``


.. index:: OSD; journal settings
.. Journal Settings

日志选项
========

默认情况下， Ceph 觉得你会把 OSD 日志存储于下列路径： ::

	/var/lib/ceph/osd/$cluster-$id/journal

When using a single device type (for example, spinning drives), the journals
should be *colocated*: the logical volume (or partition) should be in the same
device as the ``data`` logical volume.

When using a mix of fast (SSDs, NVMe) devices with slower ones (like spinning
drives) it makes sense to place the journal on the faster device, while
``data`` occupies the slower device fully.

The default ``osd journal size`` value is 5120 (5 gigabytes), but it can be
larger, in which case it will need to be set in the ``ceph.conf`` file::

	osd journal size = 10240


``osd journal``

:描述: OSD 日志路径，可以是一个文件或块设备（ SSD 的一个分区）的路径。如果是\
       文件，要先创建相应目录。我们建议用 ``osd data`` 以外的独立驱动器。

:类型: String
:默认值: ``/var/lib/ceph/osd/$cluster-$id/journal``


``osd journal size``

:描述: 日志尺寸（ MB ）。
:类型: 32-bit Integer
:默认值: ``5120``


详情见\ `日志配置参考`_\ 。


.. Monitor OSD Interaction

监视器和 OSD 的交互
===================

OSD 周期性地相互检查心跳并报告给监视器。 Ceph 默认配置可满足多数情况，但是如果你的\
网络延时大，就得用较长间隔。关于心跳的讨论参见\ `监视器与 OSD 交互的配置`_\ 。


.. Data Placement

数据归置
========

详情见\ `存储池和归置组配置参考`_\ 。


.. index:: OSD; scrubbing
.. Scrubbing

洗刷
====

除了为对象复制多个副本外， Ceph 还要洗刷归置组以确保数据完整\
性。这种洗刷类似对象存储层的 ``fsck`` ，对每个归置组， Ceph \
生成一个所有对象的目录，并比对每个主对象及其副本以确保没有对\
象丢失或错配。轻微洗刷（每天）检查对象尺寸和属性，深层洗刷（\
每周）会读出数据并用校验和方法确认数据完整性。

洗刷对维护数据完整性很重要，但会影响性能；你可以用下列选项来\
增加或减少洗刷操作。


``osd max scrubs``

:描述: 一 OSD 的最大并发洗刷操作数。
:类型: 32-bit Int
:默认值: ``1``


``osd scrub begin hour``

:描述: 被调度的洗刷操作在一天中可以运行的时间下限。
:类型: 0 到 24 之间的整数
:默认值: ``0``


``osd scrub end hour``

:描述: 被调度的洗刷操作在一天中可以运行的时间上限。本选项与 \
       ``osd scrub begin hour`` 一起定义了一个时间窗口，在此\
       期间可以进行洗刷操作。但是，在某个归置组的洗刷间隔超过 \
       ``osd scrub max interval`` 时，不管这个时间窗口是否合\
       适都会执行。
:类型: 0 到 24 之间的整数
:默认值: ``24``


``osd scrub during recovery``

:描述: 在恢复期间允许洗刷。有正在进行的恢复，且这里为 ``false``
       时，就会禁止调度新的洗刷（和深层洗刷）。已经在运行的洗\
       刷不受影响。对繁忙的集群来说，这样做可降低负载。
:类型: Boolean
:默认值: ``true``


``osd scrub thread timeout``

:描述: 洗刷线程最大死亡时值。
:类型: 32-bit Integer
:默认值: ``60``


``osd scrub finalize thread timeout``

:描述: 洗刷终结线程最大超时值。
:类型: 32-bit Integer
:默认值: ``60*10``


``osd scrub load threshold``

:描述: 标准的最大负载，当前系统负载（ ``getloadavg() / 在线 CPU 数量``
       所定义的）高于此值时 Ceph 不会洗刷。默认 ``0.5`` 。

:类型: Float
:默认值: ``0.5``


``osd scrub min interval``

:描述: 集群负载低的时候，洗刷的最小间隔时间，秒。
:类型: Float
:默认值: 每天一次。 ``60*60*24``


``osd scrub max interval``

:描述: 不论集群负载如何，都要进行洗刷的时间间隔。
:类型: Float
:默认值: 每周一次。 ``7*60*60*24``


``osd scrub chunk min``

:描述: 单个操作可洗刷的最小对象块数。数据块在洗刷期间， Ceph \
       会阻塞别人向它写入。
:类型: 32-bit Integer
:默认值: 5


``osd scrub chunk max``

:描述: 单个操作可洗刷的最大对象块数。
:类型: 32-bit Integer
:默认值: 25


``osd scrub sleep``

:描述: 洗刷下一组数据块前等待的时间。增加此值会拖慢整个洗刷进\
       度，但对客户端操作没什么影响。

:类型: Float
:默认值: 0


``osd deep scrub interval``

:描述: 深层洗刷的间隔（完整地读所有数据）。
       ``osd scrub load threshold`` 不会影响此选项。

:类型: Float
:默认值: 每周一次。 ``60*60*24*7``


``osd scrub interval randomize ratio``

:描述: 在给某一归置组调度下一个洗刷作业时，给 \
       ``osd scrub min interval`` 增加个随机延时，这个延时是个\
       小于 ``osd scrub min interval`` \* \
       ``osd scrub interval randomized ratio`` 的随机值。所以\
       在实践中，这个默认设置会把洗刷操作随机地散布到允许的时\
       间窗口内，即 ``[1, 1.5]`` \*
       ``osd scrub min interval`` 。

:类型: Float
:默认值: ``0.5``


``osd deep scrub stride``

:描述: 深层洗刷时的读取尺寸。
:类型: 32-bit Integer
:默认值: 512 KB. ``524288``



.. index:: OSD; operations settings
.. Operations

操作数
======


``osd op queue``

:描述: 本选项用于配置 OSD 内各种操作（ ops ）的优先顺序。两\
       种队列都实现了严格子队列，出常规队列之前要先出子队列，\
       它与常规队列的实现机制不同。最初的 PrioritizedQueue
       (``prio``) 使用令牌桶系统，在令牌足够多时它会先处理\
       优先级高的队列；在令牌不够多时，则按优先级从低到高依\
       次处理。 WeightedPriorityQueue (``wpq``) 会根据\
       其优先级处理所有队列，以避免出现饥饿队列。在一部分
       OSD 负载高于其它的时， WPQ 应该有优势。
       新的、基于 mClock 的 OpClassQueue (``mclock_opclass``)\
       可针对它们所属的类（ recovery 、 scrub 、 snaptrim 、
       client op 、 osd subop ）来划分优先级。还有，基于 mClock
       的 ClientQueue (``mclock_client``) 还能结合客户端标识符\
       来增进各客户端之间的公平性。见\
       `基于 mClock 的 QoS`_\ 。此配置更改后需重启。

:类型: String
:可选值: prio, wpq, mclock_opclass, mclock_client
:默认值: ``prio``


``osd op queue cut off``

:描述: 本选项用于配置把哪个优先级的操作放入严格队列、还是常\
       规队列。 ``low`` 这个选项会把所有复制操作以及优先级更\
       高的放入严格队列；而 ``high`` 选项只会把复制的确认反\
       馈操作以及优先级更高的发往严格队列。此选项设置为
       ``high`` 时，应该有助于缓解集群内某些 OSD 特别繁忙的\
       情形，尤其是配合 ``osd op queue`` 设置为 ``wpq`` 使用\
       时效果更佳。忙于处理副本流量的 OSD 们，如果没有这些配\
       置，它们的主副本（ primary client ）客户端往往比较空\
       闲。此配置更改后需重启。

:类型: String
:可选值: low, high
:默认值: ``low``


``osd client op priority``

:描述: 设置客户端操作优先级，它相对于
       ``osd recovery op priority`` 。

:类型: 32-bit Integer
:默认值: ``63``
:有效范围: 1-63


``osd recovery op priority``

:描述: 设置恢复优先级，其值相对于 ``osd client op priority`` 。
:类型: 32-bit Integer
:默认值: ``3``
:有效范围: 1-63


``osd scrub priority``

:描述: 设置洗刷操作的优先级。与 ``osd client op priority`` 有\
       关。

:类型: 32-bit Integer
:默认值: ``5``
:有效范围: 1-63


``osd requested scrub priority``

:描述: The priority set for user requested scrub on the work queue.  If
              this value were to be smaller than ``osd client op priority`` it
              can be boosted to the value of ``osd client op priority`` when
              scrub is blocking client operations.

:类型: 32-bit Integer
:默认值: ``120``


``osd snap trim priority``

:描述: 设置快照修建操作的优先级。与 ``osd client op priority``
       有关。

:类型: 32-bit Integer
:默认值: ``5``
:有效范围: 1-63


``osd snap trim sleep``

:描述: Time in seconds to sleep before next snap trim op.
              Increasing this value will slow down snap trimming.
              This option overrides backend specific variants.

:类型: Float
:默认值: ``0``


``osd snap trim sleep hdd``

:描述: Time in seconds to sleep before next snap trim op
              for HDDs.

:类型: Float
:默认值: ``5``


``osd snap trim sleep ssd``

:描述: Time in seconds to sleep before next snap trim op
              for SSDs.

:类型: Float
:默认值: ``0``


``osd snap trim sleep hybrid``

:描述: Time in seconds to sleep before next snap trim op
              when osd data is on HDD and osd journal is on SSD.

:类型: Float
:默认值: ``2``


``osd op thread timeout``

:描述: OSD 线程超时秒数。
:类型: 32-bit Integer
:默认值: ``15``


``osd op complaint time``

:描述: 一个操作进行多久后开始抱怨。
:类型: Float
:默认值: ``30``


``osd disk threads``

:描述: 硬盘线程数，用于在后台执行磁盘密集型操作，像数据洗刷和\
       快照修复。

:类型: 32-bit Integer
:默认值: ``1``


``osd disk thread ioprio class``

:描述: 警告：只有 ``osd disk thread ioprio class`` 和 \
       ``osd disk thread ioprio priority`` 同时改为非默认值时\
       此配置才生效。 OSD 用 ioprio_set(2) 为磁盘线程设置 I/O
       调度分类（ ``class`` ），当前支持 ``idle`` 、 ``be`` 或
       ``rt`` 。其中， ``idle`` 类意味着磁盘线程的优先级低于其\
       它的 OSD 线程，适合需延缓洗刷操作的情形，如 OSD 正忙于\
       处理客户端操作。 ``be`` 是默认值，将设置与其它 OSD 线程\
       相同的优先级。 ``rt`` 意为磁盘线程的优先级将高于其它任何
       OSD 线程。注：只能与 Linux 内核的 CFQ 调度器配合使用。从
       Jewel 版起，洗刷操作已经不再由磁盘的 iothread 发起了，\
       请参考 OSD 优先级选项。

:类型: String
:默认值: 空字符串


``osd disk thread ioprio priority``

:描述: 警告：只有 ``osd disk thread ioprio class`` 和 \
       ``osd disk thread ioprio priority`` 同时改为非默认值时\
       此配置才生效。它通过 ioprio_set(2) 设置磁盘线程的 I/O \
       调度优先级（ ``priority`` ），优先级从最高的 0 到最低的
       7 。如果某主机上的所有 OSD 都在 ``idle`` 类中竞争 I/O \
       资源（即控制器拥塞了），那么你就可以用此选项把某 OSD 的\
       磁盘线程优先级调低为 7 ，其它优先级为 0 的 OSD 就获得优\
       先权了。注：只能与 Linux 内核的 CFQ 调度器配合使用。

:类型: 0 到 7 间的整数， -1 禁用此功能。
:默认值: ``-1``


``osd op history size``

:描述: 要跟踪的最大已完成操作数量。
:类型: 32-bit Unsigned Integer
:默认值: ``20``


``osd op history duration``

:描述: 要跟踪的最老已完成操作。
:类型: 32-bit Unsigned Integer
:默认值: ``600``


``osd op log threshold``

:描述: 一次显示多少操作日志。
:类型: 32-bit Integer
:默认值: ``5``


.. QoS Based on mClock
.. _dmclock-qos:

基于 mClock 的 QoS
------------------

Ceph 对 mClock 的应用仍处于实验阶段，应当以探索心态试用。


.. Core Concepts

核心概念
````````

The QoS support of Ceph is implemented using a queueing scheduler
based on `the dmClock algorithm`_. This algorithm allocates the I/O
resources of the Ceph cluster in proportion to weights, and enforces
the constraints of minimum reservation and maximum limitation, so that
the services can compete for the resources fairly. Currently the
*mclock_opclass* operation queue divides Ceph services involving I/O
resources into following buckets:

- client op: the iops issued by client
- osd subop: the iops issued by primary OSD
- snap trim: the snap trimming related requests
- pg recovery: the recovery related requests
- pg scrub: the scrub related requests

And the resources are partitioned using following three sets of tags. In other
words, the share of each type of service is controlled by three tags:

#. reservation: the minimum IOPS allocated for the service.
#. limitation: the maximum IOPS allocated for the service.
#. weight: the proportional share of capacity if extra capacity or system
   oversubscribed.

In Ceph operations are graded with "cost". And the resources allocated
for serving various services are consumed by these "costs". So, for
example, the more reservation a services has, the more resource it is
guaranteed to possess, as long as it requires. Assuming there are 2
services: recovery and client ops:

- recovery: (r:1, l:5, w:1)
- client ops: (r:2, l:0, w:9)

The settings above ensure that the recovery won't get more than 5
requests per second serviced, even if it requires so (see CURRENT
IMPLEMENTATION NOTE below), and no other services are competing with
it. But if the clients start to issue large amount of I/O requests,
neither will they exhaust all the I/O resources. 1 request per second
is always allocated for recovery jobs as long as there are any such
requests. So the recovery jobs won't be starved even in a cluster with
high load. And in the meantime, the client ops can enjoy a larger
portion of the I/O resource, because its weight is "9", while its
competitor "1". In the case of client ops, it is not clamped by the
limit setting, so it can make use of all the resources if there is no
recovery ongoing.

Along with *mclock_opclass* another mclock operation queue named
*mclock_client* is available. It divides operations based on category
but also divides them based on the client making the request. This
helps not only manage the distribution of resources spent on different
classes of operations but also tries to insure fairness among clients.

CURRENT IMPLEMENTATION NOTE: the current experimental implementation
does not enforce the limit values. As a first approximation we decided
not to prevent operations that would otherwise enter the operation
sequencer from doing so.


.. Subtleties of mClock

mClock 的精妙之处
`````````````````

The reservation and limit values have a unit of requests per
second. The weight, however, does not technically have a unit and the
weights are relative to one another. So if one class of requests has a
weight of 1 and another a weight of 9, then the latter class of
requests should get 9 executed at a 9 to 1 ratio as the first class.
However that will only happen once the reservations are met and those
values include the operations executed under the reservation phase.

Even though the weights do not have units, one must be careful in
choosing their values due how the algorithm assigns weight tags to
requests. If the weight is *W*, then for a given class of requests,
the next one that comes in will have a weight tag of *1/W* plus the
previous weight tag or the current time, whichever is larger. That
means if *W* is sufficiently large and therefore *1/W* is sufficiently
small, the calculated tag may never be assigned as it will get a value
of the current time. The ultimate lesson is that values for weight
should not be too large. They should be under the number of requests
one expects to ve serviced each second.


.. Caveats

注意事项
````````

There are some factors that can reduce the impact of the mClock op
queues within Ceph. First, requests to an OSD are sharded by their
placement group identifier. Each shard has its own mClock queue and
these queues neither interact nor share information among them. The
number of shards can be controlled with the configuration options
``osd_op_num_shards``, ``osd_op_num_shards_hdd``, and
``osd_op_num_shards_ssd``. A lower number of shards will increase the
impact of the mClock queues, but may have other deleterious effects.

Second, requests are transferred from the operation queue to the
operation sequencer, in which they go through the phases of
execution. The operation queue is where mClock resides and mClock
determines the next op to transfer to the operation sequencer. The
number of operations allowed in the operation sequencer is a complex
issue. In general we want to keep enough operations in the sequencer
so it's always getting work done on some operations while it's waiting
for disk and network access to complete on other operations. On the
other hand, once an operation is transferred to the operation
sequencer, mClock no longer has control over it. Therefore to maximize
the impact of mClock, we want to keep as few operations in the
operation sequencer as possible. So we have an inherent tension.

The configuration options that influence the number of operations in
the operation sequencer are ``bluestore_throttle_bytes``,
``bluestore_throttle_deferred_bytes``,
``bluestore_throttle_cost_per_io``,
``bluestore_throttle_cost_per_io_hdd``, and
``bluestore_throttle_cost_per_io_ssd``.

A third factor that affects the impact of the mClock algorithm is that
we're using a distributed system, where requests are made to multiple
OSDs and each OSD has (can have) multiple shards. Yet we're currently
using the mClock algorithm, which is not distributed (note: dmClock is
the distributed version of mClock).

Various organizations and individuals are currently experimenting with
mClock as it exists in this code base along with their modifications
to the code base. We hope you'll share you're experiences with your
mClock and dmClock experiments in the ceph-devel mailing list.


``osd push per object cost``

:描述: 一个推送操作允许的开销。
:类型: Unsigned Integer
:默认值: 1000

``osd recovery max chunk``

:描述: 一个恢复操作可携带数据块的最大尺寸。
:类型: Unsigned Integer
:默认值: 8 MiB


``osd op queue mclock client op res``

:描述: 为客户端操作预留的值。
:类型: Float
:默认值: 1000.0


``osd op queue mclock client op wgt``

:描述: 客户端操作的权重。
:类型: Float
:默认值: 500.0


``osd op queue mclock client op lim``

:描述: 客户端操作的上限。
:类型: Float
:默认值: 1000.0


``osd op queue mclock osd subop res``

:描述: 为 OSD 子操作预留的值。
:类型: Float
:默认值: 1000.0


``osd op queue mclock osd subop wgt``

:描述: OSD 子操作的权重。
:类型: Float
:默认值: 500.0


``osd op queue mclock osd subop lim``

:描述: OSD 子操作的上限。
:类型: Float
:默认值: 0.0


``osd op queue mclock snap res``

:描述: 为快照修剪操作预留的值。
:类型: Float
:默认值: 0.0


``osd op queue mclock snap wgt``

:描述: 快照修剪操作的权重。
:类型: Float
:默认值: 1.0


``osd op queue mclock snap lim``

:描述: 快照修剪操作的上限。
:类型: Float
:默认值: 0.001


``osd op queue mclock recov res``

:描述: 为恢复预留的值。
:类型: Float
:默认值: 0.0


``osd op queue mclock recov wgt``

:描述: 恢复操作的权重。
:类型: Float
:默认值: 1.0


``osd op queue mclock recov lim``

:描述: 恢复操作的上限。
:类型: Float
:默认值: 0.001


``osd op queue mclock scrub res``

:描述: 为洗刷作业预留的值。
:类型: Float
:默认值: 0.0


``osd op queue mclock scrub wgt``

:描述: 洗刷作业的权重。
:类型: Float
:默认值: 1.0


``osd op queue mclock scrub lim``

:描述: 洗刷作业的上限。
:类型: Float
:默认值: 0.001

.. _the dmClock algorithm: https://www.usenix.org/legacy/event/osdi10/tech/full_papers/Gulati.pdf


.. Backfilling
.. index:: OSD; backfilling

回填
====

当集群新增或移除 OSD 时，按照 CRUSH 算法应该重新均衡集群，它会\
把一些归置组移出或移入多个 OSD 以回到均衡状态。归置组和对象的\
迁移会导致集群运营性能显著降低，为维持运营性能， Ceph 用 \
backfilling 来执行此迁移，它可以使得 Ceph 的回填操作优先级低于\
用户读写请求。


``osd max backfills``

:描述: 单个 OSD 允许的最大回填操作数。
:类型: 64-bit Unsigned Integer
:默认值: ``1``


``osd backfill scan min``

:描述: 集群负载低时，回填操作时扫描间隔。
:类型: 32-bit Integer
:默认值: ``64``


``osd backfill scan max``

:描述: 回填操作时最大扫描间隔。
:类型: 32-bit Integer
:默认值: ``512``


``osd backfill retry interval``

:描述: 重试回填请求前等待秒数。
:类型: Double
:默认值: ``10.0``



.. index:: OSD; osdmap
.. OSD Map

OSD 运行图
==========

OSD 运行图反映集群中运行的 OSD 守护进程，斗转星移，图元增加。
Ceph 用一些选项来确保 OSD 运行图增大时仍运行良好。


``osd map dedup``

:描述: 允许删除 OSD 图里的重复项。
:类型: Boolean
:默认值: ``true``


``osd map cache size``

:描述: 缓存的 OSD 图个数。
:类型: 32-bit Integer
:默认值: ``500``


``osd map cache bl size``

:描述: OSD 进程中，驻留内存的 OSD 图缓存尺寸。
:类型: 32-bit Integer
:默认值: ``50``


``osd map cache bl inc size``

:描述: OSD 进程中，驻留内存的 OSD 图缓存增量尺寸。
:类型: 32-bit Integer
:默认值: ``100``


``osd map message max``

:描述: 每个  MOSDMap 图消息允许的最大条目数量。
:类型: 32-bit Integer
:默认值: ``100``



.. index:: OSD; recovery
.. Recovery

恢复
====

当集群启动、或某 OSD 守护进程崩溃后重启时，此 OSD 开始与其它 \
OSD 们建立连接，这样才能正常工作。详情见\
`监控 OSD 和归置组`_\ 。

如果某 OSD 崩溃并重生，通常会落后于其他 OSD ，也就是没有同归置\
组内最新版本的对象。这时， OSD 守护进程进入恢复模式并检索最新\
数据副本，并更新运行图。根据 OSD 挂的时间长短， OSD 的对象和归\
置组可能落后得厉害，另外，如果挂的是一个失效域（如一个机柜），\
多个 OSD 会同时重生，这样恢复时间更长、更耗资源。

为保持运营性能， Ceph 进行恢复时会限制恢复请求数、线程数、对象\
块尺寸，这样在降级状态下也能保持良好的性能。


``osd recovery delay start``

:描述: 对等关系建立完毕后， Ceph 开始对象恢复前等待的时间\
       （秒）。

:类型: Float
:默认值: ``0``


``osd recovery max active``

:描述: 每个 OSD 一次可以处理的活跃恢复请求数量，增大此值能加速\
       恢复，但它们会增大集群负载。

:类型: 32-bit Integer
:默认值: ``3``


``osd recovery max chunk``

:描述: 一次推送的数据块的最大尺寸。
:类型: 64-bit Unsigned Integer
:默认值: ``8 << 20``


``osd recovery max single start``

:描述: 某个 OSD 恢复时，各 OSD 即将新开的最大恢复操作数量。
:类型: 64-bit Unsigned Integer
:默认值: ``1``


``osd recovery thread timeout``

:描述: 恢复线程最大死亡时值。
:类型: 32-bit Integer
:默认值: ``30``


``osd recover clone overlap``

:描述: 在数据恢复期间保留重叠副本。应该总是 ``true`` 。
:类型: Boolean
:默认值: ``true``


``osd recovery sleep``

:描述: 下一轮恢复或回填操作前睡眠的时间，单位为秒。增大此值会\
       减慢恢复操作，同时客户端操作受到的影响也小些了。

:类型: Float
:默认值: ``0``


``osd recovery sleep hdd``

:描述: 对于机械硬盘，下一轮恢复或回填操作前睡眠的时间，单位为\
       秒。

:类型: Float
:默认值: ``0.1``


``osd recovery sleep ssd``

:描述: 对于固态硬盘，下一轮恢复或回填操作前睡眠的时间，单位为\
       秒。

:类型: Float
:默认值: ``0``


``osd recovery sleep hybrid``

:描述: OSD 数据在机械硬盘上而 OSD 日志在固态硬盘上时，下一轮\
       恢复或回填操作前睡眠的时间，单位为秒。

:类型: Float
:默认值: ``0.025``


.. Tiering

分级缓存选项
============


``osd agent max ops``

:描述: 在高速模式下，每个分级缓存代理同时执行刷回操作的最大数\
       量。

:类型: 32-bit Integer
:默认值: ``4``


``osd agent max low ops``

:描述: 在低速模式下，每个分级缓存代理同时执行刷回操作的最大数\
       量。
:类型: 32-bit Integer
:默认值: ``2``

关于在高速模式下，分级缓存代理何时刷回脏对象，见
`cache target dirty high ratio`_ 选项。


.. Miscellaneous

杂项
====


``osd snap trim thread timeout``

:描述: 快照修复线程最大死亡时值。
:类型: 32-bit Integer
:默认值: ``60*60*1``


``osd backlog thread timeout``

:描述: 积压线程最大死亡时值。
:类型: 32-bit Integer
:默认值: ``60*60*1``


``osd default notify timeout``

:描述: OSD 默认通告超时，秒。
:类型: 32-bit Unsigned Integer
:默认值: ``30``


``osd check for log corruption``

:描述: 根据日志文件查找数据损坏，会耗费大量计算时间。
:类型: Boolean
:默认值: ``false``


``osd remove thread timeout``

:描述: OSD 删除线程的最大死亡时值。
:类型: 32-bit Integer
:默认值: ``60*60``


``osd command thread timeout``

:描述: 命令线程最大超时值。
:类型: 32-bit Integer
:默认值: ``10*60``


``osd command max records``

:描述: 限制返回的丢失对象数量。
:类型: 32-bit Integer
:默认值: ``256``


``osd fast fail on connection refused``

:描述: 如果启用此选项，崩溃的 OSD 会即刻被已互联的 OSD 和监视\
       器们标记为 down （假设已崩溃 OSD 所在主机还活着）。禁用\
       此选项即可恢复原来的行为，代价是 I/O 操作中途若有 OSD \
       崩溃可能会导致较长时间的 I/O 停顿。
:类型: Boolean
:默认值: ``true``


.. _pool: ../../operations/pools
.. _监视器与 OSD 交互的配置: ../mon-osd-interaction
.. _监控 OSD 和归置组: ../../operations/monitoring-osd-pg#peering
.. _存储池和归置组配置参考: ../pool-pg-config-ref
.. _日志配置参考: ../journal-ref
.. _cache target dirty high ratio: ../../operations/pools#cache-target-dirty-high-ratio
