文件布局
========

文件布局可控制如何把文件内容映射到各 Ceph RADOS 对象，你可以用\ \
*虚拟扩展属性*\ 或 xattrs 来读、写某一文件的布局。

布局 xattrs 的名字取决于此文件是常规文件还是目录，常规文件的布局 xattrs 叫\
作 ``ceph.file.layout`` 、目录的布局 xattrs 叫作 ``ceph.dir.layout`` 。因此\
后续实例中若用的是 ``ceph.file.layout`` ，处理目录时就要替换为 ``dir`` 。

.. tip::

    你的 Linux 发行版也许默认没提供操作 xattrs 的命令，所需软件包通常是 \
    ``attr`` 。


布局字段
--------

pool
    字符串，可指定 ID 或名字。它是文件的数据对象所在的 RADOS 存储池。

stripe_unit
    字节数、整数。一个文件的数据块按照此尺寸（字节）像 RAID 0 一样分布。一文\
    件所有条带单元的尺寸一样，最后一个条带单元通常不完整——即它包含文件末尾的\
    数据、还有数据末端到固定条带单元尺寸之间的未使用“空间”。

stripe_count
    整数。组成 RAID 0 “条带”数据的连续条带单元数量。

object_size
    整数个字节。文件数据按此尺寸分块为 RADOS 对象。


用 ``getfattr`` 读取布局
------------------------

读出的布局信息表示为单个字符串：

.. code-block:: bash

    $ touch file
    $ getfattr -n ceph.file.layout file
    # file: file
    ceph.file.layout="stripe_unit=4194304 stripe_count=1 object_size=4194304 pool=cephfs_data"

读取单个布局字段：

.. code-block:: bash

    $ getfattr -n ceph.file.layout.pool file
    # file: file
    ceph.file.layout.pool="cephfs_data"
    $ getfattr -n ceph.file.layout.stripe_unit file
    # file: file
    ceph.file.layout.stripe_unit="4194304"
    $ getfattr -n ceph.file.layout.stripe_count file
    # file: file
    ceph.file.layout.stripe_count="1"
    $ getfattr -n ceph.file.layout.object_size file
    # file: file
    ceph.file.layout.object_size="4194304"

.. note::

    读取布局时，存储池通常是以名字标识的。然而在极少数情况下，如存储池刚创建\
    时，可能会输出 ID 。

目录只有经过定制才会有显式的布局，如果从没更改过，那么读取其布局时就会失败：\
这表明它会继承父目录的显式布局设置。

.. code-block:: bash

    $ mkdir dir
    $ getfattr -n ceph.dir.layout dir
    dir: ceph.dir.layout: No such attribute
    $ setfattr -n ceph.dir.layout.stripe_count -v 2 dir
    $ getfattr -n ceph.dir.layout dir
    # file: dir
    ceph.dir.layout="stripe_unit=4194304 stripe_count=2 object_size=4194304 pool=cephfs_data"


用 ``setfattr`` 设置布局
------------------------

布局字段可用 ``setfattr`` 修改：

.. code-block:: bash

    $ ceph osd lspools
    0 rbd,1 cephfs_data,2 cephfs_metadata,

    $ setfattr -n ceph.file.layout.stripe_unit -v 1048576 file2
    $ setfattr -n ceph.file.layout.stripe_count -v 8 file2
    $ setfattr -n ceph.file.layout.object_size -v 10485760 file2
    $ setfattr -n ceph.file.layout.pool -v 1 file2  # Setting pool by ID
    $ setfattr -n ceph.file.layout.pool -v cephfs_data file2  # Setting pool by name


布局的继承
----------

文件会在创建时继承其父目录的布局，然而之后对父目录布局的更改不会影响其子孙。

.. code-block:: bash

    $ getfattr -n ceph.dir.layout dir
    # file: dir
    ceph.dir.layout="stripe_unit=4194304 stripe_count=2 object_size=4194304 pool=cephfs_data"

    # 证实 file1 继承了其父的布局
    $ touch dir/file1
    $ getfattr -n ceph.file.layout dir/file1
    # file: dir/file1
    ceph.file.layout="stripe_unit=4194304 stripe_count=2 object_size=4194304 pool=cephfs_data"

    # 现在更改目录布局，然后再创建第二个文件
    $ setfattr -n ceph.dir.layout.stripe_count -v 4 dir
    $ touch dir/file2

    # 证实 file1 的布局未变
    $ getfattr -n ceph.file.layout dir/file1
    # file: dir/file1
    ceph.file.layout="stripe_unit=4194304 stripe_count=2 object_size=4194304 pool=cephfs_data"

    # 但 file2 继承了父目录的新布局
    $ getfattr -n ceph.file.layout dir/file2
    # file: dir/file2
    ceph.file.layout="stripe_unit=4194304 stripe_count=4 object_size=4194304 pool=cephfs_data"

如果中层目录没有设置布局，那么内层目录中创建的文件也会继承此目录的布局：

.. code-block:: bash

    $ getfattr -n ceph.dir.layout dir
    # file: dir
    ceph.dir.layout="stripe_unit=4194304 stripe_count=4 object_size=4194304 pool=cephfs_data"
    $ mkdir dir/childdir
    $ getfattr -n ceph.dir.layout dir/childdir
    dir/childdir: ceph.dir.layout: No such attribute
    $ touch dir/childdir/grandchild
    $ getfattr -n ceph.file.layout dir/childdir/grandchild
    # file: dir/childdir/grandchild
    ceph.file.layout="stripe_unit=4194304 stripe_count=4 object_size=4194304 pool=cephfs_data"


把数据存储池加入 MDS
--------------------

要把存储池当 CephFS 用，你必须把它加入元数据服务器。

.. code-block:: bash

    $ ceph mds add_data_pool cephfs_data_ssd
    # 现在应该能看到存储池了
    $ ceph fs ls
    .... data pools: [cephfs_data cephfs_data_ssd ]

确保你的 cephx 密钥允许客户端访问新存储池。
