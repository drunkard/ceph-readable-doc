.. _cephfs-mount-using-kernel-driver:

=======================
 用内核驱动挂载 CephFS
=======================
.. Mount CephFS using Kernel Driver

CephFS 内核驱动程序是 Linux 内核的一部分。它可以将 CephFS
按普通文件系统进行挂载，并能达到原生内核性能。它是大多数用例的首选客户端。

.. note:: 现在挂载 CephFS 设备的字符串使用新语法("v2")。挂载辅助程序\
   可向后兼容旧语法、内核也向后兼容旧语法。
   这意味着使用较新的挂载辅助程序和内核仍然能支持旧语法。

前提条件
========
.. Prerequisites

完成通用的前提条件
------------------
.. Complete General Prerequisites

按照\ `挂载 CephFS ：前提条件`_\ 页面所述，先完成内核和 FUSE 挂载所需的先决条件。

是否有挂载辅助程序？
--------------------
.. Is mount helper is present?

``mount.ceph`` 辅助程序是随 Ceph 软件包安装的。辅助程序会传入监视器地址和
CephX 用户密钥环，这样 Ceph 管理员就不必在挂载 CephFS 时明确指定这些细节。
如果客户端计算机上没有辅助程序，仍然可以用内核驱动挂载 CephFS ，
但必须向 ``mount`` 命令明确传递这些细节。
要检查系统中是否有 ``mount.ceph`` 程序，执行以下命令：

.. prompt:: bash #

   stat /sbin/mount.ceph

哪个内核版本？
--------------
.. Which Kernel Version?

由于内核客户端是作为 Linux 内核的一部分发布的（不是随打包的
Ceph 二进制包发布），所以你得考虑客户端节点用哪个内核版本。
已知比较老的内核所带的 Ceph 客户端有诸多缺陷，而且不一定支持近期
Ceph 集群所支持的功能。

需要注意的是， Linux 发行版的稳定分支里的“最新”内核可能会比最新的\
上游 Linux 内核晚好几年，而 Ceph 的开发是与上游内核同步的\
（包括缺陷修订）。

粗略地算，从 Ceph 10.x (Jewel) 起，你最好用 4.x 以上的内核。
如果不得不用更老的内核，那你应该用 fuse 客户端，不要用内核客户端。

如果你的 Linux 发行版提供额外的 CephFS 技术支持，那就另当别论了，
这时候发行者会负责把修订补丁移植到他们的稳定版内核：请与厂商核实。


提纲
====
.. Synopsis

通常，用内核驱动挂载 CephFS 的命令格式如下：

.. prompt:: bash #

   mount -t ceph {device-string}:{path-to-mounted} {mount-point} -o {key-value-args} {other-args}

挂载 CephFS
===========
.. Mounting CephFS

Ceph 集群默认启用了 CephX 认证。用 ``mount`` 命令通过内核驱动程序挂载 CephFS：

.. prompt:: bash #

   mkdir /mnt/mycephfs
   mount -t ceph <name>@<fsid>.<fs_name>=/ /mnt/mycephfs

#. ``name`` 是我们挂载 CephFS 时使用的 CephX 用户名。
#. ``fsid`` 是 Ceph 集群的 FSID ，可以用 ``ceph fsid`` 找到。
   ``fs_name`` 是要挂载的文件系统。内核驱动要求有一个 Ceph 监视器的地址、
   和 CephX 用户的密钥。例如：

   .. prompt:: bash #

      mount -t ceph cephuser@b3acfc0d-575f-41d3-9c91-0e7ed3dbb3fa.cephfs=/ -o mon_addr=192.168.0.1:6789,secret=AQATSKdNGBnwLhAAnNDKnH65FmVKpXZJVasUeQ==

使用 mount 辅助程序时，监视器主机和 FSID 是可选项。 ``mount.ceph`` 辅助程序\
会查找和读取 ceph 配置文件来发现这些细节信息。例如：

