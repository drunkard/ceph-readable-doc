===========
 升级 Ceph
===========

Ceph 的各个版本都可能有特定的步骤，升级前请参考与此版本相关的章节和\ `发布说明`_\ \
文档，以确定有哪些特定于此版本的步骤。


概述
====

你可以在 Ceph 集群在线且提供服务时升级守护进程！某些类型的守护进程依赖其他的，如 \
Ceph 元数据服务器和 Ceph 对象网关依赖于 Ceph 监视器和 OSD 守护进程，所以我们建议按\
以下顺序升级：

#. `ceph-deploy 工具`_
#. Ceph 监视器
#. Ceph OSD 守护进程
#. Ceph 元数据服务器
#. Ceph 对象网关

作为普适规则，我们建议一次升级一类的所有守护进程（如所有 ``ceph-osd`` 、所有 \
``ceph-mon`` 等），这样才能确保它们属于同一版本。

`升级过程`_\ 相对简单，只要参照此版本特定的章节即可。基本过程有三个：

#. 用 ``ceph-deploy`` 为各主机升级软件包（用 ``ceph-deploy install`` 命令），\
   或者分别登录各主机\ `手动升级`_\ 。例如，\ `升级监视器`_\ 时， ``ceph-deploy`` \
   语法大致如此： ::

	ceph-deploy install --release {release-name} ceph-node1[ ceph-node2]
	ceph-deploy install --release firefly mon1 mon2 mon3

   **注：** ``ceph-deploy install`` 命令会把指定节点上的旧版本升级为你所指定的版\
   本，没有 ``ceph-deploy upgrade`` 这样的升级命令。

#. 登录各节点并重启各相关 Ceph 守护进程，详情参见\ `操纵集群`_ 。

#. 确认集群健康状况，详情见\ `监控集群`_\ 。

.. important:: 一旦升级完，就不能降级了。


ceph-deploy 工具
================

升级 Ceph 守护进程前，应该先升级 ``ceph-deploy`` 工具。 ::

	sudo pip install -U ceph-deploy

或者： ::

	sudo apt-get install ceph-deploy

或者： ::

	sudo yum install ceph-deploy python-pushy


Argonaut 到 Bobtail
===================

从 Argonaut 升级到 Bobtail 时，你得注意几件事：

#. 现在认证默认是\ **启用**\ 的，而以前是\ **禁用**\ 的；
#. 监视器间使用新的通讯协议；
#. 通过 RBD 使用 ``format2`` 格式的映像前要完成所有 OSD 的升级。

确保你先更新软件库路径，如： ::

	sudo rm /etc/apt/sources.list.d/ceph.list
	echo deb http://download.ceph.com/debian-bobtail/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list

详情见如下几段。


认证
----

Bobtail 版默认启用了认证，而且预置了粒度更细的认证配置选项。在先前的版本（即 v0.55 \
和更早的）中你可以简单地写： ::

	auth supported = [cephx | none]

此选项仍然能被识别，但已经过时。新版支持 ``cluster`` 、 ``service`` 、和 \
``client`` 认证选项，如下： ::

	auth cluster required = [cephx | none]  # default cephx
	auth service required = [cephx | none] # default cephx
	auth client required = [cephx | none] # default cephx,none

.. important:: 如果你现在的集群没有用 ``auth supported`` 这一行启用认证，那你升级\
   到 Bobtail 时必须显式地关闭： ::

	auth cluster required = none
	auth service required = none

   这样集群认证将被关闭，但客户端默认行为未更改，它仍可以与启用了认证、但不强求的集\
   群通讯。

.. important:: 如果你的集群配置文件之前就有 ``auth supported`` 选项，那就没必要更\
   改了。

详情见 `Ceph 认证——向后兼容性`_\ 。


监视器线上协议
--------------

我们建议把所有监视器都升级到 Bobtail ，混用 Bobtail 和 Argonaut 将不能使用新的线上\
协议，因为此协议要求所有监视器的版本都在 Bobtail 及以上。只升级大多数（如三个中的两\
个）监视器也会导致潜在风险，即再失效一个就会丧失可用性（因为未升级的守护进程不能参与\
新协议构成的通讯）。我们建议各 ``ceph-mon`` 升级期间不要等待。


