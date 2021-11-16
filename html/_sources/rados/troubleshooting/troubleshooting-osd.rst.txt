.. Troubleshooting OSDs

==============
 OSD 故障排除
==============

对 OSD 排障前，先检查一下各监视器和网络。如果命令行 ``ceph health``
或 ``ceph -s`` 返回的是 ``HEALTH_OK`` ，这意味着监视器们形成了\
法定人数。如果你还没监视器法定人数、或者监视器状态有错误，要\
`先定位监视器问题 <../troubleshooting-mon>`_\ 。核实下你的\
网络，确保它在正常运行，因为网络对 OSD 的正常运作和性能\
有显著影响，检查一下主机侧的丢包情况和交换机侧的 CRC 错误。


.. Obtaining Data About OSDs

收集 OSD 数据
=============

开始 OSD 排障的第一步最好先收集拓扑信息，另外还有\ `监控 OSD`_
时收集的，如 ``ceph osd tree`` 。


.. Ceph Logs

Ceph 日志
---------

如果你没改默认路径，可以在 ``/var/log/ceph`` 下找到日志： ::

	ls /var/log/ceph

如果看到的日志还不够详细，可以增大日志级别。请参考\
`日志记录和调试`_\ ，看看如何保证看到大量日志时又不影响集群运行。


.. Admin Socket

管理套接字
----------

用管理套接字检索运行时信息。要看详细信息，罗列一下
Ceph 守护进程的套接字： ::

	ls /var/run/ceph

然后，执行下例命令显示可用选项，把 ``{daemon-name}`` 换成实际的\
守护进程（如 ``osd.0`` ）： ::

	ceph daemon osd.0 help

另外，你也可以指定一个 ``{socket-file}`` （如 ``/var/run/ceph`` \
下的文件）： ::

	ceph daemon {socket-file} help

和其它手段相比，管理接口允许你：

- 在运行时列出配置
- 列出历史操作
- 列出操作的优先队列状态
- 列出在进行的操作
- 列出性能计数器


.. Display Freespace

显示剩余空间
------------

可能是文件系统问题，用 ``df`` 显示文件系统剩余空间。 ::

	df -h

其它用法见 ``df --help`` 。


.. I/O Statistics

I/O 统计信息
------------

用 ``iostat`` 定位 I/O 相关问题。 ::

	iostat -x


.. Diagnostic Messages

诊断消息
--------

要查看内核里的诊断信息，配合 ``less`` 、 ``more`` 、 ``grep`` 或
``tail`` 使用 ``dmesg`` ，例如： ::

	dmesg | grep scsi


.. Stopping w/out Rebalancing

停止自动重均衡
==============

你得周期性地维护集群的子系统、或解决某个失败域的问题（如一机\
架）。如果你不想在停机维护 OSD 时让 CRUSH 自动重均衡，提前设置
``noout`` ： ::

	ceph osd set noout

On Luminous or newer releases it is safer to set the flag only on affected OSDs.
You can do this individually ::

	ceph osd add-noout osd.0
	ceph osd rm-noout  osd.0

Or an entire CRUSH bucket at a time.  Say you're going to take down
``prod-ceph-data1701`` to add RAM ::

	ceph osd set-group noout prod-ceph-data1701

在集群上设置 ``noout`` 后，你就可以停机维护失败域内的 OSD
以及其它相关的 Ceph 服务了。::

	systemctl stop ceph\*.service ceph\*.target

.. note:: 在定位同一故障域内的问题时，停机 OSD 内的归置组状态\
   会变为 ``degraded`` 。

维护结束后，重启 OSD 和其它守护进程。如果维护期间重启过主机，\
无需干预它们应该就会自己回归。 ::

	sudo systemctl start ceph.target

最后，必须解除集群范围的 ``noout`` 标志。 ::

	ceph osd unset noout
	ceph osd unset-group noout prod-ceph-data1701

