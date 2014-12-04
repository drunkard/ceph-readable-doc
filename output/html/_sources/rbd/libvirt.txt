=================================
 通过 ``libvirt`` 使用 Ceph RBD
=================================

.. index:: Ceph Block Device; livirt

``libvirt`` 库是管理程序和软件应用间的一个虚拟机抽象层。有了 ``libvirt`` ，开发者\
和系统管理员只需要关注这些管理器的一个通用管理框架、通用 API 、和通用 shell 接口\
（即 ``virsh`` ）就可以了，像：

- QEMU/KVM
- XEN
- LXC
- VirtualBox
- 等等

Ceph 块设备支持 QEMU/KVM ，所以你可以在 Ceph 块设备之上运行能与 ``libvirt`` 交互\
的软件。下面的堆栈图解释了 ``libvirt`` 和 QEMU 如何通过 ``librbd`` 使用 Ceph 块设\
备。


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


``libvirt`` 常见于为云解决方案提供 Ceph 块设备，像 OpenStack 、 ClouldStack ，它\
们用 ``libvirt`` 和 QEMU/KVM 交互、 QEMU/KVM 再与 Ceph 块设备通过 ``librbd`` 交\
互。详情见\ `块设备与 OpenStack`_ 和\ `块设备与 CloudStack`_ 。关于如何安装见\ \
`安装`_\ 。

你也可以通过 ``libvirt`` 、 ``virsh`` 和 ``libvirt`` API 使用 Ceph 块设备，详\
情见 `libvirt 虚拟化 API`_ 。

要创建使用 Ceph 块设备的虚拟机，请看后面几段。具体应用时，我们用 ``libvirt-pool`` \
作为存储池名、 ``client.libvirt`` 作为用户名、 ``new-libvirt-image`` 作为映像名，\
你可以任意命名，确保在后续过程中用自己的名字替换掉对应名字即可。


配置 Ceph
=========

要把 Ceph 用于 ``libvirt`` ，执行下列步骤：

#. `创建一存储池`_\ （或者用默认的）。本例用 ``libvirt-pool`` 作存储池名，配备了 \
   128 个归置组。
   ::

	ceph osd pool create libvirt-pool 128 128

   验证存储池是否存在。
   ::

	ceph osd lspools

#. `创建一 Ceph 用户`_\ （ 0.9.7 版之前的话用 ``client.admin`` ），本例用 \
   ``client.libvirt`` 、且权限限制到 ``libvirt-pool`` 。
   ::

	ceph auth get-or-create client.libvirt mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=libvirt-pool'

   验证名字是否存在。
   ::

	ceph auth list

   **注：** ``libvirt`` 访问 Ceph 时将用 ``libvirt`` 作为 ID ，而不是 \
   ``client.libvirt`` 。关于 ID 和名字不同的详细解释请参考\ `用户管理——用户`_\ \
   和\ `用户管理——命令行界面`_\ 。

#. 用 QEMU 在 RBD 存储池中\ `创建一映像`_\ 。本例中映像名为 \
   ``new-libvirt-image`` 、存储池为 ``libvirt-pool`` 。
   ::

	qemu-img create -f rbd rbd:libvirt-pool/new-libvirt-image 2G

   验证映像是否存在。
   ::

	rbd -p libvirt-pool ls

   **注：**\ 你也可以用 `rbd create`_ 创建映像，但我们建议顺便确认下 QEMU 可正常运\
   行。


准备虚拟机管理器
================

即使没 VM 管理器你也可以用 ``libvirt`` ，但是用 ``virt-manager`` 创建域更简单。

#. 安装个虚拟机管理器，详情见 `KVM/VirtManager`_ 。
   ::

	sudo apt-get install virt-manager

#. 下载一 OS 映像。

#. 启动虚拟机管理器。
   ::

	sudo virt-manager


新建虚拟机
==========

要用 ``virt-manager`` 创建 VM ，按下列步骤：

#. 点击 **Create New Virtual Machine** 按钮。

#. 为新虚拟机域命名，本例中我们用 ``libvirt-virtual-machine`` ，你可以任意命名，在\
   后续命令行和配置实例中替换掉 ``libvirt-virtual-machine`` 即可。
   ::

	libvirt-virtual-machine

#. 导入映像。
   ::

	/path/to/image/recent-linux.img

   **注：**\ 导入一个近期映像，一些较老的映像未必能正确地重扫描虚拟设备。

#. 配置并启动 VM 。

#. 用 ``virsh list`` 验证 VM 域存在。
   ::

	sudo virsh list

#. 登入 VM （ root/root ）

#. 改配置让它使用 Ceph 前停止 VM 。


配置 VM
=======

配置 VM 使用 Ceph 时，切记尽量用 ``virsh`` 。另外， ``virsh`` 命令通常需要 root \
权限（如 ``sudo`` ），否则不会返回正确结果或提示你需要 root 权限， ``virsh`` 命令\
参考见 `Virsh 命令参考`_\ 。