RBD 映像
--------

Bobtail 版现在支持 ``format 2`` 格式的映像了！但是在所有 ``ceph-osd`` 都升级完之\
前，不要新建或使用 ``format 2`` 格式的RBD映像。注意， ``format 1`` 仍然是默认值。\
你可以用新的 ``ceph osd ls`` 和 ``ceph tell osd.N version`` 命令再次确认软件版\
本， ``ceph osd ls`` 将给出集群内所有 OSD 的 ID 列表，把它们代入 shell 循环即可查\
询所有 OSD 的版本： ::

      for i in $(ceph osd ls); do
          ceph tell osd.${i} version
      done


Argonaut 到 Cuttlefish
======================

要从 Argonaut 升级到 Cuttlefish ，请仔细阅读本段、以及从 Argonaut \
升到 Bobtail 、还有从 Bobtail 升级到 Cuttlefish 。从 Argonaut 升\
级到 Cuttlefish 时，\ **你必须先把所有监视器从 Argonaut 升级到 \
Bobtail v0.56.5** ！！！所有其他的守护进程都可以从 Argonaut 直接\
升级到 Cuttlefish ，无需间接地从 Bobtail 过渡一次。

.. important:: 确保软件仓库指定的是 Bobtail ，而不是 Cuttlefish 。

例如： ::

	sudo rm /etc/apt/sources.list.d/ceph.list
	echo deb http://download.ceph.com/debian-bobtail/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list

我们建议把监视器升级到 Cuttlefish 之前，先升级到 Bobtail 。混用 \
Bobtail 和 Argonaut 监视器将导致旧监视器不能使用新线上协议，因为\
此协议在 Bobtail 及更高版本上才可用。只升级大多数（如三个中的两\
个）监视器也会导致潜在风险，即再失效一个就会丧失可用性（因为未升\
级的守护进程不能参与新协议构成的通讯）。我们建议各 ``ceph-mon`` \
升级期间不要等待。详情见\ `升级监视器`_\ 。

.. note:: 关于 Bobtail 的认证及向后兼容性请参考\ `认证`_\ 和
   `Ceph 认证——向后兼容性`_\ 。

把监视器从 Argonaut 升级到 Bobtail 、并重启无误（可以形成法定人\
数）后，还必须从 Bobtail 再升级到 Cuttlefish 。再次升级前，记得\
更改到 Cuttlefish 软件库的引用，例如： ::

	sudo rm /etc/apt/sources.list.d/ceph.list
	echo deb http://download.ceph.com/debian-cuttlefish/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list

详情见\ `升级监视器`_\ 。

Argonaut 和 Cuttlefish 的监视器架构差异较大，详情见\ \
`监视器配置`_\ 和 `Joao 的博客文章`_\ 。完成监视器的升级后， \
OSD 和 MDS 守护进程可以按照常规步骤升级，详情见\ \
`升级单个 OSD`_ 和\ `升级单个元数据服务器`_\ 。


Bobtail 到 Cuttlefish
=====================

从 Bobtail 升级到 Cuttlefish 有几点要特别注意，首先，监视器架构\
大变，所以你应该把所有监视器都升级到 Cuttlefish ；其次，如果你的\
集群有多个元数据服务器，应该确保它们的名字都唯一。详情如下。

把较老的 ``apt`` 源替换为 Cuttlefish ，如： ::

	sudo rm /etc/apt/sources.list.d/ceph.list
	echo deb http://download.ceph.com/debian-cuttlefish/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list


监视器
------

Argonaut 和 Cuttlefish 的监视器架构差异较大，详情见\ `监视器配置`_\ 和 \
`Joao 的博客文章`_\ 。这意味着 v0.61 与较早版本不能相互通讯。在升级各监视器时，它\
会把本地数据转换为新格式存储，一旦完成了大多数监视器的升级，它们就能用新协议形成法\
定人数，并且老版本的监视器会被排斥在外。正因如此，我们建议一次性升级完所有监视器。

