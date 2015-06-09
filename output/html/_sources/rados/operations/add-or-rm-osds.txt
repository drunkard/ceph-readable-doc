===============
 增加/删除 OSD
===============

如果您的集群已经在运行，你可以在运行时添加或删除 OSD 。


增加 OSD
========

你迟早要扩容集群， Ceph 允许在运行时增加 OSD 。在 Ceph 里，一个 OSD 一般是一个 \
``ceph-osd`` 守护进程，它运行在硬盘之上，如果你有多个硬盘，可以给每个硬盘启动一个 \
``ceph-osd`` 守护进程。

通常，你应该监控集群容量，看是否达到了容量上限，因为达到了它的 ``near full`` 比率\
后，要增加一或多个 OSD 来扩容。

.. warning:: 不要等空间满了再增加 OSD ，空间使用率达到 ``near full`` 比率后， \
   OSD 失败可能导致集群空间占满。


部署硬件
--------

如果你通过增加主机来增加 OSD ，关于 OSD 服务器硬件的配置请参见\ `硬件推荐`_\ 。要\
把一台 OSD 主机加入到集群，首先要安装最新版的 Linux ，而且存储硬盘要做好必要的准\
备，详情参见\ `文件系统推荐`_\ 。

把 OSD 主机添加到集群机架上，连接好网络、确保网络通畅。详情见\ `网络配置参考`_\ 。

.. _硬件推荐: ../../../start/hardware-recommendations
.. _文件系统推荐: ../../configuration/filesystem-recommendations
.. _网络配置参考: ../../configuration/network-config-ref


安装必要软件
------------

在手动部署的集群里，你必须手动安装 Ceph 软件包，详情见\ `安装 Ceph （手动）`_\ 。\
你应该配置一个无密码登录 SSH 的用户，且他有 root 权限。

.. _安装 Ceph （手动）: ../../../install


增加 OSD （手动）
-----------------

此过程要设置一个 ``ceph-osd`` 守护进程，让它使用一个硬盘，且让集群把数据发布到 \
OSD 。如果一台主机有多个硬盘，可以重复此过程，把每个硬盘配置为一个 OSD 。

要添加 OSD ，要依次创建数据目录、把硬盘挂载到目录、把 OSD 加入集群、然后把它加入 \
CRUSH 图。

往 CRUSH 图里添加 OSD 时建议设置权重，硬盘容量每年增长 40% ，所以较新的 OSD 主机拥\
有更大的空间（即它们可以有更大的权重）。

.. tip:: Ceph 喜欢统一的硬件，与存储池无关。如果你要新增容量不一的驱动器，还\
   需调整它们的权重。但是，为实现最佳性能，CRUSH 的分级结构最好按类型、容量\
   定义。

#. 创建 OSD 。如果未指定 UUID ， OSD 启动时会自动生成一个。下列命令会输出 \
   OSD 号，后续步骤你会用到。 ::

	ceph osd create [{uuid} [{id}]]

   如果指定了可选参数 {id} ，那么它将作为 OSD id 。要注意，如果此数字已使\
   用，此命令会出错。

   .. warning:: 一般来说，我们不建议指定 {id} 。因为 ID 是按照数组分配的，\
      跳过一些依然会浪费内存；尤其是跳过太多、或者集群很大时，会更明显。若\
      未指定 {id} ，将用最小可用数字。

#. 在新 OSD 主机上创建默认目录。 ::

	ssh {new-osd-host}
	sudo mkdir /var/lib/ceph/osd/ceph-{osd-number}

#. 如果准备用于 OSD 的是单独的而非系统盘，先把它挂载到刚创建的目录下： ::

	ssh {new-osd-host}
	sudo mkfs -t {fstype} /dev/{drive}
	sudo mount -o user_xattr /dev/{hdd} /var/lib/ceph/osd/ceph-{osd-number}

#. 初始化 OSD 数据目录。 ::

	ssh {new-osd-host}
	ceph-osd -i {osd-num} --mkfs --mkkey

   运行 ``ceph-osd`` 时目录必须是空的。

