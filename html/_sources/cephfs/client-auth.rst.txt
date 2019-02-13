===================
 CephFS 客户端能力
===================

通过 Ceph 鉴权能力，你可以把文件系统客户端所需权限限制到尽可能\
低的水平。

.. note::
   路径限定和布局更改限定是 Ceph 从 Jewel 版起才具备的新功能。


路径限定
========

默认情况下，客户端不会被限定在它们被允许挂载的路径下；而且，\
当客户端挂载了一个子目录后，如 /home/user ， MDS 默认情况下也\
不会检查后续操作都“锁定”在那个目录里面。

要把客户端限定为只能挂载某个特定目录、且只能在其内工作，可以\
用基于路径的 MDS 鉴权能力实现。


语法
----

如果只想授予指定目录读写（ rw ）权限，我们在给这个客户端创建\
密钥时就要提及这个目录，语法如下： ::

        ceph auth get-or-create client.*client_name* mon 'allow r' mds 'allow r, allow rw path=/*specified_directory*' osd 'allow rw pool=data'

比如，要想把 ``foo`` 客户端限定为只能在 ``bar`` 目录下写，命\
令如下： ::

        ceph auth get-or-create client.foo mon 'allow r' mds 'allow r, allow rw path=/bar' osd 'allow rw pool=data'

要完全把此客户端限定在 ``bar`` 目录下，去掉不限制的 "allow r"
子句即可： ::

        ceph auth get-or-create client.foo mon 'allow r' mds 'allow rw path=/bar' osd 'allow rw pool=data'

需要注意的是，如果一个客户端的读权限被限定到了某一路径，它们\
就只能挂载文件系统下的这个可读路径，在挂载命令里必须指定（如\
下）。


关于用户管理的细节，请参阅\ `用户管理 - 把用户加入密钥环`_\ 。

要把客户端限定于指定的子目录，在挂载时还需指定这个目录，语法\
如下： ::

        ceph-fuse -n client.*client_name* *mount_path* -r *directory_to_be_mounted*

例如，要把客户端 ``foo`` 限定于 ``mnt/bar`` 目录，可用此命令： ::

        ceph-fuse -n client.foo mnt -r /bar


.. _Free space reporting:

报告的空闲空间
--------------

默认情况下，在客户端挂载子目录时，报告的已用空间（ ``df`` ）\
是根据这个子目录的配额计算出来的，而不是整个集群的已用空间。

如果你想让客户端报告整个文件系统的总体使用情况，而不止是已挂\
载子目录的配额使用情况，可以给客户端加如下配置： ::

    client quota df = false

如果没有启用配额、或者没有给挂载的子目录设置配额，那么不管这\
个选项配置成什么，都会报告整个文件系统的使用情况。


.. _OSD restriction:

OSD 限定
========

为防止客户端读取、或写入给 CephFS 所分配存储池之外的其它存储\
池，可设置 OSD 鉴权能力，把访问限定到 CephFS 的数据存储池内。

::

    client.0
        key: AQAz7EVWygILFRAAdIcuJ12opU/JKyfFmxhuaw==
        caps: [mds] allow rw
        caps: [mon] allow r
        caps: [osd] allow rw pool=data1, allow rw pool=data2

.. note::
   没有相呼应的 MDS 路径限定，上述 OSD 能力\ **不能**\ 限定
   ``data1`` 和 ``data2`` 之外的文件删除操作。

你也可以把 OSD 能力中的 rw 替换为 r 来限定客户端，防止它们写\
入数据。这不会影响客户端更新那些文件的文件系统元数据，但会阻\
止它们写入能被其它客户端看到的持久数据。


.. _Layout and Quota restriction (the 'p' flag):

布局和配额限定（ p 标记）
=========================

要设置布局或配额，客户端不但得有 rw 标记，还得有 p 标记。这种\
方法会限制所有以 ``ceph.`` 为前缀的特殊扩展属性、也会限制以其\
它方法配置这些字段（如对布局进行 openc 操作）。

例如，在下面的配置片段中， client.0 可以更改布局和配额，而
client.1 却不能。

::

    client.0
        key: AQAz7EVWygILFRAAdIcuJ12opU/JKyfFmxhuaw==
        caps: [mds] allow rwp
        caps: [mon] allow r
        caps: [osd] allow rw pool=data

    client.1
        key: AQAz7EVWygILFRAAdIcuJ12opU/JKyfFmxhuaw==
        caps: [mds] allow rw
        caps: [mon] allow r
        caps: [osd] allow rw pool=data


.. _用户管理 - 把用户加入密钥环: ../../rados/operations/user-management/#add-a-user-to-a-keyring
