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

#. 在管理节点上，用 ``rbd`` 工具初始化这个存储池以用于 RBD：

   .. prompt:: bash $

      rbd pool init <pool-name>

   .. note:: 没指定存储池名字时， ``rbd`` 工具会假设\
      默认的存储池名字是 rbd 。


创建一个块设备用户
==================
.. Create a Block Device User

如果不指定用户名， ``rbd`` 命令默认将\
使用 ``admin`` 用户 ID 访问 Ceph 集群。
这个 ID 有拥有集群的所有管理权限。
我们建议你用一个权限较少的 Ceph 用户 ID 访问 Ceph 集群，
而不是用 ``admin`` 访问。
我们把这样的非 ``admin`` Ceph 用户 ID 叫作“块设备用户”或“Ceph 用户”。

要 `创建一个 Ceph 用户`_ ，用 ``ceph auth get-or-create`` 命令，
再指定 Ceph 用户 ID 名、监视器能力、和 OSD 能力：

.. prompt:: bash $

   ceph auth get-or-create client.{ID} mon 'profile rbd' osd 'profile {profile name} [pool={pool-name}][, profile ...]' mgr 'profile rbd [pool={pool-name}]'

例如，要创建一个名为 ``qemu`` 的用户 ID ，
有 ``vms`` 存储池的读写权限、
和 ``images`` 存储池的只读权限，执行命令：

.. prompt:: bash $

   ceph auth get-or-create client.qemu mon 'profile rbd' osd 'profile rbd pool=vms, profile rbd-read-only pool=images' mgr 'profile rbd pool=images'

``ceph auth get-or-create`` 命令的输出是指定用户的密钥环，
可以写入 ``/etc/ceph/ceph.client.{ID}.keyring`` 。

.. note:: 使用 ``rbd`` 命令的时候可以用 ``--id {id}`` 可选参数\
   指定 Ceph 用户 ID 。


创建块设备映像
==============
.. Creating a Block Device Image

把块设备加入某节点之前，你得先在 :term:`Ceph 存储集群`\ 中给它创建一个映像。
创建一个块设备映像，执行下列命令：

.. prompt:: bash $

   rbd create --size {megabytes} {pool-name}/{image-name}

例如，要在 ``swimmingpool`` 存储池中创建一个名为 ``bar`` 、\
大小为 1GB 的映像，执行下列命令：

.. prompt:: bash $

   rbd create --size 1024 swimmingpool/bar

如果创建映像时不指定存储池，它将使用默认的 ``rbd`` 存储池。\
例如，下面的命令将在默认的 ``rbd`` 存储池中创建一个大小为 1GB 、\
名为 ``foo`` 的映像：

.. prompt:: bash $

   rbd create --size 1024 foo

.. note:: 指定此存储池前必须先创建它，详情见\ `存储池`_\ 。


罗列块设备映像
==============
.. Listing Block Device Images

要罗列 ``rbd`` 存储池中的块设备，执行下列命令：

.. prompt:: bash $

   rbd ls

.. note:: ``rbd`` 是默认存储池名字，
   而 ``rbd ls`` 就罗列默认存储池里的块设备。

罗列某个特定存储池中的块设备，执行下列命令，
用存储池名字替换掉 ``{poolname}`` ：

.. prompt:: bash $

   rbd ls {poolname}

例如：

.. prompt:: bash $

   rbd ls swimmingpool

要罗列 ``rbd`` 存储池内延期删除（ deferred delete ）的块设备，
用此命令：

.. prompt:: bash $

   rbd trash ls

要罗列指定存储池内延期删除的块设备，用下列命令，
需把 ``{poolname}`` 替换成这个存储池的名字：

.. prompt:: bash $

   rbd trash ls {poolname}

例如：

.. prompt:: bash $

   rbd trash ls swimmingpool


检索映像信息
============
.. Retrieving Image Information

查看一个指定映像的信息，执行下列命令，用 ``{image-name}`` 替换映像名字：

.. prompt:: bash $

   rbd info {image-name}

例如：

.. prompt:: bash $

   rbd info foo

查看某存储池内一映像的信息，执行下列命令。用 ``{image-name}`` \
替换掉映像名字、用 ``{pool-name}`` 替换掉存储池名字：

.. prompt:: bash $

   rbd info {pool-name}/{image-name}

例如：

.. prompt:: bash $

   rbd info swimmingpool/bar

