.. _hardware-recommendations:

==========
 硬件推荐
==========

Ceph 为普通硬件设计，这样可使建造、
维护 PB 级数据集群的费用相对低廉。
规划集群硬件时，需要均衡几方面的因素，
包括故障域、成本和性能。
硬件规划还包括，恰当地分布使用
Ceph 集群的 Ceph 守护进程和其他进程。
通常，我们推荐在一台机器上只运行一种类型的 Ceph 守护进程。
我们推荐把使用数据集群的进程（如 OpenStack 、 OpenNebula 、
CloudStack 、 Kubernetes 等）安装在别的机器上。

一个 Ceph 集群的需求与另一个集群的需求不尽相同，
下面是一些一般准则。

.. tip:: 也可以参考 `Ceph 博客文章`_\ 。


CPU
===

Ceph 元数据服务器（ MDS ）对 CPU 敏感，它们是单线程的，
而且时钟速率（频率 GHz ）越高运行越高效。
MDS 服务器不需要很多 CPU 核心，除非这些服务器还托管着别的服务，
比如用于 CephFS 元数据存储池的 SSD OSD 。
OSD 节点需要有足够的处理能力来运行 :term:`RADOS` 服务、
用 :term:`CRUSH` 计算数据存放位置、复制数据、维护它自己的集群运行图副本，

在 Ceph 的早期版本中，我们根据每个 OSD 的核心数来推荐硬件，
但这个\ *每个 OSD 的核心数*\ 指标与每个 IOP 的周期数和\
每个 OSD 的 IOPS 数指标相比，不再那样有用了。
例如，使用 NVMe OSD 驱动器时，在真正的集群上 Ceph 可以轻松用掉五六个核心，
而在隔离的单个 OSD 上最多可以利用约十四个核心。
因此，每个 OSD 的核心数这个指标不再像以前那样有效。
选择硬件时，请根据每个核心的 IOPS 进行选择。

.. tip:: 当我们谈到 CPU *核心*\ 时，我们指的是启用超线程功能后的\ *线程*\ 。
   超线程通常有利于 Ceph 服务器。

监视器节点和管理器节点对 CPU 的需求不高，只需要普通的处理器。
如果您的主机除了运行 Ceph 守护进程外还将运行 CPU 密集型进程，
请确保有足够的算力同时运行 CPU 密集型进程和 Ceph 守护进程
（比如 OpenStack Nova 就是一种 CPU 密集型进程）。
我们建议您在单独的主机上运行 Ceph 以外的 CPU 密集型进程
（就是在监视器和管理器节点的以外的主机上），以避免资源竞争。
如果集群部署了 Ceph 对象网关，那么 RGW 守护进程可以与监视器和管理器服务\
共同驻留，前提是这些节点有足够的资源。


RAM 内存
========
.. RAM

一般来说，内存越多越好。对于一般的集群，
监视器、管理器节点有 64GB 就够用了；对于有数百个 OSD 的较大集群来说，
128GB 差不多够。

.. tip:: 在谈到内存和存储需求时，
   我们通常描述的是特定类型的单个守护进程的需求。
   因此，一台服务器作为一个整体，它的最低需求要累加：
   本机上运行的所有守护进程、以及日志和其他操作系统组件所需的资源。
   记着，服务器对内存和存储的需求在启动时、组件故障或新增组件、
   还有集群重新平衡时会更大。换句话说，
   预留的余量应该大于你在小型初始集群的平静期见过它占用的空间。

用 :confval:`osd_memory_target` 选项配置 BlueStore OSD 的内存限度，
默认值是 4GB 。考虑到操作系统占用的、管理任务（像监控和指标测量）、
还有恢复时内存消耗会增加：建议\ *每个 BlueStore OSD* 配备 8GB 左右。


监视器和管理器（ ceph-mon 和 ceph-mgr ）
----------------------------------------
.. Monitors and managers (ceph-mon and ceph-mgr)

监视器和管理器守护进程的内存使用量随着集群的规模变化。
需要注意的是，在启动时、拓扑结构变化时、和恢复期间，
这些守护进程需要的内存量高于稳定状态下的，所以应该按峰值使用量规划。
对于很小的集群， 32GB 就够了；
对于比较大的集群，比如 300 个 OSD 以内，应该配备 64GB ；
对于 OSD 更多的集群（或者会扩容到这么大的），应该配备 128GB 。
你可能还得调整下面的选项：

