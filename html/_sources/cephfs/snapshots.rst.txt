===========
CephFS 快照
===========

CephFS 快照可创建一个不可变的文件系统视图，让它定格在快照拍下的那个时间点。
CephFS 支持快照，快照在名为 ``.snap`` 的特殊隐藏子目录里面管理。
快照是用 ``mkdir`` 在这个目录内创建的。

快照可以用不同的名字暴露出来，更改下列客户端配置：

- ``snapdirname`` 它是内核客户端的一个挂载选项
- ``client_snapdir`` 它是 ceph-fuse 的一个挂载选项

创建快照
========
.. Snapshot Creation

新文件系统默认启用了 CephFS 快照功能。
要在现有文件系统上启用，用下列命令。

.. code-block:: bash
    
    $ ceph fs set <fs_name> allow_new_snaps true

启用快照后， CephFS 内的所有目录都会有一个特殊的 ``.snap`` 目录。
（只要你想，还可以用客户端的 snapdir 选项配置不同的名字）。
要创建 CephFS 快照，只需在 ``.snap`` 下创建一个子目录，名字由你定。
例如，要给目录 ``/file1/`` 创建快照，调用 ``mkdir /file1/.snap/snapshot-name`` 。

.. code-block:: bash

    $ touch file1
    $ cd .snap
    $ mkdir my_snapshot

用快照恢复数据
==============
.. Using snapshot to recover data

快照也能用于恢复删掉的文件。

- ``创建文件 file1 ，然后创建快照 snap1``

.. code-block:: bash

    $ touch /mnt/cephfs/file1
    $ cd .snap
    $ mkdir snap1

- ``创建文件 file2 ，然后创建快照 snap2``

.. code-block:: bash

    $ touch /mnt/cephfs/file2
    $ cd .snap
    $ mkdir snap2

- ``删除 file1 ，然后创建新快照 snap3``

.. code-block:: bash

    $ rm /mnt/cephfs/file1
    $ cd .snap
    $ mkdir snap3

- ``恢复 file1 ，用 cp 命令和快照 snap2``

.. code-block:: bash

    $ cd .snap
    $ cd snap2
    $ cp file1 /mnt/cephfs/

删除快照
========
.. Snapshot Deletion

在目标目录里的 ``.snap`` 目录中调用 ``rmdir`` 可删除快照。
（如果一个目录的根里有快照，删除此目录会失败；
必须先删除快照）。

.. code-block:: bash

    $ cd .snap
    $ rmdir my_snapshot
