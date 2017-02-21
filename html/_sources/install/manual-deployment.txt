==========
 手动部署
==========

所有 Ceph 集群都需要至少一个监视器、且 OSD 数量不小于副本数。自举引导初始监视器是\
部署 Ceph 存储集群的第一步，监视器的部署也为整个集群奠定了重要框架，如存储池副本数、\
每个 OSD 拥有的归置组数量、心跳周期、是否需认证等，其中大多数选项都有默认值，但是建\
设生产集群时仍需要您熟知它们。

按照\ `安装（快速）`_\ 里的相同配置，我们能配置起监视器为 ``node1`` ， OSD 节点为 \
``node2`` 、 ``node3`` 的集群。



.. ditaa:: 
           /------------------\         /----------------\
           |    Admin Node    |         |     node1      |
           |                  +-------->+                |
           |                  |         | cCCC           |
           \---------+--------/         \----------------/
                     |
                     |                  /----------------\
                     |                  |     node2      |
                     +----------------->+                |
                     |                  | cCCC           |
                     |                  \----------------/
                     |
                     |                  /----------------\
                     |                  |     node3      |
                     +----------------->|                |
                                        | cCCC           |
                                        \----------------/


监视器的自举引导
================

自举引导监视器（理论上是 Ceph 存储集群）需要以下几个条件：

- **惟一标识符：** ``fsid`` 是集群的惟一标识，它是 Ceph 作为文件系统时的文件系统标\
  识符。现在， Ceph 还支持原生接口、块设备、和对象存储网关接口，所以 ``fsid`` 有点\
  名不符实了。

- **集群名称：** 每个 Ceph 集群都有自己的名字，它是个不含空格的字符串。默认名字是 \
  ``ceph`` 、但你可以更改；尤其是运营着多个集群时，需要用名字来区分要操作哪一个。
  
  比如，当你以\ `联盟架构`_\ 运营多个集群时，集群名字（如 ``us-west`` 、 \
  ``us-east`` ）将作为标识符出现在 CLI 界面上。\ **注意：**\ 要在命令行下指定某个\
  集群，可以指定以集群名为前缀的配置文件（如 ``ceph.conf`` 、  ``us-west.conf`` 、
  ``us-east.conf`` 等）；也可以参考 CLI 用法（ ``ceph --cluster {cluster-name}`` ）。
  
- **监视器名字：** 同一集群内的各监视器例程都有惟一的名字，通常都用主机名作为监视器\
  名字（我们建议每台主机只运行一个监视器、并且不要与 OSD 主机复用。短主机名可以用 \
  ``hostname -s`` 获取。

- **监视器图：** 自举引导初始监视器需要生成监视器图，为此，需要有 ``fsid`` 、集群\
  名（或用默认）、至少一个主机名及其 IP 。

- **监视器密钥环：** 监视器之间通过密钥通讯，所以你必须把监视器密钥加入密钥环，并在\
  自举引导时提供。
  
- **管理密钥环：** 要使用 ``ceph`` 这个命令行工具，你必须有 ``client.admin`` 用\
  户，所以你要创建此用户及其密钥，并把他们加入密钥环。

前述必要条件并未提及 Ceph 配置文件的创建，然而，实践中最好创建个配置文件，并写好 \
``fsid`` 、 ``mon initial members`` 和 ``mon host`` 配置。

你也可以查看或设置运行时配置。 Ceph 配置文件可以只包含非默认配置， Ceph 配置文件的\
配置将覆盖默认值，把这些配置保存在配置文件里可简化维护。

具体过程如下：

#. 登录到初始监视器节点： ::

	ssh {hostname}

   如： ::

	ssh node1

#. 确保保存 Ceph 配置文件的目录存在， Ceph 默认使用 ``/etc/ceph`` 。安装 \
   ``ceph`` 软件时，安装器也会自动创建 ``/etc/ceph/`` 目录。 ::

	ls /etc/ceph   

   **注意：**\ 部署工具在清除集群时可能删除此目录（如 ``ceph-deploy purgedata
   {node-name}`` 、 ``ceph-deploy purge {node-name}`` ）。

#. 创建 Ceph 配置文件， Ceph 默认使用 ``ceph.conf`` ，其中的 ``ceph`` 是集群名字。 ::

	sudo vim /etc/ceph/ceph.conf

