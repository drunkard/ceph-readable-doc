==============
 OSD 配置参考
==============

.. index:: OSD; configuration

你可以通过配置文件调整 OSD ，
（或者通过近期版本里的中央配置库）
但靠默认值和极少的配置 OSD 守护进程就能运行。
最简 OSD 配置需设置 ``osd journal size`` 和 ``host`` ，
其他几乎都能用默认值。

Ceph 的 OSD 守护进程用递增的数字作标识，
按惯例以 ``0`` 开始，如下： ::

	osd.0
	osd.1
	osd.2

在配置文件里， ``[osd]`` 段下的配置适用于所有 OSD ；
要添加针对特定 OSD 的选项（如 ``host`` ），
把它放到那个 OSD 段下即可，
如：

.. code-block:: ini

	[osd]
		osd journal size = 5120

	[osd.0]
		host = osd-host-a

	[osd.1]
		host = osd-host-b


.. index:: OSD; config settings

常规配置
========
.. General Settings

下列选项可配置一 OSD 的唯一标识符、以及数据和日志的路径。
Ceph 部署脚本通常会自动生成 UUID 。

.. warning:: **不要**\ 更改数据和日志的默认路径，\
   因为这样会增加后续的排障难度。

日志尺寸应该大于期望的驱动器速度和 ``filestore max sync interval`` 之乘积的两倍；
最常见的方法是为日志驱动器（通常是 SSD ）分区并挂载好，
这样 Ceph 就可以用整个分区做日志。

.. confval:: osd_uuid
.. confval:: osd_data
.. confval:: osd_max_write_size
.. confval:: osd_max_object_size
.. confval:: osd_client_message_size_cap
.. confval:: osd_class_dir
   :default: $libdir/rados-classes


.. index:: OSD; file system

文件系统选项
============
.. File System Settings

Ceph 可自动创建并挂载所需的文件系统。


``osd_mkfs_options {fs-type}``

:描述: 为 OSD 新建 {fs-type} 类型的文件系统时使用的选项。
:类型: String
:xfs 默认值: ``-f -i 2048``
:其余文件系统默认值: {empty string}

例如::

  ``osd_mkfs_options_xfs = -f -d agcount=24``


``osd_mount_options {fs-type}``

:描述: 挂载 {fs-type} 类型的文件系统作为 OSD 数据目录时所用的选项。
:类型: String
:xfs 默认值: ``rw,noatime,inode64``
:其余文件系统默认值: ``rw, noatime``

例如::

    ``osd_mount_options_xfs = rw, noatime, inode64, logbufs=8``


.. index:: OSD; journal settings

日志选项
========
.. Journal Settings

This section applies only to the older Filestore OSD back end.  Since Luminous
BlueStore has been default and preferred.

默认情况下， Ceph 希望你把 OSD 守护进程的日志放到如下路径，\
它通常是到一个设备或分区的符号链接::

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


.. confval:: osd_journal
.. confval:: osd_journal_size

详情见\ `日志配置参考`_\ 。


监视器和 OSD 的交互
===================
.. Monitor OSD Interaction

OSD 周期性地相互检查心跳并报告给监视器。 Ceph 默认配置可满足多数情况，但是如果你的\
网络延时大，就得用较长间隔。关于心跳的讨论参见\ `监视器与 OSD 交互的配置`_\ 。


数据归置
========
.. Data Placement

详情见\ `存储池和归置组配置参考`_\ 。


.. index:: OSD; scrubbing

洗刷
====
.. Scrubbing

除了为对象复制多个副本外，
Ceph 还要洗刷归置组以确保数据完整性。
这种洗刷类似对象存储层的 ``fsck`` ，对每个归置组，
Ceph 生成一个所有对象的目录，并比对每个主对象及其副本以确保没有对象丢失或错配。
轻微洗刷（每天）检查对象尺寸和属性，
深层洗刷（每周）会读出数据并用校验和方法确认数据完整性。

洗刷对维护数据完整性很重要，但会导致性能下降；
你可以用下列选项来增加或减少洗刷操作。

