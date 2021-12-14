.. _hardware-recommendations:

==========
 硬件推荐
==========

Ceph 为普通硬件设计，这可使构建、
维护 PB 级数据集群的费用相对低廉。
规划集群硬件时，需要均衡几方面的因素，
包括区域失效和潜在的性能问题。
硬件规划要包含把使用 Ceph 集群的 Ceph 守护进程和其他进程恰当分布。
通常，我们推荐在一台机器上只运行一种类型的守护进程。
我们推荐把使用数据集群的进程
（如 OpenStack 、 CloudStack 等）
安装在别的机器上。


.. tip:: 另请参考 `Ceph 博客文章`_\ 。


CPU
===

Ceph 元数据服务器对 CPU 敏感，所以它们应该有足够的处理能力
（如 4 核或更强悍的 CPU ），而且时钟速率（频率 GHz ）越高越有利。
Ceph 的 OSD 运行着 :term:`RADOS` 服务、
用 :term:`CRUSH` 计算数据存放位置、复制数据、维护它自己的集群运行图副本，
因此， OSD 需要一定的处理能力；
需求随应用场景不同差异巨大，对于轻量级的、归档应用场景，
需要为每个 OSD 配备最少一个核心，
高负载场景（比如为 VM 提供 RBD 卷）需要为每个 OSD 配备两个核心。
监视器、管理器对 CPU 要求不高，普通处理器就可以。
还得考虑机器以后是否还会运行 Ceph 守护进程以外的 CPU 密集型任务。
例如，如果服务器以后要运行用于计算的虚拟机（如 OpenStack Nova ），
你就要确保给 Ceph 守护进程们保留了足够的处理能力，
我们建议在单独的机器上运行 CPU 密集型任务，
以避免资源竞争。


RAM 内存
========
.. RAM

一般来说，内存越多越好。对于一般的集群，
监视器、管理器节点有 64GB 就够用了；对于有数百个 OSD 的较大集群来说，
128GB 差不多够。 BlueStore OSD 默认的内存限度是 4GB ，
考虑到操作系统占用的、管理任务（像监控和指标测量）、
还有恢复时内存消耗会增加：建议每个 BlueStore OSD 配备 8GB 左右。

监视器和管理器（ ceph-mon 和 ceph-mgr ）
----------------------------------------
.. Monitors and managers (ceph-mon and ceph-mgr)

监视器和管理器守护进程的内存使用量通常和集群的规模成正比。
需要注意的是，在启动时、拓扑结构变化时、和恢复期间，
这些守护进程需要的内存量高于稳定状态下的，所以应该按峰值使用量规划。
对于很小的集群， 32GB 就够了；
对于比较大的集群，比如 300 个 OSD 以内，应该配备 64GB ；
对于 OSD 更多的集群（或者会扩容到这么大的），应该配备 128GB 。
你可能还得调整配置，比如 ``mon_osd_cache_size`` 或
``rocksdb_cache_size`` ，应该仔细研究后再动手。

元数据服务器（ ceph-mds ）
--------------------------
.. Metadata servers (ceph-mds)

元数据守护进程的内存使用量取决于给它配置的缓存有多大。
对于大多数系统我们都建议 1GB 起步，
见 :confval:`mds_cache_memory` 。

OSDs (ceph-osd)
---------------


内存
====
.. Memory

Bluestore 用它自己的内存缓存数据，而不是靠操作系统的页缓存机制。
在 bluestore 里，能用 :confval:`osd_memory_target` 配置选项来\
调整 OSD 可以消耗的内存量。

- 把 osd_memory_target 设置到 2GB 以下是非常不推荐的，
  它没法保持那么低的内存使用量，还会导致非常差的性能。

- 把内存量设置为 2GB - 4GB 一般可以用，但性能提不起来，
  因为在 IO 操作时元数据得从磁盘读取，除非活跃的数据集相当较小。

- 4GB 是 osd_memory_target 选项的当前默认值，设置成这样后，
  一般的使用场景下都可以均衡内存需求量和 OSD 性能。

- 把 osd_memory_target 设置成 4GB 以上，在有很多（小）对象时、
  或者处理巨大的（256GB/OSD 或更多）数据集时可以提升性能。