.. important:: 混用版本的集群不要运行太长时间。


MDS 唯一名称
------------

现在的监视器要求集群内的 MDS 名字唯一。如果你的多个元数据服务器 ID 相同（比如都是 \
``mds.a`` ），第二个元数据服务器将把前一个显式地标记为 ``failed`` ，多个同名 MDS \
必须改得互不相同。如果你的集群只有一个元数据服务器，那你可以忽略此提醒。


ceph-deploy
-----------

现在， ``ceph-deploy`` 是我们推荐的集群部署工具。对于已经用 ``mkcephfs`` 创建、并\
想迁移到 ``ceph-deploy`` 的用户，这里提供了正确迁移的文档：\ `迁移到 ceph-deploy`_\ 。


Cuttlefish 到 Dumpling
======================

从 Cuttlefish (v0.61-v0.61.7) 开始可以滚动升级。但还有几点要特别注意：首先，你必须\
升级 ``ceph`` 这个命令行工具，因为它变动很大；其次，你必须把所有监视器升级到 \
Dumpling ，因为协议有变动。

把较老的软件库源替换为 Dumpling 源，例如用 ``apt`` 执行： ::

	sudo rm /etc/apt/sources.list.d/ceph.list
	echo deb http://download.ceph.com/debian-dumpling/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list

在 CentOS/Red Hat 发行版上可删除旧源。 ::

	sudo rm /etc/yum.repos.d/ceph.repo

然后用下列内容创建个新仓库 ``ceph.repo`` 。

.. code-block:: ini

	[ceph]
	name=Ceph Packages and Backports $basearch
	baseurl=http://download.ceph.com/rpm/el6/$basearch
	enabled=1
	gpgcheck=1
	type=rpm-md
	gpgkey=https://download.ceph.com/keys/release.asc


.. note:: 确保使用与自己发行版相匹配的 URL ，对应发行版请检查 \
   http://download.ceph.com/rpm 。

.. note:: 如果你可以用 ``ceph-deploy`` 升级软件，那你只需要把仓\
   库加到运行 ``ceph`` 或 ``ceph-deploy`` 命令的客户端节点即可。


Dumpling 到 Emperor
===================

Dumpling (v0.64) 可滚动升级。

把较老的软件库源替换为 Emperor 源，例如用 ``apt`` 执行： ::

	sudo rm /etc/apt/sources.list.d/ceph.list
	echo deb http://download.ceph.com/debian-emperor/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list

在 CentOS/Red Hat 发行版上可删除旧源。 ::

	sudo rm /etc/yum.repos.d/ceph.repo

然后新增软件库 ``ceph.repo`` ，其内容如下，要用自己的发行版名字\
（如 ``el6`` 、 ``rhel6`` 等等）替换 ``{disro}`` 。

.. code-block:: ini

	[ceph]
	name=Ceph Packages and Backports $basearch
	baseurl=http://download.ceph.com/rpm-emperor/{distro}/$basearch
	enabled=1
	gpgcheck=1
	type=rpm-md
	gpgkey=https://download.ceph.com/keys/release.asc


.. note:: 确保使用与自己发行版相匹配的 URL ，对应发行版请检查 \
   http://download.ceph.com/rpm 。

.. note:: 如果你可以用 ``ceph-deploy`` 升级软件，那你只需要把仓\
   库加到运行 ``ceph`` 或 ``ceph-deploy`` 命令的客户端节点即可。


命令行工具
----------

在 v0.65 中， ``ceph`` 命令行接口（ CLI ）工具变动很大；老的 CLI \
不能与 Dumpling 通讯，也就是说，要用 ``ceph`` 命令访问 Ceph 存储\
集群的所有节点其 ``ceph-common`` 库必须升级。 ::

	sudo apt-get update && sudo apt-get install ceph-common

确保你已安装最新版（ v0.67 或更新）。如果还没有，你也许得卸载、\
清除相关依赖，然后重新安装。