Note that most Linux distributions that Ceph supports today employ ``systemd``
for service management.  For other or older operating systems you may need
to issue equivalent ``service`` or ``start``/``stop`` commands.


.. _osd-not-running:

OSD 没运行
==========

通常情况下，简单地重启 ``ceph-osd`` 进程就可以重回集群并恢复。


.. An OSD Won't Start

OSD 起不来
----------

如果你启动集群时，其中一个 OSD 起不来，依次检查：

- **配置文件：** 如果你新装的 OSD 不能启动，检查下配置文件，确\
  保它合爻性（比如 ``host`` 而非 ``hostname`` ，等等）；

- **检查路径：** 检查你配置的路径，以及它们自己的数据和元数据\
  （日志、WAL、DB）。如果你分离了 OSD 数据和元数据、而\
  配置文件和实际挂载点存在出入，启动 OSD 时就可能遇到麻烦。\
  如果你想把元数据存储在一个独立的块设备上，应该通过\
  驱动器分区或 LVM 为各 OSD 分别分配一个分区。

- **检查最大线程数：** 如果你的节点有很多 OSD ，也许就会触碰到\
  默认的最大线程数限制（如通常是 32k 个），尤其是在恢复期间。\
  你可以用 ``sysctl`` 增大线程数，把最大线程数更改为支持的最大\
  值（即 4194303 ），看看是否有用。例如： ::

	sysctl -w kernel.pid_max=4194303

  如果增大最大线程数解决了这个问题，你可以把此配置
  ``kernel.pid_max`` 写入 ``/etc/sysctl.d`` 内的文件或\
  写入主配置 ``/etc/sysctl.conf`` ，使之永久生效，例如： ::

	kernel.pid_max = 4194303

- **Check ``nf_conntrack``:** This connection tracking and limiting system
  is the bane of many production Ceph clusters, and can be insidious in that
  everything is fine at first. As cluster topology and client workload
  grow, mysterious and intermittent connection failures and performance
  glitches manifest, becoming worse over time and at certain times of day.
  Check ``syslog`` history for table fillage events.  You can mitigate this
  bother by raising ``nf_conntrack_max`` to a much higher value via ``sysctl``.
  Be sure to raise ``nf_conntrack_buckets`` accordingly to
  ``nf_conntrack_max / 4``, which may require action outside of ``sysctl`` e.g.
  ``"echo 131072 > /sys/module/nf_conntrack/parameters/hashsize``
  More interdictive but fussier is to blacklist the associated kernel modules
  to disable processing altogether.  This is fragile in that the modules
  vary among kernel versions, as does the order in which they must be listed.
  Even when blacklisted there are situations in which ``iptables`` or ``docker``
  may activate connection tracking anyway, so a "set and forget" strategy for
  the tunables is advised.  On modern systems this will not consume appreciable
  resources.

- **内核版本：** 确认你在用的内核版本以及所用的发布版。 Ceph \
  默认依赖一些第三方工具，这些工具可能有缺陷或者与特定发布版和\
  /或内核版本冲突（如 Google ``gperftools`` 和 ``TCMalloc`` ）。\
  检查下\ `操作系统推荐`_\ 和各 Ceph 版本的发布说明，\
  以确保你已经解决了内核相关的问题。

- **段错误：** 如果出现了段错误，提高日志级别并再次启动有问题的守护进程。
  如果段错误重现了，搜索一下 Ceph 缺陷追踪器
  `https://tracker.ceph/com/projects/ceph <https://tracker.ceph.com/projects/ceph/>`_
  和 ``dev`` 、 ``ceph-users`` 邮件列表归档
  `https://ceph.io/resources <https://ceph.io/resources>`_ 。
  如果它真是一个新的、唯一的失败案例，把它发到
  ``dev`` 邮件列表，并提供运行着的 Ceph 版本号、
  ``ceph.conf`` （把私密信息抹掉）、你的监视器状态输出和\
  日志文件的节选。

.. An OSD Failed

OSD 失败
--------