.. important:: OSD 内存的自动调节机制是“尽力而为”。
   当 OSD 释放内存、让内核回收时，
   没法保证内核什么时候才会真正回收那些空闲内存。
   使用较老版本的 Ceph 时尤其是这样，
   因为内核的透明巨型页会阻止内核回收碎片化的巨型页；
   现在的 Ceph 为了避免此问题，在应用层禁用了透明巨型页，
   即使那样，仍然不能保证内核会立即回收空出来的（ unmapped ）内存。
   OSD 也会时不时地越过它的内存限值。
   我们建议另外再预留大约 20% 的内存，以防止偶尔爆发、或\
   内核回收空闲内存延期而导致 OSD 们被 OOM 清理掉。
   总之，这个值比实际需要的可多可少，
   取决于系统的实际配置。

使用旧的 FileStore 后端时，数据缓存会使用内核的页缓存机制，
所以通常不需要调整，而且 OSD 的内存消耗量一般和系统内\
每个守护进程持有的 PG 数有关。



数据存储
========
.. Data Storage

要谨慎地规划数据存储配置，因为其间涉及明显的成本和性能折衷。\
来自操作系统的并行操作和到单个硬盘的多个守护进程并发读、
写请求操作会极大地降低性能。

硬盘驱动器
----------
.. Hard Disk Drives

OSD 应该有足够的空间用于存储对象数据。
考虑到大硬盘的每 GB 成本，
我们建议用容量大于 1TB 的硬盘。
建议用 GB 数除以硬盘价格来计算每 GB 成本，
因为较大的硬盘通常会对每 GB 成本有较大影响，
例如，单价为 $75 的 1TB 硬盘其每 GB 价格为 $0.07 （ $75/1024=0.0732 ），
而单价为 $150 的 3TB 硬盘其每 GB 价格为 $0.05 （ $150/3072=0.0488 ），
这样，使用 1TB 硬盘会增加 40% 的每 GB 价格，
使得你集群的经济性较低。

.. tip:: 在单个 SAS / SATA 硬盘上运行多个 OSD **不明智**\ ！
   而对于 NVMe 驱动器，
   分割到两个或更多 OSD 可以提高性能。

.. tip:: 在一个驱动器上同时运行 OSD 和\
   监视器或元数据服务器也\ **不明智**\ ！

存储驱动器受限于寻道时间、访问时间、读写时间、还有总吞吐量，
这些物理局限性影响着整体系统性能，尤其在系统恢复期间。
因此我们推荐独立的（最好是镜像的）驱动器用于安装操作系统和软件，
另外每个 OSD 守护进程独占一个驱动器（不包括前述的 NVMe ）。
大多数 “slow OSD” 问题的起因都是在相同的硬盘上运行了操作系统、多个 OSD 、
和/或多个日志文件。对小型集群来说，
鉴于解决性能问题的成本差不多会超过另外增加磁盘驱动器，
你应该在规划设计时就避免增加 OSD 存储驱动器的负担来优化集群。

Ceph 允许你在每块硬盘驱动器上运行多个 OSD ，
但这会导致资源竞争并降低总体吞吐量。

固态硬盘
--------
.. Solid State Drives

一种提升性能的方法是使用固态硬盘（ SSD ）
来降低随机访问时间和读延时，同时增加吞吐量。
SSD 和硬盘相比每 GB 成本通常要高 10 倍以上，
但访问时间至少比硬盘快 100 倍。

SSD 没有可移动机械部件，
所以不存在和硬盘一样的局限性。
但 SSD 也有局限性，评估 SSD 时，
顺序读写性能很重要。

.. important:: 我们建议发掘 SSD 的用法来提升性能。
   然而在大量投入 SSD 前，
   我们\ **强烈建议**\ 核实 SSD 的性能指标，
   并在测试环境下衡量性能。

相对廉价的 SSD 很诱人，慎用！
可接受的 IOPS 指标对选择用于 Ceph 的 SSD 还不够。

历史上， SSD 用于对象存储曾经成本高昂，
尽管 QLC 驱动器的引进缩小的差距。基于 HDD 的 OSD
把 WAL+DB 载荷挪到 SSD 上之后可以看到明显的性能提升。

