=================
 增加/删除监视器
=================

你的集群启动并运行后，可以在运行时增加、或删除监视器。请参考\ `手动部署`_\ 或\ \
`监视器自举启动`_\ 完成初始设置。

增加监视器
==========

Ceph 监视器是轻量级进程，它维护着集群运行图的主副本。一个集群可以只有一个监视器，我\
们推荐生产环境至少 3 个监视器。 Ceph 使用 `Paxos`_ 算法的一个变种对各种图、以及其它\
对集群来说至关重要的信息达成共识。由于 Paxos 算法天生要求大部分监视器在运行，以形成\
法定人数（并因此达成共识）。

It is advisable to run an odd-number of monitors but not mandatory. An
odd-number of monitors has a higher resiliency to failures than an
even-number of monitors. For instance, on a 2 monitor deployment, no
failures can be tolerated in order to maintain a quorum; with 3 monitors,
one failure can be tolerated; in a 4 monitor deployment, one failure can
be tolerated; with 5 monitors, two failures can be tolerated.  This is
why an odd-number is advisable. Summarizing, Ceph needs a majority of
monitors to be running (and able to communicate with each other), but that
majority can be achieved using a single monitor, or 2 out of 2 monitors,
2 out of 3, 3 out of 4, etc.

For an initial deployment of a multi-node Ceph cluster, it is advisable to
deploy three monitors, increasing the number two at a time if a valid need
for more than three exists.

正因为监视器是轻量级的，所以有可能在作为 OSD 的主机上同时运行它；然而，我们推荐运行\
于单独主机，因为与内核的 fsync 问题会影响性能。

.. note:: 这里的\ *大多数*\ 监视器之间必须能互通，这样才能形成法定人数。

部署硬件
--------

如果你增加新监视器时要新增一台主机，关于其最低硬件配置请参见\ `硬件推荐`_\ 。要增加\
一个监视器主机，首先要安装最新版的 Linux （如 Ubuntu 12.04 或者 RHEL 7 ）。

把监视器主机安装上架，连通网络。

.. _硬件推荐: ../../../start/hardware-recommendations

安装必要软件
------------

手动部署的集群， Ceph 软件包必须手动装，详情参见\ `安装软件包`_\ 。应该配置一个用\
户，使之可以无密码登录 SSH 、且有 root 权限。

.. _安装软件包: ../../../install/install-storage-cluster


.. _增加监视器（手动）:

增加监视器（手动）
------------------

本步骤创建 ``ceph-mon`` 数据目录、获取监视器运行图和监视器密钥环、增加一个 \
``ceph-mon`` 守护进程。如果这导致只有 2 个监视器守护进程，你可以重演此步骤来增加一\
或多个监视器，直到你拥有足够多 ``ceph-mon`` 达到法定人数。

现在该指定监视器的标识号了。传统上，监视器曾用单个字母（ ``a`` 、 ``b`` 、 ``c`` \
……）命名，但你可以指定任何形式。在本文档里，要记住 ``{mon-id}`` 应该是你所选的标识\
号，不包含 ``mon.`` 前缀，如在 ``mon.a`` 中，其 ``{mon-id}`` 是 ``a`` 。

#. 在新监视器主机上创建默认目录： ::

	ssh {new-mon-host}
	sudo mkdir /var/lib/ceph/mon/ceph-{mon-id}

#. 创建临时目录 ``{tmp}`` ，用以保存此过程中用到的文件。此目录要不同于前面步骤创建\
   的监视器数据目录，且完成后可删除。 ::

	mkdir {tmp}

#. 获取监视器密钥环， ``{tmp}`` 是密钥环文件保存路径、 ``{filename}`` 是包含密钥的\
   文件名。 ::

	ceph auth get mon. -o {tmp}/{key-filename}

#. 获取监视器运行图， ``{tmp}`` 是获取到的监视器运行图、 ``{filename}`` 是包含监视\
   器运行图的文件名。 ::

	ceph mon getmap -o {tmp}/{map-filename}

#. 准备第一步创建的监视器数据目录。必须指定监视器运行图路径，这样才能获得监视器法定\
   人数和它们 ``fsid`` 的信息；还要指定监视器密钥环路径。 ::

	sudo ceph-mon -i {mon-id} --mkfs --monmap {tmp}/{map-filename} --keyring {tmp}/{key-filename}

#. 启动新监视器，它会自动加入机器。守护进程需知道绑定到哪个地址，\
   通过 ``--public-addr {ip:port}`` 或在 ``ceph.conf`` 里的相应段\
   设置 ``mon addr`` 可以指定。 ::

	ceph-mon -i {mon-id} --public-addr {ip:port}


