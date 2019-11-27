.. Troubleshooting OSDs

==============
 OSD 故障排除
==============

对 OSD 排障前，先检查一下监视器集群和网络。如果 ``ceph health``
或 ``ceph -s`` 返回的是健康状态，这意味着监视器们形成了\
法定人数。如果你还没监视器法定人数、或者监视器状态错误，要\
`先定位监视器问题 <../troubleshooting-mon>`_\ 。核实下你的\
网络，确保它在正常运行，因为网络对 OSD 的运行和性能有显著影响。


.. Obtaining Data About OSDs

收集 OSD 数据
=============

开始 OSD 排障的第一步最好先收集信息，另外还有\ `监控 OSD`_ 时\
收集的，如 ``ceph osd tree`` 。


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

用管理套接字检索运行时信息。所有 Ceph 套接字列表： ::

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

要查看诊断信息，配合 ``less`` 、 ``more`` 、 ``grep`` 或
``tail`` 使用 ``dmesg`` ，例如： ::

	dmesg | grep scsi


.. Stopping w/out Rebalancing

停止自动重均衡
==============

你得周期性地维护集群的子系统、或解决某个失败域的问题（如一机\
架）。如果你不想在停机维护 OSD 时让 CRUSH 自动重均衡，提前设置
``noout`` ： ::

	ceph osd set noout

在集群上设置 ``noout`` 后，你就可以停机维护失败域内的 OSD 了。 ::

	stop ceph-osd id={num}

.. note:: 在定位同一故障域内的问题时，停机 OSD 内的归置组状态\
   会变为 ``degraded`` 。

维护结束后，重启OSD。 ::

	start ceph-osd id={num}

最后，解除 ``noout`` 标志。 ::

	ceph osd unset noout


.. _osd-not-running:

OSD 没运行
==========

通常情况下，简单地重启 ``ceph-osd`` 进程就可以重回集群并恢复。


.. An OSD Won't Start

OSD 起不来
----------

如果你重启了集群，但其中一个 OSD 起不来，依次检查：

- **配置文件：** 如果你新装的 OSD 不能启动，检查下配置文件，确\
  保它合爻性（比如 ``host`` 而非 ``hostname`` ，等等）；

- **检查路径：** 检查你配置的路径，以及它们自己的数据和日志路\
  径。如果你分离了 OSD 数据和日志、而配置文件和实际挂载点存在\
  出入，启动 OSD 时就可能遇挫。如果你想把日志存储于一个块设备，\
  应该为日志硬盘分区并为各 OSD 分别分配一分区。

- **检查最大线程数：** 如果你的节点有很多 OSD ，也许就会触碰到\
  默认的最大线程数限制（如通常是 32k 个），尤其是在恢复期间。\
  你可以用 ``sysctl`` 增大线程数，把最大线程数更改为支持的最大\
  值（即 4194303 ），看看是否有用。例如： ::

	sysctl -w kernel.pid_max=4194303

  如果增大最大线程数解决了这个问题，你可以把此配置
  ``kernel.pid_max`` 写入配置文件 ``/etc/sysctl.conf`` ，使之\
  永久生效，例如： ::

	kernel.pid_max = 4194303

- **内核版本：** 确认你在用的内核版本以及所用的发布版。 Ceph \
  默认依赖一些第三方工具，这些工具可能有缺陷或者与特定发布版和\
  /或内核版本冲突（如 Google perftool ）。检查下\
  `操作系统推荐`_\ 确保你已经解决了内核相关的问题。

- **段错误：** 如果有了段错误，提高日志级别（如果还没提高），\
  再试试。如果重现了，联系 ceph-devel 并提供你的配置文件、显示\
  器输出和日志文件内容。


.. An OSD Failed

OSD 失败
--------

``ceph-osd`` 挂掉时，监视器可通过活着的 ``ceph-osd`` 了解到此\
情况，且通过 ``ceph health`` 命令报告： ::

	ceph health
	HEALTH_WARN 1/3 in osds are down

而且，有 ``ceph-osd`` 进程标记为 ``in`` 且 ``down`` 的时候，你\
会得到警告，你可以用下面的命令得知哪个 ``ceph-osd`` 进程挂了： ::

	ceph health detail
	HEALTH_WARN 1/3 in osds are down
	osd.0 is down since epoch 23, last address 192.168.106.220:6800/11080

如果有个硬盘失败或其它错误使 ``ceph-osd`` 不能正常运行或重启，\
一条错误信息将会出现在日志文件 ``/var/log/ceph/`` 里。

如果守护进程因心跳失败、或者底层文件系统无响应而停止，查看
``dmesg`` 获取硬盘或者内核错误。

如果是软件错误（失败的插入或其它意外错误），就应该回馈到
`ceph-devel`_ 邮件列表。


.. No Free Drive Space

硬盘没剩余空间
--------------

