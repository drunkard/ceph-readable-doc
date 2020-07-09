.. _ceph-file-system:

===============
 Ceph 文件系统
===============

Ceph 文件系统（ CephFS ）是个与 POSIX 标准兼容的文\件系统，\
它使用 Ceph 存储集群来存储数据。 Ceph 文件系统与 Ceph 块设备、\
同时提供 S3 和 Swift API 的 Ceph 对象存储、或者原生库\
（ librados ）一样，都使用着相同的 Ceph 存储集群系统。

.. note:: 如果你是第一次评测 CephFS ，请仔细看看部署的最佳实践\
   文档： :doc:`/cephfs/best-practices` 。

.. ditaa::
            +-----------------------+  +------------------------+
            |                       |  |      CephFS FUSE       |
            |                       |  +------------------------+
            |                       |
            |                       |  +------------------------+
            |  CephFS Kernel Object |  |     CephFS Library     |
            |                       |  +------------------------+
            |                       |
            |                       |  +------------------------+
            |                       |  |        librados        |
            +-----------------------+  +------------------------+

            +---------------+ +---------------+ +---------------+
            |      OSDs     | |      MDSs     | |    Monitors   |
            +---------------+ +---------------+ +---------------+


CephFS 使用文档
===============

Ceph 文件系统要求 Ceph 存储集群内至少有一个 :term:`Ceph 元数据服务器`\ 。


.. raw:: html

        <style type="text/css">div.body h3{margin:5px 0px 0px 0px;}</style>
        <table cellpadding="10"><colgroup><col width="33%"><col width="33%"><col width="33%"></colgroup><tbody valign="top"><tr><td><h3>步骤一：元数据服务器</h3>

要运行 Ceph 文件系统，你必须先装起至少带一个 :term:`Ceph 元数据服务器`\
的 Ceph 存储集群。


.. toctree::
        :maxdepth: 1

	配备、增加、删除 MDS <add-remove-mds>
        MDS 故障切换和灾备配置 <standby>
        MDS 配置选项 <mds-config-ref>
	客户端配置选项 <client-config-ref>
        Journaler 配置 <journaler>
        ceph-mds 手册页 <../../man/8/ceph-mds>

.. raw:: html

        </td><td><h3>步骤二：挂载 CephFS</h3>

一旦有了健康的 Ceph 存储集群，及其配套的元数据服务器，你就可以\
创建并挂载自己的 Ceph 文件系统了。首先确认下你的客户端的网络连\
通性和认证密钥。

.. toctree::
   :maxdepth: 1
   :hidden:

    创建 CephFS 文件系统 <createfs>
    挂载 CephFS 文件系统 <kernel>
    把 CephFS 挂载为 FUSE <fuse>
    通过 fstab 挂载 CephFS <fstab>
    CephFS Shell 的用法 <cephfs-shell>
    内核驱动支持的功能 <kernel-features>
    ceph-fuse 手册页 <../../man/8/ceph-fuse>
    mount.ceph 手册页 <../../man/8/mount.ceph>
    mount.fuse.ceph 手册页 <../../man/8/mount.fuse.ceph>


.. raw:: html

        </td><td><h3>其它详细信息</h3>

.. toctree::
    :maxdepth: 1
    :hidden:

    最佳部署实践 <best-practices>
    CephFS 管理命令 <administration>
    深入理解 MDS 的缓存尺寸限制 <cache-size-limits>
    MDS 的各种状态 <mds-states>
    POSIX 兼容性 <posix>
    实验性功能 <experimental-features>
    CephFS 配额管理 <quota>
    在 Ceph 上使用 Hadoop <hadoop>
    cephfs-journal-tool <cephfs-journal-tool>
    文件布局 <file-layouts>
    驱逐客户端 <eviction>
    文件系统占满的处理 <full>
    健康消息 <health-messages>
    故障排除 <troubleshooting>
    灾难恢复 <disaster-recovery>
    客户端认证 <client-auth>
    旧文件系统的升级 <upgrading>
    目录分片的配置 <dirfrags>
    多活 MDS 的配置 <multimds>
    通过 NFS 导出 <nfs>
    应用最佳实践 <app-best-practices>
    洗刷 <scrub>
    LazyIO <lazyio>
    分布式元数据缓存 <mdcache>
    FS 卷和子卷 <fs-volumes>
    CephFS 内的动态元数据管理 <dynamic-metadata-management>
    CephFS IO 路径 <cephfs-io-path>

.. raw:: html

   <!---

元数据修复
^^^^^^^^^^

.. raw:: html

   --->

.. toctree:: 
   :hidden:

    高级话题：元数据修复 <disaster-recovery-experts>


.. raw:: html

   <!---


.. Developer Guides

开发者指南
==========

.. raw:: html

   --->

.. toctree:: 
   :maxdepth: 1
   :hidden:

    Journaler 配置 <journaler>
    客户端的能力 <capabilities>
    Java 和 Python 捆绑库 <api/index>
    Mantle <mantle>


.. raw:: html

   <!---