* :confval:`mon_osd_cache_size`
* :confval:`rocksdb_cache_size`


元数据服务器（ ceph-mds ）
--------------------------
.. Metadata servers (ceph-mds)

CephFS 元数据守护进程的内存使用量取决于给它配置的缓存有多大。
对于大多数系统我们都建议 1GB 起步，见 :confval:`mds_cache_memory_limit` 。


内存
====
.. Memory

Bluestore 用它自己的内存缓存数据，而不是靠操作系统的页缓存机制。
在 Bluestore 里，能用 :confval:`osd_memory_target` 配置选项来\
调整 OSD 可以消耗的内存量。

- 不建议把 :confval:`osd_memory_target` 设置到 2GB 以下，
  Ceph 不一定能把内存消耗量压到 2GB 以下，
  还会导致非常差的性能。

- 把内存量设置为 2GB - 4GB 一般可以用，
  但性能提不起来，因为在 IO 操作时元数据得从磁盘读取，
  除非活跃的数据集相对较小。

- 4GB 是 :confval:`osd_memory_target` 选项当前的默认值，
  这个默认值适用于典型的应用场景，
  主要是为了均衡内存成本和 OSD 性能。

- 把 :confval:`osd_memory_target` 设置成 4GB 以上，
  在有很多（小）对象时、或者处理巨大的
  （256GB/OSD 或更多）数据集时可以提升性能。
  特别是在高速的 NVMe OSD 上更是如此。

.. important:: OSD 内存的自动调节机制是“尽力而为”。
   当 OSD 释放内存、让内核回收时，
   没法保证内核什么时候才会真正回收那些空闲内存。
   使用较老版本的 Ceph 时尤其是这样，
   因为内核的透明巨型页会阻止内核回收碎片化的巨型页；
   现在的 Ceph 为了避免此问题，在应用层禁用了透明巨型页，
   即使那样，仍然不能保证内核会立即回收空出来的（ unmapped ）内存。
   OSD 也会时不时地越过它的内存限值。
   我们建议另外再预留大约 20% 的内存，以防止偶尔爆发、或\
   内核回收空闲内存延期而导致 OSD 们被
   OOM （**O**\ut **O**\f **M**\emory, 内存耗尽）清理掉。
   总之，这个值比实际需要的可多可少，
   取决于系统的实际配置。

.. tip:: 对于现代操作系统，不建议给它配置交换区\
   来为守护进程提供额外的虚拟内存。
   这样做可能会导致更差的性能，而 Ceph 集群可能宁愿守护进程崩溃，
   也不要慢的像蜗牛一样。

使用旧的 FileStore 后端时，数据缓存会使用内核的页缓存机制，
所以通常不需要调整。而且在使用旧的 FileStore 后端时，
OSD 的内存消耗量一般和系统内每个守护进程持有的 PG 数有关。


数据存储
========
.. Data Storage

要谨慎地规划数据存储配置，
因为对成本和性能的权衡会导致结果会有很大起伏。\
来自操作系统的并行操作和到单个硬盘的多个守护进程并发读、
写请求操作会极大地降低性能。

OSD 需要大量硬盘空间来存储 RADOS 数据。
我们建议硬盘容量至少为 1 TB。小于 1 TB 的 OSD 硬盘\
会将其容量的很大一部分用于元数据，
而小于 100 GB 的硬盘则完全没效益。

即便要用 HDD 存储海量的 OSD 数据，
我们仍然\ *强烈建议*\ 至少为 Ceph 监视器、 Ceph 管理器主机、
CephFS 元数据服务器的元数据存储池和
Ceph 对象网关 (RGW) 索引存储池配置（企业级）固态硬盘。

要榨出 Ceph 的最佳性能，
最好在单独的硬盘上安装以下：

* 操作系统
* OSD 数据
* BlueStore WAL+DB

关于如何在 Ceph 集群中有效混合使用快速驱动器和慢速驱动器的详细信息，
参阅《 Bluestore 配置参考》中的 `block 和 block.db`_ 部分。


硬盘驱动器
----------
.. Hard Disk Drives

请仔细考虑大硬盘的每 GB 成本优势。
建议用 GB 数除以硬盘价格来计算每 GB 成本，
因为更大的硬盘通常会对每 GB 成本有比较大影响，
例如，单价为 $75 的 1TB 硬盘其每 GB 价格为 $0.07 （ $75/1024=0.0732 ），
而单价为 $150 的 3TB 硬盘其每 GB 价格为 $0.05 （ $150/3072=0.0488 ），
这样，使用 1TB 硬盘会导致每 GB 价格增加 40% ，
从而导致你的集群经济性大幅降低。

