==============
 OSD 故障排除
==============
.. Troubleshooting OSDs

对集群的 OSD 排障前，先检查一下各监视器和网络。

首先，确定一下监视器们是否形成了法定人数。
执行 ``ceph health`` 或 ``ceph -s`` 命令，
如果 Ceph 返回的是 ``HEALTH_OK`` 说明形成了监视器法定人数。

如果监视器没形成法定人数、或者监视器状态有错误，
继续下一步之前需要参考\ `监视器故障排除 <../troubleshooting-mon>`_\
资料先解决监视器问题。

接下来，核实下你的网络，确保它在正常运行，
因为网络对 OSD 的正常运作和性能有显著影响，
检查一下主机侧的丢包情况和交换机侧的 CRC 错误。


收集 OSD 数据
=============
.. Obtaining Data About OSDs

在对 OSD 进行故障排除时，收集有关 OSD 的各种信息非常有用。
有些信息来自\ `监控 OSD`_ 的实践
（例如，运行 ``ceph osd tree`` 命令获得的）。
其他信息与集群的拓扑结构有关，将在以下章节中讨论。


Ceph 日志
---------
.. Ceph Logs

Ceph 日志文件存储在 ``/var/log/ceph`` 下。除非路径改掉了
（或者你的是容器化环境，它会把日志存到别处），
日志文件可以用如下命令罗列：

.. prompt:: bash

   ls /var/log/ceph

如果看到的日志还不够详细，可以更改日志级别。
参考\ `日志记录和调试`_\ ，
确保能看到适当的日志，同时又不影响集群运行。


管理套接字
----------
.. Admin Socket

用管理套接字检索运行时信息。首先，
罗列一下 Ceph 守护进程的套接字，执行下列命令：

.. prompt:: bash

   ls /var/run/ceph

然后，执行下列命令（把 ``{daemon-name}``
替换成指定守护进程的名字比如如 ``osd.0`` ）：

.. prompt:: bash

   ceph daemon {daemon-name} help

另外，也可以指定一个 ``{socket-file}`` 执行命令
（ socket-file 套接字文件是 ``/var/run/ceph`` 下的一个特殊文件）：

.. prompt:: bash

   ceph daemon {socket-file} help

通过管理套接字可以实现很多任务，包括：

- 在运行时列出 Ceph 配置
- 列出历史操作
- 列出操作的优先队列状态
- 列出正在进行的操作
- 列出性能计数器

显示剩余空间
------------
.. Display Free Space

文件系统可能会出现问题。要显示文件系统剩余空间，执行下列命令：

.. prompt:: bash

   df -h

此命令的其它用法和选项见 ``df --help`` 。

I/O 统计信息
------------
.. I/O Statistics

用 `iostat`_ 工具定位 I/O 相关的问题。执行下列命令：

.. prompt:: bash

   iostat -x

诊断消息
--------
.. Diagnostic Messages

要查看内核里的诊断信息，配合 ``less`` 、 ``more`` 、 ``grep`` 或
``tail`` 执行 ``dmesg`` 命令，例如：

.. prompt:: bash

   dmesg | grep scsi


在不引起重均衡的情况下停机
==========================
.. Stopping without Rebalancing

有时不得不维修集群的一小部分、
或解决某个故障域的问题（如一机架）。
不过，当您停止 OSD 进行维护时，
可能不想让 CRUSH 自动重均衡集群。
要避免这种重均衡行为，可执行以下命令把集群设置为 ``noout`` ：

.. prompt:: bash

   ceph osd set noout

.. warning:: 这只是一个思想实验，目的是让读者对故障域和
   CRUSH 行为有所了解，而不是建议用 Luminous 以上版本的人\
   只要会执行 ``ceph osd set noout`` 命令就行了。
   当 OSD 恢复到 ``up`` 状态时，重均衡将恢复运行，
   ``ceph osd set noout`` 命令引入的更改也将回退。

在 Luminous 或更高版本上，可以只在受影响的 OSD 上设置此标记，
这样更安全。要给指定 OSD 增加或删除 ``noout`` 标记，
可以执行下列命令：

.. prompt:: bash

   ceph osd add-noout osd.0
   ceph osd rm-noout  osd.0

也可以一次设置整个 CRUSH 桶。比如，
你准备关闭 ``prod-ceph-data1701`` 来扩容内存，
可以执行下列命令：

