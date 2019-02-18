==============
 安装（手动）
==============


获取软件
========

获取 Ceph 软件的方法有多种，最简单、通用的\ `获取软件包`_\ 方\
法是添加软件源之后通过包管理工具（像 APT 、 YUM ）操作；也可以\
直接从 Ceph 仓库下载预编译软件包；最后，你可以下载源码包或克隆
Ceph 源码库、并自行编译。


.. toctree::
   :maxdepth: 1 

	获取二进制包 <get-packages>
	获取源码包 <get-tarballs>
	克隆源码 <clone-source>
	构建 Ceph <build-ceph>
	Ceph 镜像 <mirrors>


安装软件
========

获取到（或者软件库里有） Ceph 软件包之后，安装很简单。要在集群\
内的各\ :term:`节点`\ 安装，你可以用 ``ceph-deploy`` 、或者\
包管理工具。如果你想安装 Ceph 对象网关、或 QEMU ，那么对于使用
``yum`` 的发行版，你应该设置软件库优先级。

.. toctree:: 
   :maxdepth: 1

	安装 ceph-deploy <install-ceph-deploy>
	安装 Ceph 存储集群 <install-storage-cluster>
	安装 Ceph 对象网关 <install-ceph-gateway>
	作为虚拟化的块设备 <install-vm-cloud>


手动部署集群
============

把 Ceph 二进制包安装到各节点后，你也可以手动部署集群。手动过程\
主要是让部署脚本（如 Chef、Juju、Puppet 等）的开发者们验证程序\
的。

.. toctree:: 
   
	手动部署 <manual-deployment>
	在 FreeBSD 上手动部署 <manual-freebsd-deployment>


升级软件
========

随着新版 Ceph 的发布，你也许想升级集群以使用新功能，升级集群前\
请参考升级文档。有时候，升级 Ceph 需要严格按照升级顺序进行。

.. toctree::
   :maxdepth: 2

   升级 Ceph <upgrading-ceph>


.. _获取软件包: ../install/get-packages
