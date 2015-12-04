==============
 OSD 故障排除
==============

进行 OSD 排障前，先检查一下监视器集群和网络。如果 ``ceph health`` 或 \
``ceph -s`` 返回的是健康状态，这意味着监视器们形成了法定人数。如果你还没监\
视器法定人数、或者监视器状态错误，要先\ `解决监视器问题`_\ 。核实下你的网\
络，确保它在正常运行，因为网络对 OSD 的运行和性能有显著影响。

.. _解决监视器问题: ../troubleshooting-mon


收集 OSD 数据
=============

开始 OSD 排障的第一步最好先收集信息，另外还有\ `监控 OSD`_ 时收集的，如 ``ceph \
osd tree`` 。


Ceph 日志
---------

如果你没改默认路径，可以在 ``/var/log/ceph`` 下找到日志： ::

	ls /var/log/ceph

如果看到的日志还不够详细，可以增大日志级别。请参考\ `日志记录和调试`_\ ，看看如何保\
证看到大量日志时又不影响集群运行。


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


显示剩余空间
------------

可能是文件系统问题，用 ``df`` 显示文件系统剩余空间。 ::

	df -h

其它用法见 ``df --help`` 。


I/O 统计信息
------------

用 ``iostat`` 定位 I/O 相关问题。 ::

	iostat -x


诊断消息
--------

要查看诊断信息，配合 ``less`` 、 ``more`` 、 ``grep`` 或 ``tail`` 使用 \
``dmesg`` ，例如： ::

	dmesg | grep scsi


停止自动重均衡
==============

你得周期性地维护集群的子系统、或解决某个失败域的问题（如一机架）。如果你不想在停机维\
护 OSD 时让 CRUSH 自动重均衡，提前设置 ``noout`` ： ::

	ceph osd set noout

在集群上设置 ``noout`` 后，你就可以停机维护失败域内的 OSD 了。 ::

	stop ceph-osd id={num}

.. note:: 在定位同一故障域内的问题时，停机 OSD 内的归置组状态会变为 ``degraded`` 。

维护结束后，重启OSD。 ::

	start ceph-osd id={num}

最后，解除 ``noout`` 标志。 ::

	ceph osd unset noout


.. _osd-not-running:

OSD 没运行
==========

通常情况下，简单地重启 ``ceph-osd`` 进程就可以重回集群并恢复。


OSD 起不来
----------

如果你重启了集群，但其中一个 OSD 起不来，依次检查：

- **配置文件：** 如果你新装的 OSD 不能启动，检查下配置文件，确保它合爻性（比如 \
  ``host`` 而非 ``hostname`` ，等等）；

- **检查路径：** 检查你配置的路径，以及它们自己的数据和日志路径。如果你分离了 OSD \
  数据和日志、而配置文件和实际挂载点存在出入，启动 OSD 时就可能遇挫。如果你想把日志\
  存储于一个块设备，应该为日志硬盘分区并为各 OSD 分别分配一分区。

- **检查最大线程数：** 如果你的节点有很多 OSD ，也许就会触碰到默认的最大线程数限制\
  （如通常是 32k 个），尤其是在恢复期间。你可以用 ``sysctl`` 增大线程数，把最大线\
  程数更改为支持的最大值（即 4194303 ），看看是否有用。例如： ::

	sysctl -w kernel.pid_max=4194303

  如果增大最大线程数解决了这个问题，你可以把此配置 ``kernel.pid_max`` 写入配置文\
  件 ``/etc/sysctl.conf``，使之永久生效，例如： ::

	kernel.pid_max = 4194303

- **内核版本：** 确认你在用的内核版本以及所用的发布版。 Ceph 默认依赖一些第三方工\
  具，这些工具可能有缺陷或者与特定发布版和/或内核版本冲突（如 Google perftool ）。\
  检查下\ `操作系统推荐`_\ 确保你已经解决了内核相关的问题。

- **段错误：** 如果有了段错误，提高日志级别（如果还没提高），再试试。如果重现了，联\
  系 ceph-devel 并提供你的配置文件、显示器输出和日志文件内容。

如果问题仍未解决，电子邮件也无用，你可以联系 `Inktank`_ 寻求帮助。


OSD 失败
--------

``ceph-osd`` 挂掉时，监视器可通过活着的 ``ceph-osd`` 了解到此情况，且通过 \
``ceph health`` 命令报告： ::

	ceph health
	HEALTH_WARN 1/3 in osds are down