删除监视器
==========

从集群删除监视器时，必须认识到， Ceph 监视器用 PASOX 算法关于主集群运行图达成共识。\
必须有足够多的监视器才能对集群运行图达成共识。

.. _删除监视器（手动）:

删除监视器（手动）
------------------

本步骤从集群删除 ``ceph-mon`` 守护进程，如果此步骤导致只剩 2 个监视器了，你得增加或\
删除一个监视器，直到凑足法定人数所必需的 ``ceph-mon`` 数。

#. 停止监视器。 ::

	service ceph -a stop mon.{mon-id}

#. 从集群删除监视器。 ::

	ceph mon remove {mon-id}

#. 删除 ``ceph.conf`` 对应条目。


从不健康集群删除监视器
----------------------

本步骤从不健康集群删除 ``ceph-mon`` ，例如集群内的监视器不能形成法定人数。

#. 停止所有监视器主机上的所有 ``ceph-mon`` 守护进程。 ::

	ssh {mon-host}
	service ceph stop mon || stop ceph-mon-all
	# 要在所有监视器主机上执行

#. 找出一个活着的监视器并登录其所在主机。 ::

	ssh {mon-host}

#. 提取 monmap 副本。 ::

	ceph-mon -i {mon-id} --extract-monmap {map-path}
	# 多数情况下都是：
	ceph-mon -i `hostname` --extract-monmap /tmp/monmap

#. 删除不保留或有问题的监视器。例如，如果你有 3 个监视器 ``mon.a`` 、 \
   ``mon.b`` 和 ``mon.c`` ，其中仅保留 ``mon.a`` ，按如下步骤： ::

	monmaptool {map-path} --rm {mon-id}
	# 例如
	monmaptool /tmp/monmap --rm b
	monmaptool /tmp/monmap --rm c

#. 把去除过监视器后剩下的运行图注入存活的监视器。比如，用下列命令把一张运\
   行图注入 ``mon.a`` 监视器： ::

	ceph-mon -i {mon-id} --inject-monmap {map-path}
	# for example,
	ceph-mon -i a --inject-monmap /tmp/monmap

#. 只启动保留下来的监视器。

#. 确认这些监视器形成了法定人数（ ``ceph -s`` ）。

#. 你也许得把已删除监视器的数据目录 ``/var/lib/ceph/mon`` 备份到安全位置，\
   如果您对其余监视器很有信心、或者有足够的冗余，也可以删除。

.. _更改监视器的 IP 地址:


更改监视器的 IP 地址
====================

.. important:: 现有监视器不应该更改其 IP 地址。

监视器是 Ceph 集群的关键组件，它们要维护一个法定人数，这样整个系统才能正常工作。要\
确立法定人数，监视器得互相发现对方， Ceph 对监视器的发现要求严格。

Ceph 客户端及其它 Ceph 守护进程用 ``ceph.conf`` 发现监视器，然而，监视器之间用监视\
器运行图发现对方，而非 ``ceph.conf`` 。例如，你看过的\ `增加监视器（手动）`_\ ，\
会发现创建新监视器时得获取当前集群的 monmap ，因为它是 ``ceph-mon -i {mon-id} \
--mkfs`` 命令的必要参数。下面几段解释了 Ceph 监视器的一致性要求，和几种改 IP 的安\
全方法。


一致性要求
----------

监视器发现集群内的其它监视器时总是参照 monmap 的本地副本，用 monmap 而非 \
``ceph.conf`` 可避免因配置错误（例如在 ``ceph.conf`` 指定监视器地址或端口时拼写错\
误）而损坏集群。正因为监视器用 ``monmaps`` 相互发现、且共享于客户端和其它 Ceph 守\
护进程间，所以 monmap 给监视器提供了苛刻的一致性保证。

苛刻的一致性要求也适用于 monmap 的更新，因为任何有关监视器的更新、 monmap 的更改都\
通过名为 `Paxos`_ 的分布式一致性算法运行。为保证法定人数里的所有监视器都持有同版本 \
monmap ，所有监视器都要赞成 monmap 的每一次更新，像增加、删除监视器。 monmap 的更\
新是增量的，这样监视器都有最近商定的版本以及一系列之前版本，这样可使一个有较老 \
monmap 的监视器赶上集群当前的状态。

如果监视器通过 Ceph 配置文件而非 monmap 相互发现，就会引进额外风险，因为 Ceph 配置\
文件不会自动更新和发布。监视器有可能用了较老的 ``ceph.conf`` 而导致不能识别某监视\
器、掉出法定人数、或者发展为一种 `Paxos`_ 不能精确确定当前系统状态的情形。总之，更\
改现有监视器的 IP 地址必须慎之又慎。


