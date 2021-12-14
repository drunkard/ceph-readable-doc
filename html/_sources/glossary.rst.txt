===========
 Ceph 术语
===========

Ceph 在迅速成长。随着企业对 Ceph 的采用，技术性的术语如 RADOS 、 RBD 、 RGW
等等需要对应的营销用语，用以解释各个组件是干什么的。
这个术语表里的术语是用以完善已有技术性术语的。

有时候，一个定义有不止一个术语。通常，
第一个术语反映的是 Ceph 的营销用语，
第二个术语反映的是技术性术语、或者引用 Ceph 系统的遗留方式。


.. glossary:: 

    Ceph Project
    Ceph 项目
        关于 Ceph 的团队、软件、任务和基础架构的统称。

    cephx
        Ceph 的认证协议， Cephx 的运行机制类似 Kerberos ，但它没有单故障点。

    Ceph
    Ceph Platform
    Ceph 平台
        所有与 Ceph 相关的软件，包括所有位于 `https://github.com/ceph`_ 的源码。

    Ceph System
    Ceph Stack
    Ceph 系统
    Ceph 软件栈
        Ceph 之中两个或更多组件的组合。

    Ceph Node
    Node
    Host
    Ceph 节点
    节点
    主机
        Ceph 系统内的任意单体机器或服务器。

    Ceph Storage Cluster
    Ceph Object Store
    RADOS
    RADOS Cluster
    Reliable Autonomic Distributed Object Store
    Ceph 存储集群
    Ceph 对象存储库
    RADOS 集群
    可靠自主的分布式对象存储库
        存储软件的核心部分，存储着用户的数据（ MON+OSD ）。

    Ceph Cluster Map
    Cluster Map
    Ceph 集群运行图
    集群运行图
        一系列的运行图，包括监视器运行图、 OSD 运行图、 PG 图、 MDS 运行图\
        和 CRUSH 图。详情见 `集群运行图`_ 。

    Ceph Object Storage
    Ceph 对象存储
        对象存储“产品”、服务或者能力，至少由 Ceph 存储集群和一个 Ceph 对象网关组成。

    Ceph Object Gateway
    RADOS Gateway
    RGW
    Ceph 对象网关
    RADOS 网关
        Ceph 的 S3/Swift 网关组件。

    Ceph Block Device
    RBD
    Ceph 块设备
        Ceph 的块存储组件。

    Ceph Block Storage
    Ceph 块存储
        块存储“产品”、服务或者能力，可以配合使用的有 ``librbd`` 、
        虚拟化管理程序如 QEMU 或 Xen 、和虚拟化管理程序的抽象层，如 ``libvirt`` 。

    Ceph Filesystem
    CephFS
    Ceph FS
    Ceph 文件系统
        Ceph 的组件之一，与 POSIX 兼容的文件系统。详情见
        :ref:`CephFS 体系结构 <arch-cephfs>` 和 :ref:`ceph-file-system` 。

    Cloud Platforms
    Cloud Stacks
    云平台
    云软件栈
        第三方云服务平台，比如 OpenStack, CloudStack, OpenNebula, Proxmox VE 等等。

    Object Storage Device
    OSD
    对象存储设备
        一个物理的或逻辑的存储单元（如 LUN ）。有时候， Ceph 用户以术语 OSD 来引用
        :term:`Ceph OSD 守护进程`\ ，然而恰当的术语应该是 Ceph OSD 。

    Ceph OSD Daemon
    Ceph OSD Daemons
    Ceph OSD
    Ceph 对象存储守护进程
    Ceph OSD 守护进程
        Ceph 的 OSD 软件，它与逻辑磁盘（ :term:`OSD` ）交互。有时候，
        Ceph 用户们用 “OSD” 这个术语来指代 “Ceph OSD Daemon”，
        然而正确的术语是 “Ceph OSD”。

    OSD id
        定义一个 OSD 的整数。它是在新建 OSD 期间由监视器们生成的。

    OSD fsid
        这是一个唯一的标识符，用以提高一个 OSD 的唯一性，它位于 OSD 路径内、
        一个名为 ``osd_fsid`` 的文件里。这个 ``fsid`` 可以和 ``uuid`` 互换着用。

    OSD uuid
        就像 OSD fsid ，这是 OSD 的唯一标识符，并且可以和 ``fsid`` 互换着用。

    bluestore
        OSD BlueStore 是 OSD 守护进程的一个新后端（ kraken 及其后续版本）。
        不像 :term:`filestore` ，它是直接在给 Ceph 使用的块设备上存储对象的，
        不需要任何文件系统接口。

    filestore
        OSD 守护进程的一个后端，它需要日志、且文件是写入文件系统的。

    Ceph Monitor
    MON
    Ceph 监视器
    监视器
        Ceph 的监视器软件。

    Ceph Manager
    MGR
    Ceph 管理器
    管理器
        Ceph 管理器软件，它会把整个集群的所有状态信息收集到一起。

    Ceph Manager Dashboard
    Ceph Dashboard
    Dashboard Module
    Dashboard Plugin
    Dashboard
    Ceph 管理器仪表盘
    Ceph 仪表盘
    仪表盘模块
    仪表盘插件
    仪表盘
        一个内建的、基于网页的 Ceph 管理和监控应用程序，可用于管理集群的各方面以及对象。
        仪表盘是以一个 Ceph 管理器模块实现的。详情见 :ref:`mgr-dashboard` 。

    Ceph Metadata Server
    MDS
    Ceph 元数据服务器
    元数据服务器
        Ceph 的元数据软件。

    Ceph Clients
    Ceph Client
    Ceph 客户端
        一组能够访问 Ceph 存储集群的 Ceph 组件，其中包括 Ceph 对象网关、
        Ceph 块设备、 Ceph 文件系统、及其对应的库、内核模块和 FUSE 。

    Ceph Kernel Modules
    Ceph 内核模块
        一组能够成功和 Ceph 系统交互的内核模块（比如 ``ceph.ko`` 、 ``rbd.ko`` ）。

    Ceph Client Libraries
    Ceph 客户端库
        一组能够成功和相应 Ceph 组件交互的库。

    Ceph Release
    Ceph 发布
        任何用不同数字编号的 Ceph 版本。

    Ceph Point Release
    Ceph 修正版
    Ceph 小版本
        所有只包含缺陷或安全修正的特殊版本。

    Ceph Interim Release
    Ceph 临时发布
        尚未通过质检测试、但包含新功能的 Ceph 版本。

    Ceph Release Candidate
    Ceph 预发布
        Ceph 的一个主要版本，通过了基本的质检测试、并能够交付给普通测试者。

    Ceph Stable Release
    Ceph 稳定版
        Ceph 的一个主要版本，前面临时版本的所有功能都成功通过了质检测试。

    Ceph Test Framework
    Teuthology
    Ceph 测试框架
    测试方法学
        对 Ceph 进行脚本化测试的一系列软件。

    CRUSH
        Controlled Replication Under Scalable Hashing ，可伸缩哈希控制的复制。
        它是 Ceph 用以计算对象存储位置的算法。

    CRUSH rule
    CRUSH 规则
        应用到某个特定存储池（们）的 CRUSH 数据归置规则。

    Pool
    Pools
    存储池
        池是对象存储的逻辑部分。

    systemd oneshot
        一种 systemd 类型 ``type`` ，用于确定 ``ExecStart`` 的命令完成后是否退出
        （不想把它作为守护进程）。

    LVM tags
    LVM 标签
        LVM 卷和组的可扩展元数据，我们用它来存储 Ceph 相关的信息，
        如各设备、以及它们与 OSD 的关系。


.. _https://github.com/ceph: https://github.com/ceph
.. _集群运行图: ../architecture#cluster-map
