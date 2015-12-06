==========
 飞前检查
==========

.. versionadded:: 0.60

谢谢您尝试 Ceph ！我们建议安装一个 ``ceph-deploy`` 管理节点和一\
个三节点的 :term:`Ceph 存储集群`\ 来发掘 Ceph 的基本特性。这篇\
**飞前检查**\ 会帮你准备一个 ``ceph-deploy`` 管理节点、以及三个\
\ Ceph 节点（或虚拟机），以此构成 Ceph 存储集群。动手之前，请参\
见\ `操作系统推荐`_\ 确认你安装了合适的 Linux 发行版。如果你在\
整个生产集群中只部署了一种 Linux 发行版的同一版本，那么比较容易\
找到问题根源。

在下面的描述中\ :term:`节点`\ 代表一台机器。

.. include:: quick-common.rst


Ceph 部署工具的安装
===================

把 Ceph 仓库添加到 ``ceph-deploy`` 管理节点，然后安装 ``ceph-deploy`` 。

高级包管理工具（APT）
---------------------

在 Debian 和 Ubuntu 发行版上，可以执行下列步骤：

#. 添加发布密钥： ::

	wget -q -O- 'https://download.ceph.com/keys/release.asc' | sudo apt-key add -

#. 添加Ceph软件包源，用稳定版Ceph（如 ``cuttlefish`` 、 \
   ``dumpling`` 、 ``emperor`` 、 ``firefly`` 等等）替换掉 \
   ``{ceph-stable-release}`` 。例如： ::

	echo deb http://download.ceph.com/debian-{ceph-stable-release}/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list

#. 更新你的仓库，并安装 ``ceph-deploy`` ： ::

	sudo apt-get update && sudo apt-get install ceph-deploy

.. note:: 你也可以从欧洲镜像 eu.ceph.com 下载软件包，只需把 \
   ``http://ceph.com/`` 替换成 ``http://eu.ceph.com/`` 即可。


红帽包管理工具（RPM）
---------------------

在 Red Hat （rhel6、rhel7）、CentOS （el6、el7）和 Fedora 19-21 \
(f19-f21) 上执行下列步骤：

#. 把软件包源加入软件库，用文本编辑器创建一个 YUM (Yellowdog \
   Updater, Modified) 库文件，其路径为 \
   ``/etc/yum.repos.d/ceph.repo`` 。例如： ::

	sudo vim /etc/yum.repos.d/ceph.repo

   把如下内容粘帖进去，用最新稳定版 Ceph 名字替换 \
   ``{ceph-stable-release}`` （如 ``firefly`` ）、用你的发行版\
   名字替换 ``{distro}`` （如 ``el6`` 为 CentOS 6 、 ``el7`` \
   为 CentOS 7 、 ``rhel6`` 为 Red Hat 6.5 、 ``rhel7`` 为 \
   Red Hat 7 、 ``fc19`` 是 Fedora 19 、 ``fc20`` 是 Fedora \
   20 。最后保存到 ``/etc/yum.repos.d/ceph.repo`` 文件。 ::

	[ceph-noarch]
	name=Ceph noarch packages
	baseurl=http://download.ceph.com/rpm-{ceph-release}/{distro}/noarch
	enabled=1
	gpgcheck=1
	type=rpm-md
	gpgkey=https://download.ceph.com/keys/release.asc


#. 更新软件库并安装 ``ceph-deploy`` ： ::

	sudo yum update && sudo yum install ceph-deploy


.. note:: 你也可以从欧洲镜像 eu.ceph.com 下载软件包，只需把 \
   ``http://ceph.com/`` 替换成 ``http://eu.ceph.com/`` 即可。


Ceph 节点安装
=============

你的管理节点必须能够通过 SSH 无密码地访问各 Ceph 节点。如果 \
``ceph-deploy`` 以某个普通用户登录，那么这个用户必须有无密码\
使用 ``sudo`` 的权限。


安装 NTP
--------

我们建议把 NTP 服务安装到所有 Ceph 节点上（特别是 Ceph 监视器\
节点），以免因时钟漂移导致故障，详情见\ `时钟`_\ 。

在 CentOS / RHEL 上可执行： ::

	sudo yum install ntp ntpdate ntp-doc

在 Debian / Ubuntu 上可执行： ::

	sudo apt-get install ntp

