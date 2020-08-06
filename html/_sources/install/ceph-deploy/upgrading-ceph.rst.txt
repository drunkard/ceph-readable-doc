===========
 升级 Ceph
===========

Ceph 的各个版本都可能有特定的步骤，升级前请参考与此版本相关的\
章节和\ `此版本的发布摘要`_\ 文档，以确定有哪些特定于此版本的\
步骤。


.. Summary

概述
====

你可以在 Ceph 集群在线且提供服务时升级守护进程！某些类型的\
守护进程依赖其他的，如 Ceph 元数据服务器和 Ceph 对象网关依赖于
Ceph 监视器和 OSD 守护进程，所以我们建议按以下顺序升级：

#. `ceph-deploy 工具`_
#. Ceph 监视器
#. Ceph OSD 守护进程
#. Ceph 元数据服务器
#. Ceph 对象网关

作为普适规则，我们建议一次升级一类的所有守护进程（如所有
``ceph-osd`` 、所有 ``ceph-mon`` 等），这样才能确保它们属于\
同一版本。还有，我们建议升级完集群内的所有守护进程之后，再拿此\
版本的新功能练手。

`升级过程`_\ 相对简单，但升级前务必看一下\ `此版本的发布摘要`_\
。基本过程有三个：

#. 在你的管理节点上用 ``ceph-deploy`` 为各主机升级软件包（用
   ``ceph-deploy install`` 命令），或者分别登录各主机\
   `用你发行版自带的包管理器`_\ 升级 Ceph 软件包。例如，\
   `升级监视器`_\ 时， ``ceph-deploy`` 语法大致如此： ::

	ceph-deploy install --release {release-name} ceph-node1[ ceph-node2]
	ceph-deploy install --release firefly mon1 mon2 mon3

   **注：** ``ceph-deploy install`` 命令会把指定节点上的旧版本\
   升级为你所指定的版本，没有 ``ceph-deploy upgrade`` 这样的\
   升级命令。

#. 登录各节点并重启各相关 Ceph 守护进程，详情参见\ `操纵集群`_ 。

#. 确认集群健康状况，详情见\ `监控集群`_\ 。

.. important:: 一旦升级完，就不能降级了。


.. Ceph Deploy

ceph-deploy 工具
================

升级 Ceph 守护进程前，应该先升级 ``ceph-deploy`` 工具。 ::

	sudo pip install -U ceph-deploy

或者： ::

	sudo apt-get install ceph-deploy

或者： ::

	sudo yum install ceph-deploy python-pushy


.. Upgrade Procedures

升级过程
========

下面是具体升级进程。

.. important:: Ceph 的各版本可能有不同的步骤，所以\ **升级前**\
   请参考此版本特定的升级步骤。


.. Upgrading Monitors

升级监视器
----------

要升级监视器，执行下列步骤：

#. 升级各守护进程的二进制包。

   你可以用 ``ceph-deploy`` 一次升级所有监视器节点，如： ::

	ceph-deploy install --release {release-name} ceph-node1[ ceph-node2]
	ceph-deploy install --release hammer mon1 mon2 mon3

   你也可以用包管理器挨个升级各节点。手动升级 Debian/Ubuntu
   主机上软件包的步骤如下： ::

	ssh {mon-host}
	sudo apt-get update && sudo apt-get install ceph

   在 CentOS/Red Hat 主机上相应的命令如下： ::

	ssh {mon-host}
	sudo yum update && sudo yum install ceph

#. 重启各监视器。 Debian/Ubuntu 发行版的命令如下： ::

	sudo restart ceph-mon id={hostname}

   CentOS/Red Hat/Debian 发行版的命令如下： ::

	sudo /etc/init.d/ceph restart {mon-id}

   用 ``ceph-deploy`` 部署的 CentOS/Red Hat 发行版，其监视器 ID
   通常是 ``mon.{hostname}`` 。

#. 确保各监视器都重回法定人数。 ::

	ceph mon stat

再次确认你完成了所有监视器的升级。


.. Upgrading an OSD

升级单个 OSD
------------

升级单个 OSD 守护进程的步骤如下：

#. 升级 OSD 守护进程对应的软件包。

   你可以用 ``ceph-deploy`` 一次升级所有 OSD 守护进程，如： ::

	ceph-deploy install --release {release-name} ceph-node1[ ceph-node2]
	ceph-deploy install --release hammer osd1 osd2 osd3

   你也可以\ `用你发行版自带的包管理器`_ 挨个升级各节点的\
   软件包。手动升级 Debian/Ubuntu 主机上软件包的步骤如下。 ::

	ssh {osd-host}
	sudo apt-get update && sudo apt-get install ceph

   在 CentOS/Red Hat 主机上相应的命令如下： ::

	ssh {osd-host}
	sudo yum update && sudo yum install ceph

#. 重启 OSD ，其中 ``N`` 是 OSD 号。对 Debian/Ubuntu ，用命令： ::

	sudo restart ceph-osd id=N

   对于一主机上的多个 OSD ，你可以用 Upstart 一次性全部重启。 ::

	sudo restart ceph-osd-all

   对于CentOS/Red Hat/Debian 发行版，用： ::

	sudo /etc/init.d/ceph restart N

#. 确保升级后的 OSD 重新加入了集群： ::

	ceph osd stat

再次确认所有 OSD 守护进程已升级完。


.. Upgrading a Metadata Server

升级单个元数据服务器
--------------------

要升级单个 Ceph 元数据服务器，挨个执行下列步骤：

#. 升级二进制包。你可以用 ``ceph-deploy`` 一次升级所有
   MDS 节点，也可以在各节点用包管理器升级，如： ::

	ceph-deploy install --release {release-name} ceph-node1
	ceph-deploy install --release hammer mds1

   在 Debian/Ubuntu 主机上可这样手动升级。 ::

	ssh {mon-host}
	sudo apt-get update && sudo apt-get install ceph-mds

   在 CentOS/Red Hat 主机上则是： ::

	ssh {mon-host}
	sudo yum update && sudo yum install ceph-mds

#. 重启元数据服务器。在 Debian/Ubuntu 上用： ::

	sudo restart ceph-mds id={hostname}

   在 CentOS/Red Hat/Debian 上用： ::

	sudo /etc/init.d/ceph restart mds.{hostname}

   用 ``ceph-deploy`` 部署的集群其 ``{hostname}`` 通常是所在\
   主机的主机名。

#. 确保元数据服务器已启动，且运行着： ::

	ceph mds stat


.. Upgrading a Client

升级客户端
----------

升级软件包并重启完集群之后，我们建议同时升级下客户端节点上的
``ceph-common`` 和客户端库（ ``librbd1`` 和 ``librados2`` ）。

#. 升级软件包： ::

	ssh {client-host}
	apt-get update && sudo apt-get install ceph-common librados2 librbd1 python-rados python-rbd

#. 确认升级后的版本： ::

	ceph --version

如果不是最新版，你也许得卸载、清理掉依赖、然后重新安装。


.. _用你发行版自带的包管理器: ../install-storage-cluster/
.. _操纵集群: ../../rados/operations/operating
.. _监控集群: ../../rados/operations/monitoring
.. _此版本的发布摘要: ../../releases