.. prompt:: bash

   ceph osd set-group noout prod-ceph-data1701

设置此标记后，就可以停止 OSD 、还有失败域内\
共存的其它需要维护的 Ceph 服务： ::

    systemctl stop ceph\*.service ceph\*.target

.. note:: 一个 OSD 停机后，此 OSD 内的所有归置组\
   都会被标记为 ``degraded`` 。

维护结束后，重启 OSD 和其它停掉的守护进程们。
不过，如果维护期间重启过主机，
无需重启它们应该就会自己回归。
要重启 OSD 或其他守护进程，
执行下列命令：

.. prompt:: bash

   sudo systemctl start ceph.target

最后，必须解除集群范围的 ``noout`` 标志，
执行下列命令：

.. prompt:: bash

   ceph osd unset noout
   ceph osd unset-group noout prod-ceph-data1701

很多当代 Linux 发行版都采用了 ``systemd`` 做服务管理。
然而，对于某些操作系统（特别是比较老的），
你可能得用等价的 ``service`` 或 ``start``/``stop`` 命令。


.. _osd-not-running:

OSD 没运行
==========
.. OSD Not Running

通常情况下，重启 ``ceph-osd`` 守护进程后，它可以重回集群并恢复。


OSD 起不来
----------
.. An OSD Won't Start

如果启动集群后，其中一个 OSD 起不来，依次检查：

- **配置文件：** 如果你新装的 OSD 不能启动，
  检查下配置文件，确保它合爻性
  （比如 ``host`` 而非 ``hostname`` ，等等）；

- **检查各种路径：** 检查你配置的路径，
  对应的数据和元数据路径是否都存在
  （比如日志路径、 WAL 、 DB ）。
  分离开 OSD 数据和元数据，看看配置文件和实际挂载点是否存在出入。
  如果有，这些错误可能就是 OSD 不能启动的原因。
  可以把元数据存储在一个独立的块设备、分区、
  或 LVM 管理的驱动器上，并给每个 OSD 分别分配一个分区。

- **检查最大线程数：** 如果集群内有的节点 OSD 数量非常多，
  也许会达到到默认的最大线程数限制（通常是 32000 个），
  尤其是在恢复期间更可能发生。
  把最大线程数更改为支持的最大值（即 4194303 ），
  也许能解决此问题。要把线程数改成最大值，
  执行下列命令：

  .. prompt:: bash

     sysctl -w kernel.pid_max=4194303

  如果增大最大线程数解决了这个问题，你可以把此配置
  ``kernel.pid_max`` 写入 ``/etc/sysctl.d`` 内的文件或\
  写入主配置 ``/etc/sysctl.conf`` ，使之永久生效，
  例如： ::

    kernel.pid_max = 4194303

- **检查 ``nf_conntrack``:** 这个连接跟踪和连接限制系统
  是许多 Ceph 生产集群问题的祸根，这些问题常常是缓慢且潜移默化地发生。
  随着集群的拓扑结构和客户端载荷的增长，
  奇怪且间歇性的连接故障和性能问题发生得越来越多，
  特别是在一天中的某个时间会更糟。要定位此类问题，
  先检查 ``syslog`` 历史记录，看是否有 "table full" （表填满）事件。
  定位此类问题的其中一个方法如下：首先，
  用 ``sysctl`` 命令给 ``nf_conntrack_max`` 赋予一个非常大的值。
  然后，提高 ``nf_conntrack_buckets`` 的数值，使得
  ``nf_conntrack_buckets`` x 8 = ``nf_conntrack_max`` ；
  这一步可能需要在 ``sysctl`` 之外操作，（例如
  ``"echo 131072 > /sys/module/nf_conntrack/parameters/hashsize`` ）。
  另外一种定位此问题的方法是把相关的内核模块加入黑名单，
  防止它们一起处理。此方法很有效，但也很脆弱。
  涉及的模块及其顺序因内核版本不同可能差别很大。
  即使加了黑名单， ``iptables`` 或 ``docker``
  有时仍然会激活连接跟踪，
  所以我们建议对这种调整采用 "设置后不管" 策略。
  在现代系统中，这不会消耗太多资源。

