.. _librados-python:

===================
 Librados (Python)
===================

``rados`` 模块是 ``librados`` 的 Python 瘦封装。

安装
====

要安装 Ceph 的 Python 库，看看 `获取 librados 的 Python 接口`_ 。


开工
====

你可以用 Python 实现你自己的 Ceph 客户端。
下面的教程会向你展示如何导入 Ceph 的 Python 门口、
连接到集群、并以 ``client.admin`` 身份执行对象操作。

.. note:: 要使用 Ceph 的 Python 库，你必须有正常运行的 Ceph 集群、
   及其访问权限。要快速配起一个集群，看 `入门手册`_ 。

首先创建一个 Ceph 客户端的 Python 源文件。

.. prompt:: bash

   sudo vim client.py


导入模块
--------

要使用 ``rados`` 模块，需在源码文件里导入。

.. code-block:: python
   :linenos:

    import rados


配置集群句柄
------------
.. Configure a Cluster Handle

连接 Ceph 存储集群前，得先创建集群句柄。默认情况下，
集群句柄假设集群名为 ``ceph`` （即部署工具的默认值，
而且我们的 `入门手册`_ 也是）、用户名为 ``client.admin`` 。
你可以根据自己的环境更改默认值。

要连接到 Ceph 存储集群，你的应用程序需要知道去哪里找到 Ceph 监视器。
给你的应用程序指定 Ceph 配置文件的路径即可提供此信息，
配置文件里有初始的 Ceph 监视器位置。

.. code-block:: python
   :linenos:

    import rados, sys

    #Create Handle Examples.
    cluster = rados.Rados(conffile='ceph.conf')
    cluster = rados.Rados(conffile=sys.argv[1])
    cluster = rados.Rados(conffile = 'ceph.conf', conf = dict (keyring = '/path/to/keyring'))

确保 ``conffile`` 参数提供了 Ceph 配置文件的路径和文件名，
你可以用 ``sys`` 模块避免写死 Ceph 配置文件路径和文件名。

你的 Python 客户端还需要一个客户端密钥环，在本例中，\
我们默认使用 ``client.admin`` 的密钥。
如果你想在创建集群句柄的时候指定密钥环，可以用 ``conf`` 参数。
另外，你也可以在 Ceph 配置文件中指定密钥环文件的路径。例如，
你可以在 Ceph 配置文件里加上类似如下的： ::

    keyring = /path/to/ceph.client.admin.keyring

通过 Python 修改配置的额外细节见\  `配置`_\ 。


连接到集群
----------
.. Connect to the Cluster

配置好集群句柄后，你就可以连接到集群了。通过到集群的连接，
你可以执行库的各种方法来获取集群信息。

.. code-block:: python
   :linenos:
   :emphasize-lines: 7

    import rados, sys

    cluster = rados.Rados(conffile='ceph.conf')
    print("\nlibrados version: {}".format(str(cluster.version())))
    print("Will attempt to connect to: {}".format(str(cluster.conf_get('mon host'))))

    cluster.connect()
    print("\nCluster ID: {}".format(cluster.get_fsid()))

    print("\n\nCluster Statistics")
    print("==================")
    cluster_stats = cluster.get_cluster_stats()

    for key, value in cluster_stats.items():
        print(key, value)

默认情况下， Ceph 的认证的 ``开启的`` ，应用程序需要知道密钥环的位置。
``python-ceph`` 模块没有默认位置，所以你得指定密钥环路径。
指定密钥环最简单的方法就是写入 Ceph 配置文件，
下面的 Ceph 配置文件实例用的就是 ``client.admin`` 密钥环。

.. code-block:: ini
   :linenos:

    [global]
    # ... elided configuration
    keyring=/path/to/keyring/ceph.client.admin.keyring


管理存储池
----------
.. Manage Pools

连接到集群后，你可以用 ``Rados`` API 管理存储池。
你可以罗列存储池、检查一个存储池是否存在、新建存储池\
和删除存储池。

.. code-block:: python
   :linenos:
   :emphasize-lines: 6, 13, 18, 25

    print("\n\nPool Operations")
    print("===============")

    print("\nAvailable Pools")
    print("----------------")
    pools = cluster.list_pools()

    for pool in pools:
        print(pool)

    print("\nCreate 'test' Pool")
    print("------------------")
    cluster.create_pool('test')

    print("\nPool named 'test' exists: {}".format(str(cluster.pool_exists('test'))))
    print("\nVerify 'test' Pool Exists")
    print("-------------------------")
    pools = cluster.list_pools()

    for pool in pools:
        print(pool)

    print("\nDelete 'test' Pool")
    print("------------------")
    cluster.delete_pool('test')
    print("\nPool named 'test' exists: {}".format(str(cluster.pool_exists('test'))))


