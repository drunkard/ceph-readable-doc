.. _fs-volumes-and-subvolumes:

FS 卷和子卷
===========
.. FS volumes and subvolumes

:term:`Ceph 管理器`\ 守护进程（ ceph-mgr ）的 volumes 模块为
CephFS 导出提供了单一可信来源。 OpenStack 共享文件系统服务
（ manila_ ）和 Ceph 容器存储接口（ CSI_ ）存储管理员\
可以用 ceph-mgr ``volumes`` 模块提供的通用 CLI 来管理 CephFS 导出。

ceph-mgr ``volumes`` 模块实现了以下的文件系统导出抽象层：

* FS volumes （ FS 卷）， CephFS 文件系统的抽象

* FS subvolume groups （ FS 子卷组），在 FS 子卷之上的、目录级别的抽象。
  用于对子卷集合应用策略（例如 :doc:`/cephfs/file-layouts` ）。

* FS subvolumes （ FS 子卷）， CephFS 独立目录树的抽象。

导出抽象的可能应用场景：

* FS 子卷，用于 Manila 共享或 CSI 卷；

* FS 子卷组，用作 Manila 共享组


必备条件
--------
.. Requirements

* Nautilus (14.2.x) 或更高版本

* Cephx 客户端用户（见 :doc:`/rados/operations/user-management` ）
  至少得拥有下列能力： ::

    mon 'allow r'
    mgr 'allow rw'


FS 卷
-----
.. FS Volumes

执行下列命令，创建一个卷：

.. prompt:: bash #

   ceph fs volume create <vol_name> [placement] [--data-pool <data-pool-name>] [--meta-pool <metadata-pool-name>]

这将创建一个 CephFS 文件系统及其数据和元数据存储池。
另外，如果创建 CephFS 卷所需的数据存储池和/元数据存储池已经存在，
可以把这些存储池名字代入这个命令，这样就用这些已有存储池创建了卷。
此命令还能用 Ceph 管理器的 orchestrator 模块（如 Rook）
为文件系统部署 MDS 守护进程。参阅 :doc:`/mgr/orchestrator` 。

``<vol_name>`` 是卷名（任意字符串）。
``[placement]`` 是一个可选字符串，
用于指定 MDS 的 :ref:`orchestrator-cli-placement-spec` 。
另请参阅 :ref:`orchestrator-cli-cephfs` ，了解有关归置的更多示例。

.. note:: 系统不支持通过卷接口、用一个 YAML 文件指定归置策略。

要删除一个卷，执行下列命令：

.. prompt:: bash #

   ceph fs volume rm <vol_name> [--yes-i-really-mean-it]

此命令用于删除文件系统及其数据和元数据存储池。
如果已启用 Ceph 管理器 orchestrator 模块，还会尝试删除 MDS 守护进程。

.. note:: 卷删除后，如果在同一集群上创建了新文件系统\
   且正在使用子卷接口，我们建议重启 `ceph-mgr` 。
   详情见 https://tracker.ceph.com/issues/49605#note-5 。

.. note:: 如果 Ceph 管理器模块 snap-schedule 正用于某个卷，
   而该卷被删除了，那么 snap-schedule Ceph 管理器模块将\
   继续保持对旧存储池的引用。
   这会导致 snap-schedule Ceph 管理器模块发生故障并记录错误。
   要纠正这种情况，我们建议，
   删除卷后重启 snap-schedule Ceph 管理器模块。
   如果故障依旧，我们建议重启 `ceph-mgr` 。

罗列卷，执行下列命令：

.. prompt:: bash #

   ceph fs volume ls

重命名卷，执行下列命令：

.. prompt:: bash #

   ceph fs volume rename <vol_name> <new_vol_name> [--yes-i-really-mean-it]

重命名卷是昂贵（繁杂）的操作，需要以下必备条件：

- 重命名 orchestrator 托管的 MDS 服务，使其与 ``<new_vol_name>`` 匹配。
  这需要以 ``<new_vol_name>`` 启动 MDS 服务，
  并关闭 ``<vol_name>`` 的 MDS 服务。
- 重命名文件系统，把 ``<vol_name>`` 改成 ``<new_vol_name>`` 。
- 更改此文件系统数据存储池和元数据存储池上的应用程序标签，
  改成 ``<new_vol_name>`` 。
- 重命名此文件系统的元数据和数据存储池。

已经授权给 ``<vol_name>`` 的 CephX ID ，必须重新授权给 ``<new_vol_name>`` 。
使用这些 ID 的客户端，它们的所有正在进行的操作可能会中断。
确保在这个卷上禁用镜像功能。

要提取 CephFS 卷的信息，执行下列命令：

.. prompt:: bash #

   ceph fs volume info vol_name [--human_readable]

加上 ``--human_readable`` 选项，会以 KB/MB/GB 单位显示存储池的已用和可用空间。

输出是 JSON 格式，会包含下列字段：

* ``pools``: 数据和元数据存储池属性

        * ``avail``: 可用的空闲空间，单位是字节
        * ``used``: 已消耗的存储量，单位是字节
        * ``name``: 存储池名字

* ``mon_addrs``: Ceph 监视器地址列表
* ``used_size``: 当前已经用掉的 CephFS 卷大小，单位是字节
* ``pending_subvolume_deletions``: 等待被删除的子卷数量

``volume info`` 命令的输出样本：

.. prompt:: bash #

   ceph fs volume info vol_name

::

    {
        "mon_addrs": [
            "192.168.1.7:40977"
        ],
        "pending_subvolume_deletions": 0,
        "pools": {
            "data": [
                {
                    "avail": 106288709632,
                    "name": "cephfs.vol_name.data",
                    "used": 4096
                }
            ],
            "metadata": [
                {
                    "avail": 106288709632,
                    "name": "cephfs.vol_name.meta",
                    "used": 155648
                }
            ]
        },
        "used_size": 0
    }


FS 子卷组
---------
.. FS Subvolume groups

创建一个子卷组，执行下列命令：

.. prompt:: bash #

   ceph fs subvolumegroup create <vol_name> <group_name> [--size <size_in_bytes>] [--pool_layout <data_pool_name>] [--uid <uid>] [--gid <gid>] [--mode <octal_mode>]

即使要创建的子卷组已经存在，这个命令也会成功。

创建子卷组时，可以指定其数据存储池布局
（参阅 :doc:`/cephfs/file-layouts` ）、 uid 、 gid 、
以八进制数字表示的文件模式、以及按字节计算的大小。
子卷组的大小可设置配额来指定（参阅 :doc:`/cephfs/quota` ）。
默认情况下，创建的子卷组：文件权限位是八进制的 ``755`` 、
UID ``0`` 、 GID ``0`` 、数据存储池布局继承其父目录的。

您还可以用 ``--normalization`` 选项指定 Unicode 归一化方式。
这将用于内部文件名处理，以便可以把不同 Unicode 码位序列表示的
Unicode 字符都映射到相同的表示形式，
这意味着它们都将访问同一个文件。但是，
用户看到的还是他们在创建文件时使用的那个名称。

Unicode 归一化方式的有效值有：

    - nfd: 规范解构（默认）
    - nfc: 规范解构，而后进行规范组合
    - nfkd: 兼容解构
    - nfkc: 兼容解构，而后进行规范组合

如需了解更多 Unicode 归一化方式的信息，访问 https://unicode.org/reports/tr15

子卷组也可以配置成不区分大小写的访问，
加上 ``--casesensitive=0`` 选项即可。加了这个选项后，
只有字符大小写不同的文件名将被映射到同一个文件。
新建文件时所用文件名的大小写将保持不变。

.. note:: 设置 ``--casesensitive=0`` 选项会\
   隐式地启用子卷组上的 Unicode 归一化。

删除子卷组，执行下列命令：

.. prompt:: bash #

   ceph fs subvolumegroup rm <vol_name> <group_name> [--force]

如果子卷组不为空或不存在，则删除子卷组会失败。
当命令的参数是一个不存在的子卷组时，
``--force`` 标志可以让命令成功执行。

获取子卷组的绝对路径，执行下列命令：

.. prompt:: bash #

   ceph fs subvolumegroup getpath <vol_name> <group_name>

罗列子卷组，执行下列命令：