- **内核版本：** 确认在用的内核版本以及所用的发布版。
  默认情况下， Ceph 会使用一些第三方工具，这些工具\
  可能有缺陷或者与特定发行版或内核版本产生冲突
  （如 Google 的 ``gperftools`` 和 ``TCMalloc`` ）。
  检查下\ `操作系统推荐`_\ 和各 Ceph 版本的发布说明，
  以确保已经解决了内核相关的问题。

- **段错误：** 如果出现了段错误，提高日志级别并重启有问题的守护进程。
  如果段错误重现了，搜索一下 Ceph 缺陷追踪器
  `https://tracker.ceph/com/projects/ceph <https://tracker.ceph.com/projects/ceph/>`_ 和
  ``dev`` 、 ``ceph-users`` 邮件列表归档
  `https://ceph.io/resources <https://ceph.io/resources>`_ ，
  看看别人是否也遇到并报告过同样的问题。
  如果它真是一个新的、唯一的故障案例，把它发到 ``dev`` 邮件列表，
  并提供这些信息：运行的 Ceph 具体版本号、
  ``ceph.conf`` （把密钥用 XXX 替换掉）、
  你的监视器状态输出、和相关的日志文件节选。


OSD 故障
--------
.. An OSD Failed

某个 OSD 发生故障时，这意味着一个 ``ceph-osd`` 进程没有响应或者已经死亡，
而且对应的 OSD 已经被标记成了 ``down`` 状态。
活着的 ``ceph-osd`` 守护进程们会报告给各监视器，
说它挂了，随后它就会浮现在 ``ceph health`` 命令的新状态信息里，如下例：

.. prompt:: bash

   ceph health

::

   HEALTH_WARN 1/3 in osds are down

只要有一个或多个 OSD 被标记为 ``in`` 且 ``down`` ，
就会产生这样的健康告警。要查出哪个 OSD 是 ``down`` 的，
给命令加上 ``detail`` ，如下例：

.. prompt:: bash

   ceph health detail

::

   HEALTH_WARN 1/3 in osds are down
   osd.0 is down since epoch 23, last address 192.168.106.220:6800/11080

或者，执行下列命令：

.. prompt:: bash

   ceph osd tree down

如果有个驱动器发生故障或其它错误使
``ceph-osd`` 不能正常运行或重启，应该会有\
一条错误信息出现在 ``/var/log/ceph/`` 内的日志文件里。

如果 ``ceph-osd`` 守护进程因心跳故障、
``suicide timeout`` 错误停止，
其原因可能是底层驱动器或文件系统无响应。
查看一下 ``dmesg`` 和 `syslog` 输出，找一下驱动器错误或内核错误。
可能得加一些选项（例如 ``dmesg -T`` 显示容易看懂的时间戳），
以免把旧错误消息误读成新的。

如果整个主机的 OSD 都是 ``down`` ，检查一下，
看是不是主机网络错误或硬件问题。

如果这个 OSD 问题是软件错误（比如，
失败的断言或其它意外错误），搜索一下有没有人报告过这个问题，
在 `bug tracker <https://tracker.ceph/com/projects/ceph>`_ 、
`开发邮件列表存档 <https://lists.ceph.io/hyperkitty/list/dev@ceph.io/>`_ 、
和 `ceph-users 邮件列表存档
<https://lists.ceph.io/hyperkitty/list/ceph-users@ceph.io/>`_ 搜索。
如果没有明确修复或已经发现的缺陷，那就 :ref:`把这个问题报告给 ceph-devel
邮件列表 <Get Involved>` 。


.. _no-free-drive-space:

硬盘没剩余空间
--------------
.. No Free Drive Space

如果某一个 OSD 满了， Ceph 为防止数据丢失，会确保不再向 OSD 写入新数据。
在健康运行着的集群中，当 OSD 们和存储池\
接近设置的“占满”比率时，就会产生健康警告。
``mon_osd_full_ratio`` 阈值默认是 ``0.95`` （或容量的 95% ）：
这就是那个点，超过时客户端们就不能再写入数据了；
``mon_osd_backfillfull_ratio`` 阈值默认为 ``0.90`` （或容量的 90% ）：
这就是那个点，超过时就不会启动回填。
``mon_osd_nearfull_ratio`` 阈值默认为 ``0.85`` （或容量的 85% ）：
这就是那个点，达到时它会产生 ``OSD_NEARFULL`` 健康警告。

Ceph 集群内的 OSD 们分配到的数据量可能差别很大。
要检查“有多满”，可以显示出每个 OSD 的数据利用率，执行下列命令：

