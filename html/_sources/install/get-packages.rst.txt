.. _packages:

==============
 获取二进制包
==============

要安装 Ceph 及其依赖软件，你需要参考本手册从 Ceph 软件库下\
载，然后继续看\ `安装 Ceph 对象存储`_\ 。


.. Getting Packages

获取软件包
==========

有两种方法获取软件包：

- **增加源：** 增加源是获取二进制包的最简方法，因为多数情\
  况下包管理工具都能自动下载、并解决依赖关系。然而，这种方\
  法要求各 :term:`Ceph 节点`\ 都能连接互联网。
  
- **手动下载：** 如果你的环境不允许 :term:`Ceph 节点`\ 访\
  问互联网，手动下载软件包安装 Ceph 也不复杂。


.. Requirements

准备工作
========

所有 Ceph 部署都需要 Ceph 软件包（除非是开发），你应该安装\
相应的密钥和推荐的软件包。

- **密钥：（推荐）** 不管你是用仓库还是手动下载，你都需要用\
  密钥校验软件包。如果你没有密钥，就会收到安全警告。详情见\
  `安装密钥`_\ 。

- **Ceph:（必要）** 所有 Ceph 部署都需要 Ceph 发布的软件包，\
  除非你部署开发版软件包（仅有开发版、 QA 、和尖端部署）。\
  详情见\ `添加 Ceph 库`_\ 。

- **Ceph Development:（可选）** 如果你在做 Ceph 开发、为 Ceph
  做构建测试、或者急需开发版中的尖端功能，可以安装开发版\
  软件包，详情见 `添加 Ceph 开发库`_ 。

如果你想手动下载二进制包，请参考\ `下载软件包`_\ 。


.. Add Keys

安装密钥
========

把密钥加入你系统的可信密钥列表内，以消除安全告警。对主要发行版\
（如 ``luminous`` 、 ``mimic`` 、 ``nautilus`` ）和开发版（如 \
``release-name-rc1`` 、 ``release-name-rc2`` ）应该用 \
``release.asc`` 密钥。


APT
---

执行下列命令安装 ``release.asc`` 内的密钥： ::

	wget -q -O- 'https://download.ceph.com/keys/release.asc' | sudo apt-key add -


RPM
---

执行下列命令安装 ``release.asc`` 密钥： ::

	sudo rpm --import 'https://download.ceph.com/keys/release.asc'


.. Add Ceph

添加 Ceph 库
============

发布库用 ``release.asc`` 公钥校验软件包。要通过 APT 或 YUM
安装 Ceph 二进制包，必须先配置库。

适合 Debian/Ubuntu 的包位于： ::

	https://download.ceph.com/debian-{release-name}

适合 CentOS/RHEL 和其他发行版（通过 YUM 安装）的包位于： ::

	https://download.ceph.com/rpm-{release-name}

Ceph 的主要版本都汇总到了 :ref:`ceph-releases-general`\ 。

每两个主要发布会有一个长期稳定版（ LTS ），严重的缺陷修正会移\
植到 LTS 版，直到它退役。退役版本不再维护，所以我们建议用户们\
定期升级集群——最好升级到最新的 LTS 版。

.. tip:: 对不在美国的用户来说，你也许可以从比较近的镜像下载
   Ceph 。请参考 `Ceph 镜像`_\ 。


.. Debian Packages

Debian 二进制包
---------------

把 Ceph 库加入系统级 APT 源列表。在较新版本的 Debian/Ubuntu
上，用命令 ``lsb_release -sc`` 可获取短代码名，然后用它替换\
下列命令里的 ``{codename}`` 。 ::

	sudo apt-add-repository 'deb https://download.ceph.com/debian-luminous/ {codename} main'

对于早期 Linux 发行版，你可以执行下列命令： ::

	echo deb https://download.ceph.com/debian-luminous/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list

对于早期 Ceph 发布，可用 Ceph 发布名替换 ``{release-name}`` 。\
用命令 ``lsb_release -sc`` 可获取短代码名，然后用它替换下列\
命令里的 ``{codename}`` 。 ::

	sudo apt-add-repository 'deb https://download.ceph.com/debian-{release-name}/ {codename} main'