.. prompt:: bash #

   mount -t ceph cephuser@.cephfs=/ -o secret=AQATSKdNGBnwLhAAnNDKnH65FmVKpXZJVasUeQ==

.. note:: 注意，点（ ``cephuser@.cephfs`` 字符串里的 ``.`` ）
   是设备字符串里必不可少的一部分。

这种方法的弱点是会把密钥留在 shell 的命令历史中。
为避免这种情况，可将密钥复制到一个文件中，
然后用 ``secretfile`` 选项代替 ``secret`` ，传递这个文件。例如：

.. prompt:: bash #

   mount -t ceph cephuser@.cephfs=/ /mnt/mycephfs -o secretfile=/etc/ceph/cephuser.secret

确保这个密钥文件的权限正确（最好是 ``600`` ）。

可以传入多个监视器主机，用 ``/`` 分隔地址：

.. prompt:: bash #

   mount -t ceph cephuser@.cephfs=/ /mnt/mycephfs -o \
   mon_addr=192.168.0.1:6789/192.168.0.2:6789,secretfile=/etc/ceph/cephuser.secret

如果禁用了 CephX ，忽略所有与凭证相关的选项即可，例如：

.. prompt:: bash #

   mount -t ceph cephuser@.cephfs=/ /mnt/mycephfs

.. note:: 必须传入 Ceph 用户名，放在设备字符串里。

要想挂载 CephFS 根的一个子树，把路径加到设备字符串后面即可： ::

  mount -t ceph cephuser@.cephfs=/subvolume/dir1/dir2 /mnt/mycephfs -o secretfile=/etc/ceph/cephuser.secret

向后兼容性
==========
.. Backward Compatibility

为保持向后兼容，旧语法还支持着。

要用内核驱动挂载 CephFS ，执行以下命令：

.. prompt:: bash #

   mkdir /mnt/mycephfs
   mount -t ceph :/ /mnt/mycephfs -o name=admin

选项 ``-o`` 后面的键值参数是 CephX 凭据，
``name`` 是挂载 CephFS 的 CephX 用户名。

要想挂载非默认 FS （本例中为 ``cephfs2`` ），执行以下形式的命令。
这些命令适用于集群有多个文件系统的情形：

.. prompt:: bash #

   mount -t ceph :/ /mnt/mycephfs -o name=admin,fs=cephfs2

或者

.. prompt:: bash #

   mount -t ceph :/ /mnt/mycephfs -o name=admin,mds_namespace=cephfs2

.. note:: ``mds_namespace`` 选项废弃了，按旧语法挂载时改用 ``fs=`` 。

卸载 CephFS
===========
.. Unmounting CephFS

要卸载 Ceph 文件系统，用 ``unmount`` 命令，实例：

.. prompt:: bash #

   umount /mnt/mycephfs

.. tip:: 执行此命令时确保你不在此文件系统的目录内。

永久挂载
========
.. Persistent Mounts

要在文件系统表里用内核驱动挂载 CephFS ，把下列内容加入 ``/etc/fstab``::

  {name}@.{fs_name}=/ {mount}/{mountpoint} ceph [mon_addr={ipaddress},secret=secretkey|secretfile=/path/to/secretfile],[{mount.options}]  {fs_freq}  {fs_passno}

例如::

  cephuser@.cephfs=/     /mnt/ceph    ceph    mon_addr=192.168.0.1:6789,noatime,_netdev    0       0

如果未指定 ``secret`` 或 ``secretfile`` 选项， mount 辅助程序将尝试\
在配置的密钥环中为用户名 ``name`` 查找它的密钥。

有关 CephX 用户管理的详细信息，参阅\ `用户管理`_\ ；
参阅 mount.ceph_ 手册查看它支持的选项。
有关故障排除，参阅\ :ref:`kernel_mount_debugging`\ 。

.. _fstab: ../fstab/#kernel-driver
.. _挂载 CephFS ：前提条件: ../mount-prerequisites
.. _mount.ceph: ../../man/8/mount.ceph/
.. _用户管理: ../../rados/operations/user-management/