.. prompt:: bash

   ceph osd df

要检查整个集群“有多满”，可以显示它的总体利用率、
和各存储池的数据分布，执行下列命令：

.. prompt:: bash

   ceph df

检查 ``ceph df`` 命令的输出时，特别留意一下
**最满** 的 OSD 们，而不是原始空间的利用率。
只要有一个突兀的 OSD 被填满，就会导致它所在存储池的所有写入失败。
``ceph df`` 报告存储池的可用空间时，考虑了\
指定存储池里相对 *最满* OSD 的比率设置。
要使数据分布更加均匀，有两种方法：
(1) 用 ``reweight-by-utilization`` 命令逐步地把数据\
从过满的 OSD 移到不太满的 OSD 上；
(2) 在 Luminous 后来的修订版和后续版本上，可以试着用
``ceph-mgr`` ``balancer`` 模块来自动地执行这个任务。

要调整这个“占满”比率，
执行下列命令：

.. prompt:: bash

   ceph osd set-nearfull-ratio <float[0.0-1.0]>
   ceph osd set-full-ratio <float[0.0-1.0]>
   ceph osd set-backfillfull-ratio <float[0.0-1.0]>

有时候，集群出现用满的问题是由于一个 OSD 发生故障。
发生此问题，原因要么是测试、要么是集群很小、很满、或者不均衡。
当某一 OSD 或节点存储着很大比例的集群数据时，
组件故障甚或自然增长就能诱发 ``nearfull`` 和 ``full`` 比率超额。
如果你在小型集群上测试 Ceph 如何应对 OSD 故障，
建议留出足够的空闲空间，并且考虑临时降低 OSD 的
``full ratio`` 、 ``backfillfull ratio`` 和
OSD ``nearfull ratio`` 的值。

OSD 的“占满”情况在 ``ceph health`` 的输出中能看到，
如下例：

.. prompt:: bash

   ceph health

::

  HEALTH_WARN 1 nearfull osd(s)

要看细节，再加上 ``detail`` 命令，如下例：

.. prompt:: bash

    ceph health detail

::

    HEALTH_ERR 1 full osd(s); 1 backfillfull osd(s); 1 nearfull osd(s)
    osd.3 is full at 97%
    osd.4 is backfill full at 91%
    osd.2 is near full at 87%

处理集群用满的最好方法就是增加新 OSD 扩容，
新增 OSD 后，集群就能把数据重分布到新存储器里。
搜索一下 ``rados bench`` 遗留的数据，它们在浪费空间。

如果因满载而导致旧的 FileStore OSD 不能启动，
你可以试着删除那个 OSD 上的一些归置组数据目录，
回收一些空间。

.. important:: 如果你准备从填满的 OSD 中删除某个归置组，
   注意\ **不要**\ 删除另一个 OSD 上的同名归置组目录，
   否则\ **你会丢数据**\ 。\ **必须**\ 在多个 OSD 上\
   保留至少一份数据副本。删除归置组目录是\
   一种少见的极端干涉方法，轻易不要用。

详情见\ `监视器配置参考`_\ 。


OSD 龟速或无响应
================
.. OSDs are Slow/Unresponsive

OSD 有时会龟速或无响应。解决这种常见问题时，
你应该先确保排除了其他故障的可能性，然后再深入性能问题。
例如，确保你的网络运行正常、且 OSD 在运行，
还要检查 OSD 是否被恢复流量拖住了。

.. tip:: 在 Ceph Luminous 版以前， ``up`` 且 ``in``
   的 OSD 有时候不可用或很慢，因为恢复消耗了系统资源。
   较新的版本解决了这个问题，能更好地处理恢复。


网络问题
--------
.. Networking Issues

作为一个分布式的存储系统， Ceph 要靠网络实现 OSD 互联、
复制、故障恢复、和周期性心跳。
联网问题会导致 OSD 延时和状态抖动，详情参见\ `状态抖动的 OSD`_ 。

确保 Ceph 进程和 Ceph 依赖的进程已建立连接、且在监听，
执行下列命令：

.. prompt:: bash

   netstat -a | grep ceph
   netstat -l | grep ceph
   sudo netstat -p | grep ceph

检查网络统计信息，执行下列命令：

.. prompt:: bash

   netstat -s


驱动器配置
----------
.. Drive Configuration

