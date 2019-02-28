.. Block Device Commands

============
 块设备命令
============

.. index:: Ceph Block Device; image management

``rbd`` 命令可用于创建、罗列、自查和删除块设备映像，也可克隆\
映像、创建快照、回滚快照、查看快照等等。 ``rbd`` 命令用法详情见
`RBD – 管理 RADOS 块设备映像`_\ 。

.. important:: 要使用 Ceph 块设备命令，你必须有相应的集群\
   访问权限。


.. Create a Block Device Pool

创建一个块设备存储池
====================

#. 在管理节点上，用 ``ceph`` 工具\ `创建一个存储池`_\ 。

#. 在管理节点上，用 ``rbd`` 工具初始化这个存储池以用于 RBD： ::

        rbd pool init <pool-name>

.. note:: 没指定存储池名字时， ``rbd`` 工具会假设默认的为
   rbd 。


.. Create a Block Device User

创建一个块设备用户
==================

Unless specified, the ``rbd`` command will access the Ceph cluster using the ID
``admin``. This ID allows full administrative access to the cluster. It is
recommended that you utilize a more restricted user wherever possible.

To `创建一个 Ceph 用户`_, with ``ceph`` specify the ``auth get-or-create``
command, user name, monitor caps, and OSD caps::

        ceph auth get-or-create client.{ID} mon 'profile rbd' osd 'profile {profile name} [pool={pool-name}][, profile ...]'

For example, to create a user ID named ``qemu`` with read-write access to the
pool ``vms`` and read-only access to the pool ``images``, execute the
following::

	ceph auth get-or-create client.qemu mon 'profile rbd' osd 'profile rbd pool=vms, profile rbd-read-only pool=images'

The output from the ``ceph auth get-or-create`` command will be the keyring for
the specified user, which can be written to ``/etc/ceph/ceph.client.{ID}.keyring``.

.. note:: The user ID can be specified when using the ``rbd`` command by
        providing the ``--id {id}`` optional argument.


.. Creating a Block Device Image

创建块设备映像
==============

要想把块设备加入某节点，你得先在 :term:`Ceph 存储集群`\ 中创建\
一个映像，用下列命令： ::

	rbd create --size {megabytes} {pool-name}/{image-name}

例如，要在 ``swimmingpool`` 存储池中创建一个名为 ``bar`` 、\
大小为 1GB 的映像，执行下列命令： ::

	rbd create --size 1024 swimmingpool/bar

如果创建映像时不指定存储池，它将使用默认的 ``rbd`` 存储池。\
例如，下面的命令将默认在 ``rbd`` 存储池中创建一个大小为 1GB 、\
名为 ``foo`` 的映像： ::

	rbd create --size 1024 foo

.. note:: 指定此存储池前必须先创建它，详情见\ `存储池`_\ 。


.. Listing Block Device Images

罗列块设备映像
==============

要罗列 ``rbd`` 存储池中的块设备，用下列命令（即 ``rbd`` 是默认\
存储池名字）： ::

	rbd ls

用下列命令罗列某个特定存储池中的块设备，用存储池名字替换掉 \
``{poolname}`` ： ::

	rbd ls {poolname}

例如： ::

	rbd ls swimmingpool

要罗列 ``rbd`` 存储池内延期删除的块设备，用此命令： ::

        rbd trash ls

要罗列指定存储池内延期删除的块设备，用下列命令，需把
``{poolname}`` 替换成这个存储池的名字： ::

        rbd trash ls {poolname}

例如： ::

        rbd trash ls swimmingpool


.. Retrieving Image Information

检索映像信息
============

用下列命令检索某特定映像的信息，用 ``{image-name}`` 替换\
映像名字： ::

	rbd info {image-name}

例如： ::

	rbd info foo

用下列命令检索某存储池内一映像的信息，用 ``{image-name}`` \
替换掉映像名字、用 ``{pool-name}`` 替换掉存储池名字： ::

	rbd info {pool-name}/{image-name}

例如： ::

	rbd info swimmingpool/bar


调整块设备映像尺寸
==================

:term:`Ceph 块设备`\ 映像是瘦接口设备，只有在你开始写入数据\
时它们才会占用物理空间。然而，它们都有最大容量，就是你设置的
``--size`` 选项。如果你想增加（或减小） Ceph 块设备映像的最\
大尺寸，用下列命令： ::

	rbd resize --size 2048 foo (to increase)
	rbd resize --size 2048 foo --allow-shrink (to decrease)


删除块设备映像
==============

用下列命令删除块设备，用 ``{image-name}`` 替换映像名字： ::

	rbd rm {image-name}

例如： ::

	rbd rm foo

用下列命令从某存储池中删除一个块设备，用 ``{image-name}`` 替换\
要删除的映像名、用 ``{pool-name}`` 替换存储池名字： ::

	rbd rm {pool-name}/{image-name}

例如： ::

	rbd rm swimmingpool/bar

要从某一存储池中延期删除一个块设备，执行下列命令，但需把
``{image-name}`` 替换成要操作的映像名、把 ``{pool-name}`` 替换\
成存储池的名字： ::

        rbd trash mv {pool-name}/{image-name}

例如： ::

        rbd trash mv swimmingpool/bar

要从某一存储池删除已延期的块设备，执行下列命令，但需把
``{image-id}`` 替换成欲删除映像的 id 、把 ``{pool-name}`` 替换\
成存储池的名字： ::

        rbd trash rm {pool-name}/{image-id}

例如： ::

        rbd trash rm swimmingpool/2bf4474b0dc51

.. note::

  * 你可以把一个映像移入垃圾池，即便它有快照、或正在被克隆品\
    引用着，但不能从垃圾池删掉。

  * 你可以用 *--expires-at* 设置延期时间（默认为 ``now`` ），\
    并且，它的延期时间没到的话是不能删除的，除非你用 *--force*
    选项。


.. Restoring a Block Device Image

块设备映像的恢复
================
要恢复 rbd 存储池内的一个延期删除块设备，用下列命令，但需把
``{image-id}`` 替换成那个映像的 id ： ::

        rbd trash restore {image-id}

例如： ::

        rbd trash restore 2bf4474b0dc51

要恢复指定存储池内的一个延期删除块设备，用下列命令，但需把
``{image-id}`` 替换成映像的 id 、 ``{pool-name}`` 替换成存储池\
名字： ::

        rbd trash restore {pool-name}/{image-id}

例如： ::

        rbd trash restore swimmingpool/2bf4474b0dc51

在恢复时，你还可以加 ``--image`` 选项来重命名它。

例如： ::

        rbd trash restore swimmingpool/2bf4474b0dc51 --image new-name


.. _创建一个存储池: ../../rados/operations/pools/#create-a-pool
.. _存储池: ../../rados/operations/pools
.. _RBD – 管理 RADOS 块设备映像: ../../man/8/rbd/
.. _创建一个 Ceph 用户: ../../rados/operations/user-management#add-a-user