.. note:: 还有别的命名惯例，但是可能和这里叙述的有冲突。
   比如， ``userid/<uuid>`` 是个 RBD 映像的名字，
   而这样的名字（至少）可能有歧义。


调整块设备映像尺寸
==================
.. Resizing a Block Device Image

:term:`Ceph 块设备`\ 映像是瘦接口设备，只有在你开始写入数据时它们才会占用物理空间。
然而，它们都有最大容量，就是你用 ``--size`` 选项设置的。
如果你想增加（或减小） Ceph 块设备映像的最大尺寸，执行下列命令：

增加块设备映像的尺寸
--------------------
.. Increasing the Size of a Block Device Image

.. prompt:: bash $

   rbd resize --size 2048 foo

减小块设备映像的尺寸
--------------------
.. Decreasing the Size of a Block Device Image

.. prompt:: bash $

   rbd resize --size 2048 foo --allow-shrink


删除块设备映像
==============
.. Removing a Block Device Image

要删除块设备，执行下列命令，把 ``{image-name}`` 替换成要删除映像的名字：

.. prompt:: bash $

   rbd rm {image-name}

例如：

.. prompt:: bash $

   rbd rm foo

从一个存储池删除块设备
----------------------
.. Removing a Block Device from a Pool

要从某存储池中删除一个块设备，执行下列命令，把 ``{image-name}`` 替换成\
要删除的映像名、把 ``{pool-name}`` 替换成要删除映像所在存储池的名字：

.. prompt:: bash $

   rbd rm {pool-name}/{image-name}

例如：

.. prompt:: bash $

   rbd rm swimmingpool/bar

从一个存储池“延迟删除”块设备
----------------------------
.. "Defer Deleting" a Block Device from a Pool

要从某一存储池中延迟删除（ defer delete ）一个块设备，执行下列命令，
但需把 ``{image-name}`` 替换成要放进回收站的映像名、
把 ``{pool-name}`` 替换成它所在存储池的名字：

.. prompt:: bash $

   rbd trash mv {pool-name}/{image-name}

例如：

.. prompt:: bash $

   rbd trash mv swimmingpool/bar

从存储池删除已延期的块设备
--------------------------
.. Removing a Deferred Block Device from a Pool

要从某一存储池删除已延期的块设备，执行下列命令，
但需把 ``{image-id}`` 替换成想要删除映像的 ID 、
把 ``{pool-name}`` 替换成它所在存储池的名字：

.. prompt:: bash $

   rbd trash rm {pool-name}/{image-id}

例如：

.. prompt:: bash $

   rbd trash rm swimmingpool/2bf4474b0dc51

.. note::

  * 你可以把一个映像移入回收站，即便它有快照、
    或正在被克隆品引用着，但不能从回收站删掉。

  * 你可以用 ``--expires-at`` 设置延期时间（默认为 ``now`` ），\
    并且，它的延期时间没到的话是不能删除的，除非你用 ``--force`` 选项。


块设备映像的恢复
================
.. Restoring a Block Device Image

要恢复 rbd 存储池内的一个延期删除块设备，用下列命令，但需把
``{image-id}`` 替换成那个映像的 ID ：

.. prompt:: bash $

   rbd trash restore {image-id}

例如：

.. prompt:: bash $

   rbd trash restore 2bf4474b0dc51

恢复指定存储池里的块设备映像
----------------------------
.. Restoring a Block Device Image in a Specific Pool

要恢复指定存储池内的一个延期删除块设备，用下列命令，但需把
``{image-id}`` 替换成映像的 id 、 ``{pool-name}`` 替换成存储池名字：

.. prompt:: bash $

  rbd trash restore {pool-name}/{image-id}

例如：

.. prompt:: bash $

   rbd trash restore swimmingpool/2bf4474b0dc51

恢复映像时重命名它
------------------
.. Renaming an Image While Restoring It

在恢复时，你还可以加 ``--image`` 选项来重命名它。

例如：

.. prompt:: bash $

   rbd trash restore swimmingpool/2bf4474b0dc51 --image new-name


.. _创建一个存储池: ../../rados/operations/pools/#create-a-pool
.. _存储池: ../../rados/operations/pools
.. _RBD – 管理 RADOS 块设备映像: ../../man/8/rbd/
.. _创建一个 Ceph 用户: ../../rados/operations/user-management#add-a-user
