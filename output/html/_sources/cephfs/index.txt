===============
 Ceph 文件系统
===============

:term:`Ceph 文件系统`\ 是个 POSIX 兼容的文件系统，它使用 Ceph 存储集群来存储数\
据。 Ceph FS 使用的 Ceph 存储集群系统和 Ceph 块设备及 Ceph 对象存储相同，就像 \
Ceph 对象存储也有 S3 和 Swift API 、或本地库（ librados ）一样。

.. important:: 现在还不建议 Ceph 文件系统用于生产环境数据。

.. ditaa::
            +-----------------------+  +------------------------+
            | CephFS Kernel Object  |  |      CephFS FUSE       |
            +-----------------------+  +------------------------+

            +---------------------------------------------------+
            |            Ceph FS Library (libcephfs)            |
            +---------------------------------------------------+

            +---------------------------------------------------+
            |      Ceph Storage Cluster Protocol (librados)     |
            +---------------------------------------------------+

            +---------------+ +---------------+ +---------------+
            |      OSDs     | |      MDSs     | |    Monitors   |
            +---------------+ +---------------+ +---------------+


Ceph 文件系统要求 Ceph 存储集群内至少有一个 :term:`Ceph 元数据服务器`\ 。


.. raw:: html

	<style type="text/css">div.body h3{margin:5px 0px 0px 0px;}</style>
	<table cellpadding="10"><colgroup><col width="33%"><col width="33%"><col width="33%"></colgroup><tbody valign="top"><tr><td><h3>步骤一：元数据服务器</h3>

要运行 Ceph 文件系统，你必须先装起至少带一个 :term:`Ceph 元数据服务器`\ 的 Ceph \
存储集群。


.. toctree::
	:maxdepth: 1

	增加/删除 MDS <../../rados/deployment/ceph-deploy-mds>
	MDS 配置 <mds-config-ref>
	Journaler 配置 <journaler>
	ceph-mds 手册页 <../../man/8/ceph-mds>

.. raw:: html

	</td><td><h3>步骤二：挂载 Ceph FS</h3>

一旦有了健康的 Ceph 存储集群，及其配套的元数据服务器，你就可以创建并挂载自己的 \
Ceph 文件系统了。首先确认下你的客户端的网络连通性和认证密钥。

.. toctree::
	:maxdepth: 1

	创建 Ceph 文件系统 <createfs>
	挂载 Ceph 文件系统 <kernel>
	把 Ceph FS 挂载为 FUSE <fuse>
	通过 fstab 挂载 Ceph FS <fstab>
	cephfs 手册页 <../../man/8/cephfs>
	ceph-fuse 手册页 <../../man/8/ceph-fuse>
	mount.ceph 手册页 <../../man/8/mount.ceph>


.. raw:: html

	</td><td><h3>其它详细信息</h3>

.. toctree::
	:maxdepth: 1

	在 Ceph 上使用 Hadoop <hadoop>
	libcephfs <../../api/libcephfs-java/>
	cephfs-journal-tool <cephfs-journal-tool>
	文件布局 <file-layouts>
        驱逐客户端 <eviction>
        文件系统占满的处理 <full>
	故障排除 <troubleshooting>

.. raw:: html

	</td></tr></tbody></table>