#. 用 ``virsh edit`` 打开配置文件。
   ::

	sudo virsh edit {vm-domain-name}

   ``<devices>`` 下应该有 ``<disk>`` 条目。
   ::

	<devices>
		<emulator>/usr/bin/kvm</emulator>
		<disk type='file' device='disk'>
			<driver name='qemu' type='raw'/>
			<source file='/path/to/image/recent-linux.img'/>
			<target dev='vda' bus='virtio'/>
			<address type='drive' controller='0' bus='0' unit='0'/>
		</disk>

   用你的 OS 映像路径取代 ``/path/to/image/recent-linux.img`` ，可利用较快的 \
   ``virtio`` 总线的最低内核版本是 2.6.25 ，参见 `Virtio`_ 。

   **重要：**\ 要用 ``sudo virsh edit`` 而非文本编辑器，如果你用文本编辑器编辑了 \
   ``/etc/libvirt/qemu`` 下的配置文件， ``libvirt`` 未必能感知你做的更改。如果 \
   ``/etc/libvirt/qemu`` 下的 XML 文件和 ``sudo virsh dumpxml {vm-domain-name}`` \
   输出结果内容不同， VM 可能会运行异常。

#. 把你创建的 Ceph RBD 映像创建为 ``<disk>`` 条目。
   ::

	<disk type='network' device='disk'>
		<source protocol='rbd' name='libvirt-pool/new-libvirt-image'>
			<host name='{monitor-host}' port='6789'/>
		</source>
		<target dev='vda' bus='virtio'/>
	</disk>

   用你的主机名替换 ``{monitor-host}`` 、可能还有存储池、映像名。你可以为 Ceph 监\
   视器添加多条 ``<host>`` ， ``dev`` 属性是将出现在 VM 之 ``/dev`` 目录下的逻辑\
   设备名，可选的 ``bus`` 属性是要模拟的磁盘类型。可用和驱动相关，如 ide 、 \
   scsi 、 virtio 、 xen 、 usb 或 sata 。

   关于 ``<disk>`` 标签及其子标签和属性，详见\ `硬盘`_\ 。

#. 保存文件。

#. 如果你的 Ceph 存储集群启用了 `Ceph 认证`_\ （默认已启用），那么必须生成一个密钥。
   ::

	cat > secret.xml <<EOF
	<secret ephemeral='no' private='no'>
		<usage type='ceph'>
			<name>client.libvirt secret</name>
		</usage>
	</secret>
	EOF

#. 定义密钥。
   ::

	sudo virsh secret-define --file secret.xml
	<uuid of secret is output here>

#. 获取 ``client.libvirt`` 密钥并把字符串保存于文件。
   ::

	ceph auth get-key client.libvirt | sudo tee client.libvirt.key

#. 设置密钥的 UUID 。
   ::

	sudo virsh secret-set-value --secret {uuid of secret} --base64 $(cat client.libvirt.key) && rm client.libvirt.key secret.xml

   还必须手动设置密钥，把下面的 ``<auth>`` 条目添加到前面的 ``<disk>`` 标签内（用\
   上一命令的输出结果替换掉 ``uuid`` 值）。
   ::

	sudo virsh edit {vm-domain-name}

   然后，把 ``<auth></auth>`` 标签加进域配置文件：
   ::

	...
	</source>
	<auth username='libvirt'>
		<secret type='ceph' uuid='9ec59067-fdbc-a6c0-03ff-df165c0587b8'/>
	</auth>
	<target ...

   **注：**\ 示例 ID 是 ``libvirt`` ，不是\ `配置 Ceph`_ 生成的 Ceph 名 \
   ``client.libvirt`` ，确保你用的是 Ceph 名的 ID 部分。如果出于某些原因你需要更换\
   密钥，必须先执行 ``sudo virsh secret-undefine {uuid}`` 、然后再执行 \
   ``sudo virsh secret-set-value`` 。


总结
====

完成上面的配置后你就可以启动 VM 了，为确认 VM 和 Ceph 在通讯，你可以执行如下过程。

#. 检查 Ceph 是否在运行：
   ::

	ceph health

#. 检查 VM 是否在运行。
   ::

	sudo virsh list

#. 检查 VM 是否在和 Ceph 通讯，用你的 VM 域名字替换 ``{vm-domain-name}`` ：
   ::

	sudo virsh qemu-monitor-command --hmp {vm-domain-name} 'info block'

#. 检查一下 ``<target dev='hdb' bus='ide'/>`` 定义的设备是否出现在 ``/dev`` 或 \
   ``/proc/partitions`` 里。
   ::

	ls dev
	cat proc/partitions

如果看起来一切正常，你就可以在虚拟机内使用 Ceph 块设备了。


.. _安装: ../../install
.. _libvirt 虚拟化 API: http://www.libvirt.org
.. _块设备与 OpenStack: ../rbd-openstack
.. _块设备与 CloudStack: ../rbd-cloudstack
.. _创建一存储池: ../../rados/operations/pools#create-a-pool
.. _创建一 Ceph 用户: ../../rados/operations/user-management#add-a-user
.. _创建一映像: ../qemu-rbd#creating-images-with-qemu
.. _Virsh 命令参考: http://www.libvirt.org/virshcmdref.html
.. _KVM/VirtManager: https://help.ubuntu.com/community/KVM/VirtManager
.. _Ceph 认证: ../../rados/configuration/auth-config-ref
.. _硬盘: http://www.libvirt.org/formatdomain.html#elementsDisks
.. _rbd create: ../rados-rbd-cmds#creating-a-block-device-image
.. _用户管理——用户: ../../rados/operations/user-management#user
.. _用户管理——命令行界面: ../../rados/operations/user-management#command-line-usage
.. _Virtio: http://www.linux-kvm.org/page/Virtio
