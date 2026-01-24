.. _cephfs_mount_using_fuse:

=====================
 用 FUSE 挂载 CephFS
=====================
.. Mount CephFS using FUSE

除了 :ref:`CephFS 内核驱动<cephfs-mount-using-kernel-driver>` ，
`ceph-fuse`_ 是挂载 CephFS 的另外一种方法，
`ceph-fuse`_ 挂载是在用户空间运行的。这意味着，
`ceph-fuse`_ 挂载的性能相比内核驱动会低一些，
但它更容易管理、也容易升级。

前提条件
========

先走通二者都需要的先决条件，内核、 FUSE 挂载的，位于 `挂载 CephFS ：先决条件`_ 。

.. note:: 通过 FUSE 挂载 CephFS 需要超级用户权限（ sudo/root ）。
   libfuse 接口没有提供在内核中修剪缓存条目的机制，
   因此需要用重新挂载（ ``mount(2)`` ）系统调用来强制内核丢弃缓存的元数据。
   ``ceph-fuse`` 会周期性地发送重新挂载系统调用，
   以回应 MDS 的缓存压力或元数据缓存撤销。

提纲
====
.. Synopsis

一般来说，通过 FUSE 挂载 CephFS 的命令是这样的：

.. prompt:: bash #

   ceph-fuse {mountpoint} {options}

挂载 CephFS
===========

要以 FUSE 方式挂载 Ceph 文件系统，用 ``ceph-fuse`` 命令： ::

    mkdir /mnt/mycephfs
    ceph-fuse --id foo /mnt/mycephfs

``--id`` 选项传入的是 CephX 用户的名字，我们挂载 CephFS 时要用他的密钥环，
在这个命令里，他是 ``foo`` 。你也可以用 ``-n`` 代替，但 ``--id`` 显然更简单::

    ceph-fuse -n client.foo /mnt/mycephfs

如果密钥环不在标准位置下，你可以手动传入： ::

    ceph-fuse --id foo -k /path/to/keyring /mnt/mycephfs

你也可以传入监视器地址，虽然这不是强制的： ::

    ceph-fuse --id foo -m 192.168.0.1:6789 /mnt/mycephfs

你也可以只挂载 CephFS 里的某个特定目录，
而不是挂载 CephFS 的根目录： ::

    ceph-fuse --id foo -r /path/to/dir /mnt/mycephfs

如果你的 Ceph 集群有多个 FS ，可以用 ``--client_fs`` 选项\
挂载非默认的文件系统： ::

    ceph-fuse --id foo --client_fs mycephfs2 /mnt/mycephfs2

你也可以在 ``ceph.conf`` 里加上 ``client_fs`` 配置。
另外， ``--client_mds_namespace`` 选项可用于向后兼容。

卸载 CephFS
===========
.. Unmounting CephFS

像其它文件系统一样，用 ``umount`` 卸载 CephFS::

    umount /mnt/mycephfs

.. tip:: 执行此命令时需要确保你不在这个文件系统的目录里。

永久挂载
========
.. Persistent Mounts

要把 CephFS 挂载成用户空间文件系统，在 ``/etc/fstab`` 里加上如下内容::

       #DEVICE PATH       TYPE      OPTIONS
       none    /mnt/mycephfs  fuse.ceph ceph.id={user-ID}[,ceph.conf={path/to/conf.conf}],_netdev,defaults  0 0

例如::

       none    /mnt/mycephfs  fuse.ceph ceph.id=myuser,_netdev,defaults  0 0
       none    /mnt/mycephfs  fuse.ceph ceph.id=myuser,ceph.conf=/etc/ceph/foo.conf,_netdev,defaults  0 0

这里用的是 ID （如 ``myuser`` ，不是 ``client.myuser`` ）。
你可以这样给命令行加上任意的合法 ``ceph-fuse`` 选项。

要挂载 CephFS 的一个子目录，把下面的加进 ``/etc/fstab``::

       none    /mnt/mycephfs  fuse.ceph ceph.id=myuser,ceph.client_mountpoint=/path/to/dir,_netdev,defaults  0 0

``ceph-fuse@.service`` 和 ``ceph-fuse.target`` systemd unit 默认可用。
和往常一样，这些 unit 文件声明了默认的依赖关系和建议的 ``ceph-fuse`` 执行上下文。
写好上面的 fstab 条目后，运行下列命令： ::

    systemctl start ceph-fuse@/mnt/mycephfs.service
    systemctl enable ceph-fuse.target
    systemctl enable ceph-fuse@-mnt-mycephfs.service

关于 CephX 用户管理的细节见 :ref:`用户管理 <user-management>` ，
`ceph-fuse`_ 手册里有它支持的更多选项。
故障排除请参考 :ref:`ceph_fuse_debugging` 。

.. _ceph-fuse: ../../man/8/ceph-fuse/#options
.. _挂载 CephFS ：先决条件: ../mount-prerequisites