关于新命令行的细节在 `v0.65`_ 。

.. _v0.65: http://ceph.com/docs/master/release-notes/#v0-65


监视器
------

Dumpling (v0.67) 版本的 ``ceph-mon`` 守护进程与 v0.66 及更早版本相比，内部协议有\
所变更，也就是说它不能与 v0.66 或更老的版本通讯。大多数监视器升级完后，它们就能用新\
协议形成法定人数了，旧版监视器将被排斥在外，正因如此，我们建议要一次性升级所有监视\
器（或者较快的节奏），以最小化可能的当机时间。

.. important:: 混用版本的集群不要运行太长时间。


Dumpling 到 Firefly
===================

如果您的现有集群运行着低于 v0.67 Dumpling 的版本，请先升级到最新的 Dumpling 版，\
然后再升级到 v0.80 Firefly 版。


监视器
------

Dumpling (v0.67) 版本的 ``ceph-mon`` 守护进程与 v0.66 及更早版本相比，内部协议有\
所变更，也就是说它不能与 v0.66 或更老的版本通讯。大多数监视器升级完后，它们就能用新\
协议形成法定人数了，旧版监视器将被排斥在外，正因如此，我们建议要一次性升级所有监视\
器（或者较快的节奏），以最小化可能的当机时间。

.. important:: 混用版本的集群不要运行太长时间。


Ceph 配置文件变更
-----------------

我们建议升级前先把下列配置加入 ``ceph.conf`` 配置文件的 ``[mon]`` 段下： ::

    mon warn on legacy crush tunables = false

此配置可消除因用着老 CRUSH 归置法而引起的健康告警。虽说可以在全集群范围内重均衡已有\
数据，但我们不建议生产集群做，因为它涉及大量数据，而且重均衡会导致严重的性能降级。


命令行工具
----------

在 V0.65 版中， ``ceph`` 命令行界面（ CLI ）工具有重大改变，老版的 CLI 不能用于 \
Firefly 。也就是说，在升级 Ceph 守护进程前，要用 ``ceph`` 命令访问存储集群的节点都\
必须升级 ``ceph-common`` 库。

在 Debian/Ubuntu 上可用此命令： ::

	sudo apt-get update && sudo apt-get install ceph-common

在 CentOS/RHEL 上可用此命令： ::

	sudo yum install ceph-common

确保你已安装最新版。如果还没有，你也许得卸载、清除相关依赖，然\
后重新安装。

关于新命令行界面的详细情况请参考 `v0.65`_ 。

.. _v0.65: http://ceph.com/docs/master/release-notes/#v0-65


升级顺序
--------

把旧版软件库改为 Firefly 的，例如用 ``apt`` 命令执行此命令： ::

	sudo rm /etc/apt/sources.list.d/ceph.list
	echo deb http://download.ceph.com/debian-firefly/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list

对于 CentOS/Red Hat 发行版，先删除旧版软件库。 ::

	sudo rm /etc/yum.repos.d/ceph.repo

然后新增一个 ``ceph.repo`` 软件库条目，其内容如下，还有把 \
``{distro}`` 替换为你的发行版名字（如 ``el6`` 、 ``rhel6`` 、 \
``rhel7`` 等等）。

.. code-block:: ini

	[ceph]
	name=Ceph Packages and Backports $basearch
	baseurl=http://download.ceph.com/rpm-firefly/{distro}/$basearch
	enabled=1
	gpgcheck=1
	type=rpm-md
	gpgkey=https://download.ceph.com/keys/release.asc


按如下顺序升级守护进程：

#. **监视器：** 如果 ``ceph-mon`` 守护进程晚于 ``ceph-osd`` 守护进程启动，那么这\
   些监视器就不能正确注册其能力，新功能也不可用，除非再次重启。

#. **OSD**

#. **元数据服务器：** 如果 ``ceph-mds`` 守护进程先被重启了，它只能先等着，直到所\
   有 OSD 都升级完，它才能完全启动。