输入/输出上下文
---------------
.. Input/Output Context

读出或写入 Ceph 存储集群需要有输入输出上下文（ ioctx ）。
你可以用 ``Rados`` 类的 ``open_ioctx()`` 或 ``open_ioctx2()`` 方法创建 ioctx 。
``ioctx_name`` 参数是存储池的名字、
``pool_id`` 是你想要使用的存储池的 ID 。

.. code-block:: python
   :linenos:

    ioctx = cluster.open_ioctx('data')


或者

.. code-block:: python
   :linenos:

        ioctx = cluster.open_ioctx2(pool_id)


拿到 I/O 上下文之后，你就能读写对象、扩展属性、
以及其他很多操作了。完成操作后，
必须关闭连接。例如：

.. code-block:: python
   :linenos:

    print("\nClosing the connection.")
    ioctx.close()


对象的写入、读取和删除
----------------------
.. Writing, Reading and Removing Objects

创建 I/O 上下文之后，你就能向集群写入对象了。如果你向一个不存在的对象写入，
Ceph 会创建它。如果你向一个已存在的对象写入， Ceph 会覆盖它
（除非你指定了一个范围，那么它就只覆盖那段范围）。
你可以从集群读出对象（和对象的某一段）。也能从集群删除对象。例如：

.. code-block:: python
    :linenos:
    :emphasize-lines: 2, 5, 8

    print("\nWriting object 'hw' with contents 'Hello World!' to pool 'data'.")
    ioctx.write_full("hw", "Hello World!")

    print("\n\nContents of object 'hw'\n------------------------\n")
    print(ioctx.read("hw"))

    print("\nRemoving object 'hw'")
    ioctx.remove_object("hw")


XATTRS 的读取和写入
-------------------
.. Writing and Reading XATTRS

创建对象后，你可以向对象写入扩展属性（ XATTR ）、还能从对象读出 XATTR 。例如：

.. code-block:: python
    :linenos:
    :emphasize-lines: 2, 5

    print("\n\nWriting XATTR 'lang' with value 'en_US' to object 'hw'")
    ioctx.set_xattr("hw", "lang", "en_US")

    print("\n\nGetting XATTR 'lang' from object 'hw'\n")
    print(ioctx.get_xattr("hw", "lang"))


罗列对象
--------
.. Listing Objects

如果你想检查存储池内的对象们，你可以检索出这些对象，
并通过对象迭代器挨个迭代出来。例如：

.. code-block:: python
    :linenos:
    :emphasize-lines: 1, 6, 7

    object_iterator = ioctx.list_objects()

    while True :

        try :
            rados_object = object_iterator.__next__()
            print("Object contents = {}".format(rados_object.read()))

        except StopIteration :
            break

    # Or alternatively
    [print("Object contents = {}".format(obj.read())) for obj in ioctx.list_objects()]

``Object`` 类提供了类似文件的接口让你访问对象，
可以读写内容和扩展属性。对象操作使用 I/O 上下文，
它还提供了额外功能和异步能力。


集群句柄 API
============
.. Cluster Handle API

``Rados`` 类提供了一个进入 Ceph 存储守护进程的接口。


配置
----
.. Configuration

``Rados`` 类提供了一些方法，可以获取和设置配置选项的值、
读取 Ceph 配置文件、并分析参数。
无需连接到 Ceph 存储集群就能调用下面的方法。有关配置的细节见 `存储集群配置`_ 。

.. currentmodule:: rados
.. automethod:: Rados.conf_get(option)
.. automethod:: Rados.conf_set(option, val)
.. automethod:: Rados.conf_read_file(path=None)
.. automethod:: Rados.conf_parse_argv(args)
.. automethod:: Rados.version()   


连接管理
--------
.. Connection Management

配置好集群句柄后，你就能连接集群了，检查集群的 ``fsid`` 、检出集群统计信息、
然后断开（关闭）到集群的连接。你也可以断言集群句柄处于某种状态
（例如 configuring 、 connecting 等等）。

.. automethod:: Rados.connect(timeout=0)
.. automethod:: Rados.shutdown()
.. automethod:: Rados.get_fsid()
.. automethod:: Rados.get_cluster_stats()

.. documented manually because it raises warnings because of *args usage in the
.. signature

.. py:class:: Rados

   .. py:method:: require_state(\*args)

      检查一下 Rados 对象是否处于指定状态。

      :param args: Any number of states to check as separate arguments
      :raises: :class:`RadosStateError`


