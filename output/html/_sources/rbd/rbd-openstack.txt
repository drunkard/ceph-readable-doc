====================
 块设备与 OpenStack
====================

.. index:: Ceph Block Device; OpenStack

通过 ``libvirt`` 你可以把 Ceph 块设备用于 OpenStack ，它配置了 QEMU 到 \
``librbd`` 的接口。 Ceph 把块设备分块为对象并分布到集群中，这意味着大个的 Ceph 块\
设备映像其性能会比独立服务器更好。

要把 Ceph 块设备用于 OpenStack ，必须先安装 QEMU 、 ``libvirt`` 和 OpenStack 。我\
们建议用一台独立的物理主机安装 OpenStack ，此主机最少需 8GB 内存和一个 4 核 CPU 。\
下面的图表描述了 OpenStack/Ceph 技术栈。


.. ditaa::  +---------------------------------------------------+
            |                    OpenStack                      |
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

.. important:: 要让 OpenStack 使用 Ceph 块设备，你必须有相应的 Ceph 集群访问权限。

OpenStack 里有三个地方和 Ceph 块设备结合：

- **映像：** OpenStack 的 Glance 管理着 VM 的映像。映像相对恒定， OpenStack 把它\
  们当作大块二进制数据、并按需下载。

- **卷宗：** 卷宗是块设备， OpenStack 用它们引导虚拟机、或挂到运行着的虚拟机上。 \
  OpenStack 用 Cinder 服务管理卷宗。

- **Guest Disks**: Guest disks are guest operating system disks. By default,
  when you boot a virtual machine, its disk appears as a file on the filesystem
  of the hypervisor (usually under ``/var/lib/nova/instances/<uuid>/``). Prior
  to OpenStack Havana, the only way to boot a VM in Ceph was to use the
  boot-from-volume functionality of Cinder. However, now it is possible to boot
  every virtual machine inside Ceph directly without using Cinder, which is
  advantageous because it allows you to perform maintenance operations easily
  with the live-migration process. Additionally, if your hypervisor dies it is
  also convenient to trigger ``nova evacuate`` and  run the virtual machine
  elsewhere almost seamlessly.

你可以用 OpenStack Glance 把映像存储到 Ceph 块设备中，还可以用 Cinder 来引导映像的\
写时复制克隆品。

下面将详细指导你安装设置 Glance 、 Cinder 和 Nova ，虽然它们不一定一起用。你可以在\
本地硬盘上运行 VM 、却把映像存储于 Ceph 块设备，反之亦可。

.. important:: Ceph doesn’t support QCOW2 for hosting a virtual machine disk.
   Thus if you want to boot virtual machines in Ceph (ephemeral backend or boot
   from volume), the Glance image format must be ``RAW``.

.. tip:: This document describes using Ceph Block Devices with OpenStack Havana.
   For earlier versions of OpenStack see
   `块设备与 OpenStack (Dumpling)`_.

.. index:: pools; OpenStack

Create a Pool
=============

默认情况下， Ceph 块设备使用 ``rbd`` 存储池，你可以用任何可用存储池。但我们建议分别\
为 Cinder 和 Glance 创建存储池。确保 Ceph 集群在运行，然后创建存储池。 ::

	ceph osd pool create volumes 128
	ceph osd pool create images 128
	ceph osd pool create backups 128
	ceph osd pool create vms 128

参考\ `创建存储池`_\ 为存储池指定归置组数量，参考\ `归置组`_\ 确定应该为存储池分配\
多少归置组。

.. _创建存储池: ../../rados/operations/pools#createpool
.. _归置组: ../../rados/operations/placement-groups


配置 OpenStack 的 Ceph 客户端
=============================

运行着 ``glance-api`` 、 ``cinder-volume`` 、 ``nova-compute`` 或 \
``cinder-backup`` 的主机被当作 Ceph 客户端，它们都需要 ``ceph.conf`` 文件。 ::

	ssh {your-openstack-server} sudo tee /etc/ceph/ceph.conf </etc/ceph/ceph.conf


安装 Ceph 客户端软件包
----------------------

在运行 ``glance-api`` 的节点上你需要 ``librbd`` 的 Python 接口： ::

	sudo apt-get install python-rbd
	sudo yum install python-rbd