某个 ``ceph-osd`` 死掉时，活着的 ``ceph-osd`` 守护进程们\
会报告给各监视器，说它挂了，随后它就会浮现在
``ceph health`` 命令的新状态信息里： ::

	ceph health
	HEALTH_WARN 1/3 in osds are down

具体来说，只要有 OSD 被标记为 ``in`` 和 ``down`` ，你就会\
收到警告信息，你可以用下面的命令得知具体哪个是 ``down`` 的： ::

	ceph health detail
	HEALTH_WARN 1/3 in osds are down
	osd.0 is down since epoch 23, last address 192.168.106.220:6800/11080

或 ::

	ceph osd tree down

如果有个驱动器失败或其它错误使 ``ceph-osd`` 不能正常运行或\
重启，一条错误信息将会出现在 ``/var/log/ceph/`` 内的\
日志文件里。

如果守护进程因心跳失败、 ``suicide timeout`` 、底层驱动器或\
文件系统无响应而停止，查看一下 ``dmesg`` 和 `syslog` 输出\
获取驱动器或者其它的内核错误。你可能得用诸如 ``dmesg -T``
这样的命令加上时间戳，否则很容易把旧消息误读成新的。

如果此问题是软件错误（失败的断言或其它意外错误），搜索一下\
前述归档和追踪器，如果没发现修正或已有的缺陷，请把它反馈到
`ceph-devel`_ 邮件列表。


.. No Free Drive Space
.. _no-free-drive-space:

硬盘没剩余空间
--------------

Ceph 不允许你向满的 OSD 写入数据，以免丢失数据。在运营着的集群\
中， OSD 们和存储池接近 full ratio 时你应该会收到警告。 ``mon osd full ratio``
默认为 ``0.95`` 、或达到容量的 95% 时它将阻止客户端写入数据；
``mon osd backfillfull ratio`` 默认为 ``0.90`` 、或达到容量的
90% 以上时不会启动回填。 OSD 将满比率默认为 ``0.85`` 、\
也就是说达到容量的 85% 时它会产生健康警告。

Note that individual OSDs within a cluster will vary in how much data Ceph
allocates to them.  This utilization can be displayed for each OSD with ::

	ceph osd df

Overall cluster / pool fullness can be checked with ::

	ceph df 

Pay close attention to the **most full** OSDs, not the percentage of raw space
used as reported by ``ceph df``.  It only takes one outlier OSD filling up to
fail writes to its pool.  The space available to each pool as reported by
``ceph df`` considers the ratio settings relative to the *most full* OSD that
is part of a given pool.  The distribution can be flattened by progressively
moving data from overfull or to underfull OSDs using the ``reweight-by-utilization``
command.  With Ceph releases beginning with later revisions of Luminous one can also
exploit the ``ceph-mgr`` ``balancer`` module to perform this task automatically
and rather effectively.

这些比率可以调整：

::

    ceph osd set-nearfull-ratio <float[0.0-1.0]>
    ceph osd set-full-ratio <float[0.0-1.0]>
    ceph osd set-backfillfull-ratio <float[0.0-1.0]>

集群用满的问题一般出现在某个 OSD 失败时，起因是在小型的和/或\
非常满或失衡的集群上进行的测试之类的原因。当某一 OSD 或节点
存储着很大部分的集群数据时，因组件失败甚或自然增长就能诱发
``nearfull`` 和 ``full`` 比率超额。如果你在小型集群上测试
Ceph 如何应对 OSD 失败，应该保留足够的空闲空间，并且临时降低
OSD 的 ``full ratio`` 、 ``backfillfull ratio`` 和
``nearfull ratio`` 值，命令为：

``ceph health`` 会显示将满的 ``ceph-osds`` ： ::

	ceph health
	HEALTH_WARN 1 nearfull osd(s)

或者： ::

	ceph health detail
	HEALTH_ERR 1 full osd(s); 1 backfillfull osd(s); 1 nearfull osd(s)
	osd.3 is full at 97%
	osd.4 is backfill full at 91%
	osd.2 is near full at 87%