#. 给集群分配惟一 ID （即 ``fsid`` ）。 ::

	uuidgen

#. 把此 ID 写入 Ceph 配置文件。 ::

	fsid = {UUID}

   例如： ::

	fsid = a7f64266-0894-4f1e-a635-d0aeaca0e993

#. 把初始监视器写入 Ceph 配置文件。 ::

	mon initial members = {hostname}[,{hostname}]

   例如： ::

	mon initial members = node1

#. 把初始监视器的 IP 地址写入 Ceph 配置文件、并保存。 ::

	mon host = {ip-address}[,{ip-address}]

   例如： ::

	mon host = 192.168.0.1

   **注意：** 你也可以写 IPv6 地址，但是必须设置 ``ms bind ipv6 = true`` 。详情\
   见\ `网络配置参考`_\ 。

#. 为此集群创建密钥环、并生成监视器密钥。 ::

	ceph-authtool --create-keyring /tmp/ceph.mon.keyring --gen-key -n mon. --cap mon 'allow *'

#. 生成管理员密钥环，生成 ``client.admin`` 用户并加入密钥环。 ::

	ceph-authtool --create-keyring /etc/ceph/ceph.client.admin.keyring --gen-key -n client.admin --set-uid=0 --cap mon 'allow *' --cap osd 'allow *' --cap mds 'allow'

#. 把 ``client.admin`` 密钥加入 ``ceph.mon.keyring`` 。 ::

	ceph-authtool /tmp/ceph.mon.keyring --import-keyring /etc/ceph/ceph.client.admin.keyring

#. 用规划好的主机名、对应 IP 地址、和 FSID 生成一个监视器图，并保存为 ``/tmp/monmap`` 。 ::

	monmaptool --create --add {hostname} {ip-address} --fsid {uuid} /tmp/monmap

   例如： ::

	monmaptool --create --add node1 192.168.0.1 --fsid a7f64266-0894-4f1e-a635-d0aeaca0e993 /tmp/monmap

#. 在监视器主机上分别创建数据目录。 ::

	sudo mkdir /var/lib/ceph/mon/{cluster-name}-{hostname}

   例如： ::

	sudo mkdir /var/lib/ceph/mon/ceph-node1

   详情见\ `监视器配置参考——数据`_\ 。

#. 用监视器图和密钥环组装守护进程所需的初始数据。 ::

	ceph-mon [--cluster {cluster-name}] --mkfs -i {hostname} --monmap /tmp/monmap --keyring /tmp/ceph.mon.keyring

   例如： ::

	ceph-mon --mkfs -i node1 --monmap /tmp/monmap --keyring /tmp/ceph.mon.keyring

#. 仔细斟酌 Ceph 配置文件，公共的全局配置包括这些： ::

	[global]
	fsid = {cluster-id}
	mon initial members = {hostname}[, {hostname}]
	mon host = {ip-address}[, {ip-address}]
	public network = {network}[, {network}]
	cluster network = {network}[, {network}]
	auth cluster required = cephx
	auth service required = cephx
	auth client required = cephx
	osd journal size = {n}
	filestore xattr use omap = true
	osd pool default size = {n}  # Write an object n times.
	osd pool default min size = {n} # Allow writing n copy in a degraded state.
	osd pool default pg num = {n}
	osd pool default pgp num = {n}
	osd crush chooseleaf type = {n}

   按前述实例， ``[global]`` 段的配置大致如下： ::

	[global]
	fsid = a7f64266-0894-4f1e-a635-d0aeaca0e993
	mon initial members = node1
	mon host = 192.168.0.1
	public network = 192.168.0.0/24
	auth cluster required = cephx
	auth service required = cephx
	auth client required = cephx
	osd journal size = 1024
	filestore xattr use omap = true
	osd pool default size = 2
	osd pool default min size = 1
	osd pool default pg num = 333
	osd pool default pgp num = 333
	osd crush chooseleaf type = 1

#. 建一个空文件 ``done`` ，表示监视器已创建、可以启动了： ::

	sudo touch /var/lib/ceph/mon/ceph-node1/done