.. prompt:: bash #

   ceph fs subvolumegroup ls <vol_name>

.. note:: 子卷组的快照功能在主线 CephFS 里不再支持了，
   现有的组快照仍然能罗列、删除。

提取一个子卷组的元数据，
执行下列命令：

.. prompt:: bash #

   ceph fs subvolumegroup info <vol_name> <group_name>

输出是 JSON 格式，包含下列字段：

* ``atime``: 子卷组路径的访问时间，
  格式是 ``YYYY-MM-DD HH:MM:SS``
* ``mtime``: 子卷组路径的最近修改时间，
  格式是 ``YYYY-MM-DD HH:MM:SS``
* ``ctime``: 子卷组路径的最近更改时间，
  格式是 ``YYYY-MM-DD HH:MM:SS``
* ``uid``: 子卷组路径的 UID
* ``gid``: 子卷组路径的 GID
* ``mode``: 子卷组路径的权限位（ mode ）
* ``mon_addrs``: 监视器地址列表
* ``bytes_pcent``: 如果设置了配额，这里就显示已用配额的百分比；否则显示 "undefined"
* ``bytes_quota``: 如果设置了配额，这里就显示配额大小，字节数；否则显示 "infinite"
* ``bytes_used``: 当前用掉的子卷组大小，字节数
* ``created_at``: 子卷组的创建时间，按格式 ``YYYY-MM-DD HH:MM:SS``
* ``data_pool``: 子卷组所属的数据存储池

检查指定的子卷组是否存在，
执行下列命令：

.. prompt:: bash #

   ceph fs subvolumegroup exist <vol_name>

``exist`` 命令会输出：

* ``subvolumegroup exists``: 如果它存在
* ``no subvolumegroup exists``: 如果它不存在

.. note:: 此命令检查的是自定义组是否存在，
   而不是默认组是否存在。
   只有子卷组存在性的检查不足以确认此卷是否为空，
   还必须检查子卷是否存在，因为默认组里可能有子卷。

改变一个子卷组的大小，执行下列命令：

.. prompt:: bash #

   ceph fs subvolumegroup resize <vol_name> <group_name> <new_size> [--no_shrink]

此命令用于调整子卷组配额的大小，指定的新尺寸是 ``new_size`` 。
``--no_shrink`` 标志可防止子卷组缩小，
缩到低于当前已用大小。

子卷组的大小可以调整为无限大，
传入 ``inf`` 或 ``infinite`` 作为 ``new_size`` 。

删除子卷组的快照，
执行下列命令：

.. prompt:: bash #

   ceph fs subvolumegroup snapshot rm <vol_name> <group_name> <snap_name> [--force]

快照不存在时，此命令会失败，
加上 ``--force`` 选项命令就会成功。

罗列一个子卷组的快照，执行下列命令：

.. prompt:: bash #

   ceph fs subvolumegroup snapshot ls <vol_name> <group_name>


FS 子卷
-------
.. FS Subvolumes

创建子卷
~~~~~~~~
.. Creating a subvolume

创建子卷，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume create <vol_name> <subvol_name> [--size <size_in_bytes>] [--group_name <subvol_group_name>] [--pool_layout <data_pool_name>] [--uid <uid>] [--gid <gid>] [--mode <octal_mode>] [--namespace-isolated] [--earmark <earmark>]


即使要创建的子卷已经存在，此命令也会返回成功。

创建子卷时，可以指定它的子卷组、数据存储池布局、
UID 、 GID 、八进制的文件模式以及以字节表示的大小。
子卷的大小是用配额来指定的（参阅 :doc:`/cephfs/quota` ）。
可以在单独的 RADOS 命名空间中创建子卷，加 ``--namespace-isolated`` 选项。
默认情况下，子卷会在默认子卷组内创建，
八进制文件模式为 ``755`` ， uid 继承其子卷组的， gid 继承子卷组的，
数据存储池布局继承其父目录的、且没有大小限制。
你还可以用 ``--earmark`` 选项为子卷分配一个 earmark 。
earmark 是个唯一标识符，用于标记特定用途的子卷，
如 NFS 或 SMB 服务。默认不设置 earmark ，
这样就允许根据管理需要灵活分配。
空字符串（ "" ）可用于删除子卷上现有的所有标记。

earmark 机制能确保正确地标记和管理子卷，
有助于避免冲突，并确保每个子卷都与预期的服务或用例相关联。


可用的 earmark
~~~~~~~~~~~~~~
.. Valid Earmarks

- **对于 NFS:**

   - 可用的 earmark 格式是顶级范围（ top-level scope ）： ``'nfs'`` 。

- **对于 SMB:**

   - 可用的 earmark 格式有：

      - 顶级范围： ``'smb'`` 。
      - 顶级范围、加上模块内级别范围： ``'smb.cluster.{cluster_id}'`` ，
        其中， ``cluster_id`` 是唯一标识集群的短字符串。
      - 不含模块内范围，实例： ``smb``
      - 包含模块内范围，实例： ``smb.cluster.cluster_1``

.. note:: 如果要把 earmark 从一个范围改成另一个（比如 nfs 改为 smb ，或反过来），
   注意与先前范围相关的用户权限和 ACL 可能仍然有效。
   确保根据需要更新必要的权限，以维持正确的访问控制。

您还可以用 ``--normalization`` 选项指定 Unicode 归一化方式。
这将用于内部文件名处理，以便可以把不同 Unicode 码位序列表示的
Unicode 字符都映射到相同的表示形式，
这意味着它们都将访问同一个文件。但是，
用户看到的还是他们在创建文件时使用的那个名称。

Unicode 归一化方式的有效值有：

    - nfd: 规范解构（默认）
    - nfc: 规范解构，而后进行规范组合
    - nfkd: 兼容解构
    - nfkc: 兼容解构，而后进行规范组合

如需了解更多 Unicode 归一化方式的信息，访问 https://unicode.org/reports/tr15

子卷组也可以配置成不区分大小写的访问，
加上 ``--casesensitive=0`` 选项即可。加了这个选项后，
只有字符大小写不同的文件名将被映射到同一个文件。
新建文件时所用文件名的大小写将保持不变。

.. note:: 设置 ``--casesensitive=0`` 选项会\
   隐式地启用子卷组上的 Unicode 归一化。

加密时可使用一个单独的加密标签（ encryption tag ）。
这样可以给子卷打上一个标识符，对系统管理员或其他服务有用。
默认情况下，此标签为空，而且是可选的。

加密标签可用于把子卷与其关联的加密主密钥和策略对应起来。
可以把加密标签（ enctag ）的值与相应的主密钥进行映射。
这样就可以用于解锁子卷。这个主密钥可以存储在安全位置，如密钥管理服务器中。


删除子卷
~~~~~~~~
.. Removing a subvolume

删除子卷，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume rm <vol_name> <subvol_name> [--group_name <subvol_group_name>] [--force] [--retain-snapshots]

此命令将删除子卷及其内容，分两步完成。
首先，把子卷移至垃圾文件夹；
其次，异步地清除垃圾文件夹中的内容。

如果子卷有快照或不存在，子卷移除将失败。
加 ``--force`` 选项可以让 "non-existent subvolume remove" 的命令成功。

要在删除子卷的同时保留该子卷的快照，用 ``--retain-snapshots`` 标志。
如果保留与指定子卷相关联的快照，
那么所有与保留的快照不相干的操作，
都会把这个子卷视为空卷。

.. note:: 可以用 ``ceph fs subvolume create`` 命令重新创建保留了快照的子卷。

.. note:: 保留的快照可用作重新创建子卷、或克隆新子卷的克隆源。


改变子卷大小
~~~~~~~~~~~~
.. Resizing a subvolume

更改子卷大小，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume resize <vol_name> <subvol_name> <new_size> [--group_name <subvol_group_name>] [--no_shrink]

此命令用于调整子卷配额的大小，新尺寸是 ``new_size`` 。
``--no_shrink`` 标志可防止子卷缩小到低于当前子卷的 "used size （已用大小）"。

子卷大小可以调整为逻辑上无限大（却是稀疏的），
传入 ``inf`` 或 ``infinite`` 作为 ``<new_size>`` 。