处理集群用满的最好方法就是增加新 OSD 扩容，这样集群就能把\
数据重分布到新存储器里。

如果因满载而导致旧的 FileStore OSD 不能启动，你可以试着删除\
那个 OSD 上的一些归置组数据目录。

.. important:: 如果你准备从填满的 OSD 中删除某个归置组，注意\
   **不要**\ 删除另一个 OSD 上的同名归置组，否则\
   **你会丢数据**\ 。\ **必须**\ 在多个 OSD 上保留至少一份\
   数据副本。这是一种少见的极端干涉方法，轻易不要用。

详情见\ `监视器配置参考`_\ 。


.. OSDs are Slow/Unresponsive

OSD 龟速或无响应
================

一个反复出现的问题是龟速或无响应。在深入性能问题前，你应该先确\
保不是其他故障。例如，确保你的网络运行正常、且 OSD 在运行，还\
要检查 OSD 是否被恢复流量拖住了。

.. tip:: 较新版本的 Ceph 能更好地处理恢复，可防止恢复进程耗尽\
   系统资源而导致 ``up`` 且 ``in`` 的 OSD 不可用或响应慢。


.. Networking Issues

网络问题
--------

Ceph 是一个分布式存储系统，所以它靠网络实现 OSD 互联、复制、\
故障恢复、和心律传递。网络问题会导致 OSD 延时和状态抖动，
详情参见\ `状态抖动的 OSD`_ 。

确保 Ceph 进程和 Ceph 依赖的进程连接了、和/或在监听。 ::

	netstat -a | grep ceph
	netstat -l | grep ceph
	sudo netstat -p | grep ceph

检查网络统计信息。 ::

	netstat -s


.. Drive Configuration

驱动器配置
----------

一个 SAS 或 SATA 存储驱动器应该只用于一个 OSD ； NVMe 驱动器\
可以轻松处理两个或更多。如果有其它进程（包括日志/元数据、\
操作系统、 Ceph 监视器、 `syslog` 日志、其它 OSD 以及\
非 Ceph 进程）共享驱动器，读和写吞吐量会成为瓶颈。

Ceph 在日志记录\ *完成之后*\ 才会确认写操作，所以高速 SSD
有助于降低响应时间，尤其是在用旧的 FileStore OSD 搭配
``XFS`` 或 ``ext4`` 文件系统时。相反， ``Btrfs`` 文件系统\
可以同时写入和记日志。（然而还是得注意，我们不建议在生产环境下\
用 ``Btrfs`` 。）

.. note:: 给驱动器分区并不能改变总吞吐量或顺序读写限制。\
   把日志分离到单独的分区可能有所帮助，但最好是另外一个\
   物理驱动器。


.. Bad Sectors / Fragmented Disk

坏扇区和碎片化硬盘
------------------

检修下硬盘是否有坏块、碎片、和其它会导致性能急剧下降的错误。\
有用的工具有 ``dmesg`` 、 ``syslog`` 日志、和 ``smartctl``
（包含在 ``smartmontools`` 软件包里）。


.. Co-resident Monitors/OSDs

监视器和 OSD 蜗居
-----------------

监视器是相对轻量的进程，但它们会发出大量 ``fsync()`` 系统调用，\
这会妨碍其它工作负载，特别是监视器和 OSD 共享驱动器时。另外，\
如果你在 OSD 主机上同时运行监视器，遭遇的性能问题可能\
和这些相关：

- 运行较老的内核（低于3.0）
- 运行的内核不支持 ``syncfs(2)`` 系统调用

在这些情况下，同一主机上运行着的多个 OSD 会发出大量提交，\
导致相互拖累。这种情况经常会导致爆发写。


.. Co-resident Processes

进程蜗居
--------