一个 SAS 或 SATA 存储驱动器应该只用于一个 OSD ；
NVMe 驱动器可以轻松处理两个或更多。不过，如果\
有其它进程共享驱动器，读和写吞吐量会成为瓶颈。
这类进程包括：日志/元数据、操作系统、 Ceph 监视器、 ``syslog`` 日志、
其它 OSD 以及非 Ceph 进程。

Ceph 在日志记录\ *完成之后*\ 才会确认写操作，
所以高速 SSD 有助于降低响应时间，
尤其是在用旧的 FileStore OSD 搭配 ``XFS`` 或 ``ext4`` 文件系统时。
相反， ``Btrfs`` 文件系统可以同时写入和记日志。
（然而还是得注意，我们不建议在生产环境下用 ``Btrfs`` 。）

.. note:: 给驱动器分区并不能改变\
   总吞吐量或顺序读写上限。\
   把日志分离到单独的分区有可能提升吞吐量，
   但它最好位于另外一个物理驱动器。

.. warning:: Reef 不支持 FileStore 。 Reef 之后的版本不支持 FileStore 。
   任何提及 FileStore 的信息仅与 Ceph 的 Quincy 版本和 Quincy 之前的版本相关。


坏扇区和碎片化硬盘
------------------
.. Bad Sectors / Fragmented Disk

检修下硬盘是否有坏块、碎片、和其它会导致性能急剧下降的错误。\
检查设备错误用得上的工具有 ``dmesg`` 、 ``syslog`` 日志、和
``smartctl`` （包含在 ``smartmontools`` 软件包里）。

.. note:: ``smartmontools`` 7.0 和后续版本可提供
   NVMe 透传（ passthrough ）统计信息和 JSON 输出。


监视器和 OSD 蜗居
-----------------
.. Co-resident Monitors/OSDs

虽然监视器是相对轻量的进程，但它们与 OSD
共存于同一台主机时，仍然可能产生性能问题。
监视器会发出大量 ``fsync()`` 系统调用，这会妨碍别的工作负载，
特别是监视器和 OSD 共享驱动器时，性能问题更严重。
另外，如果监视器基于较老的内核（低于3.0）、
或者不支持 ``syncfs(2)`` 系统调用的内核运行，
那么，同一主机上运行着的多个 OSD 会发出大量提交，\
导致相互拖累。这种情况有时会导致“爆发写”。


进程蜗居
--------
.. Co-resident Processes

与 OSD 们共存于同一套硬件上、并向 Ceph （例如，基于云的解决方案、
虚拟机）写入数据的进程，可能会导致 OSD 延时明显增大。
正因为如此，所以一般不建议让这样的进程与 OSD 共存。
相反，我们建议对用于 Ceph 的主机做特别优化、
并用另外一些主机跑其它进程。实践证明，
把 Ceph 操作和其他应用程序分开可提高性能、
并简化故障排除和运维。

在同一套硬件上运行共存进程有时叫做“融合（ convergence ）”。
在使用 Ceph 时，只有具备了专业知识并经过深思熟虑后才能进行融合。


日志记录级别
------------
.. Logging Levels

高日志级别会导致性能问题。
操作员有时为追踪某问题提高了日志级别、但结束后忘了调回去。
此时， OSD 们就可能浪费宝贵的系统资源，
向硬盘写入大量没必要的详细日志。如果你想始终保持高日志级别，
可以考虑给默认日志路径挂载个硬盘（比如 ``/var/log/ceph/$cluster-$name.log`` ）。


恢复节流
--------
.. Recovery Throttling

根据你的配置， Ceph 可以降低恢复速度来维持客户端或 OSD 性能，
或者，可以不顾客户端或 OSD 性能而加快恢复速度。
检查下客户端或 OSD 是否正在恢复。


内核版本
--------
.. Kernel Version

检查下你在用的内核版本。较老的内核也许没有移植能提高 Ceph 性能的功能。


与 SyncFS 相关的内核问题
------------------------
.. Kernel Issues with SyncFS

如果你遇到了与 SyncFS 相关的内核问题，试试在一主机上只运行一个 OSD ，
看看性能是否有提升。老内核未必支持有 ``syncfs(2)`` 系统调用的新版 ``glibc`` 。


文件系统问题
------------
.. Filesystem Issues