授予 CephFS 认证 ID
~~~~~~~~~~~~~~~~~~~
.. Authorizing CephX auth IDs

授予 CephX 认证 ID ，这将授予对文件系统子卷的读/读写访问权限，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume authorize <vol_name> <sub_name> <auth_id> [--group_name=<group_name>] [--access_level=<access_level>]

``<access_level>`` 选项的值可以是 ``r`` 或 ``rw`` 。

取消授予的 CephX 认证 ID
~~~~~~~~~~~~~~~~~~~~~~~~
.. De-authorizing CephX auth IDs

取消授予的 CephX 认证 ID ，这将删除对文件系统子卷的读/读写访问权限，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume deauthorize <vol_name> <sub_name> <auth_id> [--group_name=<group_name>]

罗列 CephX 认证 ID
~~~~~~~~~~~~~~~~~~
.. Listing CephX auth IDs

罗列 CephX 认证 ID 被授予的文件系统子卷访问权限，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume authorized_list <vol_name> <sub_name> [--group_name=<group_name>]

驱逐文件系统客户端（ auth ID ）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. Evicting File System Clients (Auth ID)

驱逐用认证 ID 和挂载的子卷标识的文件系统客户端，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume evict <vol_name> <sub_name> <auth_id> [--group_name=<group_name>]

提取一个子卷的绝对路径
~~~~~~~~~~~~~~~~~~~~~~
.. Fetching the Absolute Path of a Subvolume

提取一个子卷的绝对路径，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume getpath <vol_name> <subvol_name> [--group_name <subvol_group_name>]

提取一个子卷的信息
~~~~~~~~~~~~~~~~~~
.. Fetching a Subvolume's Information

提取一个子卷的信息，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume info <vol_name> <subvol_name> [--group_name <subvol_group_name>]

其输出是 JSON 格式，包含下列字段。

* ``atime``: 子卷路径的访问时间，按格式 ``YYYY-MM-DD HH:MM:SS``
* ``mtime``: 子卷路径的修改时间，按格式 ``YYYY-MM-DD HH:MM:SS``
* ``ctime``: 子卷路径的变更时间，按格式 ``YYYY-MM-DD HH:MM:SS``
* ``uid``: 子卷路径的 UID
* ``gid``: 子卷路径的 GID
* ``mode``: 子卷路径的权限位
* ``mon_addrs``: 监视器地址列表
* ``bytes_pcent``: 如果设置了配额，这里就显示已用配额的百分比；
  否则显示 ``undefined``
* ``bytes_quota``: 如果设置了配额，这里就显示配额大小，字节数；
  否则显示 ``infinite``
* ``bytes_used``: 当前用掉的子卷组大小，字节数
* ``created_at``: 子卷组的创建时间，
  按格式 ``YYYY-MM-DD HH:MM:SS``
* ``data_pool``: 子卷组所属的数据存储池
* ``path``: 子卷的绝对路径
* ``type``: 子卷类型，标明它是 ``clone`` 还是 ``subvolume``
* ``pool_namespace``: 子卷的 RADOS 命名空间
* ``features``: 子卷支持的功能
* ``state``: 这个子卷当前的状态
* ``earmark``: 这个子卷的 earmark
* ``source``: 仅当子卷是克隆品时存在。它包含源快照的名字，
  还有此快照所在位置的卷名、子卷组和子卷。
  如果此克隆品是用 Tentacle 或更早版本创建，
  此字段的值是 'N/A' 。
* ``enctag``: 本子卷的加密标签。

如果子卷已经删除，而它的快照保留下来了，
那么输出会只包含下列字段。

* ``type``: 子卷类型，标明它是 ``clone`` 还是 ``subvolume``
* ``features``: 子卷支持的功能
* ``state``: 这个子卷当前的状态

一个子卷的 ``features`` 基于此子卷的内部版本，
是下列中的一个子集：

* ``snapshot-clone``: 支持克隆，
  用子卷的快照作为源
* ``snapshot-autoprotect``: 如果快照是有用的克隆源，
  此功能支持自动保护快照，以防删除。
* ``snapshot-retention``: 支持删除子卷内容，
  却保留所有现存快照。

一个子卷的 ``state`` 基于此子卷的当前状态，
且包含下列值之一：

* ``complete``: 子卷正常，可做任何操作
* ``snapshot-retained``: 子卷删除了，但它的快照保留着

罗列子卷
~~~~~~~~
.. Listing Subvolumes

罗列子卷，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume ls <vol_name> [--group_name <subvol_group_name>]

.. note:: 已经删除但保留了快照的那些子卷也会罗列出来。

检查一个子卷是否存在
~~~~~~~~~~~~~~~~~~~~
.. Checking for the Presence of a Subvolume

检查一个指定子卷是否存在，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume exist <vol_name> [--group_name <subvol_group_name>]

``exist`` 命令可能的结果：

* ``subvolume exists``: 如果指定的 ``group_name`` 里面有子卷
* ``no subvolume exists``: 如果指定的 ``group_name`` 里面没有子卷


在一个子卷上设置自定义元数据
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. Setting Custom Metadata On a Subvolume

在子卷上设置自定义的键值对元数据，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume metadata set <vol_name> <subvol_name> <key_name> <value> [--group_name <subvol_group_name>]

.. note:: 如果 key_name 已经存在，那么它的旧值会被新值替换。

.. note:: ``key_name`` 和 ``value`` 应该是 ASCII 字符组成的字符串
   （就是 Python 的 ``string.printable`` 指定的那些）。
   ``key_name`` 不区分大小写，总是以小写保存。

.. note:: 拍快照时不会保留子卷上的自定义元数据，
   因此，克隆子卷快照时也不会保留。


查看子卷的自定义元数据集合
~~~~~~~~~~~~~~~~~~~~~~~~~~
.. Getting The Custom Metadata Set of a Subvolume

查看设置的自定义元数据，需指定元数据键名，
执行下列命令：

.. prompt:: bash #

   ceph fs subvolume metadata get <vol_name> <subvol_name> <key_name> [--group_name <subvol_group_name>]

罗列子卷的自定义元数据集合
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. Listing The Custom Metadata Set of a Subvolume

罗列子卷上设置的自定义元数据（键值对），执行下列命令：

.. prompt:: bash #

   ceph fs subvolume metadata ls <vol_name> <subvol_name> [--group_name <subvol_group_name>]

删除子卷的一个自定义元数据集合
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. Removing a Custom Metadata Set from a Subvolume

删除子卷上设置的自定义元数据，需指定元数据键名，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume metadata rm <vol_name> <subvol_name> <key_name> [--group_name <subvol_group_name>] [--force]

加 ``--force`` 可以让此命令返回成功，否则它可能失败（如果删除的元数据键不存在）。

查看子卷的 earmark
~~~~~~~~~~~~~~~~~~
.. Getting earmark of a subvolume

查看子卷的 earmark ，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume earmark get <vol_name> <subvol_name> [--group_name <subvol_group_name>]

设置子卷的 earmark
~~~~~~~~~~~~~~~~~~
.. Setting earmark of a subvolume

设置子卷的 earmark ，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume earmark set <vol_name> <subvol_name> [--group_name <subvol_group_name>] <earmark>

删除子卷的 earmark
~~~~~~~~~~~~~~~~~~
.. Removing earmark of a subvolume

删除子卷的 earmark ，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume earmark rm <vol_name> <subvol_name> [--group_name <subvol_group_name>]

查看子卷的 enctag
~~~~~~~~~~~~~~~~~
.. Getting enctag of a subvolume

要查看一个子卷的 enctag ，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume enctag get <vol_name> <subvol_name> [--group_name <subvol_group_name>]

设置子卷的 enctag
~~~~~~~~~~~~~~~~~
.. Setting enctag of a subvolume

要给一个子卷设置 enctag ，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume enctag set <vol_name> <subvol_name> [--group_name <subvol_group_name>] <enctag>

删除子卷的 enctag
~~~~~~~~~~~~~~~~~
.. Removing enctag of a subvolume

要删除一个子卷的 enctag ，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume enctag rm <vol_name> <subvol_name> [--group_name <subvol_group_name>]

创建子卷的快照
~~~~~~~~~~~~~~
.. Creating a Snapshot of a Subvolume