在 ``nova-compute`` 、 ``cinder-backup`` 和 ``cinder-volume`` 节点上，要安装 \
Python 绑定和客户端命令行工具： ::

	sudo apt-get install ceph-common
	sudo yum install ceph


配置 Ceph 客户端认证
--------------------

如果你启用了 `cephx 认证`_\ ，需要分别为 Nova/Cinder 和 Glance 创建新用户。命令如下： ::

	ceph auth get-or-create client.cinder mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=volumes, allow rwx pool=vms, allow rx pool=images'
	ceph auth get-or-create client.glance mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=images'
	ceph auth get-or-create client.cinder-backup mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=backups'

把这些用户 ``client.cinder`` 、 ``client.glance`` 和 ``client.cinder-backup`` 的\
密钥环复制到各自所在节点，并修正所有权： ::

	ceph auth get-or-create client.glance | ssh {your-glance-api-server} sudo tee /etc/ceph/ceph.client.glance.keyring
	ssh {your-glance-api-server} sudo chown glance:glance /etc/ceph/ceph.client.glance.keyring
	ceph auth get-or-create client.cinder | ssh {your-volume-server} sudo tee /etc/ceph/ceph.client.cinder.keyring
	ssh {your-cinder-volume-server} sudo chown cinder:cinder /etc/ceph/ceph.client.cinder.keyring
	ceph auth get-or-create client.cinder-backup | ssh {your-cinder-backup-server} sudo tee /etc/ceph/ceph.client.cinder-backup.keyring
	ssh {your-cinder-backup-server} sudo chown cinder:cinder /etc/ceph/ceph.client.cinder-backup.keyring

运行 ``nova-compute`` 的节点，其进程需要密钥环文件： ::

	ceph auth get-or-create client.cinder | ssh {your-nova-compute-server} sudo tee /etc/ceph/ceph.client.cinder.keyring

还得把 ``client.cinder`` 用户的密钥存进 ``libvirt`` ， libvirt 进程从 \
Cinder 挂载块设备时要用它访问集群。

在运行 ``nova-compute`` 的节点上创建一个密钥的临时副本： ::

	ceph auth get-key client.cinder | ssh {your-compute-node} tee client.cinder.key

然后，在计算节点上把密钥加进 ``libvirt`` 、然后删除临时副本： ::

	uuidgen
	457eb676-33da-42ec-9a8c-9293d545c337

	cat > secret.xml <<EOF
	<secret ephemeral='no' private='no'>
		<uuid>457eb676-33da-42ec-9a8c-9293d545c337</uuid>
		<usage type='ceph'>
			<name>client.cinder secret</name>
		</usage>
	</secret>
	EOF
	sudo virsh secret-define --file secret.xml
	Secret 457eb676-33da-42ec-9a8c-9293d545c337 created
	sudo virsh secret-set-value --secret 457eb676-33da-42ec-9a8c-9293d545c337 --base64 $(cat client.cinder.key) && rm client.cinder.key secret.xml

保留密钥的 uuid ，稍后配置 ``nova-compute`` 要用。

.. important:: You don't necessarily need the UUID on all the compute nodes.
   However from a platform consistency perspective, it's better to keep the
   same UUID.

.. _cephx 认证: ../../rados/operations/authentication


让 OpenStack 使用 Ceph
=======================

配置 Glance
-----------

Glance 可使用多种后端存储映像，要让它默认使用 Ceph 块设备，可以这样配置 Glance 。

低于 Juno 的版本
~~~~~~~~~~~~~~~~

编辑 ``/etc/glance/glance-api.conf`` 并把下列内容加到 ``[DEFAULT]`` 段下： ::

	default_store = rbd
	rbd_store_user = glance
	rbd_store_pool = images
	rbd_store_chunk_size = 8


Juno 版
~~~~~~~

编辑 ``/etc/glance/glance-api.conf`` 并把下列内容加到 ``[glance_store]`` 段下： ::

	[DEFAULT]
	...
	default_store = rbd
	...
	[glance_store]
	stores = rbd
	rbd_store_pool = images
	rbd_store_user = glance
	rbd_store_ceph_conf = /etc/ceph/ceph.conf
	rbd_store_chunk_size = 8