提升 CephFS 文件系统性能的一种方法是从 CephFS 文件内容里分离出元数据。
Ceph 提供了默认的 ``metadata`` 存储池来存储 CephFS 元数据，
所以你不需要给 CephFS 元数据创建存储池，
但是可以给它创建一个仅指向某主机 SSD 的 CRUSH 运行图。
详情见 :ref:`CRUSH 设备类<crush-map-device-class>`\ 。


控制器
------
.. Controllers

硬盘控制器（ HBA ）对写吞吐量有显著影响，
要谨慎地选择，确保不会产生性能瓶颈。
特别是 RAID 模式（IR）的 HBA 与简单的 JBOD（IT）模式相比，
更可能出现较高延时，而且 RAID SoC 、写缓存、
和备用电池功能还会增加硬件和维护代价。
有些 RAID HBA 可以“个性化”配置成 IT 模式。

.. tip:: `Ceph 博客文章`_ 常常是优秀的 Ceph 性能问题信息源，
   见 `Ceph 写吞吐量 1`_ 和 `Ceph 写吞吐量 2`_ 。


压力测试
--------
.. Benchmarking

BlueStore 打开块设备时加了 O_DIRECT 标记，并且频繁调用 fsync ，
以确保数据安全地持久化到了媒体。
你可以用 ``fio`` 来评测一个驱动器的底层写性能。
例如， 4kB 随机写性能可以这样衡量：

.. code-block:: console

  # fio --name=/dev/sdX --ioengine=libaio --direct=1 --fsync=1 --readwrite=randwrite --blocksize=4k --runtime=300

写缓存
------
.. Write Caches

企业级 SSD 和 HDD 通常都有掉电保护功能，它们用多级缓存来加速 direct 或同步写。
这些设备可以在两种缓存模式之间切换——用 fsync 把易失性缓存刷到持久性介质上，
或者同步地写入非易失性缓存中。

这两种模式的切换可以通过在硬件配置里启用（ enabling ）或\
禁用（ disabling ）写（易失性）缓存来实现。易失性缓存启用时， 
Linux 会以 write back 模式使用此设备，禁用时以 write through 模式。

默认配置（缓存通常都是开启的）未必是最优的，
而且禁用写缓存时 OSD 性能会极大地提升，也就是 IOPS 提高、
且 commit_latency 降低。

因此，我们鼓励用户通过 ``fio`` 评测他们的设备，
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

同样可以用那些工具来禁用：

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
（用户们请注意，下次重启之后，设置了 cache_type 的同时\
还会永久保存设备的缓存模式）：

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

你可以在同一主机上运行多个 OSD ，
但要确保 OSD 硬盘总吞吐量不超过为客户端提供读写服务所需的网络带宽；
还要考虑集群在每台主机上所存储的数据占总体的百分比，
如果一台主机所占百分比太大而它挂了，
就可能导致诸如超过 ``full ratio`` 的问题，
此问题会使 Ceph 中止运作以防数据丢失。

如果每台主机运行多个 OSD ，也得保证内核是最新的。
参阅\ `操作系统推荐`_\ 里关于 ``glibc`` 和 ``syncfs(2)`` 的部分，
确保在运行多个 OSD 的时候硬件性能能达到期望值。


网络
====
.. Networks

机架之间至少要配备 10Gbps 以上的网络连接。
通过 1Gbps 的网络复制 1TB 数据需要 3 小时，而 10TB 需要 30 小时！
相比之下，如果使用 10Gbps 复制时间可分别缩减到 20 分钟和 1 小时。
在一个 PB 级集群中， OSD 磁盘失败是常态，而非异常；
在性价比合理的的前提下，系统管理员想让 PG
尽快从 ``degraded`` （降级）状态恢复到 ``active + clean`` 状态。
另外，有些部署工具使用了 VLAN 来提高硬件和网络线路的可管理性。
VLAN 使用 802.1q 协议，还需要采用支持 VLAN 功能的网卡和交换机，
增加的硬件成本可用节省的运营（网络安装、维护）成本抵消。
使用 VLAN 来处理集群和计算栈\
（如 OpenStack 、 CloudStack 等等）之间的 VM 流量时，
采用 10G 或更高速率的以太网更合算；到 2020 年，
40Gb 或 25/50/100 Gb 的网络在生产集群上很普遍。

