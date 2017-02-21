====================
 安装 Ceph 存储集群
====================

本指南说明了如何手动安装 Ceph 软件包，此方法只适用于那些没采用\
部署工具（如 ``ceph-deploy`` 、 ``chef`` 、 ``juju`` 等）的用户。

.. tip:: 你也可以用 ``ceph-deploy`` 安装 Ceph 软件包，也许它更\
   方便，因为只需一个命令就可以把 ``ceph`` 安装到多台主机。


用 APT 安装
===========

只要把正式版或开发版软件包源加入了 APT ，你就可以更新 APT 数据\
库并安装 Ceph 了： ::

	sudo apt-get update && sudo apt-get install ceph ceph-mds


用 RPM 安装
===========

要用 RPM 安装 Ceph ，可按如下步骤进行：

#. 安装 ``yum-plugin-priorities`` 。 ::

	sudo yum install yum-plugin-priorities

#. 确认 ``/etc/yum/pluginconf.d/priorities.conf`` 文件存在。

#. 确认 ``priorities.conf`` 里面打开了插件支持。 ::

	[main]
	enabled = 1

#. 确认你的 YUM ``ceph.repo`` 库文件条目包含 ``priority=2`` ，\
   详情见\ `获取软件包`_\ ： ::

	[ceph]
	name=Ceph packages for $basearch
	baseurl=http://download.ceph.com/rpm-{ceph-release}/{distro}/$basearch
	enabled=1
	priority=2
	gpgcheck=1
	type=rpm-md
	gpgkey=https://download.ceph.com/keys/release.asc

	[ceph-noarch]
	name=Ceph noarch packages
	baseurl=http://download.ceph.com/rpm-{ceph-release}/{distro}/noarch
	enabled=1
	priority=2
	gpgcheck=1
	type=rpm-md
	gpgkey=https://download.ceph.com/keys/release.asc

	[ceph-source]
	name=Ceph source packages
	baseurl=http://download.ceph.com/rpm-{ceph-release}/{distro}/SRPMS
	enabled=0
	priority=2
	gpgcheck=1
	type=rpm-md
	gpgkey=https://download.ceph.com/keys/release.asc

#. 安装依赖的的软件包： ::

	sudo yum install snappy leveldb gdisk python-argparse gperftools-libs

成功添加正式版或开发版软件包的库文件之后，或把 ``ceph.repo`` \
文件放入 ``/etc/yum.repos.d`` 之后，你就可以安装 Ceph 软件包\
了。 ::

	sudo yum install ceph


从源码安装
==========

如果你是从源码构建的 Ceph ，可以用下面的命令安装到用户区： ::

	sudo make install

如果你是本地安装的， ``make`` 会把可执行文件放到 ``usr/local/bin`` \
里面。你可以把 Ceph 配置文件放到 ``usr/local/bin`` 目录下，这样就\
能从这个目录运行 Ceph 了。


.. _获取软件包: ../get-packages