关于 Glance 里可用的其它配置选项见 http://docs.openstack.org/trunk/config-reference/content/section_glance-api.conf.html.

.. important:: Glance 还没完全迁移到 'store' ，所以我们还得在 DEFAULT 段下配\
   置 store 。


任意版 OpenStack
~~~~~~~~~~~~~~~~

如果你想允许使用映像的写时复制克隆品，把下列内容加到 ``[DEFAULT]`` 段下： ::

	show_image_direct_url = True

注意，这里通过 Glance 的 API 展示了后端位置，所以此选项启用时的终结点不能公开访问。

Disable the Glance cache management to avoid images getting cached under ``/var/lib/glance/image-cache/``,
assuming your configuration file has ``flavor = keystone+cachemanagement``::

	[paste_deploy]
	flavor = keystone


配置 Cinder
-----------

OpenStack 需要一个驱动和 Ceph 块设备交互，还得指定块设备所在的存储池名字。编辑 \
OpenStack 节点上的 ``/etc/cinder/cinder.conf`` ，添加： ::

	volume_driver = cinder.volume.drivers.rbd.RBDDriver
	rbd_pool = volumes
	rbd_ceph_conf = /etc/ceph/ceph.conf
	rbd_flatten_volume_from_snapshot = false
	rbd_max_clone_depth = 5
	rbd_store_chunk_size = 4
	rados_connect_timeout = -1
	glance_api_version = 2

如果你在用 `cephx 认证`_\ ，还需要配置用户及其密钥（前述文档中存进了 ``libvirt`` ）\
的 uuid ：

	rbd_user = cinder
	rbd_secret_uuid = 457eb676-33da-42ec-9a8c-9293d545c337

Note that if you are configuring multiple cinder back ends,
``glance_api_version = 2`` must be in the ``[DEFAULT]`` section.


Configuring Cinder Backup
-------------------------

OpenStack Cinder Backup requires a specific daemon so don't forget to install it.
On your Cinder Backup node, edit ``/etc/cinder/cinder.conf`` and add::

	backup_driver = cinder.backup.drivers.ceph
	backup_ceph_conf = /etc/ceph/ceph.conf
	backup_ceph_user = cinder-backup
	backup_ceph_chunk_size = 134217728
	backup_ceph_pool = backups
	backup_ceph_stripe_unit = 0
	backup_ceph_stripe_count = 0
	restore_discard_excess_bytes = true


Configuring Nova to attach Ceph RBD block device
------------------------------------------------

In order to attach Cinder devices (either normal block or by issuing a boot
from volume), you must tell Nova (and libvirt) which user and UUID to refer to
when attaching the device. libvirt will refer to this user when connecting and
authenticating with the Ceph cluster. ::

	rbd_user = cinder
	rbd_secret_uuid = 457eb676-33da-42ec-9a8c-9293d545c337

These two flags are also used by the Nova ephemeral backend.


Configuring Nova
----------------

In order to boot all the virtual machines directly into Ceph, you must
configure the ephemeral backend for Nova.

It is recommended to enable the RBD cache in your Ceph configuration file
(enabled by default since Giant). Moreover, enabling the admin socket
brings a lot of benefits while troubleshoothing. Having one socket
per virtual machine using a Ceph block device will help investigating performance and/or wrong behaviors.

This socket can be accessed like this::

	ceph daemon /var/run/ceph/ceph-client.cinder.19195.32310016.asok help

Now on every compute nodes edit your Ceph configuration file::

	[client]
		rbd cache = true
		rbd cache writethrough until flush = true
		admin socket = /var/run/ceph/$cluster-$type.$id.$pid.$cctid.asok

.. tip:: If your virtual machine is already running you can simply restart it to get the socket


Havana and Icehouse
~~~~~~~~~~~~~~~~~~~

Havana and Icehouse require patches to implement copy-on-write cloning and fix
bugs with image size and live migration of ephemeral disks on rbd. These are
available in branches based on upstream Nova `stable/havana`_  and
`stable/icehouse`_. Using them is not mandatory but **highly recommended** in
order to take advantage of the copy-on-write clone functionality.