.. tip:: 在单个 SAS / SATA 硬盘上运行多个 OSD
   **不明智**\ ！

.. tip:: 在一个驱动器上同时运行 OSD 和\
   监视器、管理器或 MDS 数据也\ **不明智**\ ！

.. tip:: 对于机械硬盘， SATA 和 SAS 接口\
   在容量较大时会逐渐成为瓶颈。
   另请参阅\ `存储网络行业协会的总拥有成本计算器`_ 。

存储驱动器受限于寻道时间、访问时间、
读写时间、还有总吞吐量，
这些物理限制影响着系统的整体性能，特别是在系统恢复期间。
因此我们推荐独立的（最好是带镜像的）驱动器用于安装操作系统和软件，
另外每个 OSD 守护进程独占一个驱动器。
大多数 “slow OSD” 问题（如果不是硬件故障造成的）\
的起因都是在同一个硬盘上同时运行了操作系统和多个 OSD 。
还要注意，现今的 22TB HDD 和 10 年前的 3TB 硬盘使用着同样的 SATA 接口，
这意味着同样的接口承担的数据量却是以前的 7 倍。
因此，在给 OSD 配备 HDD 时，大于 8TB 的驱动器最好\
只用来存储大文件、大对象，这些对性能不太敏感的数据。



固态硬盘
--------
.. Solid State Drives

使用固态硬盘（ SSD ）时可以大大提升 Ceph 性能，
它在增加吞吐量的同时还能降低随机访问时间和读延时。

SSD 和 HDD 相比每 GB 成本高，
但访问时间至少比硬盘快 100 倍。
SSD 可在繁忙的集群中避免热点问题和瓶颈问题，
在全面评估总体拥有成本（ TCO ）时， SSD 可能有更好的经济性。
值得注意的是，对于一定数量的 IOPS ， SSD 的驱动器摊销成本要比 HDD 低得多。
SSD 没有旋转或寻道延时，除了能提高客户端性能外，
还能大幅提高在集群有变动时的响应速度和降低客户端影响，
集群变动包括在 OSD 或监视器添加、删除、
发生故障时进行的重新平衡。

SSD 没有可移动机械部件，
所以不存在和 HDD 一样的局限性。
但 SSD 也有它自身的局限性。在评估 SSD 时，
考察顺序和随机读写性能非常重要。

.. important:: 我们建议发掘 SSD 的用法来提升性能。
   然而在大量投入 SSD 前，
   我们\ **强烈建议**\ 核实 SSD 的性能指标，
   并在测试环境下衡量性能。

相对廉价的 SSD 很诱人，慎用！
为 Ceph 选用 SSD 时，可接受的 IOPS 指标不是唯一要考虑的因素。
廉价的固态硬盘往往不划算：它们可能会出现“悬崖效应”，
即在最初的爆发之后，一旦有限的缓存被填满，
持续性能就会大幅下降。此外，还要考虑耐用性：
每日整盘写入次数（ Drive Writes Per Day, DWPD, 或等价指标）为 0.3
的驱动器可能适用于某些特定场景，比如顺序写入而读取居多的数据；
但不适宜做 Ceph 监视器的存储设备。企业级 SSD 是 Ceph 的最佳选择：
它们几乎都有掉电保护功能（ power loss protection, PLP ），
不会像消费级（桌面台式机）型号那样出现严重的断电现象。

在单个（或镜像的一对） SSD 上同时安装操作系统和
Ceph 监视器/管理器时，建议容量至少为 256GB ，最好是 480GB 。
驱动器建议选用 DWPD 值为 1+ （或等效的 TBW 值， TeraBytes Written）的型号。
不过，在写入载荷固定的情况下，比实际需求大的驱动器会更加耐用，
因为它有实质上更大的周转空间。
我们再次强调，企业级驱动器对于生产环境是最好的，
因为它们有掉电保护功能、与消费级（台式机）相比更好的耐用性，
消费级产品是为低负载、间歇性的使用场景设计的。