#. 注册 OSD 认证密钥， ``ceph-{osd-num}`` 路径里的 ``ceph`` 值应该是 \
   ``$cluster-$id`` ，如果你的集群名字不是 ``ceph`` ，那就用改过的名字。 ::

	ceph auth add osd.{osd-num} osd 'allow *' mon 'allow rwx' -i /var/lib/ceph/osd/ceph-{osd-num}/keyring

#. 把 OSD 加入 CRUSH 图，这样它才开始收数据。用 ``ceph osd crush add`` 命令\
   把 OSD 加入 CRUSH 分级结构的合适位置。如果你指定了不止一个桶，此命令会把\
   它加入你所指定的桶中最具体的一个，\ *并且*\ 把此桶挪到你指定的其它桶之内。\
   **重要：**\ 如果你只指定了 root 桶，此命令会把 OSD 直接挂到 root 下面，但\
   是 CRUSH 规则期望它位于主机内。

   若用的是 v0.48 版，执行下列命令： ::

	ceph osd crush add {id} {name} {weight}  [{bucket-type}={bucket-name} ...]

   若用的是 v0.56 及更高版，执行下列命令： ::

	ceph osd crush add {id-or-name} {weight}  [{bucket-type}={bucket-name} ...]

   你也可以反编译 CRUSH 图、把 OSD 加入设备列表、以桶的形式加入主机（如果它没在 \
   CRUSH 图里）、以条目形式把设备加入主机、分配权重、重编译并应用它。详情参见\ \
   `增加/移动 OSD`_ 。


.. topic:: Argonaut 0.48 版最佳实践

   为降低对用户 I/O 性能的影响，加入 CRUSH 图时应该把 OSD 的初始权重设为 ``0`` ，\
   然后每次增大一点、逐步增大 CRUSH 权重。例如每次增加 ``0.2`` ： ::

      ceph osd crush reweight {osd-id} .2

   迁移完成前，可以依次把权重重置为 ``0.4`` 、 ``0.6`` 等等，直到达到期望权重。

   为降低 OSD 失败的影响，你可以设置： ::

      mon osd down out interval = 0

   它防止挂了的 OSD 自动被标记为 ``out`` ，然后逐步降低其权重： ::

      ceph osd reweight {osd-num} .8

   还是等着集群完成数据迁移，然后再次调整权重，直到权重为 0 。注意，这会阻止集群在\
   发生故障时自动重复制数据，所以要确保监控的及时性，以便管理员迅速介入。

   注意，以上经验在 Bobtail 及后续版本已不再必要。


启动 OSD
--------

把 OSD 加入 Ceph 后， OSD 就在配置里了。然而它还没运行，它现在的状态为 ``down`` \
且 ``out`` 。你必须先启动 OSD 它才能收数据。可以用管理主机上的 ``service ceph`` 、\
或从 OSD 所在主机启动。

在 Debian/Ubuntu 上用 Upstart。 ::

	sudo start ceph-osd id={osd-num}

在 CentOS/RHEL 上用 sysvinit 。 ::

	sudo /etc/init.d/ceph start osd.{osd-num}

一旦你启动了 OSD ，其状态就变成了 ``up`` 且 ``in`` 。


观察数据迁移
------------

把新 OSD 加入 CRUSH 图后， Ceph 会重新均衡服务器，一些归置组会迁移到新 OSD 里，你\
可以用 `ceph`_ 命令观察此过程。 ::

	ceph -w

你会看到归置组状态从 ``active+clean`` 变为 ``active, some degraded objects`` \
（有降级的对象)、且迁移完成后回到 ``active+clean`` 状态。（ Ctrl-c 退出）


.. _增加/移动 OSD: ../crush-map#addosd
.. _ceph: ../monitoring


删除 OSD （手动）
=================

要想缩减集群尺寸或替换硬件，可在运行时删除 OSD 。在 Ceph 里，一个 OSD 通常是一台主\
机上的一个 ``ceph-osd`` 守护进程、它运行在一个硬盘之上。如果一台主机上有多个数据\
盘，你得挨个删除其对应 ``ceph-osd`` 。通常，操作前应该检查集群容量，看是否快达到上\
限了，确保删除 OSD 后不会使集群达到 ``near full`` 比率。