On every Compute node, edit ``/etc/nova/nova.conf`` and add::

	libvirt_images_type = rbd
	libvirt_images_rbd_pool = vms
	libvirt_images_rbd_ceph_conf = /etc/ceph/ceph.conf
	rbd_user = cinder
	rbd_secret_uuid = 457eb676-33da-42ec-9a8c-9293d545c337

It is also a good practice to disable file injection. While booting an
instance, Nova usually attempts to open the rootfs of the virtual machine.
Then, Nova injects values such as password, ssh keys etc. directly into the
filesystem. However, it is better to rely on the metadata service and
``cloud-init``.

On every Compute node, edit ``/etc/nova/nova.conf`` and add::

	libvirt_inject_password = false
	libvirt_inject_key = false
	libvirt_inject_partition = -2

To ensure a proper live-migration, use the following flags::

	libvirt_live_migration_flag="VIR_MIGRATE_UNDEFINE_SOURCE,VIR_MIGRATE_PEER2PEER,VIR_MIGRATE_LIVE,VIR_MIGRATE_PERSIST_DEST"


Juno
~~~~

In Juno, Ceph block device was moved under the ``[libvirt]`` section.
On every Compute node, edit ``/etc/nova/nova.conf`` under the ``[libvirt]``
section and add::

	[libvirt]
	images_type = rbd
	images_rbd_pool = vms
	images_rbd_ceph_conf = /etc/ceph/ceph.conf
	rbd_user = cinder
	rbd_secret_uuid = 457eb676-33da-42ec-9a8c-9293d545c337


It is also a good practice to disable file injection. While booting an
instance, Nova usually attempts to open the rootfs of the virtual machine.
Then, Nova injects values such as password, ssh keys etc. directly into the
filesystem. However, it is better to rely on the metadata service and
``cloud-init``.

On every Compute node, edit ``/etc/nova/nova.conf`` and add the following
under the ``[libvirt]`` section::

	inject_password = false
	inject_key = false
	inject_partition = -2

To ensure a proper live-migration, use the following flags::

	live_migration_flag="VIR_MIGRATE_UNDEFINE_SOURCE,VIR_MIGRATE_PEER2PEER,VIR_MIGRATE_LIVE,VIR_MIGRATE_PERSIST_DEST"


重启 OpenStack
==============

要激活 Ceph 块设备驱动、并把块设备存储池名载入配置，必须重启 OpenStack 。在基于 \
Debian 的系统上需在对应节点上执行这些命令： ::

	sudo glance-control api restart
	sudo service nova-compute restart
	sudo service cinder-volume restart
	sudo service cinder-backup restart

在基于 Red Hat 的系统上执行： ::

	sudo service openstack-glance-api restart
	sudo service openstack-nova-compute restart
	sudo service openstack-cinder-volume restart
	sudo service openstack-cinder-backup restart

Once OpenStack is up and running, you should be able to create a volume
and boot from it.
一旦 OpenStack 启动并运行正常，应该就可以创建卷宗并用它引导了。


从块设备引导
============

你可以用 Cinder 命令行工具从一映像创建卷宗： ::

	cinder create --image-id {id of image} --display-name {name of volume} {size of volume}

注意映像必须是 RAW 格式，你可以用 `qemu-img`_ 转换格式，如： ::

	qemu-img convert -f {source-format} -O {output-format} {source-filename} {output-filename}
	qemu-img convert -f qcow2 -O raw precise-cloudimg.img precise-cloudimg.raw

When Glance and Cinder are both using Ceph block devices, the image is a
copy-on-write clone, so it can create a new volume quickly. In the OpenStack
dashboard, you can boot from that volume by performing the following steps:

#. Launch a new instance.
#. Choose the image associated to the copy-on-write clone.
#. Select 'boot from volume'
#. Select the volume you created.

.. _qemu-img: ../qemu-rbd/#running-qemu-with-rbd
.. _块设备与 OpenStack (Dumpling): http://ceph.com/docs/dumpling/rbd/rbd-openstack
.. _stable/havana: https://github.com/jdurgin/nova/tree/havana-ephemeral-rbd
.. _stable/icehouse: https://github.com/angdraug/nova/tree/rbd-ephemeral-clone-stable-icehouse
