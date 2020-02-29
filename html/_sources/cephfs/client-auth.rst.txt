.. CephFS Client Capabilities

===================
 CephFS 客户端能力
===================

通过 Ceph 鉴权能力，你可以把文件系统客户端所需权限限制到尽可能\
低的水平。

.. note::
   路径限定和布局更改限定是 Ceph 从 Jewel 版起才具备的新功能。


.. Path restriction

路径限定
========

默认情况下，客户端不会被限定在它们被允许挂载的路径下；而且，\
当客户端挂载了一个子目录后，如 /home/user ， MDS 默认情况下也\
不会检查后续操作都“锁定”在那个目录里面。

要把客户端限定为只能挂载某个特定目录、且只能在其内工作，可以\
用基于路径的 MDS 鉴权能力实现。


.. Syntax

语法
----

如果只想授予指定目录读写（ rw ）权限，我们在给这个客户端创建\
密钥时就要提及这个目录，语法如下： ::

        ceph fs authorize *file_system_name* client.*client_name* /*specified_directory* rw

比如，要想把 ``foo`` 客户端限定为只能在 ``cephfs`` 文件系统的
``bar`` 目录下写，命令如下： ::

        ceph fs authorize cephfs client.foo / r /bar rw

        results in:

        client.foo
          key: *key*
          caps: [mds] allow r, allow rw path=/bar
          caps  [mon] allow r
          caps: [osd] allow rw tag cephfs data=cephfs_a

要完全把此客户端限定在 ``bar`` 目录下，去掉根目录即可： ::

        ceph fs authorize cephfs client.foo /bar rw

需要注意的是，如果一个客户端的读权限被限定到了某一路径，它们\
就只能挂载文件系统下的这个可读路径，在挂载命令里必须指定（如\
下）。

文件系统名指定为 ``all`` 或 ``*`` 时，权限将授予每个文件系统。\
注意，一般都得给 ``*`` 加引号，以免被 shell 误用。

关于用户管理的细节，请参阅\ `用户管理 - 把用户加入密钥环`_\ 。

要把客户端限定于指定的子目录，在挂载时还需指定这个目录，语法\
如下： ::

        ceph-fuse -n client.*client_name* *mount_path* -r *directory_to_be_mounted*

例如，要把客户端 ``foo`` 限定于 ``mnt/bar`` 目录，命令是： ::

        ceph-fuse -n client.foo mnt -r /bar


.. Free space reporting

报告的空闲空间
--------------

默认情况下，在客户端挂载子目录时，报告的已用空间（ ``df`` ）\
是根据这个子目录的配额计算出来的，而不是整个集群的已用空间。

如果你想让客户端报告整个文件系统的总体使用情况，而不止是已挂\
载子目录的配额使用情况，可以给客户端加如下配置： ::

    client quota df = false

如果没有启用配额、或者没有给挂载的子目录设置配额，那么不管这\
个选项配置成什么，都会报告整个文件系统的使用情况。


.. Layout and Quota restriction (the 'p' flag)

布局和配额限定（ p 标记）
=========================

要设置布局或配额，客户端不但得有 rw 标记，还得有 p 标记。这种\
方法会限制所有以 ``ceph.`` 为前缀的特殊扩展属性、也会限制以其\
它方法配置这些字段（如对布局进行 openc 操作）。

例如，在下面的配置片段中， client.0 可以更改 cephfs_a 文件系统\
的布局和配额，而 client.1 却不能。

::

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


.. Snapshot restriction (the 's' flag)

快照限定（ s 标记）
===================

To create or delete snapshots, clients require the 's' flag in addition to 'rw'.
Note that when capability string also contains the 'p' flag, the 's' flag must
appear after it (all flags except 'rw' must be specified in alphabetical order).

For example, in the following snippet client.0 can create or delete snapshots
in the ``bar`` directory of file system ``cephfs_a``.

::

    client.0
        key: AQAz7EVWygILFRAAdIcuJ12opU/JKyfFmxhuaw==
        caps: [mds] allow rw, allow rws path=/bar
        caps: [mon] allow r
        caps: [osd] allow rw tag cephfs data=cephfs_a


.. _用户管理 - 把用户加入密钥环: ../../rados/operations/user-management/#add-a-user-to-a-keyring

.. Network restriction

网络限定
========

::

 client.foo
   key: *key*
   caps: [mds] allow r network 10.0.0.0/8, allow rw path=/bar network 10.0.0.0/8
   caps: [mon] allow r network 10.0.0.0/8
   caps: [osd] allow rw tag cephfs data=cephfs_a network 10.0.0.0/8

The optional ``{network/prefix}`` is a standard network name and
prefix length in CIDR notation (e.g., ``10.3.0.0/16``).  If present,
the use of this capability is restricted to clients connecting from
this network.