在 Luminous 版以后，我们建议用 BlueStore 后端部署集群。
运行 Luminous 之前的版本、或由于某些原因还得用\
之前的 FileStore 后端部署 OSD 时，我们推荐 ``XFS`` 。

我们不推荐用 ``Btrfs`` 或 ``ext4`` 。 ``Btrfs`` 有很多诱人的功能，
但文件系统自身的缺陷可能导致性能问题，以及 ENOSPC 伪错误。
我们不建议使用 ``ext4`` 做 OSD 的 FileStore ，\
因为其 ``xattr`` 尺寸限制会破坏我们对长对象名的支持，而这是 RGW 必需的。

详情见\ `文件系统推荐`_\ 。

.. _文件系统推荐: ../configuration/filesystem-recommendations


内存不足
--------
.. Insufficient RAM

我们建议给每个 OSD 守护进程 *最少* 4GB 内存，而且建议在 6-8GB 之上取整。
你也许注意到了，日常操作中 ``ceph-osd`` 进程仅会用其中一小部分。
你也许想用这些空闲内存同时跑一些其他应用、\
或者克扣各节点的内存容量。然而当 OSD 在恢复时，其内存使用会激增，
如果恢复期间内存不够充足， OSD 的性能将明显降低，
而且守护进程们甚至会崩溃或被 Linux 的 ``OOM Killer`` 杀死。


Blocked Requests 或 Slow Requests 问题
--------------------------------------
.. Blocked Requests or Slow Requests

如果某个 ``ceph-osd`` 守护进程对一个请求的响应很慢，
集群日志系统会收到消息，
报告出耗时过于长的那些操作。默认警告阀值是 30 秒，\
可以用 ``osd_op_complaint_time`` 选项来配置。

Ceph 的老旧版本会抱怨 ``old requests``::

    osd.0 192.168.106.220:6800/18813 312 : [WRN] old request osd_op(client.5099.0:790 fatty_26485_object789 [write 0~4096] 2.5e54f643) v4 received at 2012-03-06 15:42:56.054801 currently waiting for sub ops

较新版本的 Ceph 抱怨 ``slow requests``::

    {date} {osd.num} [WRN] 1 slow requests, 1 included below; oldest blocked for > 30.005692 secs
    {date} {osd.num}  [WRN] slow request 30.005692 seconds old, received at {date-time}: osd_op(client.4240.0:8 benchmark_data_ceph-1_39426_object7 [write 0~4194304] 0.69848840) v4 currently waiting for subops from [610]

可能的起因有：

- 快要坏的驱动器（查验一下 ``dmesg`` 输出）；
- 内核文件系统缺陷（查验一下 ``dmesg`` 输出）；
- 集群过载（检查系统负载、 iostat 等等）；
- ``ceph-osd`` 守护进程缺陷。
- OSD 分片配置不是最优（在基于 HDD 的集群上采用 mClock 调度器）

可能的解决方法：

- 从 Ceph 主机去除 VM ；
- 升级内核；
- 升级 Ceph ；
- 重启 OSD 。
- 替换坏的或快要坏的组件；
- 覆盖 OSD 分片配置（在基于 HDD 的集群上采用 mClock 调度器）
    - 参阅 :ref:`mclock-tblshoot-hdd-shard-config` 获取解决方案


慢请求的调试
------------
.. Debugging Slow Requests

运行 ``ceph daemon osd.<id> dump_historic_ops`` 或
``ceph daemon osd.<id> dump_ops_in_flight`` 命令时，
你会看到一堆操作和各个操作过程的一个事件列表，下面简单说明一下。

信使层事件：

- ``header_read``: 信使首次从线路读取到消息的时间；
- ``throttled``: 信使尝试申请内存节流空间、用于把消息读入内存的时间；
- ``all_read``: 信使从线路读取出消息，完成的时间；
- ``dispatched``: 信使把消息发给 OSD 的时间；
- ``initiated``: 等价于 ``header_read`` ，这二者都保留着是历史遗留问题。

OSD 处理 ops （操作）时的事件：

- ``queued_for_pg``: op 已放入队列，等着它自己的 PG 来处理；
- ``reached_pg``: 其 PG 已开始执行这个 op ；
- ``waiting for \*``: 此 op 正等着某些其它工作结束，这样它才能\
  继续下一步（如，一个新 OSDMap ；等着这个对象目标完成洗刷；
  等着相应的 PG 完成互联；所有都在消息内指出了）；
