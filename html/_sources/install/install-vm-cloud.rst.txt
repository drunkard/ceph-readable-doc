========================
 为块设备安装虚拟化支持
========================

如果你想使用 Ceph 块设备、还想把 Ceph 存储集群作为\
虚拟机（ VM ）或 :term:`云平台` 的后端，
QEMU/KVM 和 ``libvirt`` 软件包对于 VM 和云平台的实现很重要。
VM 的具体实现有： QEMU/KVM, XEN, VMWare, LXC, VirtualBox 等等。
云平台的具体实现有： OpenStack 、 CloudStack 、 OpenNebula 等等。


.. ditaa::

            +---------------------------------------------------+
            |                     libvirt                       |
            +------------------------+--------------------------+
                                     |
                                     | configures
                                     v
            +---------------------------------------------------+
            |                       QEMU                        |
            +---------------------------------------------------+
            |                      librbd                       |
            +---------------------------------------------------+
            |                     librados                      |
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
.. Building QEMU

要从源码构建 QEMU ，按下列步骤： ::

	cd {your-development-directory}
	git clone git://git.qemu.org/qemu.git
	cd qemu
	./configure --enable-rbd
	make; make install


安装 libvirt
============
.. Install libvirt

要在 Ceph 上使用 ``libvirt`` ，你必须有正常运行的 Ceph 存储集群、
还必须安装并配置好 QEMU 。
用法见 `在 libvirt 上使用 Ceph 块设备`_ 。


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
.. Building ``libvirt``

要从源码构建 ``libvirt`` ，需要克隆 ``libvirt`` 软件库，
然后用 `AutoGen`_ 生成构建文件。然后执行 ``make`` 和 ``make install``
完成安装。例如： ::

	git clone git://libvirt.org/libvirt.git
	cd libvirt
	./autogen.sh
	make
	sudo make install

详情见 `libvirt 安装手册`_\ 。



.. _libvirt 安装手册: http://www.libvirt.org/compiling.html
.. _AutoGen: http://www.gnu.org/software/autogen/
.. _QEMU 和块设备: ../../rbd/qemu-rbd
.. _在 libvirt 上使用 Ceph 块设备: ../../rbd/libvirt
