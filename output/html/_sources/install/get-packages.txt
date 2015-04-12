==============
 获取二进制包
==============

要安装 Ceph 及其依赖软件，你需要参考本手册从 Ceph 软件库下载，然后继续看\ `安装 
Ceph 对象存储`_\ 。


获取软件包
==========

有两种方法获取软件包：

- **增加源：** 增加源是获取二进制包的最简方法，因为多数情况下包管理工具都能自动下\
  载、并解决依赖关系。然而，这种方法要求各 :term:`Ceph 节点`\ 都能连接互联网。
  
- **手动下载：** 如果你的环境不允许 :term:`Ceph 节点`\ 访问互联网，手动下载软件包\
  安装 Ceph 也不复杂。


准备工作
========

所有 Ceph 部署都需要 Ceph 软件包（除非是开发），你应该安装相应的密钥和推荐的软件包。

- **密钥：（推荐）** 不管你是用仓库还是手动下载，你都需要用密钥校验软件包。如果你没\
  有密钥，就会收到安全警告。有两个密钥：一个用于发布（常用）、一个用于开发（仅适用于\
  程序员和 QA ），请按需选择，详情见\ `安装密钥`_\ 。

- **Ceph Extras:（必要）** Ceph 附加库提供了它所依赖的软件包，这些软件包比发行版\
  自带的更新，它们对于 Ceph 的正常运行是必要的。比如，适合 CentOS/RHEL 发行版的 \
  QEMU 包、 iSCSI 相关的包。如果你想用其中的任何一个，都需要添加 Ceph Extra 源、\
  或手动下载；此库也包含 Ceph 的依赖，因为有些人想手动安装。详情见\ \
  `添加 Ceph Extra 库`_\ 。

- **Ceph:（必要）** 所有 Ceph 部署都需要 Ceph 发布的软件包，除非你部署开发版软件\
  包（仅有开发版、 QA 、和尖端部署）。详情见\ `添加 Ceph 库`_\ 。

- **Ceph Development:（可选）** 如果你在做 Ceph 开发、为 Ceph 做构建测试、或者急\
  需开发版中的尖端功能，可以安装开发版软件包，详情见 `添加 Ceph 开发库`_ 。

- **Apache/FastCGI:（可选）** 如果你想部署 :term:`Ceph 对象存储`\ 服务，那么必须\
  安装 Apache 和 FastCGI 。 Ceph 库提供的 Apache 和 FastCGI 二进制包和来自 \
  Apache 的是一样的，但它打开了 100-continue 支持。如果你想启用 \
  :term:`Ceph 对象网关`\ 、且支持 100-continue ，那必须从 Ceph 库下载 \
  Apache/FastCGI 软件包。详情见\ `添加 Apache/CGI 源`_\ 。


如果你想手动下载二进制包，请参考\ `下载软件包`_\ 。


安装密钥
========

把密钥加入你系统的可信密钥列表内，以消除安全告警。对主要发行版（如 ``dumpling`` 、 \
``emperor`` 、 ``firefly`` ）和开发版（如 ``release-name-rc1`` 、 \
``release-name-rc2`` ）应该用 ``release.asc`` 密钥；开发中的测试版应使用 \
``autobuild.asc`` 密钥（开发者和 QA ）。


APT
---

执行下列命令安装 ``release.asc`` 内的密钥： ::

	wget -q -O- 'https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc' | sudo apt-key add -


执行下列命令安装 ``autobuild.asc`` 密钥（仅适用于 QA 和开发者）： ::

	wget -q -O- 'https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/autobuild.asc' | sudo apt-key add -


RPM
---

执行下列命令安装 ``release.asc`` 密钥： ::

	sudo rpm --import 'https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc'

执行下列命令安装 ``autobuild.asc`` 密钥（仅对 QA 和开发者）： ::

	sudo rpm --import 'https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/autobuild.asc'



添加 Ceph Extra 库
==================

有些 Ceph 部署需要较新（与 Linux 发行版内建仓库相比）、且启用了 Ceph 支持的软件包，\
比如 Ceph Extra 包含了较新的、启用了 Ceph 支持的 SCSI 目标框架、和 QEMU RPM 包，还\
包含了 ``curl`` 、 ``leveldb`` 等其他依赖包。所以请添加 Ceph Extra 库，以确保你能\
获取到这些附加软件包。


Debian 二进制包
---------------

把 Ceph Extra 库添加到你的 APT 源列表里。 ::

	echo deb http://ceph.com/packages/ceph-extras/debian $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph-extras.list


RPM 二进制包
------------

