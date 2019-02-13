===============
 Ceph 文件系统
===============

:term:`Ceph 文件系统`\ （ Ceph FS ）是个与 POSIX 标准兼容的文\
件系统，它使用 Ceph 存储集群来存储数据。 Ceph 文件系统与 Ceph
块设备、同时提供 S3 和 Swift API 的 Ceph 对象存储、或者原生库\
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

        增加/删除 MDS <../../rados/deployment/ceph-deploy-mds>
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

        创建 CephFS 文件系统 <createfs>
        挂载 CephFS 文件系统 <kernel>
        把 CephFS 挂载为 FUSE <fuse>
        通过 fstab 挂载 CephFS <fstab>
        ceph-fuse 手册页 <../../man/8/ceph-fuse>
        mount.ceph 手册页 <../../man/8/mount.ceph>


.. raw:: html

        </td><td><h3>其它详细信息</h3>

.. toctree::
        :maxdepth: 1

        最佳部署实践 <best-practices>
        CephFS 管理命令 <administration>
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

.. raw:: html

        </td></tr></tbody></table>

.. _For developers:

开发者文档
==========

.. toctree:: 
    :maxdepth: 1

    客户端的能力 <capabilities>
    libcephfs <../../api/libcephfs-java/>
    Mantle <mantle>
