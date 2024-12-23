===================
 CephFS 客户端能力
===================
.. CephFS Client Capabilities

通过 Ceph 鉴权能力，你可以把文件系统客户端所需权限限制到尽可能\
低的水平。

.. note:: 路径限定和布局更改限定是 Ceph 从 Jewel 版起才具备的新功能。

.. note:: 只有 :term:`BlueStore` 支持把纠删码 (EC) 存储池\
   用于 CephFS 。纠删码存储池不能用作元数据存储池。
   必须在纠删码数据存储池上启用覆盖写（ overwrite ）。


路径限定
========
.. Path restriction

默认情况下，客户端不会被限制只能挂载某些目录；而且，当客户端\
挂载了一个子目录后，如 ``/home/user`` ， MDS 默认情况下也\
不会检查后续操作都“锁定”在那个目录里面。

要把客户端限定为只能挂载某个特定目录、且只能在其内工作，可以\
用基于路径的 MDS 鉴权能力实现。

这一限制\ *只影响*\ 文件系统的层次结构，换句话说，
也就是由 MDS 管理的元数据树。客户端仍可直接访问
RADOS 中的底层文件数据。要想完全隔离客户端，
可将不信任的客户端隔离在自己的 RADOS 命名空间中。
你可以用\ :ref:`文件布局 <file-layouts>`\ 将\
客户端的文件系统子树放置到特定的命名空间中，然后用
:ref:`OSD 能力<modify-user-capabilities>`\ 限制客户端对这个命名空间的 RADOS 访问。


语法
----
.. Syntax

如果只想授予指定目录 ``rw`` （读写）权限，我们在给这个客户端创建\
密钥时就要提及这个目录，命令语法如下：

.. prompt:: bash #

   ceph fs authorize <fs_name> client.<client_id> <path-in-cephfs> rw

比如，要想把 ``foo`` 客户端限定为只能在 ``cephfs_a`` 文件系统的
``bar`` 目录下写，执行下列命令：

.. prompt:: bash #

   ceph fs authorize cephfs_a client.foo / r /bar rw

此命令会输出： ::

    client.foo
      key: *key*
      caps: [mds] allow r, allow rw path=/bar
      caps  [mon] allow r
      caps: [osd] allow rw tag cephfs_a data=cephfs_a

要完全把此客户端限定在 ``bar`` 目录下，去掉根目录即可：

.. prompt:: bash #

   ceph fs authorize cephfs_a client.foo /bar rw

如果一个客户端的读权限被限定到了某一路径，
它在挂载文件系统的时候就必须指定这个有权限读取的路径，
（如下）。

文件系统名指定为 ``all`` 或 ``*`` 时，
将授予每个文件系统的访问权限。\
一般都得给 ``*`` 加引号，以免被 shell 误用。

关于用户管理的细节，请参阅\ `用户管理 - 把用户加入密钥环`_\ 。

要把客户端限定于指定的子目录，在挂载时还需指定这个目录，
命令语法如下：

.. prompt:: bash #

   ceph-fuse -n client.<client_id> <mount-path> -r *directory_to_be_mounted*

例如，要把客户端 ``foo`` 限定于 ``mnt/bar`` 目录，
命令是：

.. prompt:: bash #

   ceph-fuse -n client.foo mnt -r /bar


报告的空闲空间
--------------
.. Reporting free space 

客户端挂载了一个子目录后，已用空间（ ``df`` ）是\
根据这个子目录的配额计算出来的，而不是在 CephFS 文件系统上的已用空间总和。

如果你想让客户端报告总的文件系统占用情况，而不止是已挂载子目录的\
配额使用情况，可以给客户端加如下配置： ::

    client quota df = false

如果没有启用配额、或者没有给挂载的子目录设置配额，那么不管这\
个选项配置成什么，都会报告文件系统的总体占用情况。


.. _cephfs-layout-and-quota-restriction:

布局和配额使用条件（ p 标记）
=============================
.. Layout and Quota restriction (the 'p' flag)

要设置布局或配额，客户端不但得有 ``rw`` 标记，还得有 ``p`` 标记。这种\
方法会限制所有以 ``ceph.`` 为前缀的特殊扩展属性、也会限制以其它方法\
配置这些字段（如对布局进行 ``openc`` 操作）。

例如，在下面的配置片段中， ``client.0`` 可以更改 ``cephfs_a`` 文件系统\
的布局和配额，而 ``client.1`` 却不能。 ::

    client.0
        key: AQAz7EVWygILFRAAdIcuJ12opU/JKyfFmxhuaw==
        caps: [mds] allow rwp
        caps: [mon] allow r
        caps: [osd] allow rw tag cephfs data=data

    client.1
        key: AQAz7EVWygILFRAAdIcuJ12opU/JKyfFmxhuaw==
        caps: [mds] allow rw
        caps: [mon] allow r
        caps: [osd] allow rw tag cephfs data=data


