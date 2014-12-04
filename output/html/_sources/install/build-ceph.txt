===========
 构建 Ceph
===========

你可以下载 Ceph 源码并自行构建。首先，你得准备开发环境、编译 Ceph 、然后安装到用户\
区或者构建二进制包并安装。

构建依赖
========


.. tip:: 对照本段检查下你的 Linux/Unix 发行版是否满足这些依赖。

构建 Ceph 源码前，你得先安装如下库和工具。 Ceph 用 ``autoconf`` 和 ``automake`` \
脚本简化构建， Ceph 构建脚本依赖如下：

- ``autotools-dev``
- ``autoconf``
- ``automake``
- ``cdbs``
- ``gcc``
- ``g++``
- ``git``
- ``libboost-dev``
- ``libedit-dev``
- ``libssl-dev``
- ``libtool``
- ``libfcgi``
- ``libfcgi-dev``
- ``libfuse-dev``
- ``linux-kernel-headers``
- ``libcrypto++-dev``
- ``libcrypto++``
- ``libexpat1-dev``
- ``pkg-config``
- ``libcurl4-gnutls-dev``

在 Ubuntu 上，执行 ``sudo apt-get install`` 分别安装各缺失依赖。
::

	sudo apt-get install autoconf automake autotools-dev libbz2-dev debhelper default-jdk git javahelper junit4 libaio-dev libatomic-ops-dev libbabeltrace-ctf-dev libbabeltrace-dev libblkid-dev libboost-dev libboost-program-options-dev libboost-system-dev libboost-thread-dev libcurl4-gnutls-dev libedit-dev libexpat1-dev libfcgi-dev libfuse-dev libgoogle-perftools-dev libkeyutils-dev libleveldb-dev libnss3-dev libsnappy-dev liblttng-ust-dev libtool libudev-dev libxml2-dev pkg-config python python-argparse python-nose uuid-dev uuid-runtime xfslibs-dev yasm

在 Debian 上，可以执行 ``aptitude install`` 挨个安装还没装好的依赖包。
::

	aptitude install autoconf automake autotools-dev libbz2-dev debhelper default-jdk git javahelper junit4 libaio-dev libatomic-ops-dev libbabeltrace-ctf-dev libbabeltrace-dev libblkid-dev libboost-dev libboost-program-options-dev libboost-system-dev libboost-thread-dev libcurl4-gnutls-dev libedit-dev libexpat1-dev libfcgi-dev libfuse-dev libgoogle-perftools-dev libkeyutils-dev libleveldb-dev libnss3-dev libsnappy-dev liblttng-ust-dev libtool libudev-dev libxml2-dev pkg-config python python-argparse python-nose uuid-dev uuid-runtime xfslibs-dev yasm

.. note:: 在某些支持 Google 内存剖析工具的发行版上，名字未必如此（如 \
   ``libgoogle-perftools4`` ）。

Ubuntu
------

- ``uuid-dev``
- ``libkeyutils-dev``
- ``libgoogle-perftools-dev``
- ``libatomic-ops-dev``
- ``libaio-dev``
- ``libgdata-common``
- ``libgdata13``
- ``libsnappy-dev`` 
- ``libleveldb-dev``

执行 ``sudo apt-get install`` 挨个安装各缺失依赖。
::

	sudo apt-get install uuid-dev libkeyutils-dev libgoogle-perftools-dev libatomic-ops-dev libaio-dev libgdata-common libgdata13 libsnappy-dev libleveldb-dev


Debian
------

另外，你也可以安装：
::

	aptitude install fakeroot dpkg-dev
	aptitude install debhelper cdbs libexpat1-dev libatomic-ops-dev

openSUSE 11.2 （及后续版本）
----------------------------

- ``boost-devel``
- ``gcc-c++``
- ``libedit-devel``
- ``libopenssl-devel``
- ``fuse-devel`` (optional)

执行 ``zypper install`` 安装各缺失依赖。
::

	zypper install boost-devel gcc-c++ libedit-devel libopenssl-devel fuse-devel

Fedora 20
---------

以 root 身份运行：
::

    yum install make automake autoconf  boost-devel fuse-devel gcc-c++ libtool libuuid-devel libblkid-devel keyutils-libs-devel cryptopp-devel fcgi-devel libcurl-devel expat-devel gperftools-devel libedit-devel libatomic_ops-devel snappy-devel leveldb-devel libaio-devel xfsprogs-devel git libudev-devel


构建 Ceph
=========

Ceph 用 ``automake`` 和 ``configure`` 脚本简化构建过程。先进入刚克隆的 Ceph 源码\
库，执行下列命令开始构建：
::

	cd ceph
	./autogen.sh
	./configure
	make

.. topic:: 超线程

	你可以根据自己的硬件配置情况用 ``make -j`` 并行编译，比如在双核处理器上用 \
	``make -j4`` 可能会编译得快些。

参考\ `安装自构建软件`_\ 把构建好的软件安装到用户区。


构建 Ceph 安装包
================

要构建安装包，你必须克隆 `Ceph`_ 源码库。用 ``dpkg-buildpackage`` 基于最新代码为 \
Debian/Ubuntu 创建安装包；用 ``rpmbuild`` 为 RPM 包管理器创建安装包。

.. tip:: 在多核 CPU 上构建时，用参数 ``-j`` 、再加上核心数的 2 倍数，例如在双核处\
   理器上用 ``-j4`` 来加速构建。


高级打包工具（ APT ）
---------------------

要为 Debian/Ubuntu 创建 ``.deb`` 安装包，先要克隆 Ceph 源码库、安装好必要的\ `构\
建依赖`_\ 和 ``debhelper`` 。
::

	sudo apt-get install debhelper

装好 ``debhelper`` 之后就可以开始构建安装包了：
::

	sudo dpkg-buildpackage

在多核处理器上可以用 ``-j`` 加快构建速度。


RPM 包管理器
------------

要创建 ``.rpm`` 包，先得克隆 `Ceph`_ 源码库、安装必要的\ `构建依赖`_\ 、安装好 \
``rpm-build`` 和 ``rpmdevtools`` ：
::

	yum install rpm-build rpmdevtools

安装完这些工具后，设置 RPM 编译环境：
::

	rpmdev-setuptree

下载源码包，编译 RPM 时需要：
::

	wget -P ~/rpmbuild/SOURCES/ http://ceph.com/download/ceph-<version>.tar.bz2

或者从欧洲镜像下载：
::

	wget -P ~/rpmbuild/SOURCES/ http://eu.ceph.com/download/ceph-<version>.tar.bz2

提取规范文件：
::

    tar --strip-components=1 -C ~/rpmbuild/SPECS/ --no-anchored -xvjf ~/rpmbuild/SOURCES/ceph-<version>.tar.bz2 "ceph.spec"

开始构建 RPM 包：
::

	rpmbuild -ba ~/rpmbuild/SPECS/ceph.spec

在多核处理器上可以用 ``-j`` 加快构建速度。

.. _Ceph: ../clone-source
.. _安装自构建软件: ../install-storage-cluster#installing-a-build
