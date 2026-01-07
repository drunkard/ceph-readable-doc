===============
 增加/删除 OSD
===============
.. Adding/Removing OSDs

如果您的集群已经在运行，你可以在运行时添加或删除 OSD 。

增加 OSD
========
.. Adding OSDs

可以给集群增加 OSD ，以扩展集群的容量和恢复能力。
通常，一个 OSD 就是一个 ``ceph-osd`` 守护进程，
它运行在一台主机内的硬盘之上。如果你的机器有多个硬盘，
可以给每个硬盘启动一个 ``ceph-osd`` 守护进程。

通常，你应该监控集群容量，看是否达到了容量上限。
如果达到了它的 ``near full`` 比率，应该增加 OSD 来扩容。

.. warning:: 不要等集群空间达到 ``full ratio`` 再增加 OSD 。
   集群空间使用率达到 ``near full ratio`` 比率后，
   此后发生的 OSD 故障会导致集群空间超过其 ``full ratio`` 。


部署硬件
--------
.. Deploy your Hardware

如果你通过增加主机来增加 OSD ，\
关于 OSD 服务器硬件的配置请参见\ `硬件推荐`_\ 。\
要把一台 OSD 主机加入到集群，首先要安装最新版的 Linux ，\
而且存储硬盘要做好必要的准备，\
详情参见\ `文件系统推荐`_\ 。

把 OSD 主机安装到集群机架上，\
连接好网络、确保网络通畅。\
详情见\ `网络配置参考`_\ 。

.. _硬件推荐: ../../../start/hardware-recommendations
.. _文件系统推荐: ../../configuration/filesystem-recommendations
.. _网络配置参考: ../../configuration/network-config-ref


安装必要软件
------------
.. Installing the Required Software

在手动部署的集群里，你必须手动安装 Ceph 软件包，\
详情见\ `安装 Ceph （手动）`_\ 。\
你应该配置一个无密码登录 SSH 的用户，且他有 root 权限。

.. _安装 Ceph （手动）: ../../../install


增加 OSD （手动）
-----------------
.. Adding an OSD (Manual)

此过程要设置一个 ``ceph-osd`` 守护进程，让它使用一个硬盘，
且让集群把数据分布到这个 OSD 。
如果一台主机有多个硬盘，可以重复此过程，
把每个硬盘配置为一个 OSD 。

按照下列步骤的演示，添加 OSD 要依次创建元数据目录、
配置数据存储驱动器、把 OSD 加入集群、
然后把它加入 CRUSH 图。

把这个 OSD 加入 CRUSH 图时，需要仔细考虑给新 OSD 分配的权重。
因为单个存储驱动器的容量随时间增长，
较新的 OSD 主机与集群里的旧主机相比，很可能拥有更大的硬盘，
因此可以配置更大的权重。

.. tip:: Ceph 在存储池硬件统一的情况下运行得最好，
   但是容许增加容量不一的驱动器，并相应地调整它们的权重。
   即便如此，为实现最佳性能， CRUSH 的分级结构最好按类型、容量定义。
   最好是统一地给现有主机增加更大的驱动器，
   此工作可以逐步推进，每次新增驱动器时把较小的驱动器替换掉。

#. 创建新 OSD ，命令格式如下。
   如果操作时未指定 UUID ，
   OSD 启动时会自动生成一个。\
   OSD 号，在此命令的输出里有，后续步骤需要用到。

   .. prompt:: bash $

      ceph osd create [{uuid} [{id}]]

   如果指定了可选参数 {id} ，那么它将作为 OSD ID 。
   但是，如果此数字已经在用，此命令会出错。

   .. warning:: 不建议明确指定 ``{id}`` 参数。
      因为 ID 是按照数组分配的，跳过一些依然会浪费内存；
      尤其是跳过太多、或者集群很大时，这种内存消耗会很明显。
      如果不指定 ``{id}`` ， Ceph 将采用可用的最小 ID 数字，
      并且避免这些问题。


#. 给新 OSD 创建默认目录，执行下列命令：

   .. prompt:: bash $

      ssh {new-osd-host}
      sudo mkdir /var/lib/ceph/osd/ceph-{osd-number}


#. 如果准备用于 OSD 的是单独的而非系统盘，要先准备一下。
   执行下列命令：

   .. prompt:: bash $

      ssh {new-osd-host}
      sudo mkfs -t {fstype} /dev/{drive}
      sudo mount -o user_xattr /dev/{hdd} /var/lib/ceph/osd/ceph-{osd-number}

#. 初始化这个 OSD 的数据目录，执行下列命令。

   .. prompt:: bash $

      ssh {new-osd-host}
      ceph-osd -i {osd-num} --mkfs --mkkey

   运行 ``ceph-osd`` 时目录必须是空的。


#. 注册 OSD 认证密钥，执行下列命令：

   .. prompt:: bash $

      ceph auth add osd.{osd-num} osd 'allow *' mon 'allow rwx' -i /var/lib/ceph/osd/ceph-{osd-num}/keyring

   列出的 ``ceph-{osd-num}`` 路径里的 ``ceph`` 是集群名字。
   如果你的集群名字不是 ``ceph`` ，
   那么 ``ceph-{osd-num}`` 里的
   ``ceph`` 字符串应该换成你的集群名。例如，
   如果你的集群名是 ``cluster1`` ，那么命令里的路径应该是
   ``/var/lib/ceph/osd/cluster1-{osd-num}/keyring`` 。