快照使用条件（ s 标记）
=======================
.. Snapshot restriction (the 's' flag)

要创建或删除快照，客户端除需要 ``rw`` 标志外，还需要 ``s`` 标志。
注意，当能力字符串还包含 ``p`` 标志时， ``s`` 标志必须排在它后面
（除 ``rw`` 外的所有标志都必须按字母顺序指定）。

例如，在下面的代码段中， ``client.0`` 可以在 ``cephfs_a`` 文件系统的
``bar`` 目录里创建或删除快照： ::

    client.0
        key: AQAz7EVWygILFRAAdIcuJ12opU/JKyfFmxhuaw==
        caps: [mds] allow rw, allow rws path=/bar
        caps: [mon] allow r
        caps: [osd] allow rw tag cephfs data=cephfs_a


.. _用户管理 - 把用户加入密钥环: ../../rados/operations/user-management/#add-a-user-to-a-keyring


网络限定
========
.. Network restriction

::

 client.foo
   key: *key*
   caps: [mds] allow r network 10.0.0.0/8, allow rw path=/bar network 10.0.0.0/8
   caps: [mon] allow r network 10.0.0.0/8
   caps: [osd] allow rw tag cephfs data=cephfs_a network 10.0.0.0/8

可选的 ``{network/prefix}`` 是以 CIDR 方法表示的标准“网络名和前缀”
（例如 ``10.3.0.0/16`` ）。如果 ``{network/prefix}`` 存在，
那么此功能的使用仅限于从这个网络连接进来的客户端。


.. _fs-authorize-multifs:

文件系统信息限定
================
.. File system Information Restriction

监视器集群可以展现可用文件系统的有限视图。在这种情况下，
监视器集群只会给客户端通告管理员指定的文件系统。不会报告其他文件系统，
涉及到它们的命令也会失败，就好像那个文件系统不存在一样。

比如在下面的例子中， Ceph 集群有 2 个文件系统：

.. prompt:: bash #

   ceph fs ls

::

    name: cephfs, metadata pool: cephfs_metadata, data pools: [cephfs_data ]
    name: cephfs2, metadata pool: cephfs2_metadata, data pools: [cephfs2_data ]

我们只给 ``someuser`` 客户端授权了一个文件系统：

.. prompt:: bash #

   ceph fs authorize cephfs client.someuser / rw

::

    [client.someuser]
        key = AQAmthpf89M+JhAAiHDYQkMiCq3x+J0n9e8REQ==

.. prompt:: bash #

   cat ceph.client.someuser.keyring

::

    [client.someuser]
        key = AQAmthpf89M+JhAAiHDYQkMiCq3x+J0n9e8REQ==
        caps mds = "allow rw fsname=cephfs"
        caps mon = "allow r fsname=cephfs"
        caps osd = "allow rw tag cephfs data=cephfs"

这个客户端就只能看到被授权的文件系统：

.. prompt:: bash #

   ceph fs ls -n client.someuser -k ceph.client.someuser.keyring

::

   name: cephfs, metadata pool: cephfs_metadata, data pools: [cephfs_data ]

热备的 MDS 守护进程始终都会展示。有关受限 MDS 守护进程和文件系统的信息\
还能通过其他方式获取，比如运行 ``ceph health detail`` 。

MDS 通信限定
============
.. MDS communication restriction

默认情况下，用户应用程序可以与任何 MDS 通信，不管它们是否有权\
修改相关文件系统上的数据（参见上文的\ `路径限定`\ ）。
客户端通信可以限定到与指定文件系统相关联的 MDS 守护进程上，
给那个指定的文件系统添加 MDS 能力即可。下面的示例中，
Ceph 集群有两个文件系统：

.. prompt:: bash #

   ceph fs ls

::

    name: cephfs, metadata pool: cephfs_metadata, data pools: [cephfs_data ]
    name: cephfs2, metadata pool: cephfs2_metadata, data pools: [cephfs2_data ]

``someuser`` 客户端只有一个文件系统的授权：

.. prompt:: bash #

   ceph fs authorize cephfs client.someuser / rw

::

    [client.someuser]
        key = AQBPSARfg8hCJRAAEegIxjlm7VkHuiuntm6wsA==

.. prompt:: bash #

   ceph auth get client.someuser > ceph.client.someuser.keyring