- ``started``: 此 op 已被这个 OSD 接受为应该做的事，
  并且正在进行中；
- ``waiting for subops from``: 此 op 已发送给了副本 OSD 们。

``FileStore`` 事件:

- ``commit_queued_for_journal_write``: 此 op 已递送给了 FileStore 。
- ``write_thread_in_journal_buffer``: 此 op 已经在日志的缓冲中了、
  并等着持久化（等着下一次硬盘写操作）；
- ``journaled_completion_queued``: 此 op 已在硬盘上作了日志、\
  且它的回调已进入队列等着被调用了。

数据已传递给更底层存储之后的 OSD 事件：

- ``op_commit``: 此 op 已被主 OSD 提交
  （即已写入日志）；
- ``op_applied``: 此 op `已经用 write() 写入
  <https://www.freebsd.org/cgi/man.cgi?write(2)>`_
  主 OSD 的后端文件系统（即在内存里已应用，但还没刷入硬盘）；
- ``sub_op_applied``: （类似前面的） ``op_applied`` ，但这是发生在副本上的 subop （子操作）；
- ``sub_op_committed``: 就是 ``op_commit`` ，但这是发生在副本上的 subop （仅发生在 EC 存储池中）；
- ``sub_op_commit_rec/sub_op_apply_rec from <X>``: 主 OSD 得知\
  上述消息后会做这些标记，只是为某个特定副本（即 ``<X>`` ）做标记；
- ``commit_sent``: 我们已经把回信发给客户端（或主 OSD ，来自子操作的）了。

这些事件里，有很多看起来显得多余，但都是代码里穿越重要边界的\
（像数据加锁后传入新线程）。


.. _mclock-tblshoot-hdd-shard-config:

采用 mClock 调度器时的 Slow Requests 或者 Slow Recovery
-------------------------------------------------------
.. Slow Requests or Slow Recovery With mClock Scheduler

.. note:: 此故障排除仅适针对：使用 HDD 、且采用了 mClock 调度器、
   并具有以下 OSD 分片配置： ``osd_op_num_shards_hdd`` = 5 且
   ``osd_op_num_threads_per_shard_hdd`` = 1 。此外，参阅 :ref:`mclock-hdd-cfg` ，
   了解采用 mClock 时需要对默认的 OSD HDD 分片配置做更改的原因。

在启用了 mClock 调度器的、基于硬盘扩展的集群上，有多个 OSD 节点故障的条件下，
可能会报告或观察到以下情况：

- slow requests （请求变慢）: 这也会导致客户端 I/O 性能下降。
- slow background recoveries （后台恢复变慢）: 低于应有的恢复流量。

**故障排除步骤：**

#. 从 OSD 事件中核实 "slow requests" 主要属于
   ``queued_for_pg`` 类型。
#. 在 QoS 对后台恢复服务能有效管理的情况下，
   验证报告的恢复速率是否远低于预期速率。

如果上述任一步骤属实，那就可以采用以下解决方案。
注意，这会造成中断，因为需要重新启动 OSD 。
运行以下命令更改硬盘的默认 OSD 分片配置：

.. prompt:: bash

   ceph config set osd osd_op_num_shards_hdd 1
   ceph config set osd osd_op_num_threads_per_shard_hdd 5

上述配置不会立即生效，并且需要重启此环境中的 OSD 。
为使这一过程的破坏性最小，可谨慎地交错重启 OSD 。


.. _rados_tshooting_flapping_osd:

状态抖动的 OSD
==============
.. Flapping OSDs

"状态抖动"（打摆子）是指 OSD 的一种情形，快速地、\
接二连三地被反复标记为 ``up`` 而后 ``down`` 的现象。
本节将解释如何识别抖动，以及如何缓解抖动。

OSD 们互联和检查心跳时会优先选集群网（后端），
详情见\ `监视器与 OSD 的交互`_\ 。

