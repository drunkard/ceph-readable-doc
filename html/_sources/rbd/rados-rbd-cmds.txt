============
 块设备命令
============

.. index:: Ceph Block Device; image management

``rbd`` 命令可用于创建、罗列、内省和删除块设备映像，也可克隆映像、\
创建快照、回滚快照、查看快照等等。 ``rbd`` 命令用法详情见 \
`RBD – 管理 RADOS 块设备映像`_\ 。

.. important:: 要使用 Ceph 块设备命令，你必须有对应集群的访问权限。


创建块设备映像
==============

要想把块设备加入某节点，你得先在 :term:`Ceph 存储集群`\ 中创建一个\
映像，用下列命令： ::

	rbd create --size {megabytes} {pool-name}/{image-name}

例如，要在 ``swimmingpool`` 这个存储池中创建一个名为 ``bar`` 、大小\
为 1GB 的映像，执行下列命令： ::

	rbd create --size 1024 swimmingpool/bar

如果创建映像时不指定存储池，它将使用默认的 ``rbd`` 存储池。例如，下\
面的命令将默认在 ``rbd`` 存储池中创建一个大小为 1GB 、名为 ``foo`` \
的映像： ::

	rbd create --size 1024 foo

.. note:: 指定此存储池前必须先创建它，详情见\ `存储池`_\ 。


罗列块设备映像
==============

要罗列 ``rbd`` 存储池中的块设备，用下列命令（即 ``rbd`` 是默认存储\
池名字）： ::

	rbd ls

用下列命令罗列某个特定存储池中的块设备，用存储池名字替换掉 \
``{poolname}`` ： ::

	rbd ls {poolname}

例如： ::

	rbd ls swimmingpool


检索映像信息
============

用下列命令检索某特定映像的信息，用 ``{image-name}`` 替换映像名字： ::

	rbd info {image-name}

例如： ::

	rbd info foo

用下列命令检索某存储池内一映像的信息，用 ``{image-name}`` 替换掉映\
像名字、用 ``{pool-name}`` 替换掉存储池名字： ::

	rbd info {pool-name}/{image-name}

例如： ::

	rbd info swimmingpool/bar


调整块设备映像尺寸
==================

:term:`Ceph 块设备`\ 映像是瘦接口设备，只有在你开始写入数据时它们才\
会占用物理空间。然而，它们都有最大容量，就是你设置的 ``--size`` 选\
项。如果你想增加（或减小） Ceph 块设备映像的最大尺寸，用下列命令： ::

	rbd resize --size 2048 foo


删除块设备映像
==============

用下列命令删除块设备，用 ``{image-name}`` 替换映像名字： ::

	rbd rm {image-name}

例如： ::

	rbd rm foo

用下列命令从某存储池中删除一个块设备，用 ``{image-name}`` 替换要删\
除的映像名、用 ``{pool-name}`` 替换存储池名字： ::

	rbd rm {pool-name}/{image-name}

例如： ::

	rbd rm swimmingpool/bar



.. _存储池: ../../rados/operations/pools
.. _RBD – 管理 RADOS 块设备映像: ../../man/8/rbd/