与 OSD 们共存于同一套硬件、并向 Ceph 写入数据的进程（融合，\
像基于云的解决方案、虚拟机和其他应用程序）会导致
OSD 延时大增。一般来说，我们建议用一些主机跑 Ceph 、用\
另外一些主机跑其它进程。实践证明把 Ceph 和其他应用程序分开\
可提高性能、并简化故障排除和运维。


.. Logging Levels

日志记录级别
------------

如果你为追踪某问题提高过日志级别、但结束后忘了调回去，这个 OSD
将向硬盘写入大量日志。如果你想始终保持高日志级别，可以考虑给\
默认日志路径挂载个硬盘，即 ``/var/log/ceph/$cluster-$name.log`` 。


.. Recovery Throttling

恢复节流
--------

根据你的配置， Ceph 可以降低恢复速度来维持性能，否则它会不顾
OSD 性能而加快恢复速度。检查下 OSD 是否正在恢复。


.. Kernel Version

内核版本
--------

检查下你在用的内核版本。较老的内核也许没有移植能提高 Ceph 性能\
的功能。


.. Kernel Issues with SyncFS

内核与 SyncFS 问题
------------------

试试在一主机上只运行一个 OSD ，看看能否提升性能。老内核未必支\
持有 ``syncfs(2)`` 系统调用的 ``glibc`` 。


.. Filesystem Issues

文件系统问题
------------

当前，我们建议用 BlueStore 后端部署集群。运行
Luminous 之前的版本、或由于某些原因还得用之前的 FileStore 后端\
部署 OSD 时，我们推荐 ``XFS`` 。

我们不推荐用 ``Btrfs`` 或 ``ext4`` 。 ``Btrfs`` 有很多\
诱人的功能，但文件系统自身的缺陷可能导致性能问题，以及
ENOSPC 伪错误。我们不建议使用 ``ext4`` 做 OSD 的 FileStore ，\
因为其 ``xattr`` 尺寸限制会破坏我们对长对象名的支持，而这是
RGW 必需的。

详情见\ `文件系统推荐`_\ 。

.. _文件系统推荐: ../configuration/filesystem-recommendations


.. Insufficient RAM

内存不足
--------

我们建议给每个 OSD 守护进程 *最少* 4GB 内存，而且建议在
6-8GB 之上取整。你也许注意到了，日常操作中 ``ceph-osd`` 进程\
仅会用其中一小部分。你也许想用这些空闲内存同时跑一些其他应用、\
或者克扣各节点的内存容量。然而当 OSD 在恢复时，其内存使用\
会激增，如果内存不够充足， OSD 的性能将明显降低，而且\
守护进程们甚至会崩溃或被 Linux 的 ``OOM Killer`` 杀死。


.. Blocked Requests or Slow Requests

blocked requests 或 slow requests
---------------------------------

如果某 ``ceph-osd`` 守护进程对一请求响应很慢，它会\
记录日志消息，说出耗时过于长的那些操作。默认警告阀值是 30 秒，\
用 ``osd op complaint time`` 选项来配置。这种情况发生时，\
集群日志系统会收到消息。

老旧版本抱怨 ``old requests``::

	osd.0 192.168.106.220:6800/18813 312 : [WRN] old request osd_op(client.5099.0:790 fatty_26485_object789 [write 0~4096] 2.5e54f643) v4 received at 2012-03-06 15:42:56.054801 currently waiting for sub ops

较新版本的 Ceph 抱怨 ``slow requests``::

	{date} {osd.num} [WRN] 1 slow requests, 1 included below; oldest blocked for > 30.005692 secs
	{date} {osd.num}  [WRN] slow request 30.005692 seconds old, received at {date-time}: osd_op(client.4240.0:8 benchmark_data_ceph-1_39426_object7 [write 0~4194304] 0.69848840) v4 currently waiting for subops from [610]


可能的起因有：

- 快要坏的驱动器（查验一下 ``dmesg`` 输出）；
- 内核文件系统缺陷（查验一下 ``dmesg`` 输出）；
- 集群过载（检查系统负载、 iostat 等等）；
- ``ceph-osd`` 守护进程缺陷。