上游的 Ceph 社区传统上一直建议拆分 *公共* (前端）和 *私密*
（集群、后端、复制）网络，因为这样有如下好处：

#. 隔离 (1) 心跳流量、复制流量、恢复流量（私网）与
   (2) 客户端流量、 OSD 间流量、监视器流量（公网）。
   这样有助于防范另一边发生的 DoS 攻击，
   但同样也可能导致连锁故障。

#. 公网和私网流量都会有额外吞吐量。

过去，常用的网络速率是以 100Mb/s 和 1Gb/s 来衡量的，
这样的隔离通常是必要的。
但如今用上了 10Gb/s 、 40Gb/s 和 25/50/100Gb/s 的网络，
以往对于容量的关注一般就减少甚至不存在了。
比如，你的 OSD 节点都有 2 个网口，
一个接公网、另一个接私网，就意味着没有线路冗余。
这样降低了你在不影响集群和客户端的前提下、
容忍网络维护和故障的能力。试想一下，
如果两条链路都用于公网：
采用端口绑定（ LACP ）或等价路由（如 FRR ），
有利于增加的吞吐量空间、容错性、并减少 OSD 抖动。

当私网（甚至是单条主机链路）发生故障、或降级，同时，\
公网却正常运行， OSD 就可能无法恰当地处理这种情况。
在这种情况下， OSD 们会用公网向监视器报告彼此 ``down`` 了、\
同时把它自己标记为 ``up`` 的。然后监视器们就\
再次向公网散播更新过的集群运行图，
此图把受影响的 OSD 们标记成了 `down` ，\
这些 OSD 们又向监视器反馈“我还没死呢！”，如此循环往复。\
我们把这种情形称为 "状态抖动" （或打摆子， flapping ），
而且它很难隔离和矫正。如果没有私网，就避免了这种讨厌的动态：
OSD 们通常要么是 ``up`` 要么是 ``down`` ，没有抖动。

如果有东西导致 OSD 状态抖动（反复地被标记为 ``down`` ，
而后又 ``up`` ），你可以临时冻结它们的状态，
强制监视器们暂停打摆子：

.. prompt:: bash

   ceph osd set noup      # prevent OSDs from getting marked up
   ceph osd set nodown    # prevent OSDs from getting marked down

这些标记会记录在 osdmap 里：

.. prompt:: bash

   ceph osd dump | grep flags

::

   flags no-up,no-down

下列命令可清除标记：

.. prompt:: bash

   ceph osd unset noup
   ceph osd unset nodown

还支持其它两个标记 ``noin`` 和 ``noout`` ，
它们分别可防止正在启动的 OSD 被标记为 ``in`` （已分配数据）、
或防止 OSD 被误标记为 ``out``
（不管 ``mon osd down out interval`` 当前的值是什么）。

.. note:: ``noup`` 、 ``noout`` 和 ``nodown`` 从某种意义上说是临时的，
   一旦标记清除了，它们被阻塞的动作短时间内就会发生；\
   相反， ``noin`` 标记是阻止 OSD 启动后进入集群
   （被标记为 ``in`` ），但其它所有设置标记前就已经启动了的\
   守护进程们都维持原样。

.. note:: 打摆子的起因和影响有时候可以通过仔细地调整
   ``mon_osd_down_out_subtree_limit`` 、
   ``mon_osd_reporter_subtree_level`` 、和
   ``mon_osd_min_down_reporters`` 来消除。
   最优配置的偏离取决于集群大小、拓扑结构、和在用的 Ceph 版本。
   它们之间的相互影响很微妙，超出了本文档的范围。


.. _iostat: https://en.wikipedia.org/wiki/Iostat
.. _Ceph 日志记录和调试: ../../configuration/ceph-conf#ceph-logging-and-debugging
.. _日志记录和调试: ../log-and-debug
.. _调试和日志记录: ../debug
.. _监视器与 OSD 的交互: ../../configuration/mon-osd-interaction
.. _监视器配置参考: ../../configuration/mon-config-ref
.. _监控 OSD: ../../operations/monitoring-osd-pg

.. _monitoring OSDs: ../../operations/monitoring-osd-pg/#monitoring-osds

.. _订阅 ceph-devel 邮件列表: mailto:majordomo@vger.kernel.org?body=subscribe+ceph-devel
.. _退订 ceph-devel 邮件列表: mailto:majordomo@vger.kernel.org?body=unsubscribe+ceph-devel
.. _订阅 ceph-users 邮件列表: mailto:ceph-users-join@lists.ceph.com
.. _退订 ceph-users 邮件列表: mailto:ceph-users-leave@lists.ceph.com
.. _操作系统推荐: ../../../start/os-recommendations
.. _ceph-devel: ceph-devel@vger.kernel.org