确保在各 Ceph 节点上启动了 NTP 服务，并且要使用同一个 NTP 服务器，\
详情见 `NTP`_ 。


安装 SSH 服务器
---------------

在\ **所有** Ceph 节点上执行如下步骤：

#. 在各 Ceph 节点安装 SSH 服务器（如果还没有）： ::

	sudo apt-get install openssh-server

   或者 ::

	sudo yum install openssh-server

#. 确保\ **所有** Ceph 节点上的 SSH 服务器都在运行。


创建部署 Ceph 的用户
--------------------

``ceph-deploy`` 工具必须以普通用户登录，且此用户拥有无密码使用 \
``sudo`` 的权限，因为它需要安装软件及配置文件，中途不能输入密码。

较新版的 ``ceph-deploy`` 支持用 ``--username`` 选项提供可无密码\
使用 ``sudo`` 的用户名（包括 ``root`` ，虽然\ **不建议**\ 这样\
做）。要用 ``ceph-deploy --username {username}`` 命令，指定的用\
户必须能够通过无密码 SSH 连接到 Ceph 节点，因为 ``ceph-deploy`` \
不支持中途输入密码。

我们建议在集群内的\ **所有** Ceph 节点上都给 ``ceph-deploy`` 创\
建一个普通用户（但\ **不要**\ 用 “ceph” 这个名字），全集群统一的\
用户名可简化操作（非必需），然而你应该避免使用知名用户名，因为黑\
帽子们会用它做暴力破解（如 ``root`` 、 ``admin`` 、 \
``{productname}`` ）。后续步骤描述了如何创建无 ``sudo`` 密码的用\
户，你要用自己取的名字取代 ``{username}`` 。

.. note:: 从 `Infernalis 版`_\ 起，用户名 "ceph" 保留给了 Ceph \
   守护进程。如果 Ceph 节点上已经有了 "ceph" 用户，升级前必须先\
   删掉这个用户。

#. 在各 Ceph 节点创建新用户。 ::

	ssh user@ceph-server
	sudo useradd -d /home/{username} -m {username}
	sudo passwd {username}

#. 确保各 Ceph 节点上新创建的用户都有 ``sudo`` 权限。 ::

	echo "{username} ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/{username}
	sudo chmod 0440 /etc/sudoers.d/{username}


允许无密码 SSH 登录
-------------------

正因为 ``ceph-deploy`` 不支持输入密码，你必须在管理节点上生成 \
SSH 密钥并把其公钥散布到各 Ceph 节点。 ``ceph-deploy`` 会尝试给\
初始监视器们生成 SSH 密钥对。

#. 生成 SSH 密钥对，但不要用 ``sudo`` 或 ``root`` 用户。口令为\
   空： ::

	ssh-keygen

	Generating public/private key pair.
	Enter file in which to save the key (/ceph-admin/.ssh/id_rsa):
	Enter passphrase (empty for no passphrase):
	Enter same passphrase again:
	Your identification has been saved in /ceph-admin/.ssh/id_rsa.
	Your public key has been saved in /ceph-admin/.ssh/id_rsa.pub.

#. 把公钥拷贝到各 Ceph 节点，把下列命令中的 ``{username}`` 替换\
   成前面\ `创建部署 Ceph 的用户`_\ 里的用户名。 ::

	ssh-copy-id {username}@node1
	ssh-copy-id {username}@node2
	ssh-copy-id {username}@node3

#. （ 推荐做法）修改 ``ceph-deploy`` 管理节点上的 \
   ``~/.ssh/config`` 文件，这样 ``ceph-deploy`` 就能用你所建的\
   用户名登录 Ceph 节点了，无需每次执行 ``ceph-deploy`` 都指定 \
   ``--username {username}`` 。这样做同时也简化了 ``ssh`` 和 \
   ``scp`` 的用法。把 ``{username}`` 替换成你创建的用户名。 ::

	Host node1
	   Hostname node1
	   User {username}
	Host node2
	   Hostname node2
	   User {username}
	Host node3
	   Hostname node3
	   User {username}


引导时联网
----------

Ceph 的各 OSD 进程通过网络互联并向监视器集群报告，如果网络默认为 \
``off`` ，那么 Ceph 集群就不能在启动时就上线，直到打开网络。