可能的解决方法：

- 从 Ceph 主机去除 VM ；
- 升级内核；
- 升级 Ceph ；
- 重启 OSD 。
- 替换坏的或快要坏的组件；


.. Debugging Slow Requests

慢请求的调试
------------

运行 ``ceph daemon osd.<id> dump_historic_ops`` 或
``ceph daemon osd.<id> dump_ops_in_flight`` 命令时，你会看到一\
堆操作和各个操作过程的一个事件列表，下面简单说明一下。

信使层事件：

- ``header_read``: 信使从离线开始首次读取消息；
- ``throttled``: 信使尝试申请内存节流空间，用于读入消息时；
- ``all_read``: 信使已离线、并读完了消息；
- ``dispatched``: 信使把消息发给 OSD 时；
- ``initiated``: 等价于 ``header_read`` ，这二者都保留着是历史\
  遗留问题。

OSD 处理 ops 时的事件：

- ``queued_for_pg``: op 已放入队列，等着它自己的 PG 来处理；
- ``reached_pg``: 其 PG 已开始处理这个 op ；
- ``waiting for \*``: 此 op 正等着某些其它工作结束，这样它才能\
  继续下一步（如，一个新 OSDMap ；等着这个对象目标完成洗刷；等\
  着相应的 PG 完成互联；所有都在消息内指出了）；
- ``started``: 此 op 已被这个 OSD 接受为应该做的事，并且正在进\
  行中；
- ``waiting for subops from``: 此 op 已发送给了副本 OSD 们。

```FileStore``` 事件:

- ``commit_queued_for_journal_write``: 此 op 已递送给了 FileStore 。
- ``write_thread_in_journal_buffer``: 此 op 已经在日志的缓冲中\
  了、并等着持久化（等着下一次硬盘写操作）；
- ``journaled_completion_queued``: 此 op 已在硬盘上作了日志、\
  且它的回调已进入队列等着被调用了。

数据已传递给更底层存储之后的 OSD 事件：

- ``op_commit``: 此 op 已被主 OSD 提交（即已写入日志）；
- ``op_applied``: 此 op `已写入 write() <https://www.freebsd.org/cgi/man.cgi?write(2)>`_
  主 OSD 的后端文件系统（即在内存里已应用，但还没刷入硬盘）；
- ``sub_op_applied``: （类似前面的） ``op_applied`` ，只是这次\
  是发生在副本上的 subop （子操作）；
- ``sub_op_committed``: 就是 ``op_commit`` ，可这次是发生在副\
  本上的 subop （仅发生在 EC 存储池中）；
- ``sub_op_commit_rec/sub_op_apply_rec from <X>``: 主 OSD 得知\
  上述消息后会做这些标记，只是为某个特定副本（即 ``<X>`` ）做\
  标记；
- ``commit_sent``: 我们已经把回信发给客户端（或主 OSD ，来自\
  子操作的）了。

这些事件里，有很多看起来显得多余，但都是代码里的穿越重要边界的\
（像数据加锁后传入新线程）。


.. Flapping OSDs

状态抖动的 OSD
==============

.. note:: 译者：社区同仁讨论认为，这是随时间延续，不断地在
   ``up`` 、 ``down`` 状态之间反复的情形，状态变动的时间间隔\
   有规律或无规律，运动方向为“上下”，非“左右”、亦非“前后”，\
   也可理解为打摆子、状态翻转。总之是一种病态的、非正常的状态。

OSD 们互联和检查心跳时会优先选集群网（后端），详情见\
`监视器与 OSD 的交互`_\ 。