而且，有 ``ceph-osd`` 进程标记为 ``in`` 且 ``down`` 的时候，你会得到警告，你可以用\
下面的命令得知哪个 ``ceph-osd`` 进程挂了： ::

	ceph health detail
	HEALTH_WARN 1/3 in osds are down
	osd.0 is down since epoch 23, last address 192.168.106.220:6800/11080

如果有个硬盘失败或其它错误使 ``ceph-osd`` 不能正常运行或重启，一条错误信息将会出现\
在日志文件 ``/var/log/ceph/`` 里。

如果守护进程因心跳失败、或者底层文件系统无响应而停止，查看 ``dmesg`` 获取硬盘或者内\
核错误。

如果是软件错误（失败的插入或其它意外错误），就应该回馈到 `ceph-devel`_ 邮件列表。


硬盘没剩余空间
--------------

Ceph 不允许你向满的 OSD 写入数据，以免丢失数据。在运营着的集群中，你应该能收到集群\
空间将满的警告。 ``mon osd full ratio`` 默认为 ``0.95`` 、或达到 95% 时它将阻止客\
户端写入数据。 ``mon osd nearfull ratio`` 默认为 ``0.85`` 、也就是说达到容量的 \
85% 时它会产生健康警告。

满载集群问题一般产生于测试 Ceph 在小型集群上如何处理 OSD 失败时。当某一节点利用率较\
高时，集群能够很快掩盖将满和占满率。如果你在测试小型集群上的 Ceph 如何应对 OSD 失\
败，应该保留足够的空间，然后试着临时降低 ``mon osd full ratio`` 和 \
``mon osd nearfull ratio`` 值。

``ceph health`` 会显示将满的 ``ceph-osds`` ： ::

	ceph health
	HEALTH_WARN 1 nearfull osds
	osd.2 is near full at 85%

或者： ::

	ceph health
	HEALTH_ERR 1 nearfull osds, 1 full osds
	osd.2 is near full at 85%
	osd.3 is full at 97%

处理这种情况的最好方法就是增加新的 ``ceph-osd`` ，这允许集群把数据重分布到新 OSD 里。

如果因满载而导致 OSD 不能启动，你可以试着删除那个 OSD 上的一些归置组数据目录。

.. important:: 如果你准备从填满的 OSD 中删除某个归置组，注意\ **不要**\ 删除另一个\
   OSD 上的同名归置组，否则\ **你会丢数据**\ 。\ **必须**\ 在多个 OSD 上保留至少一\
   份数据副本。

详情见\ `监视器配置参考`_\ 。


OSD 龟速或无响应
================

一个反复出现的问题是龟速或无响应。在深入性能问题前，你应该先确保不是其他故障。例如，\
确保你的网络运行正常、且 OSD 在运行，还要检查 OSD 是否被恢复流量拖住了。

.. tip:: 较新版本的 Ceph 能更好地处理恢复，可防止恢复进程耗尽系统资源而导致 \
   ``up`` 且 ``in`` 的 OSD 不可用或响应慢。


网络问题
--------

Ceph 是一个分布式存储系统，所以它依赖于网络来互联 OSD 们、复制对象、恢复错误、和检\
查心跳。网络问题会导致 OSD 延时和打摆子，详情参见\ `打摆子的 OSD`_ 。

确保 Ceph 进程和 Ceph 依赖的进程连接了、和/或在监听。 ::

	netstat -a | grep ceph
	netstat -l | grep ceph
	sudo netstat -p | grep ceph

检查网络统计信息。 ::

	netstat -s


驱动器配置
----------

一个存储驱动器应该只用于一个 OSD 。如果有其它进程共享驱动器，顺序读和顺序写吞吐量会\
成为瓶颈，包括日志记录、操作系统、监视器、其它 OSD 和非 Ceph 进程。

Ceph 在日志记录\ *完成之后*\ 才会确认写操作，所以使用 ``ext4`` 或 XFS 文件系统时高\
速的 SSD 对降低响应延时很有吸引力。相反， ``btrfs`` 文件系统可以同时读写。

.. note:: 给驱动器分区并不能改变总吞吐量或顺序读写限制。把日志分离到单独的分区可能\
   有帮助，但最好是另外一块硬盘的分区。


坏扇区和碎片化硬盘
------------------

检修下硬盘是否有坏扇区和碎片。这会导致总吞吐量急剧下降。


监视器和 OSD 蜗居
-----------------