.. confval:: osd_max_scrubs
.. confval:: osd_scrub_begin_hour
.. confval:: osd_scrub_end_hour
.. confval:: osd_scrub_begin_week_day
.. confval:: osd_scrub_end_week_day
.. confval:: osd_scrub_during_recovery
.. confval:: osd_scrub_load_threshold
.. confval:: osd_scrub_min_interval
.. confval:: osd_scrub_max_interval
.. confval:: osd_scrub_chunk_min
.. confval:: osd_scrub_chunk_max
.. confval:: osd_scrub_sleep
.. confval:: osd_deep_scrub_interval
.. confval:: osd_scrub_interval_randomize_ratio
.. confval:: osd_deep_scrub_stride
.. confval:: osd_scrub_auto_repair
.. confval:: osd_scrub_auto_repair_num_errors


.. index:: OSD; operations settings

操作数
======
.. Operations

.. confval:: osd_op_num_shards
.. confval:: osd_op_num_shards_hdd
.. confval:: osd_op_num_shards_ssd
.. confval:: osd_op_queue
.. confval:: osd_op_queue_cut_off
.. confval:: osd_client_op_priority
.. confval:: osd_recovery_op_priority
.. confval:: osd_scrub_priority
.. confval:: osd_requested_scrub_priority
.. confval:: osd_snap_trim_priority
.. confval:: osd_snap_trim_sleep
.. confval:: osd_snap_trim_sleep_hdd
.. confval:: osd_snap_trim_sleep_ssd
.. confval:: osd_snap_trim_sleep_hybrid
.. confval:: osd_op_thread_timeout
.. confval:: osd_op_complaint_time
.. confval:: osd_op_history_size
.. confval:: osd_op_history_duration
.. confval:: osd_op_log_threshold


.. _dmclock-qos:

基于 mClock 的 QoS
------------------
.. QoS Based on mClock

现在， Ceph 对 mClock 的应用更精致了，可以按照 `mClock 配置参考`_ 里的步骤使用。

核心概念
````````
.. Core Concepts

Ceph's QoS support is implemented using a queueing scheduler
based on `the dmClock algorithm`_. This algorithm allocates the I/O
resources of the Ceph cluster in proportion to weights, and enforces
the constraints of minimum reservation and maximum limitation, so that
the services can compete for the resources fairly. Currently the
*mclock_scheduler* operation queue divides Ceph services involving I/O
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

In Ceph, operations are graded with "cost". And the resources allocated
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

CURRENT IMPLEMENTATION NOTE: the current implementation enforces the limit
values. Therefore, if a service crosses the enforced limit, the op remains
in the operation queue until the limit is restored.