Ceph 不允许你向满的 OSD 写入数据，以免丢失数据。在运营着的集群\
中，你应该能收到集群空间将满的警告。 ``mon osd full ratio``
默认为 ``0.95`` 、或达到 95% 时它将阻止客户端写入数据；
``mon osd backfillfull ratio`` 默认为 ``0.90`` 、或达到容量的
90% 时它会阻塞，防止回填启动； OSD 将满比率默认为 ``0.85`` 、\
也就是说达到容量的 85% 时它会产生健康警告。

可以用此命令更改将满比率： ::

    ceph osd set-nearfull-ratio <float[0.0-1.0]>

集群用满的问题一般出现在测试过程中，为了检验 Ceph 在小型集群上\
如何处理 OSD 失败。当某一节点存储的集群数据比例较高时，集群
能够很快掩盖将满和占满率。如果你在小型集群上测试 Ceph 如何应对
OSD 失败，应该保留足够的空闲空间，然后临时降低 OSD 的
``full ratio`` 、 ``backfillfull ratio`` 和 ``nearfull ratio``
值，命令为：

::

    ceph osd set-nearfull-ratio <float[0.0-1.0]>
    ceph osd set-full-ratio <float[0.0-1.0]>
    ceph osd set-backfillfull-ratio <float[0.0-1.0]>

``ceph health`` 会显示将满的 ``ceph-osds`` ： ::

	ceph health
	HEALTH_WARN 1 nearfull osd(s)

或者： ::

	ceph health detail
	HEALTH_ERR 1 full osd(s); 1 backfillfull osd(s); 1 nearfull osd(s)
	osd.3 is full at 97%
	osd.4 is backfill full at 91%
	osd.2 is near full at 87%

处理这种情况的最好方法就是增加新的 ``ceph-osd`` ，这允许集群把\
数据重分布到新 OSD 里。

如果因满载而导致 OSD 不能启动，你可以试着删除那个 OSD 上的一些\
归置组数据目录。

.. important:: 如果你准备从填满的 OSD 中删除某个归置组，注意\
   **不要**\ 删除另一个 OSD 上的同名归置组，否则\
   **你会丢数据**\ 。\ **必须**\ 在多个 OSD 上保留至少一份\
   数据副本。

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

Ceph 是一个分布式存储系统，所以它依赖于网络来互联 OSD 们、复制\
对象、故障恢复、和检查心跳。网络问题会导致 OSD 延时和状态抖动，
详情参见\ `状态抖动的 OSD`_ 。

确保 Ceph 进程和 Ceph 依赖的进程连接了、和/或在监听。 ::

	netstat -a | grep ceph
	netstat -l | grep ceph
	sudo netstat -p | grep ceph

检查网络统计信息。 ::

	netstat -s


.. _Drive Configuration:

驱动器配置
----------

一个存储驱动器应该只用于一个 OSD 。如果有其它进程共享驱动器，\
顺序读和顺序写吞吐量会成为瓶颈，包括日志记录、操作系统、监视\
器、其它 OSD 和非 Ceph 进程。

Ceph 在日志记录\ *完成之后*\ 才会确认写操作，所以高速 SSD 有\
助于降低响应时间，尤其是在用 ``XFS`` 或 ``ext4`` 文件系统时。\
相反， ``btrfs`` 文件系统可以同时写入和记日志。（然而还是得注\
意，我们不建议在生产环境下用 ``btrfs`` 。）

.. note:: 给驱动器分区并不能改变总吞吐量或顺序读写限制。把日\
   志分离到单独的分区可能有帮助，但最好是另外一块硬盘的分区。


.. _Bad Sectors / Fragmented Disk:

坏扇区和碎片化硬盘
------------------

检修下硬盘是否有坏扇区和碎片。这会导致总吞吐量急剧下降。


.. Co-resident Monitors/OSDs

监视器和 OSD 蜗居
-----------------

监视器是普通的轻量级进程，但它们会频繁调用 ``fsync()`` ，这会\
妨碍其它工作量，特别是监视器和 OSD 共享驱动器时。另外，如果你在
OSD 主机上同时运行监视器，遭遇的性能问题可能和这些相关：

- 运行较老的内核（低于3.0）
- 运行的内核不支持 ``syncfs(2)`` 系统调用

在这些情况下，同一主机上的多个 OSD 会相互拖垮对方。它们经常\
导致爆炸式写入。


.. Co-resident Processes

进程蜗居
--------

共存于同一套硬件、并向 Ceph 写入数据的进程（像基于云的\
解决方案、虚拟机和其他应用程序）会导致 OSD 延时大增。\
一般来说，我们建议用一主机跑 Ceph 、其它主机跑其它进程，\
实践证明把 Ceph 和其他应用程序分开可提高性能、并简化故障排除和\
维护。


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


.. _Filesystem Issues:

文件系统问题
------------

当前，我们推荐基于 xfs 部署集群。

我们不推荐用 btrfs 或 ext4 。 btrfs 有很多诱人的功能，但文件系\
统内的缺陷可能导致性能问题，以及 ENOSPC 伪错误。我们不建议使用
ext4 ，因为其 xattr 尺寸限制会破坏我们对长对象名的支持（ RGW
所需）。

详情见\ `文件系统推荐`_\ 。

.. _文件系统推荐: ../configuration/filesystem-recommendations


.. Insufficient RAM