#. 把 OSD 加入 CRUSH 图，执行下列命令。这样它才开始收数据。
   用 ``ceph osd crush add`` 命令把 OSD 加入
   CRUSH 分级结构的合适位置。如果你指定了不止一个桶，
   此命令会把它加入你所指定的桶中最具体的一个，\
   *并且*\ 把此桶挪到你指定的其它桶之内。
   **重要：**\ 如果你只指定了 root 桶，
   此命令会把 OSD 直接挂到 root 下面，
   但是 CRUSH 规则期望它位于主机内。如果 OSD 不在主机内，
   这些 OSD 们可能一点数据也收不到。

   .. prompt:: bash $

      ceph osd crush add {id-or-name} {weight}  [{bucket-type}={bucket-name} ...]

   注意下，还有另外一种方法把新 OSD 加进 CRUSH 图：
   反编译 CRUSH 图、把 OSD 加入设备列表、以桶的形式加入它所在的主机
   （如果它还没在 CRUSH 图里）、把设备作为 item 加入主机、
   给此设备分配权重、重编译 CRUSH 图、并应用它。
   详情参见\ `增加/移动 OSD`_ 。最近的版本极少需要此方法
   （这句话是在 Reef 发布的那个月写下的）。


.. _rados-replacing-an-osd:

替换一个 OSD
------------
.. Replacing an OSD