传统上，我们建议拆分 *公共* (前端）和 *私密*
（集群、后端、复制）的网络：

#. Segregation of heartbeat and replication / recovery traffic (private)
   from client and OSD <-> mon traffic (public).  This helps keep one
   from DoS-ing the other, which could in turn result in a cascading failure.

#. Additional throughput for both public and private traffic.

When common networking technologies were 100Mb/s and 1Gb/s, this separation
was often critical.  With today's 10Gb/s, 40Gb/s, and 25/50/100Gb/s
networks, the above capacity concerns are often diminished or even obviated.
For example, if your OSD nodes have two network ports, dedicating one to
the public and the other to the private network means no path redundancy.
This degrades your ability to weather network maintenance and failures without
significant cluster or client impact.  Consider instead using both links
for just a public network:  with bonding (LACP) or equal-cost routing (e.g. FRR)
you reap the benefits of increased throughput headroom, fault tolerance, and
reduced OSD flapping.

当私网（或者仅仅是主机的一条链路）失败、或降级，同时，\
公网却正常运行， OSD 就不能很好地处理这种情况。这时的情形就是，
OSD 们会用公网相互向监视器报告邻居 ``down`` 了、\
同时把它自己标记为 ``up`` 的。然后监视器们就再次向公网散播\
更新过的集群运行图，把受影响的 OSD 们标记成了 `down` ，\
这些 OSD 们又向监视器反馈“我还没死呢！”，如此循环往复。\
我们把这种情形称为状态抖动（或打摆子， flapping ），而且\
它很难隔离和矫正。如果没有私网，就避免了这种讨厌的动态：
OSD 们通常要么是 ``up`` 要么是 ``down`` ，没有抖动。

如果有东西导致 OSD 状态抖动（反复地被标记为 ``down`` ，然后又 \
``up`` ），你可以临时冻结它们的状态，强制监视器们暂停打摆子： ::

	ceph osd set noup      # prevent OSDs from getting marked up
	ceph osd set nodown    # prevent OSDs from getting marked down

这些标记记录在 osdmap 里::

	ceph osd dump | grep flags
	flags no-up,no-down

下列命令可清除标记： ::

	ceph osd unset noup
	ceph osd unset nodown

还支持其它两个标记 ``noin`` 和 ``noout`` ，它们分别可\
防止正在启动的 OSD 被标记为 ``in`` （已分配数据）、\
或防止 OSD 被误标记为 ``out``
（不管 ``mon osd down out interval`` 当前的值是什么）。

.. note:: ``noup`` 、 ``noout`` 和 ``nodown`` 从某种意义上说是\
   临时的，一旦标记清除了，它们被阻塞的动作短时间内就会发生；\
   相反， ``noin`` 标记是阻止 OSD 启动后进入集群，但其它所有\
   设置标记前就已经启动了的守护进程们都维持原样。

.. note:: The causes and effects of flapping can be somewhat mitigated through
   careful adjustments to the ``mon_osd_down_out_subtree_limit``,
   ``mon_osd_reporter_subtree_level``, and ``mon_osd_min_down_reporters``.
   Derivation of optimal settings depends on cluster size, topology, and the
   Ceph  release in use. Their interactions are subtle and beyond the scope of
   this document.


.. _iostat: https://en.wikipedia.org/wiki/Iostat
.. _Ceph 日志记录和调试: ../../configuration/ceph-conf#ceph-logging-and-debugging
.. _日志记录和调试: ../log-and-debug
.. _调试和日志记录: ../debug
.. _监视器与 OSD 的交互: ../../configuration/mon-osd-interaction
.. _监视器配置参考: ../../configuration/mon-config-ref
.. _监控 OSD: ../../operations/monitoring-osd-pg
.. _订阅 ceph-devel 邮件列表: mailto:majordomo@vger.kernel.org?body=subscribe+ceph-devel
.. _退订 ceph-devel 邮件列表: mailto:majordomo@vger.kernel.org?body=unsubscribe+ceph-devel
.. _订阅 ceph-users 邮件列表: mailto:ceph-users-join@lists.ceph.com
.. _退订 ceph-users 邮件列表: mailto:ceph-users-leave@lists.ceph.com
.. _操作系统推荐: ../../../start/os-recommendations
.. _ceph-devel: ceph-devel@vger.kernel.org
