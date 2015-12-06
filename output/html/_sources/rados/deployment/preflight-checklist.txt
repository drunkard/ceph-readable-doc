==========
 飞前检查
==========

.. versionadded:: 0.60

本篇\ **飞前检查**\ 会协助你准备运行 ``ceph-deploy`` 的节点、和服务器节点，以使用\
无密码 ``ssh`` 和 ``sudo`` 。

通过 ``ceph-deploy`` 部署 Ceph 前，先要在管理节点和运行 Ceph 守护进程的节点上确认\
一些事情。


选装一个操作系统
================

在各节点安装最新发布的 Debian 或 Ubuntu （如 12.04 LTS 、 14.04 LTS ），关\
于 Debian 或 Ubuntu 以外的操作系统详见\ `操作系统推荐`_\ 。


安装 SSH 服务器
===============

``ceph-deploy`` 工具需要 ``ssh`` ，所以你的服务器节点需要 SSH 服务器。 ::

	sudo apt-get install openssh-server


创建一用户
==========

在运行 Ceph 守护进程的节点上创建一个普通用户。

.. tip:: 我们建议选一个暴力攻击者不可能轻易猜到的用户名，如 ``root`` 、 ``ceph`` \
   之外的名字。

::

	ssh user@ceph-server
	sudo useradd -d /home/ceph -m ceph
	sudo passwd ceph


``ceph-deploy`` 会在节点安装软件包，所以你创建的用户需要无密码 ``sudo`` 权限。

.. note:: 为安全起见，我们\ **不推荐**\ 启用 ``root`` 密码。

为赋予用户所有权限，把下列加入 ``/etc/sudoers.d/ceph`` 。 ::

	echo "ceph ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ceph
	sudo chmod 0440 /etc/sudoers.d/ceph


配置 SSH
========

配置你的管理主机，使之可通过  SSH无密码访问各节点，口令留空。 ::

	ssh-keygen
	Generating public/private key pair.
	Enter file in which to save the key (/ceph-client/.ssh/id_rsa):
	Enter passphrase (empty for no passphrase):
	Enter same passphrase again:
	Your identification has been saved in /ceph-client/.ssh/id_rsa.
	Your public key has been saved in /ceph-client/.ssh/id_rsa.pub.

把公钥拷贝到各节点：
:

	ssh-copy-id ceph@ceph-server

修改你管理节点上的 ``~/.ssh/config`` ，以使未指定用户名时默认用你刚刚创建的用户名。 ::

	Host ceph-server
		Hostname ceph-server.fqdn-or-ip-address.com
		User ceph


安装 ceph-deploy
================

执行下列命令安装 ``ceph-deploy`` ： ::

	wget -q -O- 'https://download.ceph.com/keys/release.asc' | sudo apt-key add -
	echo deb http://ceph.com/debian-dumpling/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list
	sudo apt-get update
	sudo apt-get install ceph-deploy


确认连通性
==========

确保你的管理节点网络连接正常、且能连接服务器节点（如确认 ``iptables`` 、 ``ufw`` \
或其它可阻断连接的工具、流量转发等等这些组件配置正常）。

完成飞前检查后，你就可以开始用 ``ceph-deploy`` 部署了。

.. _操作系统推荐: ../../../start/os-recommendations