::

    exported keyring for client.someuser

.. prompt:: bash #

   cat ceph.client.someuser.keyring

::

    [client.someuser]
        key = AQBPSARfg8hCJRAAEegIxjlm7VkHuiuntm6wsA==
        caps mds = "allow rw fsname=cephfs"
        caps mon = "allow r"
        caps osd = "allow rw tag cephfs data=cephfs"

以 ``someuser`` 身份把 ``cephfs1`` 挂载到先前创建的 ``/mnt/cephfs1`` 下是可以的：

.. prompt:: bash #

   sudo ceph-fuse /mnt/cephfs1 -n client.someuser -k ceph.client.someuser.keyring --client-fs=cephfs

.. note:: 执行上述命令前，如果没有目录 ``/mnt/cephfs1`` ，
   执行 ``mkdir /mnt/cephfs1`` 先创建它。

::

    ceph-fuse[96634]: starting ceph client
    ceph-fuse[96634]: starting fuse

.. prompt:: bash #

   mount | grep ceph-fuse

::

    ceph-fuse on /mnt/cephfs1 type fuse.ceph-fuse (rw,nosuid,nodev,relatime,user_id=0,group_id=0,allow_other)

以 ``someuser`` 身份挂载 ``cephfs2`` 就不行：

.. prompt:: bash #

   sudo ceph-fuse /mnt/cephfs2 -n client.someuser -k ceph.client.someuser.keyring --client-fs=cephfs2

::

   ceph-fuse[96599]: starting ceph client
   ceph-fuse[96599]: ceph mount failed with (1) Operation not permitted


根目录保护
==========
.. Root squash

``root squash`` 功能是实现了一种保险措施，以预防某些情形，
像不小心强制删除某个路径（例如， ``sudo rm -rf /path`` ）。
在 MDS 能力中启用 ``root_squash`` 模式，禁止 ``uid=0`` 或 ``gid=0`` 的客户端\
执行写操作（比如 ``rm`` 、 ``rmdir`` 、 ``rmsnap`` 、 ``mkdir`` 和 ``mksnap`` ）。
此模式允许 root 客户端进行读取操作，这与其他文件系统的行为不同。

下面的例子在整个文件系统上都启用了 ``root_squash`` ，
唯独 ``/volumes`` 之下的目录树除外：

.. prompt:: bash #

   ceph fs authorize a client.test_a / rw root_squash /volumes rw
   ceph auth get client.test_a

::

    [client.test_a]
    key = AQBZcDpfEbEUKxAADk14VflBXt71rL9D966mYA==
    caps mds = "allow rw fsname=a root_squash, allow rw fsname=a path=/volumes"
    caps mon = "allow r fsname=a"
    caps osd = "allow rw tag cephfs data=a"


用 ``fs authorize`` 更改能力
============================
.. Updating Capabilities using ``fs authorize``

从 Ceph 的 Reef 版开始， ``fs authorize`` 可为现有客户端
（另一个 CephFS 或同一文件系统中的另一个路径）增加新能力。

下面的示例演示了运行 ``ceph fs authorize a client.x / rw`` 命令两次后产生的行为。

#. 创建一个新客户端：

   .. prompt:: bash #

      ceph fs authorize a client.x / rw

   ::

      [client.x]
          key = AQAOtSVk9WWtIhAAJ3gSpsjwfIQ0gQ6vfSx/0w==

#. 查看此客户端的能力：

   .. prompt:: bash #

      ceph auth get client.x

   ::

      [client.x]
            key = AQAOtSVk9WWtIhAAJ3gSpsjwfIQ0gQ6vfSx/0w==
            caps mds = "allow rw fsname=a"
            caps mon = "allow r fsname=a"
            caps osd = "allow rw tag cephfs data=a"

#. 以前，第二次运行 ``fs authorize a client.x / rw`` 会打印错误信息。
   在 Reef 版和以后的版本中，此命令会打印一条信息，说没有更新能力：

   .. prompt:: bash #

      ./bin/ceph fs authorize a client.x / rw

   ::

       no update for caps of client.x

用 ``fs authorize`` 增加新能力
------------------------------
.. Adding New Caps Using ``fs authorize``

在同一 CephFS 中给另一个路径增加能力：

.. prompt:: bash #

   ceph fs authorize a client.x /dir1 rw

::

    updated caps for client.x

.. prompt:: bash #

   ceph auth get client.x

::

   [client.x]
           key = AQAOtSVk9WWtIhAAJ3gSpsjwfIQ0gQ6vfSx/0w==
           caps mds = "allow r fsname=a, allow rw fsname=a path=some/dir"
           caps mon = "allow r fsname=a"
           caps osd = "allow rw tag cephfs data=a"