.. warning:: 删除 OSD 时不要让集群达到 ``full ratio`` 值，删除 OSD 可能导致集群达\
   到或超过 ``full ratio`` 值。


把 OSD 踢出集群
---------------

删除 OSD 前，它通常是 ``up`` 且 ``in`` 的，要先把它踢出集群，以使 Ceph 启动重新均\
衡、把数据拷贝到其他 OSD 。 ::

	ceph osd out {osd-num}


观察数据迁移
------------

一旦把 OSD 踢出（ ``out`` ）集群， Ceph 就会开始重新均衡集群、把归置组迁出将删除\
的 OSD 。你可以用 `ceph`_ 工具观察此过程。 ::

	ceph -w

你会看到归置组状态从 ``active+clean`` 变为 ``active, some degraded objects`` 、\
迁移完成后最终回到 ``active+clean`` 状态。（ Ctrl-c 中止）

.. note:: 有时候，（通常是只有几台主机的“小”集群，比如小型测试集群）拿出\
   （ ``out`` ）某个 OSD 可能会使 CRUSH 进入临界状态，这时某些 PG 一直卡\
   在 ``active+remapped`` 状态。如果遇到了这种情况，你应该把此 OSD 标记\
   为 ``in`` ，用这个命令： ::

	``ceph osd in {osd-num}``

   等回到最初的状态后，把它的权重设置为 0 ，而不是标记为 ``out`` ，用此\
   命令： ::

	``ceph osd crush reweight osd.{osd-num} 0``

   执行后，你可以观察数据迁移过程，应该可以正常结束。把某一 OSD 标记为 \
   ``out`` 和权重改为 0 的区别在于，前者，包含此 OSD 的桶、其权重没变；\
   而后一种情况下，桶的权重变了（降低了此 OSD 的权重）。某些情况下， \
   reweight 命令更适合“小”集群。


停止 OSD
--------

把 OSD 踢出集群后，它可能仍在运行，就是说其状态为 ``up`` 且 ``out`` 。删除前要先停\
止 OSD 进程。 ::

	ssh {osd-host}
	sudo /etc/init.d/ceph stop osd.{osd-num}

停止 OSD 后，状态变为 ``down`` 。


删除 OSD
--------

此步骤依次把一个 OSD 移出集群 CRUSH 图、删除认证密钥、删除 OSD 图条目、删除 \
``ceph.conf`` 条目。如果主机有多个硬盘，每个硬盘对应的 OSD 都得重复此步骤。


#. 删除 CRUSH 图的对应 OSD 条目，它就不再接收数据了。你也可以反编译 CRUSH 图、删\
   除 device 列表条目、删除对应的 host 桶条目或删除 host 桶（如果它在 CRUSH 图里，\
   而且你想删除主机），重编译 CRUSH 图并应用它。详情参见\ `删除 OSD`_ 。 ::

	ceph osd crush remove {name}

#. 删除 OSD 认证密钥： ::

	ceph auth del osd.{osd-num}

   ``ceph-{osd-num}`` 路径里的 ``ceph`` 值是 ``$cluster-$id`` ，如果集群名字不\
   是 ``ceph`` ，这里要更改。

#. 删除 OSD 。 ::

	ceph osd rm {osd-num}
	#for example
	ceph osd rm 1

#. 登录到保存 ``ceph.conf`` 主拷贝的主机。 ::

	ssh {admin-host}
	cd /etc/ceph
	vim ceph.conf

#. 从 ``ceph.conf`` 配置文件里删除对应条目。 ::

	[osd.1]
		host = {hostname}

#. 从保存 ``ceph.conf`` 主拷贝的主机，把更新过的 ``ceph.conf`` 拷贝到集群其他主机\
   的 ``/etc/ceph`` 目录下。



.. _删除 OSD: ../crush-map#removeosd