#. **网关：** 一起升级 ``radosgw`` 守护进程。多片上传功能有微小的行为变化，它会阻\
   止由新版 ``radosgw`` 发起、却由旧版 ``radosgw`` 完成的多片上传请求。

.. note:: 确保先升级完\ **所有** Ceph 监视器、\ **而且**\ 重启\ **完毕**\ ，然后\
   再升级并重启各OSD、各元数据服务器和网关。


Emperor 到 Firefly
==================

如果您的现有集群运行着低于 v0.67 Dumpling 的版本，请先升级到最新的 Dumpling 版，\
然后再升级到 v0.80 Firefly 版。详细步骤请参考 `Cuttlefish 到 Dumpling`_ 和 \
`Firefly 发布说明`_\ 。若从 Emperor 之后的版本升级，请参考 `Firefly 发布说明`_\ 。


Ceph 配置文件变更
-----------------

我们建议升级前先把下列配置加入 ``ceph.conf`` 配置文件的 ``[mon]`` \
段下： ::

    mon warn on legacy crush tunables = false

此配置可消除因用着老 CRUSH 归置法而引起的健康告警。虽说可以在全\
集群范围内重均衡已有数据，但我们不建议生产集群做，因为它涉及大量\
数据，而且重均衡会导致严重的性能降级。


升级顺序
--------

把旧版软件库改为 Firefly 的，例如用 ``apt`` 命令执行此命令： ::

	sudo rm /etc/apt/sources.list.d/ceph.list
	echo deb http://download.ceph.com/debian-firefly/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list

对于 CentOS/Red Hat 发行版，先删除旧版软件库。 ::

	sudo rm /etc/yum.repos.d/ceph.repo

然后新增一个 ``ceph.repo`` 软件库条目，其内容如下，还有把 \
``{distro}`` 替换为你的发行版名字（如 ``el6`` 、 ``rhel6`` 、 \
``rhel7`` 等等）。

.. code-block:: ini

	[ceph]
	name=Ceph Packages and Backports $basearch
	baseurl=http://download.ceph.com/rpm/{distro}/$basearch
	enabled=1
	gpgcheck=1
	type=rpm-md
	gpgkey=https://download.ceph.com/keys/release.asc


.. note:: 确保使用与自己发行版相匹配的 URL ，对应发行版请检查 \
   http://download.ceph.com/rpm 。

.. note:: 如果你可以用 ``ceph-deploy`` 升级软件，那你只需要把仓\
   库加到运行 ``ceph`` 或 ``ceph-deploy`` 命令的客户端节点即可。


按如下顺序升级守护进程：

#. **监视器：** 如果 ``ceph-mon`` 守护进程晚于 ``ceph-osd`` 守\
   护进程启动，那么这些监视器就不能正确注册其能力，新功能也不可\
   用，除非再次重启。

#. **OSD**

#. **元数据服务器：** 如果 ``ceph-mds`` 守护进程先被重启了，它\
   只能先等着，直到所有 OSD 都升级完，它才能完全启动。

#. **网关：** 一起升级 ``radosgw`` 守护进程。多片上传功能有微小\
   的行为变化，它会阻止由新版 ``radosgw`` 发起、却由旧版 \
   ``radosgw`` 完成的多片上传请求。


升级过程
========

下面是具体升级进程。

.. important:: Ceph 的各版本可能有不同的步骤，所以\ **升级前**\
   请参考此版本特定的升级步骤。


升级监视器
----------

要升级监视器，执行下列步骤：

#. 升级各守护进程的二进制包。

   你可以用 ``ceph-deploy`` 一次升级所有监视器节点，如： ::

	ceph-deploy install --release {release-name} ceph-node1[ ceph-node2]
	ceph-deploy install --release hammer mon1 mon2 mon3

   你也可以用包管理器挨个升级各节点。手动升级 Debian/Ubuntu 主机上\
   软件包的步骤如下： ::

	ssh {mon-host}
	sudo apt-get update && sudo apt-get install ceph

   在 CentOS/Red Hat 主机上相应的命令如下： ::

	ssh {mon-host}
	sudo yum update && sudo yum install ceph