#. 启动监视器。

   在 Ubuntu 上用 Upstart ： ::

	sudo start ceph-mon id=node1 [cluster={cluster-name}]

   要使此守护进程开机自启，需要创建两个空文件，像这样： ::

	sudo touch /var/lib/ceph/mon/{cluster-name}-{hostname}/upstart

   例如： ::

	sudo touch /var/lib/ceph/mon/ceph-node1/upstart

   在 Debian/CentOS/RHEL 上用 sysvinit ： ::

	sudo /etc/init.d/ceph start mon.node1

#. 验证下 Ceph 已经创建了默认存储池。 ::

	ceph osd lspools

   你应该会看到这样的输出： ::

	0 data,1 metadata,2 rbd,

#. 确认下集群在运行。 ::

	ceph -s

   你应该从输出里看到刚刚启动的监视器在正常运行，并且应该会看到一个健康错误：它表明\
   归置组卡在了 ``stuck inactive`` 状态。输出大致如此： ::

	cluster a7f64266-0894-4f1e-a635-d0aeaca0e993
	  health HEALTH_ERR 192 pgs stuck inactive; 192 pgs stuck unclean; no osds
	  monmap e1: 1 mons at {node1=192.168.0.1:6789/0}, election epoch 1, quorum 0 node1
	  osdmap e1: 0 osds: 0 up, 0 in
	  pgmap v2: 192 pgs, 3 pools, 0 bytes data, 0 objects
	     0 kB used, 0 kB / 0 kB avail
	     192 creating

   **注意：** 一旦你添加了 OSD 并启动，归置组健康错误应该消失，详情见下一节。


添加 OSD
========

你的初始监视器可以正常运行后就可以添加 OSD 了。要想让集群达到 ``active + clean`` \
状态，必须安装足够多的 OSD 来处理对象副本（如 ``osd pool default size = 2`` 需要\
至少 2 个 OSD ）。在完成监视器自举引导后，集群就有了默认的 CRUSH 图，但现在此图还是\
空的，里面没有任何 OSD 映射到 Ceph 节点。


精简型
------

Ceph 软件包提供了 ``ceph-disk`` 工具，用于准备硬盘：可以是分区或用于 Ceph 的目\
录。 ``ceph-disk`` 可通过递增索引来创建 OSD ID ；还能把 OSD 加入 CRUSH 图。 \
``ceph-disk`` 的详细用法可参考 ``ceph-disk -h`` ，此工具把后面将提到的\ \
`精简型`_\ 里面的步骤都自动化了。为按照精简型创建前两个 OSD ，在 ``node2`` 和 \
``node3`` 上执行下列命令：

#. 准备OSD。 ::

	ssh {node-name}
	sudo ceph-disk prepare --cluster {cluster-name} --cluster-uuid {uuid} --fs-type {ext4|xfs|btrfs} {data-path} [{journal-path}]

   例如： ::

	ssh node1
	sudo ceph-disk prepare --cluster ceph --cluster-uuid a7f64266-0894-4f1e-a635-d0aeaca0e993 --fs-type ext4 /dev/hdd1

#. 激活 OSD： ::

	sudo ceph-disk activate {data-path} [--activate-key {path}]

   例如： ::

	sudo ceph-disk activate /dev/hdd1

   **注：** 如果你的 Ceph 节点上没有 ``/var/lib/ceph/bootstrap-osd/{cluster}.keyring`` ，\
   那么应该外加 ``--activate-key`` 参数。


细致型
------

要是不想借助任何辅助工具，可按下列步骤创建 OSD 、将之加入集群和 CRUSH 图。按下列详\
细步骤可在 ``node2`` 和 ``node3`` 上增加前 2 个 OSD ：

#. 登录到OSD主机。 ::

	ssh {node-name}

#. 给 OSD 分配 UUID 。 ::

	uuidgen

#. 创建 OSD 。如果没有指定 UUID ，将会在 OSD 首次启动时分配一个。下列命令执行完成后\
   将输出 OSD 号，在后续步骤里还会用到这个号。 ::

	ceph osd create [{uuid} [{id}]]

#. 在新 OSD 主机上创建默认目录。 ::

	ssh {new-osd-host}
	sudo mkdir /var/lib/ceph/osd/{cluster-name}-{osd-number}

