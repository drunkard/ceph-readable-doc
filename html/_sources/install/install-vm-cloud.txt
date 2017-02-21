========================
 为块设备安装虚拟化支持
========================

If you intend to use Ceph Block Devices and the Ceph Storage Cluster as a
backend for Virtual Machines (VMs) or  :term:`Cloud Platforms` the QEMU/KVM and
``libvirt`` packages are important for enabling VMs and cloud platforms.
Examples of VMs include: QEMU/KVM, XEN, VMWare, LXC, VirtualBox, etc. Examples
of Cloud Platforms include OpenStack, CloudStack, OpenNebula, etc.


.. ditaa::  +---------------------------------------------------+
            |                     libvirt                       |
            +------------------------+--------------------------+
                                     |
                                     | configures
                                     v
            +---------------------------------------------------+
            |                       QEMU                        |
            +---------------------------------------------------+
            |                      librbd                       |
            +------------------------+-+------------------------+
            |          OSDs          | |        Monitors        |
            +------------------------+ +------------------------+


安装 QEMU
=========

QEMU KVM 能通过 ``librbd`` 库使用 Ceph 块设备，这对云平台是否能\
采用 Ceph 来说很重要。装好 QEMU 后，可以参考 `QEMU 和块设备`_\ \
把它用起来。


Debian 软件包
-------------

QEMU 二进制包从 Ubuntu 12.04 Precise Pangolin 版起就被合并到了 \
Ubuntu 官方库。执行下面的命令安装 QEMU ： ::

	sudo apt-get install qemu


RPM 软件包
----------

要安装 QEMU ，可按如下步骤：

#. 更新软件库： ::

	sudo yum update

#. 安装能和 Ceph 对接的 QEMU ： ::

	sudo yum install qemu-kvm qemu-kvm-tools qemu-img

#. 安装其它的 QEMU 软件包（可选的）： ::

	sudo yum install qemu-guest-agent qemu-guest-agent-win32


构建 QEMU
---------

To build QEMU from source, use the following procedure::

	cd {your-development-directory}
	git clone git://git.qemu.org/qemu.git
	cd qemu
	./configure --enable-rbd
	make; make install



安装 libvirt
============

To use ``libvirt`` with Ceph, you must have a running Ceph Storage Cluster, and
you must have installed and configured QEMU. See `Using libvirt with Ceph Block
Device`_ for usage.


Debian 软件包
-------------

``libvirt`` 软件包从 Ubuntu 12.04 Precise Pangolin 起就被并入\
了 Ubuntu 官方库。要在这些发行版上安装 ``libvirt`` ，可用下面\
的命令： ::

	sudo apt-get update && sudo apt-get install libvirt-bin


RPM 软件包
----------

要通过 ``libvirt`` 使用 Ceph 存储集群，你必须有个正常运行的 \
Ceph 集群、还必须安装支持 ``rbd`` 格式的 QEMU 。详情见 \
`安装 QEMU`_ 。

近期的 CentOS/RHEL 发行版都集成了 ``libvirt`` 软件包。可执行\
如下命令安装 ``libvirt`` ： ::

	sudo yum install libvirt


构建 ``libvirt``
----------------

To build ``libvirt`` from source, clone the ``libvirt`` repository and use
`AutoGen`_ to generate the build. Then, execute ``make`` and ``make install`` to
complete the installation. For example::

	git clone git://libvirt.org/libvirt.git
	cd libvirt
	./autogen.sh
	make
	sudo make install

详情见 `libvirt 安装手册`_\ 。


.. _libvirt 安装手册: http://www.libvirt.org/compiling.html
.. _AutoGen: http://www.gnu.org/software/autogen/
.. _QEMU 和块设备: ../../rbd/qemu-rbd
.. _Using libvirt with Ceph Block Device: ../../rbd/libvirt
