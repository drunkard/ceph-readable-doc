.. _ceph-file-system:

===============
 Ceph 文件系统
===============

Ceph 文件系统、或者 **CephFS** ，是个与 POSIX 标准兼容的文件系统，
它建立在 Ceph 的分布式对象存储 -- **RADOS** 之上。
CephFS 致力于为各种应用程序（包括传统应用场景，
像共享家目录、 HPC 暂存空间、和分布式工作流的共享存储），
提供技术先进的、多用途的、高可用的、而且性能优异的文件存储功能

CephFS 通过采用一些新颖的架构实现了这些目标。
特别是，文件元数据存储在与文件数据相独立的 RADOS 存储池内、
并通过尺寸可变的 *元数据服务器* （缩写 **MDS** ）集群来提供服务，
集群扩容就能支持更高的元数据载荷吞吐量。
这个文件系统的客户端是直连到 RADOS 来读取和写入文件数据块的。
正因为这样，工作载荷可以随底层 RADOS 存储的规模线性增长；
就是说，没有网关或者代理为客户端转接数据 I/O 。

到数据的访问是通过集群的 MDS 来协调的，
客户端和 MDS 共同维护分布式的元数据缓存，而 MDS 是权威。
对元数据的变更由各个 MDS 汇总成一系列、并高效地写入 RADOS 内的日志；
MDS 本身不会存储元数据状态。这种模式使客户端们能够连贯且快速地协作，
而且还是在 POSIX 文件系统的范畴内。

.. image:: cephfs-architecture.svg

CephFS 因其新颖的设计、和对文件系统的研究而成了很多学术论文的主题。
它是 Ceph 里最老的存储接口，而且曾经还是 RADOS 的主要用途。
现在，又加入了两个其它的存储接口，
形成了一个现代的、大一统的存储系统：
RBD （ Ceph 块设备）和 RGW （Ceph 对象存储网关）。


CephFS 入门
^^^^^^^^^^^
.. Getting Started with CephFS

对绝大多数部署好的 Ceph ，建立一个 CephFS 文件系统都如此简单：

.. prompt:: bash

    # Create a CephFS volume named (for example) "cephfs":
    ceph fs volume create cephfs

如果后端的部署技术支持的话（见 `编排器部署表`_ ），
Ceph `编排器`_ 会自动给你的文件系统创建和配置 MDS ，
否则，请 `根据需要手动部署 MDS`_ 。

最后，要想在你的客户端节点上挂载 CephFS ，
参见 `挂载 CephFS ：先决条件`_ 。另外，还有一个命令行的 shell 工具，
可以用 `cephfs-shell`_ 交互地访问或者编写脚本。

.. _编排器: ../mgr/orchestrator
.. _根据需要手动部署 MDS: add-remove-mds
.. _编排器部署表: ../mgr/orchestrator/#current-implementation-status
.. _挂载 CephFS ：先决条件: mount-prerequisites
.. _cephfs-shell: cephfs-shell


.. raw:: html

   <!---

管理
^^^^
.. Administration

.. raw:: html

   --->

.. toctree:: 
   :maxdepth: 1
   :hidden:

    创建 CephFS 文件系统 <createfs>
    管理命令 <administration>
    创建多个文件系统 <multifs>
    配备、增加、删除 MDS <add-remove-mds>
    MDS 故障切换和灾备配置 <standby>
    MDS 缓存配置 <cache-configuration>
    MDS 配置选项 <mds-config-ref>
    ceph-mds 手册页 <../../man/8/ceph-mds>
    通过 NFS 导出 <nfs>
    应用最佳实践 <app-best-practices>
    FS 卷和子卷 <fs-volumes>
    CephFS 配额管理 <quota>
    健康消息 <health-messages>
    升级旧文件系统 <upgrading>
    CephFS Top 工具 <cephfs-top>
    定时快照 <snap-schedule>
    CephFS 快照镜像 <cephfs-mirroring>

.. raw:: html

   <!---

挂载 CephFS
^^^^^^^^^^^
.. Mounting CephFS

.. raw:: html

   --->

.. toctree:: 
   :maxdepth: 1
   :hidden:

    客户端配置选项 <client-config-ref>
    客户端认证 <client-auth>
    挂载 CephFS: 前提条件 <mount-prerequisites>
    用内核驱动挂载 CephFS 文件系统 <mount-using-kernel-driver>
    用 FUSE 挂载 CephFS <mount-using-fuse>
    在 Windows 上挂载 CephFS <ceph-dokan>
    CephFS Shell 的用法 <../../man/8/cephfs-shell>
    内核驱动支持的功能 <kernel-features>
    ceph-fuse 手册页 <../../man/8/ceph-fuse>
    mount.ceph 手册页 <../../man/8/mount.ceph>
    mount.fuse.ceph 手册页 <../../man/8/mount.fuse.ceph>

.. raw:: html

   <!---

CephFS 内幕
^^^^^^^^^^^
.. CephFS Concepts

.. raw:: html

   --->

.. toctree:: 
   :maxdepth: 1
   :hidden:

    MDS 的各种状态 <mds-states>
    POSIX 兼容性 <posix>
    MDS 日志记录 <mds-journaling>
    文件布局 <file-layouts>
    分布式元数据缓存 <mdcache>
    CephFS 内的动态元数据管理 <dynamic-metadata-management>
    CephFS IO 路径 <cephfs-io-path>
    LazyIO <lazyio>
    目录分片的配置 <dirfrags>
    多活 MDS 的配置 <multimds>


.. raw:: html

   <!---

故障排除和灾难恢复
^^^^^^^^^^^^^^^^^^
.. Troubleshooting and Disaster Recovery

.. raw:: html

   --->

.. toctree:: 
   :hidden:

    驱逐客户端 <eviction>
    洗刷 <scrub>
    文件系统占满的处理 <full>
    元数据修复 <disaster-recovery-experts>
    故障排除 <troubleshooting>
    灾难恢复 <disaster-recovery>
    cephfs-journal-tool <cephfs-journal-tool>
    监视器存储丢失后恢复文件系统 <recover-fs-after-mon-store-loss>


.. raw:: html

   <!---

开发者指南
==========
.. Developer Guides

.. raw:: html

   --->

.. toctree:: 
   :maxdepth: 1
   :hidden:

    Journaler 配置 <journaler>
    客户端的能力 <capabilities>
    Java 和 Python 捆绑库 <api/index>
    Mantle <mantle>
    Metrics <metrics>


.. raw:: html

   <!---

更多细节
^^^^^^^^
.. Additional Details

.. raw:: html

   --->

.. toctree::
   :maxdepth: 1
   :hidden:

    实验性功能 <experimental-features>