#. 如果要把 OSD 装到非系统盘的独立硬盘上，先创建文件系统、然后挂载到刚创建的目录下： ::

	ssh {new-osd-host}
	sudo mkfs -t {fstype} /dev/{hdd}
	sudo mount -o user_xattr /dev/{hdd} /var/lib/ceph/osd/{cluster-name}-{osd-number}

#. 初始化 OSD 数据目录： ::

	ssh {new-osd-host}
	sudo ceph-osd -i {osd-num} --mkfs --mkkey --osd-uuid [{uuid}]

   加 ``--mkkey`` 选项运行 ``ceph-osd`` 之前，此目录必须是空的；另外，如果集群名字\
   不是默认值，还要给 ``ceph-osd`` 指定 ``--cluster`` 选项。

#. 注册此 OSD 的密钥。路径内 ``ceph-{osd-num}`` 里的 ``ceph`` 其含义为 \
   ``$cluster-$id`` ，如果你的集群名字不是 ``ceph`` ，请指定自己的集群名： ::

	sudo ceph auth add osd.{osd-num} osd 'allow *' mon 'allow profile osd' -i /var/lib/ceph/osd/{cluster-name}-{osd-num}/keyring

#. 把此节点加入 CRUSH 图。 ::

	ceph [--cluster {cluster-name}] osd crush add-bucket {hostname} host

   例如： ::

	ceph osd crush add-bucket node1 host

#. 把此 Ceph 节点放入 ``default`` 根下。 ::

	ceph osd crush move node1 root=default

#. 把此 OSD 加入 CRUSH 图之后，它就能接收数据了。你也可以反编译 CRUSH 图、把此 \
   OSD 加入设备列表、对应主机作为桶加入（如果它还不在 CRUSH 图里）、然后此设备作为\
   主机的一个条目、分配权重、重新编译、注入集群。 ::

	ceph [--cluster {cluster-name}] osd crush add {id-or-name} {weight} [{bucket-type}={bucket-name} ...]

   例如： ::

	ceph osd crush add osd.0 1.0 host=node1

#. 把 OSD 加入 Ceph 后， OSD 已经在配置里了。但它还没开始运行，这时处于 ``down`` \
   且 ``in`` 状态，要启动进程才能收数据。

   在 Ubuntu 系统上用 Upstart 启动： ::

	sudo start ceph-osd id={osd-num} [cluster={cluster-name}]

   例如： ::

	sudo start ceph-osd id=0
	sudo start ceph-osd id=1

   在 Debian/CentOS/RHEL 上用 sysvinit 启动： ::

	sudo /etc/init.d/ceph start osd.{osd-num} [--cluster {cluster-name}]

   例如： ::

	sudo /etc/init.d/ceph start osd.0
	sudo /etc/init.d/ceph start osd.1

   要让守护进程开机自启，必须创建一个空文件： ::

	sudo touch /var/lib/ceph/osd/{cluster-name}-{osd-num}/sysvinit

   例如： ::

	sudo touch /var/lib/ceph/osd/ceph-0/sysvinit
	sudo touch /var/lib/ceph/osd/ceph-1/sysvinit

   OSD 启动后，它应该处于 ``up`` 且 ``in`` 状态。


总结
====

监视器和两个 OSD 开始正常运行后，你就可以通过下列命令观察归置组互联过程了： ::

	ceph -w

执行下列命令查看 OSD树： ::

	ceph osd tree

你应该会看到类似如下的输出： ::

	# id	weight	type name	up/down	reweight
	-1	2	root default
	-2	2		host node1
	0	1			osd.0	up	1
	-3	1		host node2
	1	1			osd.1	up	1

要增加（或删除）额外监视器，参见\ `增加/删除监视器`_\ 。要增加（或删除）额外 OSD ，\
参见\ `增加/删除 OSD`_ 。


.. _联盟架构: ../../radosgw/federated-config
.. _安装（快速）: ../../start
.. _增加/删除监视器: ../../rados/operations/add-or-rm-mons
.. _增加/删除 OSD: ../../rados/operations/add-or-rm-osds
.. _网络配置参考: ../../rados/configuration/network-config-ref
.. _监视器配置参考——数据: ../../rados/configuration/mon-config-ref#data
