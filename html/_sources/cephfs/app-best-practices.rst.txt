.. Application best practices for distributed file systems

分布式文件系统之上的应用程序最佳实践
====================================

CephFS 与 POSIX 标准兼容的、所以它应该适用于使用 POSIX
文件系统的任意应用程序。然而，由于它是一个网络文件系统（不像\
诸如 XFS 的）并且是强一致性的（不像诸如 NFS 的），所以，\
应用程序作者了解一下某些后果是有好处的。

下面几节描述了分布式文件系统与本地文件系统在性能表现上有明显\
差异的一些地方。


ls -l
-----
当你运行 ``ls -l`` 命令时， ``ls`` 程序会先罗列目录、然后对\
其内每一个文件调用一次 ``stat`` 。

这通常远超一个应用程序的真实需求，而且在大目录上会很慢。如果\
你不是真的需要各文件的所有此类元数据，那就只用一个 ``ls`` 吧。


ls/stat on files being extended
-------------------------------

If another client is currently extending files in the listed directory,
then an ``ls -l`` may take an exceptionally long time to complete, as
the lister must wait for the writer to flush data in order to do a valid
read of the every file's size.  So unless you *really* need to know the
exact size of every file in the directory, just don't do it!

This would also apply to any application code that was directly
issuing ``stat`` system calls on files being appended from
another node.


.. Very large directories

巨型目录
--------

Do you really need that 10,000,000 file directory?  While directory
fragmentation enables CephFS to handle it, it is always going to be
less efficient than splitting your files into more modest-sized directories.

Even standard userspace tools can become quite slow when operating on very
large directories. For example, the default behaviour of ``ls``
is to give an alphabetically ordered result, but ``readdir`` system
calls do not give an ordered result (this is true in general, not just
with CephFS).  So when you ``ls`` on a million file directory, it is
loading a list of a million names into memory, sorting the list, then writing
it out to the display.


.. Hard links

硬链接
------

Hard links have an intrinsic cost in terms of the internal housekeeping
that a file system has to do to keep two references to the same data.  In
CephFS there is a particular performance cost, because with normal files
the inode is embedded in the directory (i.e. there is no extra fetch of
the inode after looking up the path).


Working set size
----------------

The MDS acts as a cache for the metadata stored in RADOS.  Metadata
performance is very different for workloads whose metadata fits within
that cache.

If your workload has more files than fit in your cache (configured using 
``mds_cache_memory_limit`` settings), then make sure you test it
appropriately: don't test your system with a small number of files and then
expect equivalent performance when you move to a much larger number of files.


.. Do you need a file system?

你真的需要文件系统么？
----------------------

Remember that Ceph also includes an object storage interface.  If your
application needs to store huge flat collections of files where you just
read and write whole files at once, then you might well be better off
using the :ref:`Object Gateway <object-gateway>`