更改监视器 IP 地址（正确方法）
------------------------------

仅仅在 ``ceph.conf`` 里更改监视器的 IP 不足以让集群内的其它监视器接受更新。要更改\
一个监视器的 IP 地址，你必须以先以想用的 IP 地址增加一个监视器（见\ `增加监视器（手\
动）`_\ ），确保新监视器成功加入法定人数，然后删除用旧 IP 的监视器，最后更新 \
``ceph.conf`` 以确保客户端和其它守护进程得知新监视器的 IP 地址。

例如，我们假设有 3 个监视器，如下： ::

	[mon.a]
		host = host01
		addr = 10.0.0.1:6789
	[mon.b]
		host = host02
		addr = 10.0.0.2:6789
	[mon.c]
		host = host03
		addr = 10.0.0.3:6789

要把 ``host04`` 上 ``mon.c`` 的 IP 改为 ``10.0.0.4`` ，按照\ `增加监视器（手\
动）`_\ 里的步骤增加一个新监视器 ``mon.d`` ，确认它运行正常后再删除 ``mon.c`` ，否\
则会破坏法定人数；最后依照\ `删除监视器（手动）`_\ 删除 ``mon.c`` 。 3 个监视器都\
要更改的话，每次都要重复一次。


更改监视器 IP 地址（凌乱方法）
------------------------------

可能有时候监视器不得不挪到不同的网络、数据中心的不同位置、甚至不同的数据中心，这是可\
能的，但过程有点惊险。

在这种情形下，一种方法是用所有监视器的新 IP 地址生成新 monmap ，并注入到集群内的所\
有监视器。对大多数用户来说，这并不简单，好在它不常见。再次重申，监视器不应该更改 \
IP 地址。

以前面的监视器配置为例，假设你想把所有监视器的 IP 从 ``10.0.0.x`` 改为 \
``10.1.0.x`` ，并且两个网络互不相通，步骤如下：

#. 获取监视器运行图，其中 ``{tmp}`` 是所获取的运行图路径， ``{filename}`` 是监视器\
   运行图的文件名。 ::

	ceph mon getmap -o {tmp}/{filename}

#. 下面是一个 monmap 内容示例： ::

	$ monmaptool --print {tmp}/{filename}

	monmaptool: monmap file {tmp}/{filename}
	epoch 1
	fsid 224e376d-c5fe-4504-96bb-ea6332a19e61
	last_changed 2012-12-17 02:46:41.591248
	created 2012-12-17 02:46:41.591248
	0: 10.0.0.1:6789/0 mon.a
	1: 10.0.0.2:6789/0 mon.b
	2: 10.0.0.3:6789/0 mon.c

#. 删除现有监视器。 ::

	$ monmaptool --rm a --rm b --rm c {tmp}/{filename}

	monmaptool: monmap file {tmp}/{filename}
	monmaptool: removing a
	monmaptool: removing b
	monmaptool: removing c
	monmaptool: writing epoch 1 to {tmp}/{filename} (0 monitors)

#. 添加新监视器位置。 ::

	$ monmaptool --add a 10.1.0.1:6789 --add b 10.1.0.2:6789 --add c 10.1.0.3:6789 {tmp}/{filename}

	monmaptool: monmap file {tmp}/{filename}
	monmaptool: writing epoch 1 to {tmp}/{filename} (3 monitors)

#. 检查新内容。 ::

	$ monmaptool --print {tmp}/{filename}

	monmaptool: monmap file {tmp}/{filename}
	epoch 1
	fsid 224e376d-c5fe-4504-96bb-ea6332a19e61
	last_changed 2012-12-17 02:46:41.591248
	created 2012-12-17 02:46:41.591248
	0: 10.1.0.1:6789/0 mon.a
	1: 10.1.0.2:6789/0 mon.b
	2: 10.1.0.3:6789/0 mon.c

从这里开始，假设监视器（及存储）已经被安装到了新位置。下一步把修正的 monmap 散播到\
新监视器，并且注入每个监视器。

#. 首先，停止所有监视器，注入必须在守护进程停止时进行。

#. 注入 monmap 。 ::

	ceph-mon -i {mon-id} --inject-monmap {tmp}/{filename}

#. 重启监视器。

到这里，到新位置的迁移完成，监视器应该照常运行了。


.. _手动部署: ../../../install/manual-deployment
.. _监视器自举启动: ../../../dev/mon-bootstrap
.. _Paxos: http://en.wikipedia.org/wiki/Paxos_(computer_science)
