================
 块设备基础命令
================
.. Basic Block Device Commands

.. index:: Ceph Block Device; image management

``rbd`` 命令可用于创建、罗列、自查和删除块设备映像，也可克隆\
映像、创建快照、回滚快照、查看快照等等。 ``rbd`` 命令用法详情见
`RBD – 管理 RADOS 块设备映像`_\ 。

.. important:: 要使用 Ceph 块设备命令，你必须有相应的集群访问权限。


创建一个块设备存储池
====================
.. Create a Block Device Pool

#. 在管理节点上，用 ``ceph`` 工具\ `创建一个存储池`_\ 。

#. 在管理节点上，用 ``rbd`` 工具初始化这个存储池以用于 RBD： ::

        rbd pool init <pool-name>

.. note:: 没指定存储池名字时， ``rbd`` 工具会假设默认的是 rbd 。


创建一个块设备用户
==================
.. Create a Block Device User

如果不指定用户名， ``rbd`` 命令默认将使用 ``admin`` 用户 ID 访问 Ceph 集群。
这个 ID 有拥有集群的所有管理权限。可能的话，建议你用一个限制了权限的用户访问。

要 `创建一个 Ceph 用户`_ ，用 ``ceph`` 命令加上 ``auth get-or-create`` 子命令、
用户名、监视器能力、和 OSD 能力： ::

        ceph auth get-or-create client.{ID} mon 'profile rbd' osd 'profile {profile name} [pool={pool-name}][, profile ...]' mgr 'profile rbd [pool={pool-name}]'

例如，要创建一个名为 ``qemu`` 的用户 ID ，有 ``vms`` 存储池的读写权限、
和 ``images`` 存储池的只读权限，执行命令： ::

	ceph auth get-or-create client.qemu mon 'profile rbd' osd 'profile rbd pool=vms, profile rbd-read-only pool=images' mgr 'profile rbd pool=images'

``ceph auth get-or-create`` 命令的输出是指定用户的密钥环，
可以写入 ``/etc/ceph/ceph.client.{ID}.keyring`` 。

.. note:: 使用 ``rbd`` 命令的时候可以用 ``--id {id}`` 可选参数\
   指定用户 ID 。


创建块设备映像
==============
.. Creating a Block Device Image

要想把块设备加入某节点，你得先在
:term:`Ceph 存储集群`\ 中创建一个映像。
创建一个块设备映像用下列命令： ::

	rbd create --size {megabytes} {pool-name}/{image-name}

例如，要在 ``swimmingpool`` 存储池中创建一个名为 ``bar`` 、\
大小为 1GB 的映像，执行下列命令： ::

	rbd create --size 1024 swimmingpool/bar

如果创建映像时不指定存储池，它将使用默认的 ``rbd`` 存储池。\
例如，下面的命令将默认在 ``rbd`` 存储池中创建一个大小为 1GB 、\
名为 ``foo`` 的映像： ::

	rbd create --size 1024 foo

.. note:: 指定此存储池前必须先创建它，详情见\ `存储池`_\ 。


罗列块设备映像
==============
.. Listing Block Device Images

要罗列 ``rbd`` 存储池中的块设备，用下列命令（即 ``rbd`` 是默认存储池名字）： ::

	rbd ls

用下列命令罗列某个特定存储池中的块设备，用存储池名字替换掉 ``{poolname}`` ： ::

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


检索映像信息
============
.. Retrieving Image Information

用下列命令检索某特定映像的信息，用 ``{image-name}`` 替换映像名字： ::

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
.. Resizing a Block Device Image

:term:`Ceph 块设备`\ 映像是瘦接口设备，只有在你开始写入数据时它们才会占用\
物理空间。然而，它们都有最大容量，就是你设置的 ``--size`` 选项。
如果你想增加（或减小） Ceph 块设备映像的最大尺寸，用下列命令： ::

	rbd resize --size 2048 foo (to increase)
	rbd resize --size 2048 foo --allow-shrink (to decrease)


删除块设备映像
==============
.. Removing a Block Device Image

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


块设备映像的恢复
================
.. Restoring a Block Device Image

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