mClock 的精妙之处
`````````````````
.. Subtleties of mClock

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
one expects to be serviced each second.

注意事项
````````
.. Caveats

There are some factors that can reduce the impact of the mClock op
queues within Ceph. First, requests to an OSD are sharded by their
placement group identifier. Each shard has its own mClock queue and
these queues neither interact nor share information among them. The
number of shards can be controlled with the configuration options
:confval:`osd_op_num_shards`, :confval:`osd_op_num_shards_hdd`, and
:confval:`osd_op_num_shards_ssd`. A lower number of shards will increase the
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
the operation sequencer are :confval:`bluestore_throttle_bytes`,
:confval:`bluestore_throttle_deferred_bytes`,
:confval:`bluestore_throttle_cost_per_io`,
:confval:`bluestore_throttle_cost_per_io_hdd`, and
:confval:`bluestore_throttle_cost_per_io_ssd`.

A third factor that affects the impact of the mClock algorithm is that
we're using a distributed system, where requests are made to multiple
OSDs and each OSD has (can have) multiple shards. Yet we're currently
using the mClock algorithm, which is not distributed (note: dmClock is
the distributed version of mClock).

Various organizations and individuals are currently experimenting with
mClock as it exists in this code base along with their modifications
to the code base. We hope you'll share you're experiences with your
mClock and dmClock experiments on the ``ceph-devel`` mailing list.

.. confval:: osd_async_recovery_min_cost
.. confval:: osd_push_per_object_cost
.. confval:: osd_mclock_scheduler_client_res
.. confval:: osd_mclock_scheduler_client_wgt
.. confval:: osd_mclock_scheduler_client_lim
.. confval:: osd_mclock_scheduler_background_recovery_res
.. confval:: osd_mclock_scheduler_background_recovery_wgt
.. confval:: osd_mclock_scheduler_background_recovery_lim
.. confval:: osd_mclock_scheduler_background_best_effort_res
.. confval:: osd_mclock_scheduler_background_best_effort_wgt
.. confval:: osd_mclock_scheduler_background_best_effort_lim

.. _the dmClock algorithm: https://www.usenix.org/legacy/event/osdi10/tech/full_papers/Gulati.pdf


.. index:: OSD; backfilling

回填
====
.. Backfilling

当集群新增或移除 OSD 时，按照 CRUSH 算法应该重新均衡集群，
它会把一些归置组移出或移入多个 OSD 以回到均衡状态。
归置组和对象的迁移会导致集群运营性能显著降低，为维持运营性能，
Ceph 用 backfilling 来执行此迁移，
它可以使得 Ceph 的回填操作优先级低于用户读写请求。

.. confval:: osd_max_backfills
.. confval:: osd_backfill_scan_min
.. confval:: osd_backfill_scan_max
.. confval:: osd_backfill_retry_interval


.. index:: OSD; osdmap

OSD 运行图
==========
.. OSD Map

OSD 运行图反映集群中运行的 OSD 守护进程，斗转星移，图元增加。
Ceph 用一些选项来确保 OSD 运行图增大时仍运行良好。

.. confval:: osd_map_dedup
.. confval:: osd_map_cache_size
.. confval:: osd_map_message_max


.. index:: OSD; recovery

恢复
====
.. Recovery

当集群启动、或某 OSD 守护进程崩溃后重启时，此 OSD 开始与其它 \
OSD 们建立连接，这样才能正常工作。详情见\ `监控 OSD 和归置组`_\ 。

如果某 OSD 崩溃并重生，通常会落后于其他 OSD ，也就是没有同归置\
组内最新版本的对象。这时， OSD 守护进程进入恢复模式并检索最新\
数据副本，并更新运行图。根据 OSD 挂的时间长短， OSD 的对象和归\
置组可能落后得厉害，另外，如果挂的是一个失效域（如一个机柜），\
多个 OSD 会同时重生，这样恢复时间更长、更耗资源。

为保持运营性能， Ceph 进行恢复时会限制恢复请求数、线程数、对象\
块尺寸，这样在降级状态下也能保持良好的性能。

.. confval:: osd_recovery_delay_start
.. confval:: osd_recovery_max_active
.. confval:: osd_recovery_max_active_hdd
.. confval:: osd_recovery_max_active_ssd
.. confval:: osd_recovery_max_chunk
.. confval:: osd_recovery_max_single_start
.. confval:: osd_recover_clone_overlap
.. confval:: osd_recovery_sleep
.. confval:: osd_recovery_sleep_hdd
.. confval:: osd_recovery_sleep_ssd
.. confval:: osd_recovery_sleep_hybrid
.. confval:: osd_recovery_priority


分级缓存选项
============
.. Tiering

.. confval:: osd_agent_max_ops
.. confval:: osd_agent_max_low_ops

关于在高速模式下，分级缓存代理何时刷回脏对象，见 `cache target dirty high ratio`_ 选项。


杂项
====
.. Miscellaneous

.. confval:: osd_default_notify_timeout
.. confval:: osd_check_for_log_corruption
.. confval:: osd_delete_sleep
.. confval:: osd_delete_sleep_hdd
.. confval:: osd_delete_sleep_ssd
.. confval:: osd_delete_sleep_hybrid
.. confval:: osd_command_max_records
.. confval:: osd_fast_fail_on_connection_refused


.. _pool: ../../operations/pools
.. _监视器与 OSD 交互的配置: ../mon-osd-interaction
.. _监控 OSD 和归置组: ../../operations/monitoring-osd-pg#peering
.. _存储池和归置组配置参考: ../pool-pg-config-ref
.. _日志配置参考: ../journal-ref
.. _cache target dirty high ratio: ../../operations/pools#cache-target-dirty-high-ratio
.. _mClock 配置参考: ../mclock-config-ref