给子卷创建一个快照，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume snapshot create <vol_name> <subvol_name> <snap_name> [--group_name <subvol_group_name>]

删除子卷的快照
~~~~~~~~~~~~~~
.. Removing a Snapshot of a Subvolume

删除子卷的一个快照，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume snapshot rm <vol_name> <subvol_name> <snap_name> [--group_name <subvol_group_name>] [--force]

加 ``--force`` 选项可以让此命令成功，否则它可能失败（假如快照不存在）。

.. note:: 对于一个保留了快照的子卷，如果它的最后一个快照删掉了，这个子卷也会删掉。

获取一个子卷快照的路径
----------------------
.. Fetching Path of a Snapshot of a Subvolume

要获取一个子卷快照的绝对路径，执行下列命令：

.. prompt:: base #

    ceph fs subvolume snapshot getpath <volname> <subvol_name> <snap_name> [<group_name>]

罗列子卷的快照
~~~~~~~~~~~~~~
.. Listing the Snapshots of a Subvolume

罗列子卷的快照，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume snapshot ls <vol_name> <subvol_name> [--group_name <subvol_group_name>]

提取一个快照的信息
~~~~~~~~~~~~~~~~~~
.. Fetching a Snapshot's Information

提取一个快照的信息，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume snapshot info <vol_name> <subvol_name> <snap_name> [--group_name <subvol_group_name>]

其输出是 JSON 格式的，包含下列字段。

* ``created_at``: 快照的创建时间，按格式 ``YYYY-MM-DD HH:MM:SS:ffffff``
* ``data_pool``: 快照所属的数据存储池
* ``has_pending_clones``: 如果快照克隆正在进行中就是 ``yes`` ，否则就是 ``no``
* ``pending_clones``: 正在进行的或待定的克隆操作列表，
  如果有目标组也会一并列出；否则此字段不显示。
* ``orphan_clones_count``: 如果有孤儿克隆，
  这里就是孤儿克隆的数量，否则此字段不显示。

有快照克隆正在进行或待定时的输出样本：

.. prompt:: bash #

   ceph fs subvolume snapshot info cephfs subvol snap

::

    {
        "created_at": "2022-06-14 13:54:58.618769",
        "data_pool": "cephfs.cephfs.data",
        "has_pending_clones": "yes",
        "pending_clones": [
            {
                "name": "clone_1",
                "target_group": "target_subvol_group"
            },
            {
                "name": "clone_2"
            },
            {
                "name": "clone_3",
                "target_group": "target_subvol_group"
            }
        ]
    }

没有快照克隆正在进行或待定时的输出样本：

.. prompt:: bash #

   ceph fs subvolume snapshot info cephfs subvol snap

::

    {
        "created_at": "2022-06-14 13:54:58.618769",
        "data_pool": "cephfs.cephfs.data",
        "has_pending_clones": "no"
    }

在一个快照上设置自定义键值对元数据
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. Setting Custom Key-Value Pair Metadata on a Snapshot

在快照上设置自定义键值对元数据，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume snapshot metadata set <vol_name> <subvol_name> <snap_name> <key_name> <value> [--group_name <subvol_group_name>]

.. note:: 如果 ``key_name`` 已经存在，那么旧值会被新值替换。

.. note:: ``key_name`` 和它的值应该是 ASCII 字符组成的字符串
   （就是 Python 的 ``string.printable`` 里指定的那些）。
   ``key_name`` 不区分大小写，且总是存储小写的。

.. note:: 拍子卷快照的时候，自定义元数据不会保留下来，
   因此，克隆子卷快照时也不会保留。

查看一个快照上设置的自定义元数据
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. Getting Custom Metadata That Has Been Set on a Snapshot

用元数据键、查看之前在快照上设置的自定义元数据，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume snapshot metadata get <vol_name> <subvol_name> <snap_name> <key_name> [--group_name <subvol_group_name>]

罗列一个快照上设置的自定义元数据
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. Listing Custom Metadata that has been Set on a Snapshot

罗列快照上设置的自定义元数据（键值对），执行下列命令：

.. prompt:: bash #

   ceph fs subvolume snapshot metadata ls <vol_name> <subvol_name> <snap_name> [--group_name <subvol_group_name>]

删除快照的自定义元数据
~~~~~~~~~~~~~~~~~~~~~~
.. Removing Custom Metadata from a Snapshot

用元数据键、删除快照上设置的自定义元数据，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume snapshot metadata rm <vol_name> <subvol_name> <snap_name> <key_name> [--group_name <subvol_group_name>] [--force]

加 ``--force`` 选项可以让此命令成功，否则它可能失败（如果指定的元数据键不存在）。

克隆快照
--------
.. Cloning Snapshots

可以通过克隆子卷快照来创建子卷。克隆是一种异步操作，可将数据从快照复制到子卷。
由于克隆是一种涉及大批量复制的操作，因此对于非常大的数据集来说，速度会比较慢。

.. note:: 如果有待定或正在进行的克隆操作，删除快照（源子卷）会失败。

在 Nautilus 版里，克隆之前先保护快照是前提条件。
为此引入了可以保护和解除保护快照的命令。
这一前提条件已废弃，可能会在未来的版本中删除。

正在废弃的命令有：

.. prompt:: bash #

   ceph fs subvolume snapshot protect <vol_name> <subvol_name> <snap_name> [--group_name <subvol_group_name>]
   ceph fs subvolume snapshot unprotect <vol_name> <subvol_name> <snap_name> [--group_name <subvol_group_name>]

.. note:: 使用上面的命令不会产生错误，但是它们没有实际用途。

.. note:: 根据 ``snapshot-autoprotect`` （快照自动保护）功能是否可用，
   用 ``subvolume info`` 命令提取子卷元数据，里面有支持的 ``features`` ，
   可用来帮助决定是否需要保护/取消保护快照。

启动一个克隆操作，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume snapshot clone <vol_name> <subvol_name> <snap_name> <target_subvol_name>

.. note:: ``subvolume snapshot clone`` 命令依赖于上面提过的\
   配置选项 ``snapshot_clone_no_wait``

快照（源子卷）属于非默认组时，执行下列命令。注意，需要指定组名：

.. prompt:: bash #

   ceph fs subvolume snapshot clone <vol_name> <subvol_name> <snap_name> <target_subvol_name> --group_name <subvol_group_name>

克隆的子卷可以位于不同于源快照所属的组（默认情况下，
会在默认组中创建克隆的子卷）。克隆到指定组，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume snapshot clone <vol_name> <subvol_name> <snap_name> <target_subvol_name> --target_group_name <subvol_group_name>

创建克隆子卷时可以指定存储池布局，
指定方式类似于创建子卷时指定存储池布局。
创建具有指定存储池布局的克隆子卷，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume snapshot clone <vol_name> <subvol_name> <snap_name> <target_subvol_name> --pool_layout <pool_layout>

检查克隆操作的状态，执行下列命令：

.. prompt:: bash #

   ceph fs clone status <vol_name> <clone_name> [--group_name <group_name>]

克隆操作的状态可以是下列之一：

#. ``pending``     : 克隆操作尚未开始
#. ``in-progress`` : 克隆操作正在进行
#. ``complete``    : 克隆操作已成功完成
#. ``failed``      : 克隆操作失败了
#. ``canceled``    : 用户取消了克隆操作

克隆失败的原因有如下几个：

#. ``errno``     : 错误号
#. ``error_msg`` : 失败的报错字符串

这是一个 ``in-progress`` （正在进行的）克隆实例:

.. prompt:: bash #

   ceph fs subvolume snapshot clone cephfs subvol1 snap1 clone1
   ceph fs clone status cephfs clone1

::

    {
      "status": {
        "state": "in-progress",
        "source": {
          "volume": "cephfs",
          "subvolume": "subvol1",
          "snapshot": "snap1"
        },
        "progress_report": {
          "percentage cloned": "12.24%",
          "amount cloned": "376M/3.0G",
          "files cloned": "4/6"
        }
      }
    }

当克隆正在进行时，输出中还会打印一份进度报告。
这里只报告指定克隆的进度。
对于所有正在进行的克隆的总体进度，
``ceph status`` 命令的输出结果底部会打印一个进度条： ::

  progress:
    3 ongoing clones - average progress is 47.569% (10s)
      [=============...............] (remaining: 11s)