.. note:: 在基于 RPM 的系统中，只有基于 EL6 的发行版（ RHEL 6 、 CentOS 6 、 \
   Scientific Linux 6）需要 ceph-extras ； Fedora 或 RHEL 7+ 系统不需要。

对于 RPM 包，仓库的定义需要加到 ``/etc/yum.repos.d/`` 下面（如 \
``ceph-extras.repo`` ）。某些包（如 QEMU ）必须优先于标准包，所以要设置 \
``priority=2`` 。 ::

	[ceph-extras]
	name=Ceph Extras Packages
	baseurl=http://ceph.com/packages/ceph-extras/rpm/{distro}/$basearch
	enabled=1
	priority=2
	gpgcheck=1
	type=rpm-md
	gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc

	[ceph-extras-noarch]
	name=Ceph Extras noarch
	baseurl=http://ceph.com/packages/ceph-extras/rpm/{distro}/noarch
	enabled=1
	priority=2
	gpgcheck=1
	type=rpm-md
	gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc

	[ceph-extras-source]
	name=Ceph Extras Sources
	baseurl=http://ceph.com/packages/ceph-extras/rpm/{distro}/SRPMS
	enabled=1
	priority=2
	gpgcheck=1
	type=rpm-md
	gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc


添加 Ceph 库
============

发布库用 ``release.asc`` 公钥校验软件包。要通过 APT 或 YUM 安装 Ceph 二进制包，必\
须先配置库。

适合 Debian/Ubuntu 的包位于： ::

	http://ceph.com/debian-{release-name}

适合 CentOS/RHEL 和其他发行版（通过 YUM 安装）的包位于： ::

	http://ceph.com/rpm-{release-name}

Ceph 的主要发布包括：

- **Giant:** 是 Ceph 的最新主要发布。这些包适合于生产环境，重要的缺陷修正会\
  移植回来、并在必要时发布修正版。

- **Firefly:** 是 Ceph 的第六个主要发布。这些包适合于生产环境，重要的缺陷\
  修正会移植回来、并在必要时发布修正版。

- **Emperor:** 是 Ceph 的第五个主要发布。这些包比较老了，而且已没有后续支\
  持，所以我们建议升级到 Firefly 。
  
- **Dumpling:** 是 Ceph 的第四个主要发布。这些包比较老了，不建议新用户使\
  用，但重大缺陷在必要时仍会修复。我们鼓励所有 Dumpling 用户尽快升级到 Firefly 。

- **Argonaut, Bobtail, Cuttlefish:** 这些是 Ceph 的前三个主要发布，它们都很老了，且不再维护，所以请升级到仍支持的版本。

.. tip:: 荷兰有个镜像 http://eu.ceph.com/ ，适合欧洲用户使用。


Debian 二进制包
---------------

把 Ceph 库加入系统级 APT 源列表。在较新版本的 Debian/Ubuntu 上，用命令 \
``lsb_release -sc`` 可获取短代码名，然后用它替换下列命令里的 ``{codename}`` 。 ::

	sudo apt-add-repository 'deb http://ceph.com/debian-firefly/ {codename} main'

对于早期 Linux 发行版，你可以执行下列命令： ::

	echo deb http://ceph.com/debian-firefly/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list

对于早期 Ceph 发布，可用 Ceph 发布名替换 ``{release-name}`` 。用命令 \
``lsb_release -sc`` 可获取短代码名，然后用它替换下列命令里的 ``{codename}`` 。 ::

	sudo apt-add-repository 'deb http://ceph.com/debian-{release-name}/ {codename} main'

对较老的 Linux 发行版，用发布名替换 ``{release-name}`` 。 ::

	echo deb http://ceph.com/debian-{release-name}/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list

要在 ARM 处理器上运行 Ceph 的话，需要 Google 的内存剖析工具（\ ``google-perftools`` ）， \
Ceph 库里有： http://ceph.com/packages/google-perftools/debian 。 ::

	echo deb http://ceph.com/packages/google-perftools/debian  $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/google-perftools.list


对于开发版，把我们的软件库加入 APT 源。这里 `Debian 测试版软件库`_ 是已支持的 \
Debian/Ubuntu 列表。 ::

	echo deb http://ceph.com/debian-testing/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list


RPM 二进制包
------------

