===========
 Ceph 术语
===========

.. glossary::

    Application
    应用程序
        更恰当的叫法是 :term:`客户端` ，它是任何一个 Ceph 外部的程序，
        使用 Ceph 集群来存储和复制数据。

    :ref:`BlueStore<rados_config_storage_devices_bluestore>`
        OSD BlueStore 是 OSD 守护进程使用的一个存储后端，\
        是特意为 Ceph 设计的 。BlueStore 最早在 Kraken 版本引进，\
        从 Luminous 版起， BlueStore 取代 FileStore 被提升为\
        默认 OSD 后端。到 Reef 版时，就不能再使用 FileStore
        作为存储后端了。

        BlueStore 直接在原始块设备或分区上存储对象，不会与\
        挂载的文件系统产生交互操作。 BlueStore 利用 RocksDB
        的键/值数据库来把对象名映射到磁盘上的块位置。

    Bucket
    桶
        在 :term:`RGW` 的语境中，“桶”是一组对象。
        以文件系统类比的话，对象对应文件，桶对应目录。
        要想对一个区域到另一个区域的数据移动进行细粒度控制，
        可以在桶上设置
        :ref:`多站点同步策略 <radosgw-multisite-sync-policy>` 。

        桶的概念是从 AWS S3 借鉴来的。
        请参考 `AWS S3 网页上有关创建桶 <https://docs.aws.amazon.com/AmazonS3/latest/userguide/creating-buckets-s3.html>`_
        和 `AWS S3 “桶概览”页面 <https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingBucket.html>`_ 。

        在 RGW 和 AWS 上称为“桶”的东西，到了 OpenStack Swift 上叫作术语“容器（ containers ）”。
        见 `OpenStack Storage API 概览页面 <https://docs.openstack.org/swift/latest/api/object_api_v1_overview.html>`_ 。

    Ceph
        Ceph 是一套支持分布式元数据管理和 POSIX 语义的
        分布式网络存储和文件系统。

        `ceph-ansible <https://docs.ceph.com/projects/ceph-ansible/en/latest/index.html>`_
        A GitHub repository, supported from the Jewel release to the
        Quincy release, that facilitates the installation of a Ceph
        cluster.

    Ceph Block Device
    Ceph 块设备
        Also called "RADOS Block Device" and :term:`RBD`. A software
        instrument that orchestrates the storage of block-based data in
        Ceph. Ceph Block Device splits block-based application data
        into "chunks". RADOS stores these chunks as objects. Ceph Block
        Device orchestrates the storage of those objects across the
        storage cluster.

    Ceph Block Storage
    Ceph 块存储
        Ceph 支持的三种存储之一（另外两种是对象存储和文件存储）。Ceph 块存储是
        块存储“产品”，它在与三类软件套件对接使用时指的是块存储相关的服务及其功能，
        可以对接 (1) ``librbd`` （一个 Python 模块，它能使我们像访问文件一样
        访问 :term:`RBD` 映像）、(2) 虚拟化管理程序如 QEMU 或 Xen 、和
        (3) 虚拟化管理程序抽象层，如 ``libvirt`` 。

    :ref:`Ceph Client <architecture_ceph_clients>`
    Ceph 客户端
        任何能够访问 Ceph 存储集群的 Ceph 组件，其中包括 Ceph 对象网关、
        Ceph 块设备、 Ceph 文件系统、及其对应的库，也包括内核模块和 FUSE
        （用户空间文件系统）。

    Ceph Client Libraries
    Ceph 客户端库
        能够和相应 Ceph 集群的组件相交互的一组库。

    Ceph Cluster Map
    Ceph 集群运行图
        见 :term:`Cluster Map`

    Ceph Dashboard
    Ceph 仪表盘
        一个内建的、基于网页的 Ceph 管理和监控应用程序，可用于管理集群的各方面以及对象。
        仪表盘是以一个 Ceph 管理器模块实现的。详情见 :ref:`mgr-dashboard` 。

    Ceph Filesystem
    Ceph 文件系统
        见 :term:`CephFS`

    :ref:`CephFS<ceph-file-system>`
        Ceph 文件系统（ **Ceph F**\ile **S**\ystem ）或者 CephFS ，
        是一个与 POSIX 兼容的文件系统，底层基于 Ceph 的分布式对象存储 RADOS 。
        详情见 :ref:`CephFS 体系结构 <arch-cephfs>` 。

        :ref:`ceph-fuse <man-ceph-fuse>`
                :ref:`ceph-fuse <man-ceph-fuse>` is a FUSE ("**F**\ilesystem in
                **USE**\rspace") client for CephFS. ceph-fuse mounts a Ceph FS
                ata  specified mount point.

    Ceph Interim Release
    Ceph 临时发布
        见 :term:`Releases`.

    Ceph Kernel Modules
    Ceph 内核模块
        一组能够成功和 Ceph 系统交互的内核模块（比如 ``ceph.ko`` 、 ``rbd.ko`` ）。

    :ref:`Ceph Manager<ceph-manager-daemon>`
    Ceph 管理器
        Ceph 管理器软件，它会把整个集群的所有状态信息收集到一起。

    Ceph Manager Dashboard
    Ceph 管理器仪表盘
        See :term:`Ceph Dashboard`.

    Ceph Metadata Server
    Ceph 元数据服务器
        Ceph 的元数据软件。

    Ceph Monitor
    Ceph 监视器
        A daemon that maintains a map of the state of the cluster. This
        "cluster state" includes the monitor map, the manager map, the
        OSD map, and the CRUSH map. A Ceph cluster must contain a
        minimum of three running monitors in order to be both redundant
        and highly-available. Ceph monitors and the nodes on which they
        run are often referred to as "mon"s. See :ref:`Monitor Config
        Reference <monitor-config-reference>`.

    Ceph Node
    Ceph 节点
        Ceph 系统内的任意单体机器或服务器。
        A Ceph node is a unit of the Ceph Cluster that communicates with
        other nodes in the Ceph Cluster in order to replicate and
        redistribute data. All of the nodes together are called the
        :term:`Ceph Storage Cluster`. Ceph nodes include :term:`OSD`\s,
        :term:`Ceph Monitor`\s, :term:`Ceph Manager`\s, and
        :term:`MDS`\es. The term "node" is usually equivalent to "host"
        in the Ceph documentation. If you have a running Ceph Cluster,
        you can list all of the nodes in it by running the command
        ``ceph node ls all``.

    :ref:`Ceph Object Gateway<object-gateway>`
    Ceph 对象网关
                An object storage interface built on top of librados. Ceph
                Object Gateway provides a RESTful gateway between applications
                and Ceph storage clusters.

    Ceph Object Storage
    Ceph 对象存储
        见 :term:`Ceph Object Store` 。

    Ceph Object Store
    Ceph 对象存储系统
        Ceph 对象存储库由 :term:`Ceph 存储集群` 和
        :term:`Ceph 对象网关` （RGW）组成。

    :ref:`Ceph OSD<rados_configuration_storage-devices_ceph_osd>`
        Ceph 的 OSD 软件，它与逻辑磁盘（ :term:`OSD` ）交互。有时候，
        Ceph 用户们用 “OSD” 这个术语来指代 “Ceph OSD Daemon”，
        然而正确的术语是 “Ceph OSD”。
        Ceph **O**\bject **S**\torage **D**\aemon. The Ceph OSD
        software, which interacts with logical disks (:term:`OSD`).
        Around 2013, there was an attempt by "research and industry"
        (Sage's own words) to insist on using the term "OSD" to mean
        only "Object Storage Device", but the Ceph community has always
        persisted in using the term to mean "Object Storage Daemon" and
        no less an authority than Sage Weil himself confirms in
        November of 2022 that "Daemon is more accurate for how Ceph is
        built" (private correspondence between Zac Dover and Sage Weil,
        07 Nov 2022).

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
        See :term:`Releases`.

    Ceph Project
    Ceph 项目
        关于 Ceph 的团队、软件、任务和基础架构的统称。

    Ceph Release
    Ceph 发布
        任何用不同数字编号的 Ceph 版本。

    Ceph Release Candidate
    Ceph 预发布
        见 :term:`Releases` 。

    Ceph Stable Release
    Ceph 稳定版
        见 :term:`Releases` 。

    Ceph Stack
    Ceph 软件栈
        Ceph 之中两个或更多组件的组合。

    :ref:`Ceph Storage Cluster<arch-ceph-storage-cluster>`
    Ceph 存储集群
        The collection of :term:`Ceph Monitor`\s, :term:`Ceph
        Manager`\s, :term:`Ceph Metadata Server`\s, and :term:`OSD`\s
        that work together to store and replicate data for use by
        applications, Ceph Users, and :term:`Ceph Client`\s. Ceph
        Storage Clusters receive data from :term:`Ceph Client`\s.

    CephX
        Ceph 的认证协议， Cephx 的运行机制类似 Kerberos ，但它没有单故障点。

    Client
    客户端
        A client is any program external to Ceph that uses a Ceph
        Cluster to store and replicate data.

    Cloud Platforms
    Cloud Stacks
    云平台
    云软件栈
        第三方云服务平台，比如 OpenStack, CloudStack, OpenNebula, Proxmox VE 等等。

    Cluster Map
    集群运行图
        一系列的运行图，包括监视器运行图、 OSD 运行图、 PG 图、 MDS 运行图\
        和 CRUSH 图。详情见 `集群运行图`_ 。

    Crimson
        A next-generation OSD architecture whose main aim is the
        reduction of latency costs incurred due to cross-core
        communications. A re-design of the OSD reduces lock
        contention by reducing communication between shards in the data
        path. Crimson improves upon the performance of classic Ceph
        OSDs by eliminating reliance on thread pools. See `Crimson:
        Next-generation Ceph OSD for Multi-core Scalability
        <https://ceph.io/en/news/blog/2023/crimson-multi-core-scalability/>`_.
        See the :ref:`Crimson developer
        documentation<crimson_dev_doc>`.

    CRUSH
        Controlled Replication Under Scalable Hashing ，可伸缩哈希控制的复制。
        它是 Ceph 用以计算对象存储位置的算法。

    CRUSH rule
    CRUSH 规则
        应用到某个特定存储池（们）的 CRUSH 数据归置规则。

    DAS
    直连存储器
        **D**\irect-\ **A**\ttached **S**\torage. Storage that is
        attached directly to the computer accessing it, without passing
        through a network.  Contrast with NAS and SAN.

    :ref:`Dashboard<mgr-dashboard>`
    仪表盘
        A built-in web-based Ceph management and monitoring application
        to administer various aspects and objects of the cluster. The
        dashboard is implemented as a Ceph Manager module. See
        :ref:`mgr-dashboard` for more details.

    Dashboard Module
    仪表盘模块
        Another name for :term:`Dashboard`.

    Dashboard Plugin
    仪表盘插件
        <原文空>

    Flapping OSD
        An OSD that is repeatedly marked ``up`` and then ``down`` in
        rapid succession. See :ref:`rados_tshooting_flapping_osd`.

    FQDN
    全资域名
        **F**\ully **Q**\ualified **D**\omain **N**\ame. A domain name
        that is applied to a node in a network and that specifies the
        node's exact location in the tree hierarchy of the DNS.

        In the context of Ceph cluster administration, FQDNs are often
        applied to hosts. In this documentation, the term "FQDN" is
        used mostly to distinguish between FQDNs and relatively simpler
        hostnames, which do not specify the exact location of the host
        in the tree hierarchy of the DNS but merely name the host.

    Host
    主机
        Any single machine or server in a Ceph Cluster. See :term:`Ceph
        Node`.

    Hybrid OSD
    混合式 OSD
        Refers to an OSD that has both HDD and SSD drives.

    librados
        An API that can be used to create a custom interface to a Ceph
        storage cluster. ``librados`` makes it possible to interact
        with Ceph Monitors and with OSDs. See :ref:`Introduction to
        librados <librados-intro>`. See :ref:`librados (Python)
        <librados-python>`.

    LVM tags
    LVM 标签
        LVM 卷和组的可扩展元数据，我们用它来存储 Ceph 相关的信息，
        如各设备、以及它们与 OSD 的关系。

    MDS
    元数据服务器
        The Ceph **M**\eta\ **D**\ata **S**\erver daemon. Also referred
        to as "ceph-mds". The Ceph metadata server daemon must be
        running in any Ceph cluster that runs the CephFS file system.
        The MDS stores all filesystem metadata. :term:`Client`\s work
        together with either a single MDS or a group of MDSes to
        maintain a distributed metadata cache that is required by
        CephFS.

        See :ref:`Deploying Metadata Servers<cephfs_add_remote_mds>`.

        See the :ref:`ceph-mds man page<ceph_mds_man>`.

    MGR
    管理器
        The Ceph manager software, which collects all the state from
        the whole cluster in one place.

    :ref:`MON<arch_monitor>`
    监视器
        The Ceph monitor software.

    Monitor Store
    监视器存储系统
        The persistent storage that is used by the Monitor. This
        includes the Monitor's RocksDB and all related files in
        ``/var/lib/ceph``.

    Node
        See :term:`Ceph Node`.

    Object Storage
                Object storage is one of three kinds of storage relevant to
                Ceph. The other two kinds of storage relevant to Ceph are file
                storage and block storage. Object storage is the category of
                storage most fundamental to Ceph.

    Object Storage Device
    对象存储设备（实体）
        一个物理的或逻辑的存储单元（如 LUN ）。有时候， Ceph 用户以术语 OSD 来引用
        :term:`Ceph OSD 守护进程`\ ，然而恰当的术语应该是 Ceph OSD 。

    OMAP
        "object map". A key-value store (a database) that is used to
        reduce the time it takes to read data from and to write to the
        Ceph cluster. RGW bucket indexes are stored as OMAPs.
        Erasure-coded pools cannot store RADOS OMAP data structures.

        Run the command ``ceph osd df`` to see your OMAPs.

        See Eleanor Cawthon's 2012 paper `A Distributed Key-Value Store
        using Ceph
        <https://ceph.io/assets/pdfs/CawthonKeyValueStore.pdf>`_ (17
        pages).

    OpenStack Swift
        In the context of Ceph, OpenStack Swift is one of the two APIs
        supported by the Ceph Object Store. The other API supported by
        the Ceph Object Store is S3.

        See `the OpenStack Storage API overview page
        <https://docs.openstack.org/swift/latest/api/object_api_v1_overview.html>`_.

    OSD
    对象存储设备
        Probably :term:`Ceph OSD`, but not necessarily. Sometimes
        (especially in older correspondence, and especially in
        documentation that is not written specifically for Ceph), "OSD"
        means "**O**\bject **S**\torage **D**\evice", which refers to a
        physical or logical storage unit (for example: LUN). The Ceph
        community has always used the term "OSD" to refer to
        :term:`Ceph OSD Daemon` despite an industry push in the
        mid-2010s to insist that "OSD" should refer to "Object Storage
        Device", so it is important to know which meaning is intended.

    OSD, flapping
        见 :term:`Flapping OSD`.

    OSD FSID
        The OSD fsid is a unique identifier that is used to identify an
        OSD. It is found in the OSD path in a file called ``osd_fsid``.
        The term ``FSID`` is used interchangeably with ``UUID``.
        这是一个唯一的标识符，用以提高一个 OSD 的唯一性，它位于 OSD 路径内、
        一个名为 ``osd_fsid`` 的文件里。这个 ``fsid`` 可以和 ``uuid`` 互换着用。

    OSD ID
        定义一个 OSD 的整数。它是在新建 OSD 期间由监视器们生成的。

    OSD UUID
        就像 OSD fsid ，这是 OSD 的唯一标识符，并且可以和 ``fsid`` 互换着用。

    Period
        In the context of :term:`RGW`, a period is the configuration
        state of the :term:`Realm`. The period stores the configuration
        state of a multi-site configuration. When the period is updated,
        the "epoch" is said thereby to have been changed.

    Placement Groups (PGs)
    归置组（ PGs ）
        Placement groups (PGs) are subsets of each logical Ceph pool.
        Placement groups perform the function of placing objects (as a
        group) into OSDs. Ceph manages data internally at
        placement-group granularity: this scales better than would
        managing individual (and therefore more numerous) RADOS
        objects. A cluster that has a larger number of placement groups
        (for example, 100 per OSD) is better balanced than an otherwise
        identical cluster with a smaller number of placement groups.

        Ceph's internal RADOS objects are each mapped to a specific
        placement group, and each placement group belongs to exactly
        one Ceph pool.

    PLP
            **P**\ower **L**\oss **P**\rotection. A technology that
            protects the data of solid-state drives by using capacitors to
            extend the amount of time available for transferring data from
            the DRAM cache to the SSD's permanent memory. Consumer-grade
            SSDs are rarely equipped with PLP.

    :ref:`Pool<rados_pools>`
    存储池
        A pool is a logical partition used to store objects.

    Pools
        See :term:`pool`.

    :ref:`Primary Affinity <rados_ops_primary_affinity>`
    主亲和性
        The characteristic of an OSD that governs the likelihood that
        a given OSD will be selected as the primary OSD (or "lead
        OSD") in an acting set. Primary affinity was introduced in
        Firefly (v. 0.80). See :ref:`Primary Affinity
        <rados_ops_primary_affinity>`.

    :ref:`Prometheus <mgr-prometheus>`
            An open-source monitoring and alerting toolkit. Ceph offers a
            :ref:`"Prometheus module" <mgr-prometheus>`, which provides a
            Prometheus exporter that passes performance counters from a
            collection point in ``ceph-mgr`` to Prometheus.

    Quorum
    法定人数
        Quorum is the state that exists when a majority of the
        :ref:`Monitors<arch_monitor>` in the cluster are ``up``. A
        minimum of three :ref:`Monitors<arch_monitor>` must exist in
        the cluster in order for Quorum to be possible.

    RADOS
        **R**\eliable **A**\utonomic **D**\istributed **O**\bject
        **S**\tore. RADOS is the object store that provides a scalable
        service for variably-sized objects. The RADOS object store is
        the core component of a Ceph cluster.  `This blog post from
        2009
        <https://ceph.io/en/news/blog/2009/the-rados-distributed-object-store/>`_
        provides a beginner's introduction to RADOS. Readers interested
        in a deeper understanding of RADOS are directed to `RADOS: A
        Scalable, Reliable Storage Service for Petabyte-scale Storage
        Clusters <https://ceph.io/assets/pdfs/weil-rados-pdsw07.pdf>`_.

    RADOS Cluster
    RADOS 集群
        A proper subset of the Ceph Cluster consisting of
        :term:`OSD`\s, :term:`Ceph Monitor`\s, and :term:`Ceph
        Manager`\s.

    RADOS Gateway
    RADOS 网关
        See :term:`RGW`.

    RBD
        **R**\ADOS **B**\lock **D**\evice. See :term:`Ceph Block
        Device`.

    :ref:`Realm<rgw-realms>`
        In the context of RADOS Gateway (RGW), a realm is a globally
        unique namespace that consists of one or more zonegroups.

    Releases
    版本

        Ceph Interim Release
        Ceph 临时发布
            尚未通过质检测试、但包含新功能的 Ceph 版本。
            A version of Ceph that has not yet been put through
            quality assurance testing. May contain new features.

        Ceph Point Release
            Any ad hoc release that includes only bug fixes and
            security fixes.
            所有只包含缺陷或安全修正的特殊版本。

        Ceph Release
            Any distinct numbered version of Ceph.

        Ceph Release Candidate
            A major version of Ceph that has undergone initial
            quality assurance testing and is ready for beta
            testers.

        Ceph Stable Release
            A major version of Ceph where all features from the
            preceding interim releases have been put through
            quality assurance testing successfully.

    Reliable Autonomic Distributed Object Store
        The core set of storage software which stores the user's data
        (MON+OSD). See also :term:`RADOS`.

    :ref:`RGW<object-gateway>`
        **R**\ADOS **G**\ate\ **w**\ay.

        Also called "Ceph Object Gateway". The component of Ceph that
        provides a gateway to both the Amazon S3 RESTful API and the
        OpenStack Swift API.

    S3
            In the context of Ceph, S3 is one of the two APIs supported by
            the Ceph Object Store. The other API supported by the Ceph
            Object Store is OpenStack Swift.

            See `the Amazon S3 overview page
            <https://aws.amazon.com/s3/>`_.

    scrubs
        The processes by which Ceph ensures data integrity. During the
        process of scrubbing, Ceph generates a catalog of all objects
        in a placement group, then ensures that none of the objects are
        missing or mismatched by comparing each primary object against
        its replicas, which are stored across other OSDs. Any PG
        is determined to have a copy of an object that is different
        than the other copies or is missing entirely is marked
        "inconsistent" (that is, the PG is marked "inconsistent").

        There are two kinds of scrubbing: light scrubbing and deep
        scrubbing (also called "shallow scrubbing" and "deep scrubbing",
        respectively). Light scrubbing is performed daily and does
        nothing more than confirm that a given object exists and that
        its metadata is correct. Deep scrubbing is performed weekly and
        reads the data and uses checksums to ensure data integrity.

        See :ref:`Scrubbing <rados_config_scrubbing>` in the RADOS OSD
        Configuration Reference Guide and page 141 of *Mastering Ceph,
        second edition* (Fisk, Nick. 2019).

    secrets
        Secrets are credentials used to perform digital authentication
        whenever privileged users must access systems that require
        authentication. Secrets can be passwords, API keys, tokens, SSH
        keys, private certificates, or encryption keys.

    SDS
        **S**\oftware-**d**\efined **S**\torage.

    systemd oneshot
        一种 systemd 类型 ``type`` ，用于确定 ``ExecStart`` 的命令完成后是否退出
        （不想把它作为守护进程）。

    Swift
            See :term:`OpenStack Swift`.

    Teuthology
    测试方法学
        对 Ceph 进行脚本化测试的一系列软件。

    User
        An individual or a system actor (for example, an application)
        that uses Ceph clients to interact with the :term:`Ceph Storage
        Cluster`. See :ref:`User<rados-ops-user>` and :ref:`User
        Management<user-management>`.

    Zone
        In the context of :term:`RGW`, a zone is a logical group that
        consists of one or more :term:`RGW` instances.  A zone's
        configuration state is stored in the :term:`period`. See
        :ref:`Zones<radosgw-zones>`.


.. _https://github.com/ceph: https://github.com/ceph
.. _集群运行图: ../architecture#cluster-map