存储池操作
----------
.. Pool Operations

要使用存储池操作方法，你必须先连接到 Ceph 存储集群。你可以列出可用存储池、
创建一个存储池、检验一个存储池是否存在、还能删除存储池。

.. automethod:: Rados.list_pools()
.. automethod:: Rados.create_pool(pool_name, crush_rule=None)
.. automethod:: Rados.pool_exists()
.. automethod:: Rados.delete_pool(pool_name)


CLI 命令
--------
.. CLI Commands

Ceph CLI 命令的内部用的是下面的 librados Python 绑定的方法。

要发出一个命令，选对方法和正确的目标。

.. automethod:: Rados.mon_command
.. automethod:: Rados.osd_command
.. automethod:: Rados.mgr_command
.. automethod:: Rados.pg_command


输入/输出上下文 API
===================
.. Input/Output Context API

要向 Ceph 对象存储系统写入和读出数据，必须创建一个输入/输出上下文（ ioctx ）。
`Rados` 类提供了 `open_ioctx()` 和 `open_ioctx2()` 方法。
其余 ``ioctx`` 操作包括 `Ioctx` 和其它类的调用方法。

.. automethod:: Rados.open_ioctx(ioctx_name)
.. automethod:: Ioctx.require_ioctx_open()
.. automethod:: Ioctx.get_stats()
.. automethod:: Ioctx.get_last_version()
.. automethod:: Ioctx.close()


.. Pool Snapshots
.. --------------

.. The Ceph Storage Cluster allows you to make a snapshot of a pool's state.
.. Whereas, basic pool operations only require a connection to the cluster, 
.. snapshots require an I/O context.

.. Ioctx.create_snap(self, snap_name)
.. Ioctx.list_snaps(self)
.. SnapIterator.next(self)
.. Snap.get_timestamp(self)
.. Ioctx.lookup_snap(self, snap_name)
.. Ioctx.remove_snap(self, snap_name)

.. not published. This doesn't seem ready yet.


对象操作
--------
.. Object Operations

Ceph 存储集群把数据存储成了对象。你可以同步或异步地读写对象。
你可以从某个偏移量开始读写，每个对象有一个名字（键名）和数据。

.. automethod:: Ioctx.aio_write(object_name, to_write, offset=0, oncomplete=None, onsafe=None)
.. automethod:: Ioctx.aio_write_full(object_name, to_write, oncomplete=None, onsafe=None)
.. automethod:: Ioctx.aio_append(object_name, to_append, oncomplete=None, onsafe=None)
.. automethod:: Ioctx.write(key, data, offset=0)
.. automethod:: Ioctx.write_full(key, data)
.. automethod:: Ioctx.aio_flush()
.. automethod:: Ioctx.set_locator_key(loc_key)
.. automethod:: Ioctx.aio_read(object_name, length, offset, oncomplete)
.. automethod:: Ioctx.read(key, length=8192, offset=0)
.. automethod:: Ioctx.stat(key)
.. automethod:: Ioctx.trunc(key, size)
.. automethod:: Ioctx.remove_object(key)


对象的扩展属性
--------------
.. Object Extended Attributes

你可以给一个对象设置扩展属性（ XATTR ）。你可以检出一些对象或扩展属性、并挨个迭代它们。

.. automethod:: Ioctx.set_xattr(key, xattr_name, xattr_value)
.. automethod:: Ioctx.get_xattrs(oid)
.. automethod:: XattrIterator.__next__()
.. automethod:: Ioctx.get_xattr(key, xattr_name)
.. automethod:: Ioctx.rm_xattr(key, xattr_name)


对象接口
========
.. Object Interface

从一个 I/O 上下文，可以从一个存储池检出一些对象、并挨个迭代它们。
对象接口使得各个对象看起来像文件，你可以在对象上执行同步操作。
异步操作你应该用 I/O 上下文方法。

.. automethod:: Ioctx.list_objects()
.. automethod:: ObjectIterator.__next__()
.. automethod:: Object.read(length = 1024*1024)
.. automethod:: Object.write(string_to_write)
.. automethod:: Object.get_xattrs()
.. automethod:: Object.get_xattr(xattr_name)
.. automethod:: Object.set_xattr(xattr_name, xattr_value)
.. automethod:: Object.rm_xattr(xattr_name)
.. automethod:: Object.stat()
.. automethod:: Object.remove()




.. _入门手册: ../../../start
.. _存储集群配置: ../../configuration
.. _获取 librados 的 Python 接口: ../librados-intro#getting-librados-for-python