对于主要发布，你可以在 ``/etc/yum.repos.d/`` 目录下新增一个 Ceph 库：创建 \
``ceph.repo`` 。在下例中，需要用 Ceph 主要发布名（如 ``dumpling`` 、 \
``emperor`` ）替换 ``{ceph-release}`` 、用 Linux 发行版名（ ``el6`` 、 \
``rhel6`` 等）替换 ``{distro}`` 。你可以到 http://ceph.com/rpm-{ceph-release}/ \
看看 Ceph 支持哪些发行版。有些 Ceph 包（如 EPEL ）必须优先于标准包，所以你必须确保\
设置了 ``priority=2`` 。 ::

	[ceph]
	name=Ceph packages for $basearch
	baseurl=http://ceph.com/rpm-{ceph-release}/{distro}/$basearch
	enabled=1
	priority=2
	gpgcheck=1
	type=rpm-md
	gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc

	[ceph-noarch]
	name=Ceph noarch packages
	baseurl=http://ceph.com/rpm-{ceph-release}/{distro}/noarch
	enabled=1
	priority=2
	gpgcheck=1
	type=rpm-md
	gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc

	[ceph-source]
	name=Ceph source packages
	baseurl=http://ceph.com/rpm-{ceph-release}/{distro}/SRPMS
	enabled=0
	priority=2
	gpgcheck=1
	type=rpm-md
	gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc


如果想用开发版，你也可以用相应配置： ::

	[ceph]
	name=Ceph packages for $basearch/$releasever
	baseurl=http://ceph.com/rpm-testing/{distro}/$basearch
	enabled=1
	priority=2
	gpgcheck=1
	type=rpm-md
	gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc

	[ceph-noarch]
	name=Ceph noarch packages
	baseurl=http://ceph.com/rpm-testing/{distro}/noarch
	enabled=1
	priority=2
	gpgcheck=1
	type=rpm-md
	gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc

	[ceph-source]
	name=Ceph source packages
	baseurl=http://ceph.com/rpm-testing/{distro}/SRPMS
	enabled=0
	priority=2
	gpgcheck=1
	type=rpm-md
	gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc


对于某些包，你可以通过名字直接下载。按照我们的开发进度，每 3-4 周会发布一次。这些包\
的变动比主要发布频繁，开发版会迅速地集成新功能，然而这些新功能需要几周时间的质检才会\
发布。

软件库包会把软件库的具体配置安装到本机，以便 ``yum`` 或 ``up2date`` 使用。把 \
``{distro}`` 替换成你的 Linux 发行版名字，把 ``{release}`` 换成 Ceph 的某个发布名。 ::

    su -c 'rpm -Uvh http://ceph.com/rpms/{distro}/x86_64/ceph-{release}.el6.noarch.rpm'

你可以从这个地址直接下载 RPM ： ::

     http://ceph.com/rpm-testing


添加 Ceph 开发库
================

开发库用 ``autobuild.asc`` 密钥校验软件包。如果你在参与 Ceph 开发，想要部署并测试\
某个分支，确保先删除（或禁用）主要版本库的配置文件。


Debian 二进制包
--------------- 

我们自动为 Debian 和 Ubuntu 构建 Ceph 当前分支的二进制包，这些包只适合开发者和质检\
人员。

把此仓库加入 APT 源，用你要测试的分支名（如 chef-3、wip-hack、master ）替换 \
``{BRANCH}`` 。我们所构建的完整分支列表在 `the gitbuilder page`_ 。 ::

	echo deb http://gitbuilder.ceph.com/ceph-deb-$(lsb_release -sc)-x86_64-basic/ref/{BRANCH} $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list


RPM 二进制包
------------

对于当前开发分支，你可以在 ``/etc/yum.repos.d/`` 目录下创建 ``ceph.repo`` 文件，\
内容如下，用你的 Linux 发行版名字（ ``centos6`` 、 ``rhel6`` 等）替换 \
``{distro}`` 、用你想安装的分支名替换 ``{branch}`` 。 ::

	[ceph-source]
	name=Ceph source packages
	baseurl=http://gitbuilder.ceph.com/ceph-rpm-{distro}-x86_64-basic/ref/{branch}/SRPMS
	enabled=0
	gpgcheck=1
	type=rpm-md
	gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/autobuild.asc


你可以到 http://gitbuilder.ceph.com 查看 Ceph 支持哪些发行版。


添加 Apache/CGI 源
==================

Ceph 对象存储与普通的 Apache 和 FastCGI 库对接，只是 Ceph 要求 Apache 和 FastCGI \
支持 100-continue 功能。请配置相应的软件库，以使用对应的 Apache 和 FastCGI 包。

Debian 二进制包
---------------

如果想要 100-continue 功能，请把我们的源加入 APT 源列表。 ::

	echo deb http://gitbuilder.ceph.com/apache2-deb-$(lsb_release -sc)-x86_64-basic/ref/master $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph-apache.list
	echo deb http://gitbuilder.ceph.com/libapache-mod-fastcgi-deb-$(lsb_release -sc)-x86_64-basic/ref/master $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph-fastcgi.list