每个网络的机架路由器到骨干网路由器的带宽应该更大，
通常 40Gbp/s 以上。

服务器硬件应配置底板管理控制器
（ Baseboard Management Controller, BMC ），
管理和部署工具也普遍会使用 BMC ，
尤其是通过 IPMI 或 Redfish ，
所以请权衡带外网络管理的成本/效益，
此程序管理着 SSH 访问、 VM 映像上传、操作系统安装、端口管理、等等，
会徒增网络负载。运营 3 个网络有点夸张了，
但是每条流量路径都表明，部署一个大型数据集群前\
要仔细考虑潜在容量、吞吐量、性能瓶颈。


故障域
======
.. Failure Domains

故障域指任何导致不能访问一个或多个 OSD 的故障，
可以是主机上停止的进程、硬盘故障、操作系统崩溃、
有问题的网卡、损坏的电源、断网、断电等等。
规划硬件需求时，要在多个需求间寻求平衡点，
像付出很多努力减少故障域带来的成本削减、
隔离每个潜在故障域增加的成本。


最低硬件推荐
============
.. Minimum Hardware Recommendations

Ceph 可以运行在廉价的普通硬件上，小型生产集群和开发集群可以在\
一般的硬件上。

+--------------+----------------+-----------------------------------------+
|  进程        | 规范           | 最低建议                                |
+==============+================+=========================================+
| ``ceph-osd`` | 处理器         | - 1 core minimum                        |
|              |                | - 1 core per 200-500 MB/s               |
|              |                | - 1 core per 1000-3000 IOPS             |
|              |                |                                         |
|              |                | * Results are before replication.       |
|              |                | * Results may vary with different       |
|              |                |   CPU models and Ceph features.         |
|              |                |   (erasure coding, compression, etc)    |
|              |                | * ARM processors specifically may       |
|              |                |   require additional cores.             |
|              |                | * Actual performance depends on many    |
|              |                |   factors including drives, net, and    |
|              |                |   client throughput and latency.        |
|              |                |   Benchmarking is highly recommended.   |
|              +----------------+-----------------------------------------+
|              | RAM            | - 4GB+ per daemon (more is better)      |
|              |                | - 2-4GB often functions (may be slow)   |
|              |                | - Less than 2GB not recommended         |
|              +----------------+-----------------------------------------+
|              | 存储卷宗       |  1x storage drive per daemon            |
|              +----------------+-----------------------------------------+
|              | DB/WAL         |  1x SSD partition per daemon (optional) |
|              +----------------+-----------------------------------------+
|              | 网络           |  1x 1GbE+ NICs (10GbE+ recommended)     |
+--------------+----------------+-----------------------------------------+
| ``ceph-mon`` | 处理器         | - 2 cores minimum                       |
|              +----------------+-----------------------------------------+
|              | RAM            |  2-4GB+ per daemon                      |
|              +----------------+-----------------------------------------+
|              | 磁盘空间       |  60 GB per daemon                       |
|              +----------------+-----------------------------------------+
|              | 网络           |  1x 1GbE+ NICs                          |
+--------------+----------------+-----------------------------------------+
| ``ceph-mds`` | 处理器         | - 2 cores minimum                       |
|              +----------------+-----------------------------------------+
|              | RAM            |  2GB+ per daemon                        |
|              +----------------+-----------------------------------------+
|              | 磁盘空间       |  1 MB per daemon                        |
|              +----------------+-----------------------------------------+
|              | 网络           |  1x 1GbE+ NICs                          |
+--------------+----------------+-----------------------------------------+

.. tip:: 如果在只有一块硬盘的机器上运行 OSD ，
   要把数据和操作系统分别放到不同分区；
   一般来说，我们推荐操作系统和数据\
   分别使用不同的硬盘。





.. _Ceph 博客文章: https://ceph.com/community/blog/
.. _Ceph 写吞吐量 1: http://ceph.com/community/ceph-performance-part-1-disk-controller-write-throughput/
.. _Ceph 写吞吐量 2: http://ceph.com/community/ceph-performance-part-2-write-throughput-without-ssd-journals/
.. _给存储池指定 OSD: ../../rados/operations/crush-map#placing-different-pools-on-different-osds
.. _操作系统推荐: ../os-recommendations
