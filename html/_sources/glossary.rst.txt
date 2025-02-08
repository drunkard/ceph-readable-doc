===========
 Ceph 术语
===========

.. glossary::

    Application
    应用程序
        更恰当的叫法是 :term:`客户端` ，它是相对于 Ceph 之外的任意程序，
        使用 Ceph 集群来存储和复制数据。

    :ref:`BlueStore<rados_config_storage_devices_bluestore>`
        OSD BlueStore 是 OSD 守护进程使用的一个存储后端，是特意为 Ceph 设计的 。
        BlueStore 最早在 Kraken 版本引进，从 Luminous 版起， BlueStore 取代
        FileStore 被提升为默认 OSD 后端，完全取代了 FileStore 。到 Reef 版时，
        就不能再使用 FileStore 作为存储后端了。

        BlueStore 直接在原始块设备或分区上存储对象，不会与挂载的文件系统\
        产生交互操作。 BlueStore 利用 RocksDB 的键/值数据库来把对象名映射到\
        磁盘上的块位置。

    Bucket
    桶
        在 :term:`RGW` 的语境中，“桶”是一组对象。
        以文件系统类比的话，对象对应文件，桶对应目录。
        要想对一个区域到另一个区域的数据移动进行细粒度控制，可以在桶上配置
        :ref:`多站点同步策略 <radosgw-multisite-sync-policy>` 。

        桶的概念是从 AWS S3 借鉴来的。请参考 `AWS S3 网页上有关创建桶
        <https://docs.aws.amazon.com/AmazonS3/latest/userguide/creating-buckets-s3.html>`_
        和 `AWS S3 “桶概览”页面 <https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingBucket.html>`_ 。

        在 RGW 和 AWS 上称为“桶”的东西，到了 OpenStack Swift 上叫作术语“容器（ containers ）”。
        见 `OpenStack Storage API 概览页面 <https://docs.openstack.org/swift/latest/api/object_api_v1_overview.html>`_ 。

    Ceph
        Ceph 是一套支持分布式元数据管理和 POSIX 语义的
        分布式网络存储和文件系统。

    `ceph-ansible <https://docs.ceph.com/projects/ceph-ansible/en/latest/index.html>`_
        一个 GitHub 仓库，支持从 Jewel 版到 Quincy 版的
        Ceph 集群安装。

    Ceph Block Device
    Ceph 块设备
        也叫作 “RADOS Block Device” 和 :term:`RBD` 。
        是个软件工具，在 Ceph 中组织基于块数据的存储。
        Ceph 块设备把基于块的应用数据分割成“块”，
        RADOS 将这些块存储为对象。
        Ceph 块设备功能组织着这些对象的在集群中的存储。

    Ceph Block Storage
    Ceph 块存储
        Ceph 支持的三种存储之一
        （另外两种是对象存储和文件存储）。
        Ceph 块存储是块存储“产品”，
        它在与三类软件套件对接使用时指的是块存储相关的服务及其功能，
        可以对接 (1) ``librbd`` （一个 Python 模块，它能使我们\
        像访问文件一样访问 :term:`RBD` 映像）、
        (2) 虚拟化管理程序如QEMU 或 Xen 、和
        (3) 虚拟化管理程序抽象层，如 ``libvirt`` 。

    :ref:`Ceph Client <architecture_ceph_clients>`
    Ceph 客户端
        任何能够访问 Ceph 存储集群的 Ceph 组件，
        其中包括 Ceph 对象网关、 Ceph 块设备、
        Ceph 文件系统、及其对应的库，
        也包括内核模块和 FUSE （用户空间文件系统）。

    Ceph Client Libraries
    Ceph 客户端库
        能够和相应 Ceph 集群的组件相交互的一组库。

    Ceph Cluster Map
    Ceph 集群运行图
        见 :term:`Cluster Map`

    Ceph Dashboard
    Ceph 仪表盘
        :ref:`Ceph 仪表盘<mgr-dashboard>` 是一个内建的、
        基于网页的 Ceph 管理和监控应用程序，
        可用于调查和管理集群内的各种资源。
        仪表盘是以一个 Ceph 管理器模块（ :ref:`ceph-manager-daemon` ）实现的。

    Ceph Filesystem
    Ceph 文件系统
        见 :term:`CephFS`

    :ref:`CephFS<ceph-file-system>`
        Ceph 文件系统（ **Ceph F**\ile **S**\ystem ）或者 CephFS ，
        是一个与 POSIX 兼容的文件系统，
        底层基于 Ceph 的分布式对象存储 RADOS 。
        详情见 :ref:`CephFS 体系结构 <arch-cephfs>` 。

    :ref:`ceph-fuse <man-ceph-fuse>`
        :ref:`ceph-fuse <man-ceph-fuse>` 是 CephFS 的一个 FUSE
        (用户空间文件系统， "**F**\ilesystem in **USE**\rspace") 客户端。
        ceph-fuse 可以把 Ceph FS 挂载到指定的挂载点。

    Ceph Interim Release
    Ceph 临时版
        见 :term:`Releases` 。

    Ceph Kernel Modules
    Ceph 内核模块
        一组能够成功和 Ceph 系统交互的内核模块
        （比如 ``ceph.ko`` 、 ``rbd.ko`` ）。

    :ref:`Ceph Manager<ceph-manager-daemon>`
    Ceph 管理器
        Ceph 管理器守护进程（ ceph-mgr ）是与监控守护进程\
        一起运行的守护进程，可提供监控功能并与外部监控和管理系统连接。
        从 Luminous 版（12.x）起，所有 Ceph 集群要想正常运作，
        都必须有正常运行的 ceph-mgr 守护进程。

    Ceph Manager Dashboard
    Ceph 管理器仪表盘
        见 :term:`Ceph Dashboard` 。

    Ceph Metadata Server
    Ceph 元数据服务器
        见 :term:`MDS` 。

    Ceph Monitor
    Ceph 监视器
        维护集群状态图的守护进程。
        集群状态包括监控器运行图、管理器运行图、OSD 运行图和 CRUSH 运行图。
        一套 Ceph 集群必须至少包含三个正常运行的监控器，
        才能同时实现冗余和高可用性。
        Ceph 监视器以及它所在的节点通常也叫做 mon 。
        见 :ref:`监视器配置参考 <monitor-config-reference>` 。

    Ceph Node
    Ceph 节点
        Ceph 节点是 Ceph 集群的一个单元，它与 Ceph 集群中的其他节点通信，
        以便复制和重新分布数据。所有节点加在一起称为 :term:`Ceph 存储集群`\ 。
        Ceph 节点包括 :term:`OSD` 、 :term:`Ceph 监视器`\ 、
        :term:`Ceph 管理器`\ 和 :term:`MDS`\ 。在 Ceph 文档中，
        术语 “节点（ node）” 通常等同于 “主机（ host ）”。
        如果您有一个正常运行的 Ceph 群集，可以用
        ``ceph node ls all`` 命令列出其内的所有节点。

    :ref:`Ceph Object Gateway<object-gateway>`
    Ceph 对象网关
        建立在 librados 基础上的一个对象存储接口。
        Ceph 对象网关符合 REST 规范，它位于应用程序和 Ceph 存储集群之间。

    Ceph Object Storage
    Ceph 对象存储
        见 :term:`Ceph Object Store` 。

    Ceph Object Store
    Ceph 对象存储库
        Ceph 对象存储库由 :term:`Ceph 存储集群` 和
        :term:`Ceph 对象网关` （RGW）组成。

    :ref:`Ceph OSD<rados_configuration_storage-devices_ceph_osd>`
        Ceph **O**\bject **S**\torage **D**\aemon （ Ceph 对象存储守护进程）。
        是指 Ceph OSD 软件，负责与逻辑磁盘（ :term:`OSD` ）交互。2013 年前后，
        “研究和行业”（ Sage 自己说的）曾试图坚持让 “OSD” 这个术语仅指
        “对象存储设备”，但 Ceph 社区此前一直用该术语指 “对象存储守护进程”，
        而且不会有人比 Sage Weil 本人更权威了，
        他在 2022 年 11 月确认，“守护进程这个说法对于 Ceph 的构建方式更精确”
        （ Zac Dover 和 Sage Weil 之间的私人通信，2022 年 11 月 7 日）。

    Ceph OSD Daemon
    Ceph OSD 守护进程
        见 :term:`Ceph OSD`.

    Ceph OSD Daemons
        见 :term:`Ceph OSD`.

    Ceph Platform
    Ceph 平台
        所有与 Ceph 相关的软件，包括所有位于 `https://github.com/ceph`_ 的源码。

    Ceph Point Release
    Ceph 修正版
    Ceph 小版本
        见 :term:`Releases`.

    Ceph Project
    Ceph 项目
        关于 Ceph 的团队、软件、任务和基础架构的统称。

    Ceph Release
    Ceph 版本
        见 :term:`Releases` 。

    Ceph Release Candidate
    Ceph 发布候选版
        见 :term:`Releases` 。

    Ceph Stable Release
    Ceph 稳定版
        见 :term:`Releases` 。

    Ceph Stack
    Ceph 软件栈
        Ceph 之中两个或更多组件的组合。

    :ref:`Ceph Storage Cluster<arch-ceph-storage-cluster>`
    :ref:`Ceph 存储集群<arch-ceph-storage-cluster>`
        :term:`Ceph 监视器`\ 、:term:`Ceph 管理器`\ 、 :term:`Ceph 元数据服务器`
        和 :term:`OSD` 的集合，它们共同存储和复制数据，供应用程序、 Ceph 用户和
        Ceph 客户端使用。 Ceph 存储集群负责接收来自 :term:`Ceph 客户端`\ 的数据。

    CephX
        Ceph 的认证协议， CephX 可认证用户和各种守护进程。
        Cephx 的运行机制类似 Kerberos ，但它没有单故障点。请看
        :ref:`高可用性认证一节 <arch_high_availability_authentication>` 和
        :ref:`CephX 配置参考 <rados-cephx-config-ref>` 。

    Client
    客户端
        客户端是处于 Ceph 集群外部、但是用 Ceph 存储和复制数据的任意程序。

    Cloud Platforms
    Cloud Stacks
    云平台
    云软件栈
        第三方云服务平台，比如 OpenStack, CloudStack, OpenNebula, Proxmox VE 。

    Cluster Map
    集群运行图
        一系列的运行图，包括监视器运行图、 OSD 运行图、 PG 图、 MDS 运行图\
        和 CRUSH 图，它们合在一起就是 Ceph 集群的状态。详情见
        :ref:`体系结构文档的“集群运行图”一节<architecture_cluster_map>` 。

    Crimson
        下一代 OSD 架构，其主要目标是减少跨核通信所导致的延迟成本。
        通过减少数据路径中分片之间的通信，重新设计的 OSD 减少了锁争用。
        Crimson 通过消除对线程池的依赖，在经典 Ceph OSD 的基础上提高了性能。
        参见 `Crimson：支持多核可扩展性的下一代 Ceph OSD
        <https://ceph.io/en/news/blog/2023/crimson-multi-core-scalability/>`_ 。
        参见 :ref:`Crimson 开发者文档<crimson_dev_doc>` 。

    CRUSH
        **C**\ontrolled **R**\eplication **U**\nder **S**\calable **H**\ashing ，
        可扩展散列计算下的受控复制。它是 Ceph 用以计算对象存储位置的算法。
        见 `CRUSH: 受控的、可扩展的、去中心化的多副本数据归置
        <https://ceph.com/assets/pdfs/weil-crush-sc06.pdf>`_ 。

    CRUSH rule
    CRUSH 规则
        应用到某个特定存储池（们）的 CRUSH 数据归置规则。

    DAS
    直连存储器
        **D**\irect-\ **A**\ttached **S**\torage, 直连存储。
        直接连接到访问它的计算机的存储设备，不通过网络传输。与 NAS 和 SAN 相反。

    :ref:`Dashboard<mgr-dashboard>`
    :ref:`仪表盘<mgr-dashboard>`
        内置的、基于网页的 Ceph 管理和监控应用程序，用于管理集群的各个方面以及\
        对象。仪表盘被做成了 Ceph 管理器的一个模块。详情见 :ref:`mgr-dashboard` 。

    Dashboard Module
    仪表盘模块
        :term:`仪表盘` 的另一种叫法。

    Dashboard Plugin
    仪表盘插件
        <原文空>

    DC
        **D**\ata **C**\enter, 数据中心。

    Flapping OSD
    打摆子的 OSD
        OSD 的状态接连被反复标记为 ``up`` 而后 ``down`` 。
        见 :ref:`rados_tshooting_flapping_osd` 。

    FQDN
    全资域名
        **F**\ully **Q**\ualified **D**\omain **N**\ame, 全资域名
        （完全符合规范的域名）。域名应用在网络中某一节点上、
        可指明这个节点在 DNS 树状层次结构中的确切位置。

        在 Ceph 集群管理工作这个语境中， FQDN 通常是指主机。在本文档中，
        术语 “FQDN” 主要用于区分 FQDN 和相对简单的主机名，后者并不指明\
        主机在 DNS 树状层次结构中的确切位置，而只是命名主机。

    Host
    主机
        Ceph 集群中的任何单机或服务器。请参阅 :term:`Ceph 节点`\ 。

    Hybrid OSD
    混合式 OSD
        指同时具有 HDD 和 SSD 驱动器的 OSD 。

    librados
        是个 API ，可以用于创建访问 Ceph 存储集群的定制接口。
        有了 ``librados`` 才能与 Ceph 监视器和 OSD 们交互。
        请参阅 :ref:`librados 简介<librados-intro>` 。
        请参阅 :ref:`librados (Python)<librados-python>` 。

    LVM tags
    LVM 标签
        **L**\ogical **V**\olume **M**\anager, 逻辑卷管理器标签。
        LVM 卷和组的可扩展元数据，它们用于存储 Ceph 特有的信息，
        如各设备、以及它们与 OSD 的关系。

    MDS
    元数据服务器
        Ceph 元数据服务器守护进程（ **M**\eta\ **D**\ata **S**\erver ），
        也称为 ceph-mds 。任何存在 CephFS 文件系统的 Ceph 集群都必须运行着
        Ceph 元数据服务器守护进程。 MDS 存储着所有的文件系统元数据。
        :term:`客户端`\ 们与单个或一组 MDS 一起维护 CephFS 所需的分布式元数据缓存。

        参阅 :ref:`部署元数据服务器<cephfs_add_remote_mds>` 。

        参阅 :ref:`ceph-mds 手册页<ceph_mds_man>` 。

    MGR
    管理器
        Ceph 管理器软件，它把整个集群的所有状态信息都收集到一个地方。

    :ref:`MON<arch_monitor>`
    :ref:`监视器<arch_monitor>`
        Ceph 监视器软件。

    Monitor Store
    监视器存储系统
        监视器使用的永久存储，包括监视器的 RocksDB 、以及
        ``/var/lib/ceph`` 内的所有相关文件。

    Node
    节点
        参阅 :term:`Ceph 节点` 。

    Object Storage
    对象存储
        对象存储是与 Ceph 相关的三种存储之一。与 Ceph 相关的其他两种存储是
        文件存储和块存储。对象存储是 Ceph 最基本的存储类别。

    Object Storage Device
    对象存储设备（实体）
        参阅 :term:`OSD` 。

    OMAP
        “对象映射（ object map ）"。键值存储库（一种数据库），用于缩短\
        从 Ceph 集群读出数据和向 Ceph 集群写入数据的时间。
        RGW 桶索引存储为 OMAP。纠删码存储池不能存储 RADOS OMAP 数据结构。

        运行 ``ceph osd df`` 命令查看 OMAP 。

        参阅 Eleanor Cawthon 2012 年发表的论文\ `用 Ceph 实现分布式键值存储库
        <https://ceph.io/assets/pdfs/CawthonKeyValueStore.pdf>`_ （17 页）。

    OpenStack Swift
        在 Ceph 语境下， OpenStack Swift 是 Ceph 对象存储库支持的两种 API 之一。
        Ceph 对象存储库支持的另外一种 API 是 S3 。

        参阅 `OpenStack 存储 API 概览
        <https://docs.openstack.org/swift/latest/api/object_api_v1_overview.html>`_ 。

    OSD
    对象存储设备
        可能是 :term:`Ceph OSD` ，但也不一定。
        有时（特别是在较早的通信中，
        尤其是在不是专门为 Ceph 编写的文档中），
        “OSD” 指的是 “对象存储设备（ **O**\bject **S**\torage **D**\evice ）”，
        它指的是物理或逻辑存储单元（例如： LUN ）。
        Ceph 社区一直用术语 OSD 来指代 :term:`Ceph OSD Daemon` ，
        尽管业界在 2010 年代中期坚持推动 “OSD” 应该表示
        “对象存储设备（ Object Storage Device ）”，
        因此有必要了解其（在不同语境下的）含义。

    OSD, flapping
        见 :term:`Flapping OSD`.

    OSD FSID
        OSD fsid 是用于确定一个 OSD 的唯一标识符。
        可以在 OSD 路径内、一个名为 ``osd_fsid`` 的文件里找到。
        术语 ``FSID`` 可以和 ``uuid`` 互换着用。

    OSD ID
        OSD id 是对每个 OSD 来说都是独一无二的整数
        （每个 OSD 都有一个唯一的 OSD ID ）。
        它是在新建与之相关的 OSD 期间由监视器们生成的。

    OSD UUID
        OSD UUID 是一个 OSD 的唯一标识符，这个术语可以和 ``FSID`` 互换着用。

    Period
        在 :term:`RGW` 中，一个 period 是 :term:`Realm` 的配置状态。
        period 存储着多站点配置的配置状态。
        当 period 更新时， epoch 也随之改变。

    Placement Groups (PGs)
    归置组（ PGs ）
        归置组（ PG ）是每个 Ceph 逻辑存储池的子集。
        归置组实现的功能是将对象（作为一个组）归置到 OSD 内。
        Ceph 内部以归置组这样的粒度管理数据：
        这比管理单个（相应地数量更庞大） RADOS 对象的扩展性更好。
        拥有较多归置组（例如每 OSD 100 个）的集群比\
        归置组较少的同等集群的平衡性更好。

        Ceph 内部的每一个 RADOS 对象都各自映射到一个特定的归置组，
        每个归置组只能属于一个 Ceph 存储池。

    PLP
        **P**\ower **L**\oss **P**\rotection, 断电保护。
        这是一种保护固态硬盘数据的技术，
        通过使用电容器来延长通电时间，
        以便数据从 DRAM 缓存写入 SSD 的永久存储器。
        消费级固态硬盘极少配备 PLP。

    :ref:`Pool<rados_pools>`
    :ref:`存储池<rados_pools>`
        存储池是用于存储对象的逻辑分区。

    Pools
        见 :term:`pool` 。

    :ref:`Primary Affinity <rados_ops_primary_affinity>`
    :ref:`主亲和性 <rados_ops_primary_affinity>`
        OSD 的特性，它用于决定某一个 OSD 在 acting set 中
        被选为主 OSD（或“领导 OSD, lead OSD”）的可能性。
        主亲和性在 Firefly（ 0.80 版）中引入。
        参阅 :ref:`主亲和性<rados_ops_primary_affinity>` 。

    :ref:`Prometheus <mgr-prometheus>`
        一个开源监控和警报工具包。
        Ceph 自带一个 :ref:`Prometheus 模块<mgr-prometheus>` ，
        它有一个 Prometheus exporter ，
        可将 ``ceph-mgr`` 内收集点里的性能计数器传给 Prometheus。

    Quorum
    法定人数
        法定人数是一种状态，此时集群中大多数\
        :ref:`监视器<arch_monitor>`\ 状态为 ``up`` 。
        集群中至少要有三个\ :ref:`监视器<arch_monitor>`\
        才有可能达到法定人数。

    RADOS
        可靠、自主的分布式对象存储， **R**\eliable **A**\utonomic
        **D**\istributed **O**\bject **S**\tore 。
        RADOS 只是个对象存储库，可为不同大小的对象提供可扩展服务。
        RADOS 对象存储是 Ceph 集群的核心组件。
        `这篇 2009 年的博文
        <https://ceph.io/en/news/blog/2009/the-rados-distributed-object-store/>`_
        为入门者介绍了 RADOS 。
        有兴趣深入了解 RADOS 的读者可参阅
        `《RADOS：一个可扩展、可靠的 PB 级存储集群》
        <https://ceph.io/assets/pdfs/weil-rados-pdsw07.pdf>`_ 。

    RADOS Cluster
    RADOS 集群
        Ceph 集群的一个比较完整的子集，
        由 :term:`OSD`\s 、 :term:`Ceph Monitor`\s 、
        和 :term:`Ceph Manager`\s 组成。

    RADOS Gateway
    RADOS 网关
        见 :term:`RGW` 。

    RBD
        RADOS 块设备， **R**\ADOS **B**\lock **D**\evice 。
        见 :term:`Ceph 块设备` 。

    :ref:`Realm<rgw-realms>`
        在 RADOS Gateway (RGW) 中，
        realm 是一个全局唯一的命名空间，
        它由一个或多个 zonegroup 组成。

    Releases

        Ceph Interim Release
            Ceph 临时版。
            尚未通过质检测试的一个 Ceph 版本。会包含新功能。

        Ceph Point Release
            Ceph 修正版。
            所有只包含缺陷修正或安全修正的特殊版本。

        Ceph Release
            Ceph 版本。
            任何版本号不同的 Ceph 。

        Ceph Release Candidate
            Ceph 发布候选版。
            Ceph 的一个主要版本，已经通过了初步的质检测试，
            并且准备好给测试员试用了。

        Ceph Stable Release
            Ceph 稳定版。
            Ceph 的主要版本，之前临时版本里所有的功能
            都成功通过了质检测试。

    Reliable Autonomic Distributed Object Store
    可靠自主的分布式对象存储库
        存储软件的核心部分，它存储着用户数据（ MON+OSD ）。
        参阅 :term:`RADOS` 。

    :ref:`RGW<object-gateway>`
        **R**\ADOS **G**\ate\ **w**\ay. RADOS 网关。

        也叫做 “Ceph Object Gateway， Ceph 对象网关”。
        Ceph 的组件，可同时为 Amazon S3 REST API 和
        OpenStack Swift API 提供网关。

    S3
        就 Ceph 而言， S3 是 Ceph Object Store 支持的两个 API 之一。
        Ceph Object Store 支持的另一个 API 是 OpenStack Swift 。

        参阅 `Amazon S3 概述页面
        <https://aws.amazon.com/s3/>`_\ 。

    scrubs
    洗刷
        Ceph 确保数据完整性的过程。
        在洗刷过程中， Ceph 会生成一个归置组中所有对象的目录，
        然后将每个主对象与其副本（存储在其他 OSD 中）进行比对，
        确保所有对象都存在且匹配。
        如果发现任何 PG 中有一个对象副本与其他副本不同或完全丢失，
        则标记为 “inconsistent （不一致）”
        （也就是这个 PG 被标记为 inconsistent ）。

        洗刷分两种：轻度洗刷（ light scrubbing ）和深度洗刷（ deep scrubbing ）
        （也叫“浅度洗刷（ shallow scrubbing ）”和“深度洗刷（ deep scrubbing ）”）。
        轻度洗刷每天执行一次，其作用只是确认某个对象存在，
        以及它的元数据正确。
        深度洗刷每周执行一次，
        会读出数据并使用校验和来确保数据完整性。

        参见《 RADOS OSD 配置参考指南》中的\
        :ref:`洗刷 <rados_config_scrubbing>`\ 和\
        *《精通 Ceph 》第二版*\ 第 141 页（Fisk，Nick，2019 年）。

    secrets
    密钥
        密钥是用于执行数字身份验证的凭证，
        只要特权用户要访问要求身份验证的系统就需要。
        密钥可以是口令、 API 密钥、令牌、
        SSH 密钥、私人证书或加密密钥。

    SDS
        **S**\oftware-**d**\efined **S**\torage ，软件定义的存储。

    systemd oneshot
        一种 systemd 类型 ``type`` ，其命令定义在 ``ExecStart`` 里，
        完成后就退出（不想把它作为守护进程）。

    Swift
        参阅 :term:`OpenStack Swift`.

    Teuthology
    测试方法学
        对 Ceph 进行脚本化测试的一系列软件。

    User
    用户
        使用 Ceph 客户端与 :term:`Ceph 存储集群`\ 交互的\
        个人或系统角色（例如应用程序）。
        请参阅 :ref:`用户<rados-ops-user>`\ 和\
        :ref:`用户管理<user-management>`\ 。

    Zone
    域
        在 :term:`RGW` 中，域是由一个或多个 :term:`RGW` 实例组成的逻辑组。
        域的配置状态存储在 :term:`period` 内。参阅 :ref:`域<radosgw-zones>` 。


.. _https://github.com/ceph: https://github.com/ceph
.. _集群运行图: ../architecture#cluster-map
