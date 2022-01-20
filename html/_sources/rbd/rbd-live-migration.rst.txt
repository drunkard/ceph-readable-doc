==============
 映像在线迁移
==============
.. Image Live-Migration

.. index:: Ceph Block Device; live-migration

在同一集群内， RBD 映像可以在不同存储池之间在线迁移、
在不同映像格式和布局之间、或者从外部数据源迁移。
启动后，源映像会被深度复制到目标映像，
会拉取所有快照历史，同时尽可能保留稀疏分配的数据。

默认情况下，如果在线迁移的 RBD 映像们位于同一集群内，
源映像会被标记为只读，并且所有客户端会把 IO 重定向到新的目标映像。
另外，这个模式可以选择性地保留它到源映像的父映像的链接，
以保留稀疏数据块；或者可以在迁移映像时拍平映像，
以去除到源映像的父映像的依赖关系。

在线迁移进程还可以用在纯导入模式，
此时，源映像保持不变，而目标映像可以被链接到外部数据源，
像备份的文件、 HTTP(S) 文件、或者 S3 对象。

新的目标映像正在被使用时，
在线迁移复制进程可以安全地在后台运行。
当前，如果不是用纯导入模式操作，
在准备迁移前需要先暂时停止源映像的使用。
这是为了帮助确保客户端使用的映像和新的目标映像位于同一个更新点。

.. note::
   映像的在线迁移需要 Ceph Nautilus 或更高版本。
   在 Ceph Pacific 或更高版本才支持外部数据源。
   ``krbd`` 内核模块现在还不支持在线迁移。


.. ditaa::

           +-------------+               +-------------+
           | {s} c999    |               | {s}         |
           |  Live       | Target refers |  Live       |
           |  migration  |<-------------*|  migration  |
           |  source     |  to Source    |  target     |
           |             |               |             |
           | (read only) |               | (writable)  |
           +-------------+               +-------------+

               Source                        Target


在线迁移过程分三步：

#. **准备迁移（ Prepare Migration ）:** 初始步骤，
   会创建新的目标映像，并链接到源映像。
   如果没配置成纯导入模式，源映像还会链接到目标映像，
   并标记为只读。

   与 `分层映像`_ 相似，尝试读取目标映像内未初始化的数据区时，
   会在程序内部把它重定向到源映像，
   而写入目标映像内未初始化的数据区时，
   程序内部会把重叠的源映像块深度复制到目标映像。


#. **执行迁移（ Execute Migration ）:** 这是后台操作，
   会把所有初始化的数据块从源映像深度复制到目标映像。
   这一步可以在客户端们正使用新目标映像时运行。


#. **完成迁移（ Finish Migration ）:**
   后台迁移过程完成后，迁移操作可以提交或中止。
   提交本次迁移会去除源和目标映像之间的链接，
   如果不是在纯导入模式下，还会删除源映像。
   中止迁移会删除交叉链接、并删除目标映像。


准备迁移事宜
============
.. Prepare Migration

在同一一个 Ceph 集群内，映像的默认在线迁移过程由
`rbd migration prepare` 命令初始化，需提供源和目的映像： ::

    $ rbd migration prepare migration_source [migration_target]

`rbd migration prepare` 命令支持的布局可选参数和
`rbd create` 命令的一样，可以用来更改不可变对象的磁盘布局。
如果只是为了更改磁盘上的布局，可以跳过 `migration_target` ，
还沿用最初的映像名。

在准备在线迁移前，所有使用这个源映像的客户端都必须先停止。
如果发现有客户端打开了客户端，不管是读还是写模式，
准备这步都会失败。准备步骤完成后，
客户端们可以用新的目标映像名重启。
仍然尝试用源映像名重启的客户端会失败。

`rbd status` 命令可以显示在线迁移当前的状态： ::

    $ rbd status migration_target
    Watchers: none
    Migration:
        	source: rbd/migration_source (5e2cba2f62e)
        	destination: rbd/migration_target (5e2ed95ed806)
        	state: prepared