对较老的 Linux 发行版，用发布名替换 ``{release-name}`` 。 ::

	echo deb https://download.ceph.com/debian-{release-name}/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list

对于开发版，把我们的软件库加入 APT 源。这里 \
`Debian 测试版软件库`_ 是已支持的 Debian/Ubuntu 列表。 ::

	echo deb https://download.ceph.com/debian-testing/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list

.. tip:: 对不在美国的用户来说，你也许可以从比较近的镜像下载
   Ceph 。请参考 `Ceph 镜像`_\ 。


.. RPM Packages

RPM 二进制包
------------

RHEL
----

对于主要发布，你可以在 ``/etc/yum.repos.d/`` 目录下新增一个
Ceph 库：创建 ``ceph.repo`` 文件。在下例中，需要用 Ceph 主要\
发布名（如 ``luminous`` 、 ``mimic`` 、 ``nautilus`` 等等）替换
``{ceph-release}`` 、用 Linux 发行版名（ ``el7`` 等等）替换
``{distro}`` 。你可以到 https://download.ceph.com/rpm-{ceph-release}/
看看 Ceph 支持哪些发行版。有些 Ceph 包（如 EPEL ）必须优先于\
标准包，所以你必须确保设置了 ``priority=2`` 。 ::

	[ceph]
	name=Ceph packages for $basearch
	baseurl=https://download.ceph.com/rpm-{ceph-release}/{distro}/$basearch
	enabled=1
	priority=2
	gpgcheck=1
	gpgkey=https://download.ceph.com/keys/release.asc

	[ceph-noarch]
	name=Ceph noarch packages
	baseurl=https://download.ceph.com/rpm-{ceph-release}/{distro}/noarch
	enabled=1
	priority=2
	gpgcheck=1
	gpgkey=https://download.ceph.com/keys/release.asc

	[ceph-source]
	name=Ceph source packages
	baseurl=https://download.ceph.com/rpm-{ceph-release}/{distro}/SRPMS
	enabled=0
	priority=2
	gpgcheck=1
	gpgkey=https://download.ceph.com/keys/release.asc

对于某些包，你可以通过名字直接下载。按照我们的开发进度，每 3-4
周会发布一次。这些包的变动比主要发布频繁，开发版会迅速地集成新\
功能，然而这些新功能需要几周时间的质检才会发布。

软件库包会把软件库的具体配置安装到本机，以便 ``yum`` 使用。把
``{distro}`` 替换成你的 Linux 发行版名字，把 ``{release}`` 换\
成 Ceph 的某个发布名。 ::

	su -c 'rpm -Uvh https://download.ceph.com/rpms/{distro}/x86_64/ceph-{release}.el7.noarch.rpm'

你可以从这个地址直接下载 RPM ： ::

	https://download.ceph.com/rpm-testing

.. tip:: 对于不在美国的用户来说，你也许可以从比较近的镜像下载
   Ceph 。请参考 `Ceph 镜像`_\ 。


openSUSE Leap 15.1
------------------
You need to add the Ceph package repository to your list of zypper sources. This can be done with the following command ::

    zypper ar https://download.opensuse.org/repositories/filesystems:/ceph/openSUSE_Leap_15.1/filesystems:ceph.repo


openSUSE Tumbleweed
-------------------
The newest major release of Ceph is already available through the normal Tumbleweed repositories.
There's no need to add another package repository manually.


.. Add Ceph Development

添加 Ceph 开发库
================

如果你在参与 Ceph 开发，想要部署并测试某个分支，确保先删除\
主版本库的配置文件。


.. DEB Packages

DEB 二进制包
------------ 

我们自动为 Ubuntu 构建 Ceph 当前开发分支的二进制包，这\
些包只适合开发者和质检人员。

把此仓库加进你的 APT 源，用你要测试的分支名（如 wip-hack 、 \
master ）替换 ``{BRANCH}`` 。我们所构建发布的完整列表在 \
`shaman 网页`\ 。 ::

    curl -L https://shaman.ceph.com/api/repos/ceph/{BRANCH}/latest/ubuntu/$(lsb_release -sc)/repo/ | sudo tee /etc/apt/sources.list.d/shaman.list