如果克隆的任务数多于克隆线程数，则会打印两个进度条，
一个是正在进行的克隆（与上述相同），
另一个是所有（正在进行的+待处理的）克隆： ::

  progress:
    4 ongoing clones - average progress is 27.669% (15s)
      [=======.....................] (remaining: 41s)
    Total 5 clones - average progress is 41.667% (3s)
      [===========.................] (remaining: 4s)

.. note:: 只有在克隆状态为 ``failed`` 或 ``canceled`` 时，才会显示 ``failure`` 段。

这是个 ``failed`` 克隆的实例：

.. prompt:: bash #

   ceph fs subvolume snapshot clone cephfs subvol1 snap1 clone1
   ceph fs clone status cephfs clone1

::

    {
        "status": {
            "state": "failed",
            "source": {
                "volume": "cephfs",
                "subvolume": "subvol1",
                "snapshot": "snap1"
                "size": "104857600"
            },
            "failure": {
                "errno": "122",
                "errstr": "Disk quota exceeded"
            }
        }
    }

.. note::  由于 ``subvol1`` 位于默认组里，所以，
   ``source`` 对象的 ``clone status`` 没有包括组名。

.. note:: 只有克隆操作成功完成后，
   才能访问到克隆的子卷。

克隆操作成功完成后，
``clone status`` 结果如下：

.. prompt:: bash #

   ceph fs clone status cephfs clone1

::

    {
        "status": {
            "state": "complete"
        }
    }

如果克隆操作不成功， ``state`` 的值将是 ``failed`` 。

要重试失败的克隆操作，必须先删除未完成的克隆，
并再次发起克隆操作。

要删除一个部分完成的克隆，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume rm <vol_name> <clone_name> [--group_name <group_name>] --force

.. note:: 克隆操作只同步目录、常规文件和符号链接。
   inode 时间戳（访问和修改时间）的同步粒度\
   能达到秒级。

处于 ``in-progress`` 或者 ``pending`` 状态的克隆操作可以取消。
取消克隆操作用 ``clone cancel`` 命令：

.. prompt:: bash #

   ceph fs clone cancel <vol_name> <clone_name> [--group_name <group_name>]

成功取消后，克隆的子卷状态会变成 ``canceled`` ：

.. prompt:: bash #

   ceph fs subvolume snapshot clone cephfs subvol1 snap1 clone1
   ceph fs clone cancel cephfs clone1
   ceph fs clone status cephfs clone1

::

    {
        "status": {
            "state": "canceled",
            "source": {
                "volume": "cephfs",
                "subvolume": "subvol1",
                "snapshot": "snap1"
            }
        }
    }

.. note:: 删除已取消的克隆品用 ``fs subvolume rm`` 命令，
   要加 ``--force`` 选项。

可配置选项
~~~~~~~~~~
.. Configurables

配置克隆操作的最大并行数量，默认为 4 ：

.. prompt:: bash #

   ceph config set mgr mgr/volumes/max_concurrent_clones <value>

暂停异步清除回收站子卷的线程。
此选项在集群恢复期间有用：

.. prompt:: bash #

    ceph config set mgr/volumes/pause_purging true

恢复用于清除的线程：

.. prompt:: bash #

    ceph config set mgr/volumes/pause_purging false


暂停异步克隆子卷快照的线程。此选项在集群恢复期间有用：

.. prompt:: bash #

    ceph config set mgr/volumes/pause_cloning true

恢复用于克隆的线程：

.. prompt:: bash #

    ceph config set mgr/volumes/pause_cloning false


配置 ``snapshot_clone_no_wait`` 选项：

``snapshot_clone_no_wait`` 配置选项用于在克隆线程
（可用上述选项进行配置，例如 ``max_concurrent_clones`` ）
不可用时拒绝克隆创建请求。此选项默认是启用的。
意思是该值被设置成了 ``True`` ，但可以用下列命令进行配置：

.. prompt:: bash #

   ceph config set mgr mgr/volumes/snapshot_clone_no_wait <bool>

``snapshot_clone_no_wait`` 当前的值可以用下列命令提取。

.. prompt:: bash #

   ceph config get mgr mgr/volumes/snapshot_clone_no_wait


控制子卷快照的可见性
--------------------
.. Controlling Subvolume Snapshot Visibility

.. note:: 当前只有 FUSE/libcephfs 客户端支持此功能。
   内核客户端支持正在实现：进度可以在这里查看
   https://tracker.ceph.com/issues/72589 。

通过以下两个操作，可以使子卷的快照对兼容客户端隐藏：

#. 将子卷的 ``snapshot_visibility`` 标志设置为 ``false`` （默认为 ``true`` ）。
#. 将目标客户端的客户端一侧配置选项 ``client_respect_subvolume_snapshot_visibility``
   设置为 ``true`` （默认为 ``false`` ）。

切换 ``snapshot_visibility`` 的 CLI 命令如下：

.. prompt:: bash #

   ceph fs subvolume snapshot_visibility set <vol_name> <sub_volname> [--group-name <subvol_group_name>] <true|false>

此命令用于更新内部 vxattr ``ceph.dir.subvolume.snaps.visible`` ，
并在 dirinode （即子卷）的 SnapRealm 中设置
``is_snapdir_visible`` 标志。

.. note:: 虽然可以直接修改，但建议使用子卷 API 。
          这样更方便，并且在设置 vxattr 时\
          可以避免可能出现的 ``EPERM`` （无权限被拒）错误，
          尤其是在客户端缺乏所需能力的情况下。
          设置此 vxattr 的方法是：

          .. prompt:: bash #

             setfattr -n ceph.dir.subvolume.snaps.visible -v 0|1 <subvolume_path>

``client_respect_subvolume_snapshot_visibility`` 选项决定着\
一个客户端是否会遵守子卷的可见性标志。
可以对每个客户端设置，用命令：

.. prompt:: bash #

   ceph config set client.<id> client_respect_subvolume_snapshot_visibility <true|false>

.. note:: 其中的 ``<id>`` 参数是 CephX 用户。

要给所有客户端全局地配置 ``client_respect_subvolume_snapshot_visibility`` ，
执行命令时不要指定 ``id`` 即可：

.. prompt:: bash #

   ceph config set client client_respect_subvolume_snapshot_visibility <true|false>

.. note:: MGR 守护进程作为具有特权的 CephFS 客户端运行，
   因此可以绕过快照可见性限制。
   这种行为对于确保快照调度和快照克隆等操作的可靠执行是必要的。
   因此，修改 ``client_respect_subvolume_snapshot_visibility``
   配置选项对运行在 MGR 守护进程内的 CephFS 实例没有影响。


如何在一个子卷上禁用快照可见性？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. How to Disable Snapshot Visibility on a Subvolume?

例如，要想防止以 ``client.user1`` CephX 用户验证过身份的客户端\
查看 ``vol1`` 卷下的 ``sv1`` 子卷的快照，
首先，用下列命令禁用 ``sv1`` 上的 ``snapshot_visibility`` 标志：

.. prompt:: bash #

   ceph fs subvolume snapshot_visibility set vol1 sv1 false

然后切换此客户端的配置，用命令：

.. prompt:: bash #

   ceph config set client.user1 client_respect_subvolume_snapshot_visibility true

这有效地防止了使用 ``client.user1`` 挂载的客户端\
对 ``sv1`` 子卷的 ``.snap`` 目录执行 ``lookup()`` 调用，
从而隐藏了其快照。

.. note:: 当子卷的快照可见性被禁用时，
   任何快照操作（包括快照创建、删除或重命名）都将被阻止，
   因为这些操作需要先成功地找到 ``.snap`` 目录。

.. note:: 子卷快照的可见性完全取决于客户端是否配置为\
   遵守子卷的 ``snapshot_visibility`` 标志。
   也就是说，不管子卷上的这个标志设置为 ``true`` 还是 ``false`` ，
   除非客户端的 ``client_respect_subvolume_snapshot_visibility``
   配置选项明确设置为 ``true`` ，
   否则该标志都将被忽略。


.. _subvol-pinning:

锁定子卷和子卷组
----------------
.. Pinning Subvolumes and Subvolume Groups

子卷和子卷组可根据策略自动锁定（ pinned ）到 rank 。
这样可以按可预测且稳定的方式在 MDS rank 之间分配负载。
详细了解锁定机制，请看 :ref:`cephfs-pinning` 和
:ref:`cephfs-ephemeral-pinning` 。

配置子卷组的锁定，执行下列命令：

.. prompt:: bash #

   ceph fs subvolumegroup pin <vol_name> <group_name> <pin_type> <pin_setting>

配置子卷的锁定，执行下列命令：

.. prompt:: bash #

   ceph fs subvolume pin <vol_name> <group_name> <pin_type> <pin_setting>

在大多数情况下，都需要设置子卷组锁定。 ``pin_type`` 可以是 ``export`` （导出）、
``distributed`` （分布式）或 ``random`` （随机）。
``pin_setting`` 对应扩展属性的 "value" ，这在上文提到的锁定文档里有。

下面是个实例，在子卷组上设置分布式锁定策略：

.. prompt:: bash #

   ceph fs subvolumegroup pin cephfilesystem-a csi distributed 1

此命令将在 csi 子卷组上启用分布式子树分区策略。
这会让组内的每个子卷自动锁定到\
文件系统内的一个可用 rank 。


归一化和大小写敏感
------------------
.. Normalization and Case Sensitivity

subvolumegroup 和 subvolume 接口有一个 porcelain layer API （统一抽象层），
用于操作 ``ceph.dir.charmap`` 配置（另请参阅 :ref:`charmap` ）。


charmap 的配置
~~~~~~~~~~~~~~
.. Configuring the charmap

要给一个 subvolumegroup 配置 charmap ：

.. prompt:: bash #

    ceph fs subvolumegroup charmap set <vol_name> <group_name> <setting> <value>

或者给一个 subvolume 配置：

.. prompt:: bash #

    ceph fs subvolume charmap set <vol_name> <subvol> <--group_name=name> <setting> <value>

例如：

.. prompt:: bash #

    ceph fs subvolumegroup charmap set vol csi normalization nfd

命令输出：

::

    {"casesensitive":true,"normalization":"nfd","encoding":"utf8"}


读取 charmap
~~~~~~~~~~~~
.. Reading the charmap

要读取一个 subvolumegroup 的 ``charmap`` 配置：

.. prompt:: bash #

    ceph fs subvolumegroup charmap get <vol_name> <group_name> <setting>

或者一个 subvolume 的：

.. prompt:: bash #

    ceph fs subvolume charmap get <vol_name> <subvol> <--group_name=name> <setting>

例如：

.. prompt:: bash #

    ceph fs subvolume charmap get vol subvol --group_name=csi casesensitive

::

    0

要读取一个 subvolumegroup 的完整 ``charmap`` ：

.. prompt:: bash #

    ceph fs subvolumegroup charmap get <vol_name> <group_name>

或者读取 subvolume 的：

.. prompt:: bash #

    ceph fs subvolume charmap get <vol_name> <subvol> <--group_name=name>

例如：

.. prompt:: bash #

    ceph fs subvolumegroup charmap get vol csi

命令输出：

::

    {"casesensitive":false,"normalization":"nfd","encoding":"utf8"}


删除 charmap
~~~~~~~~~~~~
.. Removing the charmap

要删除一个 subvolumegroup 的 ``charmap`` 配置：

.. prompt:: bash #

    ceph fs subvolumegroup charmap rm <vol_name> <group_name

或者一个 subvolume 的：

.. prompt:: bash #

    ceph fs subvolume charmap rm <vol_name> <subvol> <--group_name=name>

例如：

.. prompt:: bash #

    ceph fs subvolumegroup charmap rm vol csi

输出：

::

    {}

.. note:: 只有当 subvolumegroup 或 subvolume 为空的时候才能删除其 ``charmap`` 。


子卷静默（ subvolume quiesce ）
-------------------------------
.. Subvolume quiesce

.. note:: 此段落信息只适用于 Squid 以及更高版本。

CephFS 快照不能保证强一致性，因为在多个客户端执行写操作的情况下，
一致性备份和灾难恢复是分布式应用程序不得不面临的严峻挑战。
即使应用程序能够使用文件系统刷回功能来同步其分布式组件中的检查点的情况下，
也不能保证所有确认的写入都会进入指定快照。

为此，开发出了子卷静默（ quiesce ）功能，目的是为多客户端应用程序提供\
企业级一致性保证，这些应用程序使用着一个或多个子卷。有了这个功能，
可以暂停指定卷（文件系统）的一组子卷的 IO 。通过在所有客户端强制执行这种暂停，
可以保证应用程序在暂停前到达的所有持久（已写入的）检查点、
都可以从暂停期间拍下的快照中恢复。

``volumes`` 插件提供了一个 CLI ，用于启动和等待一组子卷的暂停。
这种暂停称为 `quiesce` （静默），也用作命令名称：

.. prompt:: bash $ auto

  $ ceph fs quiesce <vol_name> --set-id myset1 <[group_name/]sub_name...> --await
  # 在 IO 暂停生效后执行动作，比如拍快照
  $ ceph fs quiesce <vol_name> --set-id myset1 --release --await
  # 如果成功，就认为此集合的所有成员仍被暂停着，然后释放它们

``fs quiesce`` 功能基于更底层的 ``quiesce db`` 服务，它是由 MDS 守护进程提供的，
操作粒度可达到文件系统路径。 ``volumes`` 插件只是把子卷名映射到\
指定文件系统内的相应路径，然后向 MDS 发出相应的 ``quiesce db`` 命令。
有关底层服务的更多信息，参阅\ :ref:`开发者指南 <dev_mds_internals_quiesce>`\ 。


可用操作
~~~~~~~~
.. Operations

quiesce （静默）操作可以作用于一个或多个子卷（就是文件系统中的路径）组成的集合，
此集合称为 `quiesce set` 。每个 `quiesce set` 都用一个唯一的 `set id` 标识。
可以通过以下方式操作 `quiesce set` ：

* **include** （包含）一或多个子卷 - quiesce set 成员
* **exclude** （排除）一或多个成员
* **cancel** （取消）此集合，异步地中止当前所有成员的暂停
* **release** （释放）此集合，请求结束所有成员的暂停，并需要得到所有客户端的确认。
* **query** （查询）集合当前的状态，用 id 指定单个集合、或所有有效集合、或所有已知集合
* **cancel all** （取消所有）有效集合，这是需要立即恢复 IO

上述操作都是非阻塞操作：它们只是尝试想做的修改，
并返回目标集合的最新版本，而不管操作是否会成功。
集合的状态可能会因修改而改变，而响应返回的版本能保证与此操作、
还有同一控制循环批次中其他可能的成功操作的状态一致。

有些集合的状态是 `awaitable` 。我们将在下文讨论这些状态，但现在有必要提及的是，
上述所有命令都可以用 **await** 修饰符进行修改，这会让它们在应用想要的修改后\
一直阻塞在集合上，前提是此集合状态为 `awaitable` 。这样的命令会一直阻塞，
直到集合到达等待的那个状态、或被其他命令修改、或转换成另一个状态。
返回码会明确说明退出的条件，响应内容将始终包含已知的最新集合状态。

.. image:: quiesce-set-states.svg

图中的 `Awaitable` 状态用 ``(a)`` 或 ``(A)`` 标记了。当此集合处于 ``(a)`` 状态时，
阻塞着的操作将处于等待状态；如果集合到达 ``(A)`` 状态，操作就成功地完成。
如果此集合已经处于 ``(A)`` 状态，那么操作会立即成功完成。

大多数操作都需要带集合 ID （ set-id ），例外的有：

* 创建新集合，却没有指定集合 id
* 查询有效的、或所有已知集合，还有
* 取消所有

通过 `include` 或 `reset` 命令拉进成员，即可创建一个新集合。可以指定集合 id ，
而且如果它是新 id ，那么这个集合带着指定成员一创建就处于 `QUIESCING` 状态。
如果在包含或重置成员时没有指定集合 id ，就会创建一个具有唯一集合 id 的新集合。
在输出里就能找到它的集合 id ：