对于对象存储而言， SSD 的成本历来过高，
但 QLC SSD 正在缩小差距，以更低的功耗、
更低的冷却成本提供更高的密度。此外，
通过将 WAL+DB 载荷卸载到固态硬盘上， HDD OSD 的写延迟也会得到显著改善。
现实中很多 Ceph OSD 不需要耐用性超过 1 DWPD （又称“读优化的”）的固态硬盘。
3 DWPD 级别的“混合用途（ Mixed-use ）”固态硬盘通常远超这一需求，
而且成本显著升高。

要更好地了解决定存储总成本的各种因素，
可以用\ `存储网络行业协会的总拥有成本计算器`_ 。


分区对齐
~~~~~~~~
.. Partition Alignment

在 Ceph 中使用固态硬盘时，要确保正确对齐分区。
没有正确对齐的分区比正确对齐的分区数据传输速度更慢。
关于正确对齐分区，以及查看如何正确对齐分区的示例命令，
参阅 `Werner Fischer 有关分区对齐的博文`_\ 。


CephFS 元数据分离
~~~~~~~~~~~~~~~~~
.. CephFS Metadata Segregation

提升 CephFS 文件系统性能的一种方法是\
把 CephFS 元数据的存储和文件内容的存储分开。
Ceph 提供了默认的 ``metadata`` 存储池来存储 CephFS 元数据，
所以你不需要手动给 CephFS 元数据创建存储池，但是可以给
CephFS 元数据单独创建一个 CRUSH 运行图分级结构，其中只包含 SSD 存储器。
详情见 :ref:`CRUSH 设备类<crush-map-device-class>`\ 。


控制器
------
.. Controllers

硬盘控制器（ HBA ）对写吞吐量有显著影响，
要谨慎地选择，确保不会产生性能瓶颈。
特别是 RAID 模式（IR）的 HBA 与简单的 JBOD（IT）模式相比，
更可能出现较高延时，而且 RAID SoC 、写缓存、
和备用电池功能还会增加硬件和维护成本。
很多 RAID HBA 可以配置成 IT 模式，"personality" 或 "JBOD mode" ，
以简化它的操作。

您不需要 RoC （带 RAID 功能的） HBA 。 ZFS 或 Linux MD 软件镜像\
可以很好地保证启动卷的耐用性。使用 SAS 或 SATA 数据驱动器时，
放弃 HBA RAID 功能可缩小硬盘和固态硬盘之间的成本差距。
此外，使用 NVMe 固态硬盘时，\ *根本不需要* HBA ，
从整个系统考虑，这将进一步缩小硬盘与固态硬盘的成本差距。
一个高级 RAID HBA 、加上板载高速缓存、再加上备用电池（ BBU 或超级电容器）\
的初始成本即使打折后也很容易超过 1000 美元 ——
加上这个就和使用 SSD 的成本差不多了。
如果购买年度维护合同或延长保修期，
不含 HBA 的系统每年还会少花数百美元。

.. tip:: `Ceph 博客文章`_\ 常常是优秀的 Ceph 性能问题信息源，
   详情见 `Ceph 写吞吐量 1`_ 和 `Ceph 写吞吐量 2`_ 。


压力测试
--------
.. Benchmarking

BlueStore 打开块设备时加了 ``O_DIRECT`` 标记，并且频繁调用 ``fsync`` ，
以确保数据安全地持久化到了媒体介质中。你可以用 ``fio`` 来评测\
一个驱动器的底层写性能。例如， 4kB 随机写性能可以这样衡量：

.. code-block:: console

   # fio --name=/dev/sdX --ioengine=libaio --direct=1 --fsync=1 --readwrite=randwrite --blocksize=4k --runtime=300


写缓存
------
.. Write Caches

企业级 SSD 和 HDD 通常都有掉电保护功能，
它能确保操作时掉电的数据持久性，
并且用多级缓存来加速 direct 或同步写。
这些设备可以在两种缓存模式之间切换——用 fsync 把易失性缓存刷到持久性介质上，
或者同步地写入非易失性缓存中。

这两种模式的切换可以通过在硬件配置里启用（ enabling ）或\
禁用（ disabling ）写（易失性）缓存来实现。易失性缓存启用时，
Linux 会以 write back 模式使用此设备；禁用时以 write through 模式使用。

默认配置（缓存通常都是开启的）未必是最优的，
禁用写缓存时可显著提高 OSD 性能、提高 IOPS 、
并降低提交延时。

因此，我们鼓励用户用 ``fio`` 评测他们的设备，
用前面描述过的方法，并保存好设备的最优缓存配置。