#. 重启各监视器。 Debian/Ubuntu 发行版的命令如下： ::

	sudo restart ceph-mon id={hostname}

   CentOS/Red Hat/Debian 发行版的命令如下： ::

	sudo /etc/init.d/ceph restart {mon-id}

   用 ``ceph-deploy`` 部署的 CentOS/Red Hat 发行版，其监视器 ID \
   通常是 ``mon.{hostname}`` 。

#. 确保各监视器都重回法定人数。 ::

	ceph mon stat

再次确认你完成了所有监视器的升级。


升级单个 OSD
------------

升级单个 OSD 守护进程的步骤如下：

#. 升级 OSD 守护进程对应的软件包。

   你可以用 ``ceph-deploy`` 一次升级所有 OSD 守护进程，如： ::

	ceph-deploy install --release {release-name} ceph-node1[ ceph-node2]
	ceph-deploy install --release hammer osd1 osd2 osd3

   你也可以用包管理器挨个升级各节点。手动升级 Debian/Ubuntu 主机上\
   软件包的步骤如下。 ::

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


升级单个元数据服务器
--------------------

要升级单个 Ceph 元数据服务器，挨个执行下列步骤：

#. 升级二进制包。你可以用 ``ceph-deploy`` 一次升级所有 MDS 节点，\
   也可以在各节点用包管理器升级，如： ::

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

   用 ``ceph-deploy`` 部署的集群其 ``{hostname}`` 通常是所在主机的主机名。

#. 确保元数据服务器已启动，且运行着： ::

	ceph mds stat


升级客户端
----------

升级软件包并重启完集群之后，我们建议同时升级下客户端节点上的 ``ceph-common`` 和客户\
端库（ ``librbd1`` 和 ``librados2`` ）。

#. 升级软件包： ::

	ssh {client-host}
	apt-get update && sudo apt-get install ceph-common librados2 librbd1 python-rados python-rbd

#. 确认升级后的版本： ::

	ceph --version

如果不是最新版，你也许得卸载、清理掉依赖、然后重新安装。


迁移到 ceph-deploy
==================

如果您的现有集群是用 ``mkcephfs`` 部署的（通常是 Argonaut 或 Bobtail ），那么有些\
地方要改一下才能切换到 ``ceph-deploy`` 。


监视器密钥环
------------

要把 ``caps mon = "allow *"`` 加进监视器密钥环（如果还没加的话）。默认情况下监视器\
密钥环位于 ``/var/lib/ceph/mon/ceph-$id/keyring`` ，增加 ``caps`` 配置后它应该类\
似如下： ::

	[mon.]
		key = AQBJIHhRuHCwDRAAZjBTSJcIBIoGpdOR9ToiyQ==
		caps mon = "allow *"

添加 ``caps mon = "allow *"`` 将简化 ``mkcephfs`` 到 ``ceph-deploy`` 的迁移，这\
样 ``ceph-create-keys`` 就可以用 ``$mon_data`` 里的 ``mon.`` 密钥环获取它所需的权\
限了。


用默认路径
----------

``mon`` 和 ``osd`` 目录需用 ``/var/lib/ceph`` 之下的默认子目录。

- **OSDs**: 路径应该是 ``/var/lib/ceph/osd/ceph-$id``
- **MON**: 路径应该是 ``/var/lib/ceph/mon/ceph-$id``

在这些目录下都应该有一个名为 ``keyring`` 的密钥环文件。


.. _监视器配置: ../../rados/configuration/mon-config-ref
.. _Joao 的博客文章: http://ceph.com/dev-notes/cephs-new-monitor-changes
.. _Ceph 认证: ../../rados/operations/authentication/
.. _Ceph 认证——向后兼容性: ../../rados/operations/authentication/#backward-compatibility
.. _手动升级: ../install-storage-cluster/
.. _操纵集群: ../../rados/operations/operating
.. _监控集群: ../../rados/operations/monitoring
.. _Firefly 发布说明: ../../release-notes/#v0-80-firefly
.. _发布说明: ../../release-notes