RPM 二进制包
------------

你可以在 ``/etc/yum.repos.d/`` 目录下创建 ``ceph-apache.repo`` 文件，内容如下，\
用你的 Linux 发行版名字（如 ``el6`` 、 ``rhel6`` ）替换 ``{distro}`` ， \
http://gitbuilder.ceph.com  列出了支持的发行版。 ::

	[apache2-ceph-noarch]
	name=Apache noarch packages for Ceph
	baseurl=http://gitbuilder.ceph.com/apache2-rpm-{distro}-x86_64-basic/ref/master
	enabled=1
	priority=2
	gpgcheck=1
	type=rpm-md
	gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/autobuild.asc

	[apache2-ceph-source]
	name=Apache source packages for Ceph
	baseurl=http://gitbuilder.ceph.com/apache2-rpm-{distro}-x86_64-basic/ref/master
	enabled=0
	priority=2
	gpgcheck=1
	type=rpm-md
	gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/autobuild.asc


仿照前述步骤创建 ``ceph-fastcgi.repo`` 文件。 ::

	[fastcgi-ceph-basearch]
	name=FastCGI basearch packages for Ceph
	baseurl=http://gitbuilder.ceph.com/mod_fastcgi-rpm-{distro}-x86_64-basic/ref/master
	enabled=1
	priority=2
	gpgcheck=1
	type=rpm-md
	gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/autobuild.asc

	[fastcgi-ceph-noarch]
	name=FastCGI noarch packages for Ceph
	baseurl=http://gitbuilder.ceph.com/mod_fastcgi-rpm-{distro}-x86_64-basic/ref/master
	enabled=1
	priority=2
	gpgcheck=1
	type=rpm-md
	gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/autobuild.asc

	[fastcgi-ceph-source]
	name=FastCGI source packages for Ceph
	baseurl=http://gitbuilder.ceph.com/mod_fastcgi-rpm-{distro}-x86_64-basic/ref/master
	enabled=0
	priority=2
	gpgcheck=1
	type=rpm-md
	gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/autobuild.asc


下载软件包
==========

如果你位于防火墙之内，不能访问互联网，那你必须先下载齐所需软件包（镜像所有依赖）。

Debian 二进制包
---------------

Ceph 依赖这些第三方库。

- libaio1
- libsnappy1
- libcurl3
- curl
- libgoogle-perftools4
- google-perftools
- libleveldb1


这个软件库包会装好所需的 ``apt`` 软件库的配置文件。需用最新 Ceph 发布替换掉 \
``{release}`` 、用最新 Ceph 版本号替换 ``{version}`` 、用自己的 Linux 发行版代号\
替换 ``{distro}`` 、用自己的 CPU 架构替换 ``{arch}`` 。 ::

	wget -q http://ceph.com/debian-{release}/pool/main/c/ceph/ceph_{version}{distro}_{arch}.deb


RPM 二进制包
------------

Ceph 依赖一些第三方库。执行下列命令添加 EPEL 库： ::

	su -c 'rpm -Uvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm'

Ceph依赖下列包：

- snappy
- leveldb
- gdisk
- python-argparse
- gperftools-libs


当前，我们为这些平台 RHEL/CentOS6 （ ``el6`` ）、 Fedora 18 和 19（ ``f18`` 和 \
``f19`` ）、 OpenSUSE 12.2 （ ``opensuse12.2`` ）和 SLES （ ``sles11`` ）分别构\
建二进制包，仓库包会在本地系统上装好 Ceph 库配置文件，这样 ``yum`` 或 ``up2date`` \
就可以使用这些配置文件自动安装了。用自己的发行版名字替换 ``{distro}`` 。 ::

	su -c 'rpm -Uvh http://ceph.com/rpm-firefly/{distro}/noarch/ceph-{version}.{distro}.noarch.rpm'

例如，对于 CentOS 6 （ ``el6`` ）： ::

	su -c 'rpm -Uvh http://ceph.com/rpm-firefly/el6/noarch/ceph-release-1-0.el6.noarch.rpm'

你可以从这里直接下载RPM包： ::

	http://ceph.com/rpm-firefly


对较老的 Ceph 发布，用 Ceph 发布名替换 ``{release-name}`` ，你可以执行 \
``lsb_release -sc`` 命令获取发行版代号。 ::

	su -c 'rpm -Uvh http://ceph.com/rpm-{release-name}/{distro}/noarch/ceph-{version}.{distro}.noarch.rpm'




.. _安装 Ceph 对象存储: ../install-storage-cluster
.. _Debian 测试版软件库: http://ceph.com/debian-testing/dists
.. _the gitbuilder page: http://gitbuilder.ceph.com