缓存配置可以用 ``hdparm`` 、 ``sdparm`` 、 ``smartctl`` 或\
读取 ``/sys/class/scsi_disk/*/cache_type`` 里的数值来查询，
例如：

.. code-block:: console

    # hdparm -W /dev/sda

    /dev/sda:
     write-caching =  1 (on)

    # sdparm --get WCE /dev/sda
        /dev/sda: ATA       TOSHIBA MG07ACA1  0101
    WCE           1  [cha: y]
    # smartctl -g wcache /dev/sda
    smartctl 7.1 2020-04-05 r5049 [x86_64-linux-4.18.0-305.19.1.el8_4.x86_64] (local build)
    Copyright (C) 2002-19, Bruce Allen, Christian Franke, www.smartmontools.org

    Write cache is:   Enabled

    # cat /sys/class/scsi_disk/0\:0\:0\:0/cache_type
    write back

同样可以用那些工具来禁用写缓存：

.. code-block:: console

    # hdparm -W0 /dev/sda

    /dev/sda:
     setting drive write-caching to 0 (off)
     write-caching =  0 (off)

    # sdparm --clear WCE /dev/sda
        /dev/sda: ATA       TOSHIBA MG07ACA1  0101
    # smartctl -s wcache,off /dev/sda
    smartctl 7.1 2020-04-05 r5049 [x86_64-linux-4.18.0-305.19.1.el8_4.x86_64] (local build)
    Copyright (C) 2002-19, Bruce Allen, Christian Franke, www.smartmontools.org

    === START OF ENABLE/DISABLE COMMANDS SECTION ===
    Write cache disabled

通常，用 ``hdparm`` 、 ``sdparm`` 、或 ``smartctl`` 禁用缓存会\
导致 cache_type 自动切换为 write through 。
如果没有自动切换，你可以按下面的方法直接设置。
（用户需确保设置了 cache_type 的同时还永久保存了设备的缓存模式，
这些配置在重启之前都有效，因为有些驱动器需要在每次重启之后重新设置）：

.. code-block:: console

    # echo "write through" > /sys/class/scsi_disk/0\:0\:0\:0/cache_type

    # hdparm -W /dev/sda

    /dev/sda:
     write-caching =  0 (off)

.. tip:: 这条 udev 规则（在 CentOS 8 上已经测试过）会把\
   所有 SATA/SAS 设备的 cache_type 设置为 write through ：

  .. code-block:: console

    # cat /etc/udev/rules.d/99-ceph-write-through.rules
    ACTION=="add", SUBSYSTEM=="scsi_disk", ATTR{cache_type}:="write through"

.. tip:: 这条 udev 规则（在 CentOS 7 上已经测试过）会把\
   所有 SATA/SAS 设备的 cache_type 设置为 write through ：

  .. code-block:: console

    # cat /etc/udev/rules.d/99-ceph-write-through-el7.rules
    ACTION=="add", SUBSYSTEM=="scsi_disk", RUN+="/bin/sh -c 'echo write through > /sys/class/scsi_disk/$kernel/cache_type'"

.. tip:: ``sdparm`` 工具可以用于一次性查看、更改\
   多个设备的易失性写缓存：

  .. code-block:: console

    # sdparm --get WCE /dev/sd*
        /dev/sda: ATA       TOSHIBA MG07ACA1  0101
    WCE           0  [cha: y]
        /dev/sdb: ATA       TOSHIBA MG07ACA1  0101
    WCE           0  [cha: y]
    # sdparm --clear WCE /dev/sd*
        /dev/sda: ATA       TOSHIBA MG07ACA1  0101
        /dev/sdb: ATA       TOSHIBA MG07ACA1  0101


其他注意事项
------------
.. Additional Considerations

Ceph 运维人员通常会给每台主机配置多个 OSD ，
但要确保 OSD 硬盘总吞吐量不超过为客户端提供读写服务所需的网络带宽；
还要考虑每台主机占集群总容量的百分比，
如果一台主机所占百分比太大而它挂了，
就可能导致恢复时出现超过 ``full ratio`` 的问题，
进而导致 Ceph 中止运行以防数据丢失。

如果每台主机运行多个 OSD ，也得保证内核是最新的。
参阅\ `操作系统推荐`_\ 里关于 ``glibc`` 和 ``syncfs(2)`` 的部分，
确保每台主机在运行多个 OSD 的时候硬件性能能达到期望值。


网络
====
.. Networks

数据中心的网络至少要 10Gbps 以上，包括 Ceph 主机之间的、\
客户端和 Ceph 集群之间的。强烈建议\
网络链路采用 active/active 绑定并连接到不同的交换机，
这样在增加吞吐量的同时，还能为网络故障和日常维护提供冗余。
注意，绑定的散列策略（ hash policy ）会在链路间分配流量。

速度
----
.. Speed

通过 1Gbps 的网络复制 1TB 数据需要 3 小时，
而 10TB 需要 30 小时！
相比之下，如果使用 10Gbps 复制时间可分别缩减到 20 分钟和 1 小时。

注意，一个 40 Gb/s 的网络链路实际上是四个并行的 10 Gb/s 通道，
而一个 100 Gb/s 的网络链路实际上是四个并行的 25 Gb/s 通道。
因此，与 40 Gb/s 网络相比， 25 Gb/s 网络上的单个数据包的延迟时间稍短，
这个事有点违背直觉。

成本
----
.. Cost

Ceph 集群规模越大， OSD 故障越常见；
PG 从降级（ ``degraded`` ）状态恢复到 ``active + clean`` 状态\
的速度越快越好。应该注意，
快速恢复可最大限度减少多次、重叠的故障，
它们有可能导致数据暂时不能访问甚至丢失。当然，
建设网络时，还得均衡造价和性能。

有些部署工具使用了 VLAN 来提高硬件和网络线路的可管理性。
VLAN 使用 802.1q 协议，还需要采用支持 VLAN 功能的网卡和交换机，
增加的硬件成本可以用节省的运营（网络安装、维护）成本抵消。
使用 VLAN 来处理集群和计算栈\
（如 OpenStack 、 CloudStack 等等）之间的 VM 流量时，
采用 10G 或更高速率的以太网更合算；到 2022 年，
40Gb/s 或 25/50/100 Gb/s 的网络在生产集群上很普遍。

机架交换机（Top-of-rack, TOR ）到核心/骨干交换机或路由器的上联链路\
也要做到高速、有冗余，带宽应该更大，通常 40Gbp/s 以上。

底板管理控制器（ BMC ）
-----------------------
.. Baseboard Management Controller (BMC)

服务器硬件应配置底板管理控制器（ Baseboard Management Controller, BMC ），
常见的有 iDRAC （戴尔）、 CIMC (Cisco UCS) 、 iLO (HPE) 。
管理和部署工具也普遍会使用 BMC ，尤其是通过 IPMI 或 Redfish ，
所以请权衡带外网络在安全和管理方面的成本/效益，
此程序管理着 SSH 访问、 VM 映像上传、
操作系统映像安装、管理通道、等等，这些会显著增加网络负载。
运营多张网络看似夸张（过分小心），但是每条流量路径都代表着\
潜在容量、吞吐量和/或性能瓶颈，
部署一个大型数据集群前，这些都要仔细考虑。

此外，截至 2023 年， BMC 很少有速度超过 1 Gb/s 的网络连接，
所以，专门用于 BMC 管理流量的廉价 1 Gb/s 交换机\
可以节省昂贵的、更快的主机交换机端口，从而降低成本。


故障域
======
.. Failure Domains

故障域可以认为是有组件丢失，导致不能访问一个或多个 OSD ，
或者是其它的 Ceph 守护进程。可以是主机上停止的进程、
存储驱动器故障、操作系统崩溃、有问题的网卡、损坏的电源、
断网、断电等等。规划硬件需求时，
不得不找准平衡点：把太多责任划给少数几个故障域风险太大，
隔离每个潜在故障域又会增加成本。


最低硬件推荐
============
.. Minimum Hardware Recommendations

Ceph 可以运行在廉价的普通硬件上，小型生产集群\
和开发集群可以在普通硬件上运行。
如上所述：当我们谈到 CPU *核心*\ 时，我们指的是\
启用超线程（ HT ）后的\ *线程*\ 。每个现代物理 x64 CPU 核心\
通常会提供两个逻辑 CPU 线程；其他 CPU 架构可能有所不同。

注意，影响资源选择的因素很多。
一种用途所需的最低资源对于另一种用途来说不一定够。
在装有 VirtualBox 的笔记本电脑或三台树莓派（Raspberry PI）上\
建立只有一个 OSD 的沙盒集群所需的资源要少于\
为五千个 RBD 客户端提供服务的一千个 OSD 的生产部署所需的资源。
经典的 Fisher Price PXL 2000 可以捕捉视频， IMAX 或 RED 摄像机也是如此，
但我们不能指望前者可以完成后者的工作。
因此我们特别强调：
在生产环境下使用企业级存储器至关重要。

有关生产集群资源规划的其他见解，
请参阅上文和本文档的其他部分。

+--------------+----------------+-----------------------------------------+
|  进程        | 规范           | 最低标准和建议标准                      |
+==============+================+=========================================+
| ``ceph-osd`` | 处理器         | - 最低 1 核，建议 2 核                  |
|              |                | - 每 200-500 MB/s 一个核心              |
|              |                | - 每 1000-3000 IOPS 一个核心            |
|              |                |                                         |
|              |                | * Results are before replication.       |
|              |                | * Results may vary across CPU and drive |
|              |                |   models and Ceph configuration:        |
|              |                |   (erasure coding, compression, etc)    |
|              |                | * ARM processors specifically may       |
|              |                |   require more cores for performance.   |
|              |                | * SSD OSDs, especially NVMe, will       |
|              |                |   benefit from additional cores per OSD.|
|              |                | * Actual performance depends on many    |
|              |                |   factors including drives, net, and    |
|              |                |   client throughput and latency.        |
|              |                |   Benchmarking is highly recommended.   |
|              +----------------+-----------------------------------------+
|              | RAM            | - 4GB+ per daemon (more is better)      |
|              |                | - 2-4GB often functions (may be slow)   |
|              |                | - Less than 2GB not recommended         |
|              +----------------+-----------------------------------------+
|              | 存储驱动器     |  1x storage drive per daemon            |
|              +----------------+-----------------------------------------+
|              | DB/WAL         |  1x SSD partion per HDD OSD             |
|              | (可选的)       |  4-5x HDD OSDs per DB/WAL SATA SSD      |
|              |                |  <= 10 HDD OSDss per DB/WAL NVMe SSD    |
|              +----------------+-----------------------------------------+
|              | 网络           |  1x 1Gb/s (bonded 10+ Gb/s recommended) |
+--------------+----------------+-----------------------------------------+
| ``ceph-mon`` | 处理器         | - 2 cores minimum                       |
|              +----------------+-----------------------------------------+
|              | RAM            |  5GB+ per daemon (large / production    |
|              |                |  clusters need more)                    |
|              +----------------+-----------------------------------------+
|              | 存储           |  每个守护进程 100 GB ，建议用 SSD       |
|              +----------------+-----------------------------------------+
|              | 网络           |  1x 1Gb/s (10+ Gb/s recommended)        |
+--------------+----------------+-----------------------------------------+
| ``ceph-mds`` | 处理器         | - 2 cores minimum                       |
|              +----------------+-----------------------------------------+
|              | RAM            |  每个守护进程 2GB+ ，生产环境配更多     |
|              +----------------+-----------------------------------------+
|              | 磁盘空间       |  1 GB per daemon                        |
|              +----------------+-----------------------------------------+
|              | 网络           |  1x 1Gb/s (10+ Gb/s recommended)        |
+--------------+----------------+-----------------------------------------+

.. tip:: 如果在只有一块硬盘的机器上运行一个 OSD 节点，
   要把数据和操作系统分别放到不同分区；
   我们建议操作系统和 OSD 存储\
   分别使用不同的驱动器。



.. _block 和 block.db: https://docs.ceph.com/en/latest/rados/configuration/bluestore-config-ref/#block-and-block-db
.. _Ceph 博客文章: https://ceph.com/community/blog/
.. _Ceph 写吞吐量 1: http://ceph.com/community/ceph-performance-part-1-disk-controller-write-throughput/
.. _Ceph 写吞吐量 2: http://ceph.com/community/ceph-performance-part-2-write-throughput-without-ssd-journals/
.. _给存储池指定 OSD: ../../rados/operations/crush-map#placing-different-pools-on-different-osds
.. _操作系统推荐: ../os-recommendations
.. _存储网络行业协会的总拥有成本计算器: https://www.snia.org/forums/cmsi/programs/TCOcalc
.. _Werner Fischer 有关分区对齐的博文: https://www.thomas-krenn.com/en/wiki/Partition_Alignment_detailed_explanation
