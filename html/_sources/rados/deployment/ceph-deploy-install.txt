============
 软件包管理
============

安装
====

要在集群主机上安装 Ceph 软件包，在管理主机上打开命令行并执行下列命令： ::

	ceph-deploy install {hostname [hostname] ...}

没提供额外选项的话 ``ceph-deploy`` 默认会把最新稳定版安装到集群主机，要指定某个软\
件包可以用下列参数：

- ``--release <code-name>``
- ``--testing``
- ``--dev <branch-or-tag>``

例如： ::

	ceph-deploy install --release cuttlefish hostname1
	ceph-deploy install --testing hostname2
	ceph-deploy install --dev wip-some-branch hostname{1,2,3,4,5}

其余用法见： ::

	ceph-deploy install -h


卸载
====

要从集群主机卸载 Ceph 软件包，在管理主机的终端下执行： ::

	ceph-deploy uninstall {hostname [hostname] ...}

在 Debian 或 Ubuntu 系统上你也可以： ::

	ceph-deploy purge {hostname [hostname] ...}

此工具会从指定主机上卸载 ``ceph`` 软件包，另外 ``purge`` 会删除配置文件。
