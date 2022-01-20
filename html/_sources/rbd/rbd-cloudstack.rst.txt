=====================
 块设备与 CloudStack
=====================

CloudStack 4.0 及以上版本可以通过 ``libvirt`` 使用 Ceph 块设\
备， ``libvirt`` 会配置 QEMU 与 ``librbd`` 交互。 Ceph 会把\
块设备映像条带化为对象并分布到整个集群，这意味着大个的 Ceph
块设备性能会优于单体服务器。

要让 CloudStack 4.0 及更高版使用 Ceph 块设备，你得先安装
QEMU 、 ``libvirt`` 、和 CloudStack 。我们建议在另外一台物理\
服务器上安装 CloudStack ，此软件最低需要 4GB 内存和一个双核
CPU ，但是资源越多越好。下图描述了 CloudStack/Ceph 技术栈。


.. ditaa::

            +---------------------------------------------------+
            |                   CloudStack                      |
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

.. important:: 要让 CloudStack 使用 Ceph 块设备，你必须有
   Ceph 存储集群的访问权限。

CloudStack 集成了 Ceph 的块设备作为它的主要存储（ Primary
Storage ），下列指令详述了 CloudStack 的安装。

.. note:: 我们建议您安装 Ubuntu 14.04 或更高版本，这样就不用\
   手动编译 libvirt 了。

安装、配置 QEMU 用于 CloudStack 不需要任何特殊处理。确保你的
Ceph 存储集群在运行，配置好 QEMU 即可；然后安装 ``libvirt``
0.9.13 或更高版本（也许得手动编译）并确保它与 Ceph 磨合正常。

.. note:: Ubuntu 14.04 和 CentOS 7.2 版搭载了 ``libvirt`` ，
   而且默认启用了 RBD 存储池支持。


.. index:: pools; CloudStack

创建存储池
==========
.. Create a Pool

默认情况下， Ceph 块设备使用 ``rbd`` 存储池，建议为 CloudStack
NFS 主存储新建一存储池。确保 Ceph 集群在运行，再创建存储池： ::

   ceph osd pool create cloudstack

参考\ `创建存储池`_\ 为存储池指定归置组数量，参考\ `归置组`_\
确定应该为存储池分配多少归置组。

新建的存储池在使用前必须先初始化，用 ``rbd`` 工具初始化此存储池： ::

        rbd pool init cloudstack


创建 Ceph 用户
==============
.. Create a Ceph User

要访问 Ceph 集群，我们需要一个有足够权限的 Ceph 用户，以便访问\
我们刚刚创建的 ``cloudstack`` 存储池。虽说我们可以用 ``client.admin`` ，
还是建议新建一个只能访问 ``cloudstack`` 存储池的用户。 ::

  ceph auth get-or-create client.cloudstack mon 'profile rbd' osd 'profile rbd pool=cloudstack'

下一步，添加主存储（ Primary Storage ）时，我们需要用这个命令返回的信息。

详情见 `用户管理`_ 。


添加主存储
==========
.. Add Primary Storage

要添加一个 Ceph 块设备作为 Primary Storage ，步骤包括：

#. 登录 CloudStack 界面；
#. 点击左侧导航条的 **Infrastructure** ；
#. 选择 **Primary Storage** 下的 **View All** ；
#. 点击右上角的 **Add Primary Storage** 按钮；
#. 按照你的基础设施配置，填入下列信息：

   - Scope （就是说，集群或 zone 范围的）

   - Zone.

   - Pod.

   - Cluster.

   - 主存储的名字。

   - **Protocol** 那里选择 ``RBD`` ；

   - **Provider** ，这里选择适宜你的供应商类型（即 DefaultPrimary, SolidFire,
     SolidFireShared, 或 CloudByte）。根据选定的供应商，填上对应的相关信息。

#. 添加集群信息（支持 ``cephx`` ）。

   - **RADOS Monitor** 这里填入一个 Ceph 监视器节点的 IP 地址。
   
   - **RADOS Pool** 这里填入一个 RBD 存储池的名字。
   
   - **RADOS User** 这里填入用户，这个用户应该有此存储池的足够\
     的权限。注：不要加用户名的 ``client.`` 部分；
   
   - **RADOS Secret** 这里填入用户的密钥；
   
   - **Storage Tags** 是可选的。用不用标签由你，关于 CloudStack
     的存储标签，请参考\ `存储标签`_ 。
   
#. 点击 **OK** 。


创建存储服务
============
.. Create a Disk Offering

要新建硬盘存储服务，参考\ `创建一个新磁盘服务服务`_\ 。 创建\
一存储服务以与 ``rbd`` 标签相配，这样 ``StoragePoolAllocator``
查找合适存储池时就会选择 ``rbd`` 存储池；如果磁盘服务没有与
``rbd`` 标签相配， ``StoragePoolAllocator`` 就会选用你创建的\
存储池（即 ``clouldstack`` ）。


局限性
======
.. Limitations

- ClouldStack 只能绑定一个监视器（但你可以创建一个轮询域名来\
  滚动多个监视器）


.. _创建存储池: ../../rados/operations/pools#createpool
.. _归置组: ../../rados/operations/placement-groups
.. _安装和配置 QEMU: ../qemu-rbd
.. _安装和配置 libvirt: ../libvirt
.. _KVM Hypervisor Host Installation: http://docs.cloudstack.apache.org/en/latest/installguide/hypervisor/kvm.html
.. _存储标签: http://docs.cloudstack.apache.org/en/latest/adminguide/storage.html#storage-tags
.. _创建一个新磁盘服务服务: http://docs.cloudstack.apache.org/en/latest/adminguide/service_offerings.html#creating-a-new-disk-offering
.. _用户管理: ../../rados/operations/user-management