注意，在迁移进程运行期间，源映像会被挪到 RBD 垃圾桶，
以免被误用： ::

    $ rbd info migration_source
    rbd: error opening image migration_source: (2) No such file or directory
    $ rbd trash ls --all
    5e2cba2f62e migration_source


为单纯导入的迁移做准备
======================
.. Prepare Import-Only Migration

单纯导入（ import-only ）在线迁移进程同样是用 `rbd migration prepare` 命令\
初始化的，但是得加上 `--import-only` 选项，
还得提供一个 JSON 编码的 ``source-spec`` ，
用以描述如何访问源映像数据。这份 ``source-spec``
可以直接用 `--source-spec` 选项传入、
或者通过 `--source-spec-path` 选项传入一个文件或 STDIN::

        $ rbd migration prepare --import-only --source-spec "<JSON>" migration_target

`rbd migration prepare` 命令支持与 `rbd create` 命令相同的\
所有布局选项。

`rbd status` 命令可以显示在线迁移当前的状态： ::

        $ rbd status migration_target
        Watchers: none
        Migration:
	        source: {"stream":{"file_path":"/mnt/image.raw","type":"file"},"type":"raw"}
        	destination: rbd/migration_target (ac69113dc1d7)
	        state: prepared

``source-spec`` JSON 文件的通用格式如下： ::

        {
            "type": "<format-type>",
            <format unique parameters>
            "stream": {
                "type": "<stream-type>",
                <stream unique parameters>
            }
        }

当前支持这几种格式： ``native`` 、 ``qcow`` 、和 ``raw`` 。
当前支持这几种流格式： ``file`` 、 ``http`` 、和 ``s3`` 。

格式
~~~~
.. Formats

``native`` 格式用于描述与源映像位于同一集群内的本地 RBD 映像，
它的 ``source-spec`` JSON 编码成这样： ::

        {
            "type": "native",
            "pool_name": "<pool-name>",
            ["pool_id": <pool-id>,] (optional alternative to "pool_name")
            ["pool_namespace": "<pool-namespace",] (optional)
            "image_name": "<image-name>",
            ["image_id": "<image-id>",] (optional if image in trash)
            "snap_name": "<snap-name>",
            ["snap_id": "<snap-id>",] (optional alternative to "snap_name")
        }

需要注意的是， ``native`` 格式不包含 ``stream`` 对象，因为它利用的是
Ceph 的本地操作。例如，要从 ``rbd/ns1/image1@snap1`` 映像导入，
对应的 ``source-spec`` 应该是： ::

        {
            "type": "native",
            "pool_name": "rbd",
            "pool_namespace": "ns1",
            "image_name": "image1",
            "snap_name": "snap1"
        }


``qcow`` 格式可以用于描述 QCOW (QEMU copy-on-write) 块设备。
当前， QCOW (v1) 和 QCOW2 格式都支持，除了一些高级功能，
如压缩、加密、 backing files 、和外部数据文件。
这些缺失的功能可能在以后的版本中加上。
``qcow`` 格式的数据可以被链接到下文描述的任意 stream 源。
例如，它的基础 ``source-spec`` JSON 编码成这样： ::

        {
            "type": "qcow",
            "stream": {
                <stream unique parameters>
            }
        }


``raw`` 格式可以用于描述一个完备配置的、 raw 块设备导出
（即 `rbd export --export-format 1 <snap-spec>` ）。
``raw`` 格式的数据可以被链接到下文描述的任意 stream 源。
例如，它的基础 ``source-spec`` JSON 编码成这样： ::

        {
            "type": "raw",
            "stream": {
                <stream unique parameters for HEAD, non-snapshot revision>
            },
            "snapshots": [
                {
                    "type": "raw",
                    "name": "<snapshot-name>",
                    "stream": {
                        <stream unique parameters for snapshot>
                    }
                },
            ] (可选，按最老到最新排序的快照)
        }

``snapshots`` 数组是可选的，而且当前\
仅支持全配 ``raw`` 快照的导出。

其它格式，像 RBD export-format v2 和
RBD export-diff 快照会在未来版本中增加。

流格式
~~~~~~
.. Streams