内存不足
--------

我们建议为每 OSD 进程规划 1GB 内存。你也许注意到了，通常情况下
OSD 仅会用一小部分（如 100-200MB ）。你也许想用这些空闲内存跑\
一些其他应用，如虚拟机等等，然而当 OSD 进入恢复状态时，其内存\
利用率激增，如果没有可用内存，此 OSD 的性能将差的多。


.. Old Requests or Slow Requests

old requests 或  slow requests
------------------------------

如果某 ``ceph-osd`` 守护进程对一请求响应很慢，它会生成日志消息\
来抱怨请求耗费的时间过长。默认警告阀值是 30 秒，用
``osd op complaint time`` 选项来配置。这种情况发生时，集群\
日志系统会收到这些消息。

很老的版本抱怨 "old requests" ： ::

	osd.0 192.168.106.220:6800/18813 312 : [WRN] old request osd_op(client.5099.0:790 fatty_26485_object789 [write 0~4096] 2.5e54f643) v4 received at 2012-03-06 15:42:56.054801 currently waiting for sub ops

较新版本的 Ceph 抱怨 "slow requests" ： ::

	{date} {osd.num} [WRN] 1 slow requests, 1 included below; oldest blocked for > 30.005692 secs
	{date} {osd.num}  [WRN] slow request 30.005692 seconds old, received at {date-time}: osd_op(client.4240.0:8 benchmark_data_ceph-1_39426_object7 [write 0~4194304] 0.69848840) v4 currently waiting for subops from [610]


可能起因有：

- 坏驱动器（查看 ``dmesg`` 输出）；
- 内核文件系统缺陷（查看 ``dmesg`` 输出）；
- 集群过载（检查系统负载、 iostat 等等）；
- ``ceph-osd`` 守护进程缺陷。

可能的解决方法：

- 从 Ceph 主机去除 VM 云解决方案；
- 升级内核；
- 升级 Ceph ；
- 重启 OSD 。


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

OSD 事件，因为它负责为操作作准备：

- ``queued_for_pg``: op 已放入队列，等着它自己的 PG 来处理；
- ``reached_pg``: 其 PG 已开始处理这个 op ；
- ``waiting for \*``: 此 op 正等着某些其它工作结束，这样它才能\
  继续下一步（如，一个新 OSDMap ；等着这个对象目标完成洗刷；等\
  着相应的 PG 完成互联；所有都在消息内指出了）；
- ``started``: 此 op 已被这个 OSD 接受为应该做的事，并且正在进\
  行中；
- ``waiting for subops from``: 此 op 已发送给了副本 OSD 们。

FileStore 事件:

- ``commit_queued_for_journal_write``: 此 op 已递送给了 FileStore 。
- ``write_thread_in_journal_buffer``: 此 op 已经在日志的缓冲中\
  了、并等着持久化（等着下一次硬盘写操作）；
- ``journaled_completion_queued``: 此 op 已在硬盘上作了日志、\
  且它的回调已进入队列等着被调用了。

OSD 事件，数据已存入本地硬盘后的：

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

我们建议同时部署公网（前端）和集群网（后端），这样能更好地满足\
对象复制的容量需求。另一个优点是你可以运营一个不连接互联网的集\
群，以此避免拒绝服务攻击。 OSD 们互联和检查心跳时会优选集群网\
（后端），详情见\ `监视器与 OSD 的交互`_\ 。

然而，如果集群网（后端）失败、或出现了明显的延时，同时公网（前\
端）却运行良好， OSD 现在不能很好地处理这种情况。这时 OSD 们会\
向监视器报告邻居 ``down`` 了、同时报告自己是 ``up`` 的，我们把\
这种情形称为状态抖动（ flapping ）。

.. note:: 译者：社区同仁讨论认为，这是随时间延续，不断地在
   ``up`` 、 ``down`` 状态之间反复的情形，状态变动的时间间隔\
   有规律或无规律，运动方向为“上下”，非“左右”、亦非“前后”，\
   也可理解为打摆子、状态翻转。总之是一种病态的、非正常的状态。

如果有东西导致 OSD 状态抖动（反复地被标记为 ``down`` ，然后又 \
``up`` ），你可以强制监视器停止： ::

	ceph osd set noup      # prevent OSDs from getting marked up
	ceph osd set nodown    # prevent OSDs from getting marked down

这些标记记录在 osdmap 数据结构里： ::

	ceph osd dump | grep flags
	flags no-up,no-down

下列命令可清除标记： ::

	ceph osd unset noup
	ceph osd unset nodown

还支持其它两个标记 ``noin`` 和 ``noout`` ，它们分别可防止\
正在启动的 OSD 被标记为 ``in`` 、或被误标记为 ``out`` （不管
`` mon osd down out interval`` 的值是什么）。

.. note:: ``noup`` 、 ``noout`` 和 ``nodown`` 从某种意义上说是\
   临时的，一旦标记清除了，它们被阻塞的动作短时间内就会发生；\
   相反， ``noin`` 标记阻止 OSD 启动后进入集群，但其它守护进程\
   都维持原样。


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