.. prompt:: bash $ auto

  $ ceph fs quiesce fs1 sub1 --set-id=unique-id
  {
      "epoch": 3,
      "set_version": 1,
      "sets": {
          "unique-id": {
              "version": 1,
              "age_ref": 0.0,
              "state": {
                  "name": "TIMEDOUT",
                  "age": 0.0
              },
              "timeout": 0.0,
              "expiration": 0.0,
              "members": {
                  "file:/volumes/_nogroup/sub1/b1fcce76-3418-42dd-aa76-f9076d047dd3": {
                      "excluded": false,
                      "state": {
                          "name": "QUIESCING",
                          "age": 0.0
                      }
                  }
              }
          }
      }
  }

输出中包含了我们刚刚成功创建的集合，但它已经超时了（ `TIMEDOUT` ）。
这是符合预期的，因为我们没有给这个 quiesce 指定超时时间，
而且我们可以从输出中看到，它的默认初始化值为 0 ，同时还有过期时间。

超时选项
~~~~~~~~
.. Timeouts

两个超时参数，即 `timeout` 和 `expiration` ，是防止应用程序意外引起 DOS 状况
（ DOS condition, Denial-of-Service 拒绝服务？）的主要防护措施。
任何操作有效集合的命令都可以加 ``--timeout`` 或 ``--expiration`` 参数，
用以更新这个集合的值。如果存在，那么此命令请求执行的操作之前，会先应用更新的值。

.. prompt:: bash # auto

  # ceph fs quiesce fs1 --set-id=unique-id --timeout=10 > /dev/null
  Error EPERM:  

对我们的 ``unique-id`` 集合来说已经太晚了，因为它已处于终结状态。
处于终结状态（即非活动、无效状态）时，就不允许再更改集合了。我们新建一个集合：

.. prompt:: bash $ auto

  $ ceph fs quiesce fs1 sub1 --timeout 60
  {
      "epoch": 3,
      "set_version": 2,
      "sets": {
          "8988b419": {
              "version": 2,
              "age_ref": 0.0,
              "state": {
                  "name": "QUIESCING",
                  "age": 0.0
              },
              "timeout": 60.0,
              "expiration": 0.0,
              "members": {
                  "file:/volumes/_nogroup/sub1/b1fcce76-3418-42dd-aa76-f9076d047dd3": {
                      "excluded": false,
                      "state": {
                          "name": "QUIESCING",
                          "age": 0.0
                      }
                  }
              }
          }
      }
  }

这次，我们没有指定集合 id ，因此系统创建了一个新 id 。
我们在输出中看到了它的 id ，是 ``8988b419`` 。命令执行成功了，
我们可以看到这次的集合处于 `QUIESCING` 状态。此时，我们可以向此集合添加更多成员：

.. prompt:: bash $ auto

  $ ceph fs quiesce fs1 --set-id 8988b419 --include sub2 sub3
  {
      "epoch": 3,
      "set_version": 3,
      "sets": {
          "8988b419": {
              "version": 3,
              "age_ref": 0.0,
              "state": {
                  "name": "QUIESCING",
                  "age": 30.7
              },
              "timeout": 60.0,
              "expiration": 0.0,
              "members": {
                  "file:/volumes/_nogroup/sub1/b1fcce76-3418-42dd-aa76-f9076d047dd3": {
                      "excluded": false,
                      "state": {
                          "name": "QUIESCING",
                          "age": 30.7
                      }
                  },
                  "file:/volumes/_nogroup/sub2/bc8f770e-7a43-48f3-aa26-d6d76ef98d3e": {
                      "excluded": false,
                      "state": {
                          "name": "QUIESCING",
                          "age": 0.0
                      }
                  },
                  "file:/volumes/_nogroup/sub3/24c4b57b-e249-4b89-b4fa-7a810edcd35b": {
                      "excluded": false,
                      "state": {
                          "name": "QUIESCING",
                          "age": 0.0
                      }
                  }
              }
          }
      }
  }

``--include`` 位是可选的，因为，如果在提供成员时没有指定操作，
那就认为是 "include" 操作。

正如我们所看到的，超时参数指定了我们准备给系统多少时间等这个集合到达
`QUIESCED` 状态。不过，由于新成员可以随时添加进有效集合中，
因此从集合创建时间开始算超时时间并不公平。因此，超时是按成员来跟踪的：
每个成员都有 `timeout` 秒数的时间进入静默状态，如果任何一个成员的静默时间\
超过了这一时间，整个集合就会被标记为 `TIMEDOUT` ，并释放暂停。

一旦集合进入 `QUIESCED` 状态，它就会启动过期计时器（ expiration timer ）。
该计时器是按整个集合跟踪的，而不是按每个成员。 `expiration` 秒数一过，
集合就会变成 `EXPIRED` （已过期）状态，除非主动操作去释放或取消。

可以向 `QUIESCED` （已静默）的集合添加新成员。在这种情况下，
它会回到 `QUIESCING` （正进入静默）状态，新成员会有自己的静默超时。
如果新成员成功，那么该集合将再次 `QUIESCED` ，过期计时器将重启。

.. warning:: 
   * 集合处于 `QUIESCING` 状态时， `expiration timer` （过期计时器）不会启动；
     当这个\ **集合**\ 变成 `QUIESCED` 状态后，它的值会重置成 `expiration` 属性的值。
   * `timeout` 不会影响已经处于 `QUIESCED` 状态的\ **成员**\ 。

await （等待）
~~~~~~~~~~~~~~
.. Awaiting

注意，上述命令都是非阻塞的。如果我们想等待静默集合达到 `QUIESCED` 状态，
就应该在某个点等待它。 ``--await`` 可以和其他参数一并送出，让系统知道我们的意图。

await 有两种类型： `quiesce await` 和 `release await` 。前者是默认的，
后者只有在参数里有 ``--release`` 时才能做到。为了避免混淆，当集合不是
`QUIESCING` 时，不允许发出 `quiesce await` 。同样地，无论是否加 await ，试图
``--release`` （释放）一个未进入 `QUIESCED` 状态的集合也会得到 ``EPERM`` 错误。
不过， `release await` 一个已释放的集合、或 `quiesce await` 一个已静默的集合，
都不是错误 -- 这些都是成功的未操作（ no-op ）。

由于集合在等着应用程序（命令加了 ``--await`` 参数）， await 操作可能会用它自己的\
错误掩盖成功的结果。一个很好的例子就是尝试 cancel-await （等着取消）一个集合：

.. prompt:: bash # auto

  # ceph fs quiesce fs1 --set-id set1 --cancel --await
  {
      // ...
      "sets": {
          "set1": {
              // ...
              "state": {
                  "name": "CANCELED",
                  "age": 0
              },
              // ...
          }
      }
  }
  Error EPERM: 

虽然对处于有效状态的集合， ``--cancel`` 会同步地成功，但不允许等待已经取消的集合，
因此这个调用会导致 ``EPERM`` 。这是有意与返回 ``EINVAL`` 错误
（表示用户方面出错）不同的，目的是简化请求 ``--await`` 时的系统行为。
这样，对于用户来说，这也是一个更简单的模型。

在等待时，用户可以指定这个等待请求的最长期限，与前文讨论过的两个超时一致。
如果在指定的期限内未达到期望等到的状态，则会返回 ``EINPROGRESS`` 。为此，
应该使用参数 ``--await-for=<seconds>`` 。我们可以认为 ``--await`` 相当于
``--await-for=Infinity`` （无限等待）。虽然同时指定这两个参数不合理，
但不会被当作错误。如果同时存在 ``--await`` 和 ``--await-for`` 参数，
那么前者将被忽略，而采纳 ``--await-for`` 的时间限制。

.. prompt:: bash # auto

  # time ceph fs quiesce fs1 sub1 --timeout=10 --await-for=2
  {
      "epoch": 6,
      "set_version": 3,
      "sets": {
          "c3c1d8de": {
              "version": 3,
              "age_ref": 0.0,
              "state": {
                  "name": "QUIESCING",
                  "age": 2.0
              },
              "timeout": 10.0,
              "expiration": 0.0,
              "members": {
                  "file:/volumes/_nogroup/sub1/b1fcce76-3418-42dd-aa76-f9076d047dd3": {
                      "excluded": false,
                      "state": {
                          "name": "QUIESCING",
                          "age": 2.0
                      }
                  }
              }
          }
      }
  }
  Error EINPROGRESS: 
  ceph fs quiesce fs1 sub1 --timeout=10 --await-for=2  0.41s user 0.04s system 17% cpu 2.563 total