给这个 Ceph 集群上的另一个 CephFS 增加能力：

.. prompt:: bash #

   ceph fs authorize b client.x / rw

::

    updated caps for client.x

.. prompt:: bash #

   ceph auth get client.x

::

   [client.x]
           key = AQD6tiVk0uJdARAABMaQuLRotxTi3Qdj47FkBA==
           caps mds = "allow rw fsname=a, allow rw fsname=b"
           caps mon = "allow r fsname=a, allow r fsname=b"
           caps osd = "allow rw tag cephfs data=a, allow rw tag cephfs data=b"

更改能力中的 rw 权限
--------------------
.. Changing rw permissions in caps

只有在不得不更改读/写权限的时候，才能运行 ``fs authorize`` 来更改能力。
这是因为此时 ``fs authorize`` 命令可能会含糊不清。例如，
用户运行 ``fs authorize cephfs1 client.x /dir1 rw`` 创建客户端，
然后运行 ``fs authorize cephfs1 client.x /dir2 rw`` （注意
``/dir1`` 已更改为 ``/dir2`` ）。运行第二条命令可以解释为：
以当前能力将 ``/dir1`` 更改为 ``/dir2`` ，
也可以解释为给客户端的路径 ``/dir2`` 授予新权限。
之前已经展示过，命令是按第二种解释执行的，也就因此不可能更改授予的部分权限，
除了 ``rw`` 权限以外。下面显示了如何更改 ``client.x`` 的读/写权限：

.. prompt:: bash #

   ceph fs authorize a client.x / r
    [client.x]
        key = AQBBKjBkIFhBDBAA6q5PmDDWaZtYjd+jafeVUQ==

.. prompt:: bash #

   ceph auth get client.x

::

    [client.x]
            key = AQBBKjBkIFhBDBAA6q5PmDDWaZtYjd+jafeVUQ==
            caps mds = "allow r fsname=a"
            caps mon = "allow r fsname=a"
            caps osd = "allow r tag cephfs data=a"

``fs authorize`` 从不削减能力的任何部分
---------------------------------------
.. ``fs authorize`` never deducts any part of caps

授权给客户端的能力不能通过再次运行 ``fs authorize`` 来删除。例如，
假设一个客户端在某个 CephFS 上的能力里面有 ``root_squash`` ，
那么在同一个 CephFS 上再次运行 ``fs authorize`` 但不加 ``root_squash``
将不会有任何更改，这个客户端的能力仍将保持不变：

.. prompt:: bash #

   ceph fs authorize a client.x / rw root_squash

::

    [client.x]
            key = AQD61CVkcA1QCRAAd0XYqPbHvcc+lpUAuc6Vcw==

.. prompt:: bash #

   ceph auth get client.x

::

    [client.x]
            key = AQD61CVkcA1QCRAAd0XYqPbHvcc+lpUAuc6Vcw==
            caps mds = "allow rw fsname=a root_squash"
            caps mon = "allow r fsname=a"
            caps osd = "allow rw tag cephfs data=a"

.. prompt:: bash #

   ceph fs authorize a client.x / rw

::

    [client.x]
            key = AQD61CVkcA1QCRAAd0XYqPbHvcc+lpUAuc6Vcw==
    no update was performed for caps of client.x. caps of client.x remains unchanged.

如果客户端已经拥有文件系统名字 ``a`` 和路径 ``dir1`` 的能力，
再次带入文件系统名字 ``a`` 和路径 ``dir2`` 执行 ``fs authorize`` 命令的话，
不会修改客户端已经拥有的能力，而是会授予 ``dir2`` 一套新能力：

.. prompt:: bash #

   ceph fs authorize a client.x /dir1 rw
   ceph auth get client.x

::

    [client.x]
            key = AQC1tyVknMt+JxAAp0pVnbZGbSr/nJrmkMNKqA==
            caps mds = "allow rw fsname=a path=/dir1"
            caps mon = "allow r fsname=a"
            caps osd = "allow rw tag cephfs data=a"

.. prompt:: bash #
   
   ceph fs authorize a client.x /dir2 rw

::

    updated caps for client.x

.. prompt:: bash #

   ceph auth get client.x

::

    [client.x]
            key = AQC1tyVknMt+JxAAp0pVnbZGbSr/nJrmkMNKqA==
            caps mds = "allow rw fsname=a path=dir1, allow rw fsname=a path=dir2"
            caps mon = "allow r fsname=a"
            caps osd = "allow rw tag cephfs data=a"
