===========
 Ceph 术语
===========

Ceph is growing rapidly. As firms deploy Ceph, the technical terms such as
"RADOS", "RBD," "RGW" and so forth require corresponding marketing terms
that explain what each component does. The terms in this glossary are 
intended to complement the existing technical terminology.

Sometimes more than one term applies to a definition. Generally, the first
term reflects a term consistent with Ceph's marketing, and secondary terms
reflect either technical terms or legacy ways of referring to Ceph systems.


.. glossary:: 

	Ceph Project
        Ceph 项目
		关于 Ceph 的团队、软件、任务和基础架构的统称。

	cephx
		Ceph 的认证协议， Cephx 的运行机制类似 Kerberos ，但它没有单故障点。

	Ceph
	Ceph Platform
        Ceph 平台
		所有与 Ceph 相关的软件，包括所有位于 `https://github.com/ceph`_ \
		的源码。

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
        Ceph 对象存储
        RADOS 集群
        可靠自主的分布式对象存储
		The core set of storage software which stores the user's data (MON+OSD).

	Ceph Cluster Map
	cluster map
        Ceph 集群运行图
        集群运行图
		The set of maps comprising the monitor map, OSD map, PG map, MDS map and 
		CRUSH map. See `Cluster Map`_ for details.

	Ceph Object Storage
        Ceph 对象存储
		The object storage "product", service or capabilities, which consists
		essentially of a Ceph Storage Cluster and a Ceph Object Gateway.

	Ceph Object Gateway
	RADOS Gateway
	RGW
        Ceph 对象网关
        RADOS 网关
		The S3/Swift gateway component of Ceph.

	Ceph Block Device
	RBD
        Ceph 块设备
        RBD
		The block storage component of Ceph.

	Ceph Block Storage
        Ceph 块存储
		The block storage "product," service or capabilities when used in 
		conjunction with ``librbd``, a hypervisor such as QEMU or Xen, and a
		hypervisor abstraction layer such as ``libvirt``.

	Ceph Filesystem
	CephFS
	Ceph FS
        Ceph 文件系统
		Ceph 的组件之一，与 POSIX 兼容的文件系统。详情\
                见 :ref:`CephFS 体系结构 <arch-cephfs>` 和
                :ref:`ceph-file-system` 。

	Cloud Platforms
	Cloud Stacks
        云平台
        云软件栈
		Third party cloud provisioning platforms such as OpenStack, CloudStack, 
		OpenNebula, ProxMox, etc.

	Object Storage Device
	OSD
        对象存储设备
                一个物理的或逻辑的存储单元（如 LUN ）。
                有时候， Ceph 用户以术语 OSD 来引用 :term:`Ceph OSD 守护进程`\ ，\
                然而恰当的术语应该是 Ceph OSD 。

	Ceph OSD Daemon
	Ceph OSD Daemons
	Ceph OSD
        Ceph 对象存储守护进程
        Ceph OSD 守护进程
		The Ceph OSD software, which interacts with a logical
		disk (:term:`OSD`). Sometimes, Ceph users use the
		term "OSD" to refer to "Ceph OSD Daemon", though the
		proper term is "Ceph OSD".

	OSD id
                定义一个 OSD 的整数。它是在新建 OSD 期间由\
                监视器们生成的。

	OSD fsid
		This is a unique identifier used to further improve the uniqueness of an
		OSD and it is found in the OSD path in a file called ``osd_fsid``. This
		``fsid`` term is used interchangeably with ``uuid``

	OSD uuid
		Just like the OSD fsid, this is the OSD unique identifier and is used
		interchangeably with ``fsid``

	bluestore
		OSD BlueStore is a new back end for OSD daemons (kraken and newer
		versions). Unlike :term:`filestore` it stores objects directly on the
		Ceph block devices without any file system interface.

	filestore
                OSD 守护进程的一个后端，它需要日志、且文件是\
                写入文件系统的。

	Ceph Monitor
	MON
        Ceph 监视器
        监视器
		The Ceph monitor software.

	Ceph Manager
	MGR
        Ceph 管理器
        管理器
                Ceph 管理器软件，它会把整个集群的所有状态信息\
                收集到一起。

	Ceph Manager Dashboard
	Ceph Dashboard
	Dashboard Plugin
	Dashboard
        Ceph 管理器仪表盘
        Ceph 仪表盘
        仪表盘插件
        仪表盘
                一个内建的、基于网页的 Ceph 管理和监控应用程序，\
                可用于管理集群的各方面以及对象。仪表盘是以一个
                Ceph 管理器模块实现的。详情见 :ref:`mgr-dashboard` 。

	Ceph Metadata Server
	MDS
        Ceph 元数据服务器
        元数据服务器
		The Ceph metadata software.

	Ceph Clients
	Ceph Client
        Ceph 客户端
		The collection of Ceph components which can access a Ceph Storage 
		Cluster. These include the Ceph Object Gateway, the Ceph Block Device, 
		the Ceph Filesystem, and their corresponding libraries, kernel modules, 
		and FUSEs.

	Ceph Kernel Modules
        Ceph 内核模块
		The collection of kernel modules which can be used to interact with the 
		Ceph System (e.g,. ``ceph.ko``, ``rbd.ko``).

	Ceph Client Libraries
        Ceph 客户端库
		The collection of libraries that can be used to interact with components 
		of the Ceph System.

	Ceph Release
        Ceph 发布
		Any distinct numbered version of Ceph.

	Ceph Point Release
        Ceph 修正版
        Ceph 小版本
		Any ad-hoc release that includes only bug or security fixes.

	Ceph Interim Release
        Ceph 临时发布
		Versions of Ceph that have not yet been put through quality assurance
		testing, but may contain new features.

	Ceph Release Candidate
        Ceph 预发布
		A major version of Ceph that has undergone initial quality assurance 
		testing and is ready for beta testers.

	Ceph Stable Release
        Ceph 稳定版
		A major version of Ceph where all features from the preceding interim 
		releases have been put through quality assurance testing successfully.

	Ceph Test Framework
	Teuthology
        Ceph 测试框架
        测试方法学
		对 Ceph 进行脚本化测试的一系列软件。

	CRUSH
		Controlled Replication Under Scalable Hashing. It is the algorithm
		Ceph uses to compute object storage locations.

	CRUSH rule
        CRUSH 规则
		The CRUSH data placement rule that applies to a particular pool(s).

	Pool
	Pools
        存储池
		池是对象存储的逻辑部分。

	systemd oneshot
                一种 systemd 类型 ``type`` ，用于确定
                ``ExecStart`` 的命令完成后是否退出（不想把它\
                作为守护进程）。

	LVM tags
        LVM 标签
		LVM 卷和组的可扩展元数据，我们用它来存储 Ceph
                相关的信息，如各设备、以及它们与 OSD 的关系。


.. _https://github.com/ceph: https://github.com/ceph
.. _Cluster Map: ../architecture#cluster-map
