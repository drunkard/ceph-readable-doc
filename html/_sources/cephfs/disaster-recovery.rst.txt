.. Disaster recovery
.. _cephfs-disaster-recovery:

灾难恢复
========


.. Metadata damage and repair

元数据损坏及其修复
------------------

If a file system has inconsistent or missing metadata, it is considered
*damaged*.  You may find out about damage from a health message, or in some
unfortunate cases from an assertion in a running MDS daemon.

Metadata damage can result either from data loss in the underlying RADOS
layer (e.g. multiple disk failures that lose all copies of a PG), or from
software bugs.

CephFS includes some tools that may be able to recover a damaged file system,
but to use them safely requires a solid understanding of CephFS internals.
The documentation for these potentially dangerous operations is on a
separate page: :ref:`disaster-recovery-experts`.


.. Data pool damage (files affected by lost data PGs)

数据存储池损坏（受数据 PG 丢失影响的文件）
------------------------------------------

If a PG is lost in a *data* pool, then the file system will continue
to operate normally, but some parts of some files will simply
be missing (reads will return zeros).

丢失一个数据 PG 会影响很多文件。文件被拆分成了很多对象，所以要\
找出哪些文件受这些 PG 丢失的影响，需要扫描所有的对象 ID 以确认\
文件的副本数合格。这种扫描有助于找出哪些文件需要从备份恢复。

.. danger::

   这个命令不会修复任何元数据，所以在恢复文件时，你必须\ *删除*\
   损坏的文件，然后再替换它，这样才会获得完好的 inode 。不要\
   原地替换损坏的文件。

如果你已经知道了哪些 PG 丢失了对象，可以用 ``pg_files`` 子命令\
扫描可能损坏的文件，命令为： ::

    cephfs-data-scan pg_files <path> <pg id> [<pg id>...]

例如，假设你已经知道 PG 1.4 和 4.5 有数据丢失，然后你想知道
/home/bob 下面的哪些文件可能损坏了： ::

    cephfs-data-scan pg_files /home/bob 1.4 4.5

输出是可能损坏的文件路径列表，每行一个。

请注意，此命令是作为一个普通的 CephFS 客户端来搜寻文件系统内的\
所有文件、并读取它们的布局的，所以 MDS 必须是正常运行的。