.. note:: 如果本节中的步骤对您没用，
   试试 ``cephadm``` 文档中的说明：
   :ref:`cephadm-replacing-an-osd` 。

有时需要替换 OSD ：比如磁盘损坏时、
或者管理员想用新后端重新开通 OSD ，比如从
FileStore 切换到 BlueStore 。不像\ `删除 OSD`_ ，
要更换的 OSD ，在经历销毁后，其 ID 和
CRUSH 图条目都需要保持不变。

#. 确保销毁此 OSD 不会有问题：

   .. prompt:: bash $

      while ! ceph osd safe-to-destroy osd.{id} ; do sleep 10 ; done

#. 先销毁这个 OSD:

   .. prompt:: bash $

      ceph osd destroy {id} --yes-i-really-mean-it

#. *可选*\ ：如果这个硬盘不是新的，
   之前另作他用，先擦除它：

   .. prompt:: bash $

      ceph-volume lvm zap /dev/sdX

#. 用销毁前的 OSD ID 来准备这个磁盘：

   .. prompt:: bash $

      ceph-volume lvm prepare --osd-id {id} --data /dev/sdX

#. 最后，激活此 OSD ：

   .. prompt:: bash $

      ceph-volume lvm activate {id} {fsid}

或者，不需要最后两步（准备硬盘、再激活 OSD ），
只需一个命令即可重建此 OSD ，执行下列命令：

   .. prompt:: bash $

      ceph-volume lvm create --osd-id {id} --data /dev/sdX


启动 OSD
--------
.. Starting the OSD

把 OSD 加入 Ceph 后， OSD 就在集群里了。然而，在启动前，
它的状态一直是 ``down`` 且 ``in`` 。这个 OSD 没在运行，
且不能接收数据。要启动 OSD ，
可以用管理主机上的 ``service ceph`` 、或从 OSD 所在主机启动：

   .. prompt:: bash $

      sudo systemctl start ceph-osd@{osd-num}

一旦你启动了 OSD ，其状态就变成了 ``up`` 且 ``in`` 。


观察数据迁移
------------
.. Observe the Data Migration

把新 OSD 加入 CRUSH 图后， Ceph 开始重新均衡集群，一些归置组（ PG ）
会迁移到新 OSD 里，可以用 `ceph`_ 工具观察此过程，执行下列命令：

   .. prompt:: bash $

      ceph -w

或者：

   .. prompt:: bash $

      watch ceph status

你会看到归置组状态从 ``active+clean`` 变为 ``active, some degraded objects``
（有降级的对象）、且迁移完成后回到 ``active+clean`` 状态。
观察完后，键盘按 Ctrl-C 退出。

.. _增加/移动 OSD: ../crush-map#addosd
.. _ceph: ../monitoring


删除 OSD （手动）
=================
.. Removing OSDs (Manual)

可以在集群运行时手动删除 OSD ：在缩减集群尺寸或替换硬件时可以这样操作。
通常，一个 OSD 就是一个 ``ceph-osd`` 守护进程、
它运行在主机内的一个硬盘之上。
另外，如果一台主机上有多个存储驱动器，你可能得删除多个
``ceph-osd`` ：主机上一个守护进程对应一个驱动器。

.. warning:: 删除 OSD 前，确保集群没接近 ``full ratio`` 比率。
   否则，删除 OSD 可能导致集群达到或超过 ``full ratio`` 值。


把 OSD 踢出集群（ ``out`` ）
----------------------------
.. Taking the OSD ``out`` of the Cluster

删除 OSD 前，它通常是 ``up`` 且 ``in`` 的。要想从集群删除此 OSD ，
要先把它踢出（ ``out`` ）集群，以使 Ceph 启动重新均衡、
把数据复制到其他 OSD 。要把 OSD 踢出（ ``out`` ）集群，
执行下列命令：

   .. prompt:: bash $

      ceph osd out {osd-num}


观察数据迁移
------------
.. Observing the Data Migration

一旦把 OSD 踢出（ ``out`` ）集群， Ceph 就会开始重新均衡集群、\
把归置组迁出将删除的 OSD 。可以用 `ceph`_ 工具观察此过程，执行下列命令：

   .. prompt:: bash $

      ceph -w

你会看到 PG 状态从 ``active+clean`` 变为 ``active, some degraded objects`` 、
迁移完成后最终回到 ``active+clean`` 状态。
观察完后，键盘按 Ctrl-C 退出。

.. note:: 有时候，踢出（ ``out`` ）某个 OSD
   可能会使 CRUSH 进入临界状态，
   这时某些 PG 一直卡在 ``active+remapped`` 状态。\
   这个问题有时发生在只有几台主机的小集群上（比如，小型测试集群）。
   要解决这个问题，把此 OSD 标记为 ``in`` ，执行下列命令：

   .. prompt:: bash $

      ceph osd in {osd-num}

   等这个 OSD 回到最初的状态后，不要再把它标记为 ``out`` 。
   而是把此 OSD 的权重设置为 ``0`` ，
   执行下列命令：

   .. prompt:: bash $

      ceph osd crush reweight osd.{osd-num} 0

   调整此 OSD 的权重后，观察引发的数据迁移，
   确认它成功完成。把某一 OSD 标记为 ``out`` 和\
   权重改为 ``0`` 的区别在于对包含此 OSD 的桶做出的操作，
   OSD 被标记为 ``out`` 时，它所在桶的权重没变；
   而 OSD 权重改成 ``0`` 后，它所在桶的权重更新了
   （也就是，这个 OSD 的权重从桶的总权重中减掉了）。
   操作小型集群时，有时候用上述的 reweight 命令更好。



停止 OSD
--------
.. Stopping the OSD

把 OSD 踢出（ ``out`` ）集群后，它可能仍在运行，
此时，此 OSD 的状态仍然是 ``up`` 且 ``out`` 。
从集群删除此 OSD 前，必须先停止这个 OSD 。执行下列命令：

   .. prompt:: bash $

      ssh {osd-host}
      sudo systemctl stop ceph-osd@{osd-num}

停止 OSD 后，状态变为 ``down`` 。


删除 OSD
--------
.. Removing the OSD

下列步骤把一个 OSD 移出集群运行图、删除此 OSD 的认证密钥、
删除 OSD 运行图条目、删除 ``ceph.conf`` 条目。
如果主机有多个硬盘，每个硬盘对应的 OSD 都得重复此步骤。

#. 首先，让集群忘掉这个 OSD 。这一步会从 CRUSH 图中删掉这个
   OSD 、删除其认证密钥，也会从 OSD 运行图中删掉。
   （ :ref:`purge 子命令 <ceph-admin-osd>` 在 Luminous 中才引进，\
   更老的版本看 :ref:`这里链接的步骤
   <ceph_osd_purge_procedure_pre_luminous>` ）：

   .. prompt:: bash $

      ceph osd purge {id} --yes-i-really-mean-it


#. 登录保存着 ``ceph.conf`` 主副本的主机：

   .. prompt:: bash $

      ssh {admin-host}
      cd /etc/ceph
      vim ceph.conf


#. 删除 ``ceph.conf`` 文件内的相关 OSD 条目（如果还有的话）： ::

	[osd.1]
		host = {hostname}


#. 在保存着 ``ceph.conf`` 文件主副本的主机上操作，把更新过的
   ``ceph.conf`` 文件复制到集群内其余主机的 ``/etc/ceph`` 目录下。


.. _ceph_osd_purge_procedure_pre_luminous:

如果你的 Ceph 集群版本低于 Luminous ，就不能用
``ceph osd purge`` 子命令。手动执行如下步骤：

#. 删除 CRUSH 图的对应 OSD 条目，它就不再接收数据了。
   详情参见\ `删除 OSD`_ ：

   .. prompt:: bash $

      ceph osd crush remove {name}

   除了从 CRUSH 图删除此 OSD ，你还有两个选择，可以按其中之一操作：
   (1) 反编译 CRUSH 图、删除 device 列表条目、删除对应的 host 桶条目；
   (2) 从 CRUSH 图删除这个 host 桶
   （如果它在 CRUSH 图里，而且你想删除这个主机），
   重编译 CRUSH 图并应用它。

#. 删除 OSD 认证密钥：

   .. prompt:: bash $

      ceph auth del osd.{osd-num}

#. 删除 OSD 。

   .. prompt:: bash $

      ceph osd rm {osd-num}

   例如：

   .. prompt:: bash $

      ceph osd rm 1


.. _删除 OSD: ../crush-map#removeosd