``file`` 流可以用于从一个本地可访问的 POSIX 文件源导入。
它的 ``source-spec`` JSON 编码成这样： ::

        {
            <format unique parameters>
            "stream": {
                "type": "file",
                "file_path": "<file-path>"
            }
        }

例如，要导入一个位于 /mnt/image.raw 、格式为 raw 的映像，
它的 ``source-spec`` JSON 编码成这样： ::

        {
            "type": "raw",
            "stream": {
                "type": "file",
                "file_path": "/mnt/image.raw"
            }
        }

``http`` 流可以用于从远端 HTTP 或 HTTPS web 服务器导入。
它的 ``source-spec`` JSON 编码成这样： ::

        {
            <format unique parameters>
            "stream": {
                "type": "http",
                "url": "<url-path>"
            }
        }

例如，要导入一个位于 ``http://download.ceph.com/image.raw`` 、
格式为 raw 的映像，它的 ``source-spec`` JSON 编码成这样： ::

        {
            "type": "raw",
            "stream": {
                "type": "http",
                "url": "http://download.ceph.com/image.raw"
            }
        }

``s3`` 流可以用于从一个远端 S3 桶导入。
它的 ``source-spec`` JSON 编码成这样： ::

        {
            <format unique parameters>
            "stream": {
                "type": "s3",
                "url": "<url-path>",
                "access_key": "<access-key>",
                "secret_key": "<secret-key>"
            }
        }

例如，要导入一个位于 `http://s3.ceph.com/bucket/image.raw` 、
格式为 raw 的映像，它的 ``source-spec`` JSON 编码成这样： ::

        {
            "type": "raw",
            "stream": {
                "type": "s3",
                "url": "http://s3.ceph.com/bucket/image.raw",
                "access_key": "NX5QOQKC6BH2IDN8HC7A",
                "secret_key": "LnEsqNNqZIpkzauboDcLXLcYaWwLQ3Kop0zAnKIn"
            }
        }

.. note::
   ``access_key`` 和 ``secret_key`` 参数可以利用 MON 的 config-key 存储库，
   引用时加 ``config://`` 前缀、然后是那个值在 MON config-key 库里的路径。
   存入 config-key 库时可以用 ``ceph config-key set <key-path> <value>`` 命令，
   （例如 ``ceph config-key set rbd/s3/access_key NX5QOQKC6BH2IDN8HC7A`` ）。


执行迁移
========
.. Execute Migration

准备好在线迁移后，源映像里的数据块必须复制到目标映像。
要运行 `rbd migration execute` 命令来完成： ::

    $ rbd migration execute migration_target
    Image migration: 100% complete...done.

`rbd status` 命令可以反馈迁移数据块的深度复制进度： ::

    $ rbd status migration_target
    Watchers:
    	watcher=1.2.3.4:0/3695551461 client.123 cookie=123
    Migration:
        	source: rbd/migration_source (5e2cba2f62e)
        	destination: rbd/migration_target (5e2ed95ed806)
        	state: executing (32% complete)


提交迁移
========
.. Commit Migration

在线迁移把所有数据块深度复制到目标映像之后，就可以提交这个迁移了： ::

    $ rbd status migration_target
    Watchers: none
    Migration:
        	source: rbd/migration_source (5e2cba2f62e)
        	destination: rbd/migration_target (5e2ed95ed806)
        	state: executed
    $ rbd migration commit migration_target
    Commit image migration: 100% complete...done.

如果 `migration_source` 映像是一个或多个克隆品的父映像，
在确保所有派生出的映像都不在使用之后，需要加上 `--force` 选项。

提交在线迁移后，源和目标映像之间的交叉链接会被删除，
还会删除源映像： ::

    $ rbd trash list --all


中止迁移
========
.. Abort Migration

如果你想回退准备或执行步骤，可以用 `rbd migration abort` 命令\
来回退迁移进程： ::

        $ rbd migration abort migration_target
        Abort image migration: 100% complete...done.

中止迁移进程后，目标映像会被删除、
到源映像的访问会恢复： ::

        $ rbd ls
        migration_source


.. _分层映像: ../rbd-snapshot/#layering