（ Ceph 客户端会增加大约 0.5 秒的开销，至少在本地调试集群中如此）

静默-等待和过期（ Quiesce-Await and Expiration ）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

静默等待有个副作用：它会重置内部过期计时器。可以用看门狗方式处理\
长期运行的多步进程，在 IO 暂停状态下，通过重复等待（ ``--await`` ）\
已静默（ `QUIESCED` ）的集合。看下面的示例脚本：

.. prompt:: bash # auto

  # set -e   # (1)
  # ceph fs quiesce fs1 sub1 sub2 sub3 --timeout=30 --expiration=10 --set-id="snapshots" --await # (2)
  # ceph fs subvolume snapshot create a sub1 snap1-sub1  # (3)
  # ceph fs quiesce fs1 --set-id="snapshots" --await  # (4)
  # ceph fs subvolume snapshot create a sub2 snap1-sub2  # (3)
  # ceph fs quiesce fs1 --set-id="snapshots" --await  # (4)
  # ceph fs subvolume snapshot create a sub3 snap1-sub3  # (3)
  # ceph fs quiesce fs1 --set-id="snapshots" --release --await  # (5)

.. warning:: 本例用任意超时来阐述这一概念。现实中，
   这个数值必须根据实际的系统要求和规格谨慎选择。

脚本的目标是为 3 个子卷拍下一致的快照。
我们首先设置 bash 的 ``-e`` 选项 `(1)` ，
以便在后续命令返回非零状态码时退出脚本。

我们继续请求三个子卷的 IO 暂停 `(2)` 。我们设置了超时，
允许系统花最多 30 秒时间让所有成员进入静默状态，
并在静默到期和 IO 恢复之前保持静默状态最多 10 秒。
我们还指定了 ``--await`` ，只有在达到静默状态后才继续下一步。

然后，我们继续使用一组命令对、拍“下一个快照”，并在命令集中调用 ``--await`` ，
将过期超时时间再延长 10 秒 `(3,4)` 。这种方法为每个快照提供了最多 10 秒的时间，
而且还能在不失去 IO 暂停和保持一致性的情况下，根据需要拍摄任意数量的快照。
如果我们愿意，可以在每次调用了等待时更新过期时间（ `expiration` ）。

如果有快照卡住，需要 10 秒以上才能完成，那么下一次调用 ``--await`` 时就会\
返回错误，因为此集合会是过期状态（ `EXPIRED` ），不是可等待状态（ awaitable ）。
这就限制住了不利情况下对应用程序的影响。

我们本可以在 `(2)` 的一开始就把 `expiration` 超时设置为 30 ，
但这意味着一个卡住的快照会让应用程序在这段时间内一直处于等待状态。


If Version （判断版本）
~~~~~~~~~~~~~~~~~~~~~~~

有时，仅仅观察到成功的退出或释放还不够。
原因可能是另一个客户端同时更改了这个集合。看这个例子：

.. prompt:: bash # auto

  # ceph fs quiesce fs1 sub1 sub2 sub3 --timeout=30 --expiration=60 --set-id="snapshots" --await  # (1)
  # ceph fs subvolume snapshot create a sub1 snap1-sub1  # (2)
  # ceph fs subvolume snapshot create a sub2 snap1-sub2  # (3)
  # ceph fs subvolume snapshot create a sub3 snap1-sub3  # (4)
  # ceph fs quiesce fs1 --set-id="snapshots" --release --await  # (5)

顺序看起来没问题，释放 `(5)` 也成功完成了。但是，有可能在 sub3 `(4)` 的\
快照拍下之前，另一个会话把 sub3 从集合中排除掉了，恢复了它的 IO 。

.. prompt:: bash # auto

   # ceph fs quiesce fs1 --set-id="snapshots" --exclude sub3

由于从集合中删除成员不会影响它的 `QUIESCED` 状态，因此 release 命令 `(5)`
没理由失败。它会确认 sub1 和 sub2 这两个未被排除的成员，并报告成功。

为了解决这个问题或此类问题， quiesce 命令支持乐观并发模式。
要激活该模式，需要传递一个 ``--if-version=<version>`` 参数，
该参数将与集合的 db 版本进行比较，只有当数值匹配时，操作才会继续。
否则，命令不会执行，并返回状态 ``ESTALE`` 。

要知道一个集合应该期待的版本很容易，因为每条修改集合的命令都会在 stdout 上\
返回该集合，不管退出状态如何。在上面的示例中，我们可以看到，
每个集合都带一个 ``"version"`` 属性，每次修改这个集合，
无论是用户显式修改还是隐式修改，这个属性都会更新。

在本小节开头的示例中，初始 quiesce 命令 `(1)` 会返回新创建的集合，
其 id 为 ``"snapshots"`` ，版本为 13 。
由于用命令 `(2,3,4)` 拍快照时，我们不希望对集合做任何更改，
因此 release （释放）命令 `(5)` 可能是这样的：

.. prompt:: bash # auto

  # ceph fs quiesce fs1 --set-id="snapshots" --release --await --if-version=13 # (5)

这样， release 命令的结果就会是 ``ESTALE`` ，而不是 0 ，
我们就能知道 quiesce 集合不对劲，拍下的快照也可能不一致。

.. tip:: 当 ``--if-version`` 和命令返回 ``ESTALE`` 时，
   请求的操作\ **不会**\ 执行。这意味着脚本可能还需要对这个集合执行\
   某些无条件命令（ unconditional command ），以根据要求调整其状态。

对于自动化软件来说， ``--if-version`` 参数还有另一种用途。
正如我们前面所讨论的，可以用指定集合 id 创建个新的 quiesce 集合。
像用于 Kubernetes 的 CSI 这样的驱动程序，可以用其内部请求 ID ，
这样就无需维护与 quiesce 集合 ID 的额外映射。不过，为了保证唯一性，
驱动程序可能需要验证该集合确实是新的。为此，可以用 ``if-version=0`` ，
而且只有当数据库中没有这个集合 id 时，才会创建新集合。

.. prompt:: bash $ auto

  $ ceph fs quiesce fs1 sub1 sub2 sub3 --set-id="external-id" --if-version=0


.. _disabling-volumes-plugin:

禁用 volumes 插件
-----------------
.. Disabling Volumes Plugin

默认情况下， volumes 插件是启用的、且设置成了 ``always on`` （始终打开）。
但在某些情况下，禁用它可能更合适。例如，当 CephFS 处于降级状态时，
卷插件命令可能会堆积在 MGR 中，而不是顺利执行。
这最终会导致策略节流启动，而且 MGR 变得反应迟钝。

在这种情况下，可以禁用 volumes 插件，即便它在 MGR 中是始终开启的
（ ``always on`` ）模块。要禁用，可执行
``ceph mgr module disable volumes --yes-i-really-mean-it`` 命令。
注意，此命令会禁用 volume 插件的操作并删除 volumes 插件的命令，
因为它要禁用 Ceph 集群上、所有通过这个插件访问的 CephFS 服务。

在采取类似这样激烈的措施之前，最好先尝试不那么激烈的措施，
然后评估文件系统体验是否因此得到改善。类似措施比如，
禁用 volumes 插件启动的、用于克隆和清除垃圾的异步线程。
有关详情见 :ref:`暂停清除线程 <pause-purge-threads>` 和
:ref:`暂停克隆线程 <pause-clone-threads>`\ 。

.. note:: 直到 Tentacle ， CephFS 卷所属存储池命名空间的名字\
   都是按这个格式： "fsvolumens__<subvol-name>" 。然而，
   当两个同名子卷分别位于不同子卷组时，这可能导致命名空间冲突。
   因此，在 Tentacle 版以后，存储池命名格式改成了
   "fsvolumens__<subvol-grp-name>_<subvol-name>" 。


.. _manila: https://github.com/openstack/manila
.. _CSI: https://github.com/ceph/ceph-csi