.. note:: 如果某个仓库还没准备好，你就会遇到 HTTP 504 。

上面 URL 里用了 ``latest`` ，它用来指示本次构建的最后一个\
提交。另外，还能指定某个特定的 sha1 号码。要给 Ubuntu Xenial
构建 Ceph 的 master 分支，命令如下： ::

    curl -L https://shaman.ceph.com/api/repos/ceph/master/53e772a45fdf2d211c0c383106a66e1feedec8fd/ubuntu/xenial/repo/ | sudo tee /etc/apt/sources.list.d/shaman.list

.. warning:: 两周后开发库就不再可用了。


.. RPM Packages

RPM 二进制包
------------

对于当前开发分支，你可以在 ``/etc/yum.repos.d/`` 目录下创建 \
Ceph 条目。你可以从 `shaman 网页`\ 获取软件库文件的所有细节，\
可以通过 HTTP 请求获取，例如： ::

    curl -L https://shaman.ceph.com/api/repos/ceph/{BRANCH}/latest/centos/7/repo/ | sudo tee /etc/yum.repos.d/shaman.repo

上面 URL 里用了 ``latest`` ，它用来指示本次构建的最后一个\
提交。另外，还能指定某个特定的 sha1 号码。要给 CentOS 7
构建 Ceph 的 master 分支，命令如下： ::

    curl -L https://shaman.ceph.com/api/repos/ceph/master/53e772a45fdf2d211c0c383106a66e1feedec8fd/centos/7/repo/ | sudo tee /etc/apt/sources.list.d/shaman.list

.. warning:: 两周后开发库就不再可用了。

.. note:: 如果某个仓库还没准备好，你就会遇到 HTTP 504 。


.. Download Packages

下载软件包
==========

如果你位于防火墙之内，不能访问互联网，那你必须先下载齐所需\
软件包（镜像所有依赖）。


.. Debian Packages

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

这个软件库包会装好所需的 ``apt`` 软件库的配置文件。需用最新 Ceph \
发布替换掉 ``{release}`` 、用最新 Ceph 版本号替换 ``{version}`` 、\
用自己的 Linux 发行版代号替换 ``{distro}`` 、用自己的 CPU 架构替\
换 ``{arch}`` 。 ::

	wget -q https://download.ceph.com/debian-{release}/pool/main/c/ceph/ceph_{version}{distro}_{arch}.deb


.. RPM Packages

RPM 二进制包
------------

Ceph 依赖一些第三方库。执行下列命令添加 EPEL 库： ::

        sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

Ceph 依赖下列包：

- snappy
- leveldb
- gdisk
- python-argparse
- gperftools-libs

当前，我们为 RHEL/CentOS7 （ ``el7`` ）平台构建二进制包，\
软件库包会在本地系统上安装 Ceph 库配置文件，这样 ``yum`` 就\
可以使用这些配置文件自动安装了。用你自己的发行版名字替换
``{distro}`` 。 ::

    su -c 'rpm -Uvh https://download.ceph.com/rpm-luminous/{distro}/noarch/ceph-{version}.{distro}.noarch.rpm'

例如，对于 CentOS 7 （ ``el7`` ）： ::

    su -c 'rpm -Uvh https://download.ceph.com/rpm-luminous/el7/noarch/ceph-release-1-0.el7.noarch.rpm'

你可以从这里直接下载RPM包： ::

    https://download.ceph.com/rpm-luminous

对较老的 Ceph 发布，用 Ceph 发布名替换 ``{release-name}`` ，你\
可以执行 ``lsb_release -sc`` 命令获取发行版代号。 ::

	su -c 'rpm -Uvh https://download.ceph.com/rpm-{release-name}/{distro}/noarch/ceph-{version}.{distro}.noarch.rpm'



.. _安装 Ceph 对象存储: ../install-storage-cluster
.. _Debian 测试版软件库: https://download.ceph.com/debian-testing/dists
.. _shaman 网页: https://shaman.ceph.com
.. _Ceph 镜像: ../mirrors