某些发行版（如 CentOS ）默认关闭网络接口，故此需确保网卡在系统启\
动时都能启动，这样 Ceph 守护进程才能通过网络互联。例如，在 Red \
Hat 和 CentOS 上，需进入 ``/etc/sysconfig/network-scripts`` 目录\
并确保 ``ifcfg-{iface}`` 文件中的 ``ONBOOT`` 设置成了 ``yes`` 。


确保联通性
----------

用 ``ping`` 短主机名（ ``hostname -s`` ）的方式确认网络没问题，\
解决掉可能存在的主机名解析问题。

.. note:: 主机名应该解析为网络 IP 地址，而非回环接口 IP 地址（即\
   主机名应该解析成非 ``127.0.0.1`` 的IP地址）。如果你的管理节点\
   同时也是一个 Ceph 节点，也要确认它能正确解析主机名和 IP 地址\
   （即非回环 IP 地址）。


放通所需端口
------------

Ceph 监视器之间默认用 ``6789`` 端口通信， OSD 之间默认用 \
``6800:7300`` 这个范围内的端口通信。详情见\ `网络配置参考`_\ 。 \
Ceph OSD 能利用多个网络连接与客户端、监视器、其他副本 OSD 、其它\
心跳 OSD 分别进行通信。

某些发行版（如 RHEL ）的默认防火墙配置非常严格，你得调整防火墙，\
允许相应的入栈请求，这样客户端才能与 Ceph 节点通信。

对于 RHEL 7 上的 ``firewalld`` ，要对公共域放通 Ceph 监视器所使\
用的 ``6789`` 端口、以及 OSD 所使用的 ``6800:7300`` ，并且要配置\
为永久规则，这样重启后规则仍有效。例如： ::

	sudo firewall-cmd --zone=public --add-port=6789/tcp --permanent

若用 ``iptables`` 命令，要放通 Ceph 监视器所用的 ``6789`` 端口和 \
OSD 所用的 ``6800:7300`` 端口范围，命令如下： ::

	sudo iptables -A INPUT -i {iface} -p tcp -s {ip-address}/{netmask} --dport 6789 -j ACCEPT

配置好 ``iptables`` 之后要保存配置，这样重启之后才依然有效。\
例如： ::

	/sbin/service iptables save


终端（ TTY ）
-------------

在 CentOS 和 RHEL 上执行 ``ceph-deploy`` 命令时，如果你的 Ceph \
节点默认设置了 ``requiretty`` 那就会遇到报错。可以这样禁用它，\
执行 ``sudo visudo`` ，找到 ``Defaults requiretty`` 选项，把它\
改为 ``Defaults:ceph !requiretty`` 或者干脆注释掉，这样 \
``ceph-deploy`` 就可以用之前创建的用户（\
`创建部署 Ceph 的用户`_ ）连接了。

.. note:: 编辑配置文件 ``/etc/sudoers`` 时，必须用 \
   ``sudo visudo`` 而不是文本编辑器。


SELinux
-------

在 CentOS 和 RHEL 上， SELinux 默认开启为 ``Enforcing`` 。为简化\
安装，我们建议把 SELinux 设置为 ``Permissive`` 或者完全禁用，也\
就是在加固系统配置前先确保集群的安装、配置没问题。用下列命令把 \
SELinux 设置为 ``Permissive`` ： ::

	sudo setenforce 0

要使 SELinux 配置永久生效（如果它确是问题根源），需修改其配置文件 \
``/etc/selinux/config`` 。


优先级或首选项
--------------

确保你的包管理器安装了优先级或首选项包、且已启用。在 CentOS 上\
你也许得安装 EPEL ，在 RHEL 上你也许得启用可选软件库。 ::

	sudo yum install yum-plugin-priorities

比如在 RHEL 7 服务器上，可用下列命令安装 ``yum-plugin-priorities``\
\ 并启用 ``rhel-7-server-optional-rpms`` 软件库： ::

	sudo yum install yum-plugin-priorities --enablerepo=rhel-7-server-optional-rpms


总结
====

快速入门的飞前部分到此结束，请继续\ `存储集群快速入门`_\ 。


.. _存储集群快速入门: ../quick-ceph-deploy
.. _操作系统推荐: ../os-recommendations
.. _网络配置参考: ../../rados/configuration/network-config-ref
.. _时钟: ../../rados/configuration/mon-config-ref#clock
.. _NTP: http://www.ntp.org/
.. _Infernalis 版: ../../release-notes/#v9-1-0-infernalis-release-candidate