监视器是普通的轻量级进程，但它们会频繁调用 ``fsync()`` ，这会妨碍其它工作量，特别是\
监视器和 OSD 共享驱动器时。另外，如果你在 OSD 主机上同时运行监视器，遭遇的性能问题\
可能和这些相关：

- 运行较老的内核（低于3.0）
- v0.48 版运行在老的 ``glibc`` 之上
- 运行的内核不支持 ``syncfs(2)`` 系统调用

在这些情况下，同一主机上的多个 OSD 会相互拖垮对方。它们经常导致爆炸式写入。


进程蜗居
--------

共存于同一套硬件、并向 Ceph 写入数据的进程（像基于云的解决方案、虚拟机和其他应用程\
序）会导致 OSD 延时大增。一般来说，我们建议用一主机跑 Ceph 、其它主机跑其它进程，实\
践证明把 Ceph 和其他应用程序分开可提高性能、并简化故障排除和维护。


日志记录级别
------------

如果你为追踪某问题提高过日志级别、但结束后忘了调回去，这个 OSD 将向硬盘写入大量日\
志。如果你想始终保持高日志级别，可以考虑给默认日志路径挂载个硬盘，即 \
``/var/log/ceph/$cluster-$name.log`` 。


恢复节流
--------

根据你的配置， Ceph 可以降低恢复速度来维持性能，否则它会不顾 OSD 性能而加快恢复速\
度。检查下 OSD 是否正在恢复。


内核版本
--------

检查下你在用的内核版本。较老的内核也许没有移植能提高 Ceph 性能的功能。


内核与 SyncFS 问题
------------------

试试在一主机上只运行一个 OSD ，看看能否提升性能。老内核未必支持有 ``syncfs(2)`` 系\
统调用的 ``glibc`` 。


文件系统问题
------------

当前，我们推荐基于 xfs 或 ext4 部署集群。 btrfs 有很多诱人的功能，但文件系统内的缺\
陷可能导致性能问题。


内存不足
--------

我们建议为每 OSD 进程规划 1GB 内存。你也许注意到了，通常情况下 OSD 仅会用一小部分\
（如 100-200MB ）。你也许想用这些空闲内存跑一些其他应用，如虚拟机等等，然而当 OSD \
进入恢复状态时，其内存利用率激增，如果没有可用内存，此 OSD 的性能将差的多。


old requests 或  slow requests
------------------------------

如果某 ``ceph-osd`` 守护进程对一请求响应很慢，它会生成日志消息来抱怨请求耗费的时间\
过长。默认警告阀值是 30 秒，用 ``osd op complaint time`` 选项来配置。这种情况发生\
时，集群日志系统会收到这些消息。

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


打摆子的 OSD
============

我们建议同时部署公网（前端）和集群网（后端），这样能更好地满足对象复制的容量需求。另\
一个优点是你可以运营一个不连接互联网的集群，以此避免拒绝估计。 OSD 们互联和检查心跳\
时会优选集群网（后端），详情见\ `监视器与 OSD 的交互`_\ 。

然而，如果集群网（后端）失败、或出现了明显的延时，同时公网（前端）却运行良好， OSD \
现在不能很好地处理这种情况。这时 OSD 们会向监视器报告邻居 ``down`` 了、同时报告自己\
是 ``up`` 的，我们把这种情形称为打摆子（ flapping ）。

如果有东西导致 OSD 打摆子（反复地被标记为 ``down`` ，然后又 ``up`` ），你可以强制\
监视器停止： ::

	ceph osd set noup      # prevent OSDs from getting marked up
	ceph osd set nodown    # prevent OSDs from getting marked down

这些标记记录在 osdmap 数据结构里： ::

	ceph osd dump | grep flags
	flags no-up,no-down

下列命令可清除标记： ::

	ceph osd unset noup
	ceph osd unset nodown

``mon osd down out interval`` is).
还支持其它两个标记 ``noin`` 和 ``noout`` ，它们分别可防止正在启动的 OSD 被标记为 \
``in`` 、或被误标记为 ``out`` （不管 `` mon osd down out interval`` 的值是什么）。

.. note:: ``noup`` 、 ``noout`` 和 ``nodown`` 从某种意义上说是临时的，一旦标记清\
   除了，它们被阻塞的动作短时间内就会发生；相反， ``noin`` 标记阻止 OSD 启动后进入\
   集群，但其它守护进程都维持原样。


.. _iostat: http://en.wikipedia.org/wiki/Iostat
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
.. _Inktank: http://inktank.com
.. _操作系统推荐: ../../../install/os-recommendations
.. _ceph-devel: ceph-devel@vger.kernel.org
