=====================
 与 POSIX 标准的差异
=====================

CephFS 尽可能地严格遵循 POSIX 语义，比如，与其他很多网络文件\
系统（如 NFS ）相反， CephFS 与各个客户端之间维持着缓存的强一\
致性。目的就是为了让不同主机上的进程通过这个文件系统通讯时，\
可以表现得与本机进程间的通讯一样。

然而，CephFS 确实由于各种原因在某些地方偏离了严谨的 POSIX
语义：

- If a client is writing to a file and fails, its writes are not
  necessarily atomic. That is, the client may call write(2) on a file
  opened with O_SYNC with an 8 MB buffer and then crash and the write
  may be only partially applied.  (Almost all file systems, even local
  file systems, have this behavior.)
- In shared simultaneous writer situations, a write that crosses
  object boundaries is not necessarily atomic. This means that you
  could have writer A write "aa|aa" and writer B write "bb|bb"
  simultaneously (where | is the object boundary), and end up with
  "aa|bb" rather than the proper "aa|aa" or "bb|bb".
- POSIX includes the telldir(2) and seekdir(2) system calls that allow
  you to obtain the current directory offset and seek back to it.
  Because CephFS may refragment directories at any time, it is
  difficult to return a stable integer offset for a directory.  As
  such, a seekdir to a non-zero offset may often work but is not
  guaranteed to do so.  A seekdir to offset 0 will always work (and is
  equivalent to rewinddir(2)).
- Sparse files propagate incorrectly to the stat(2) st_blocks field.
  Because CephFS does not explicitly track which parts of a file are
  allocated/written, the st_blocks field is always populated by the
  file size divided by the block size.  This will cause tools like
  du(1) to overestimate consumed space.  (The recursive size field,
  maintained by CephFS, also includes file "holes" in its count.)
- When a file is mapped into memory via mmap(2) on multiple hosts,
  writes are not coherently propagated to other clients' caches.  That
  is, if a page is cached on host A, and then updated on host B, host
  A's page is not coherently invalidated.  (Shared writable mmap
  appears to be quite rare--we have yet to here any complaints about this
  behavior, and implementing cache coherency properly is complex.)
- CephFS clients present a hidden ``.snap`` directory that is used to
  access, create, delete, and rename snapshots.  Although the virtual
  directory is excluded from readdir(2), any process that tries to
  create a file or directory with the same name will get an error
  code.  The name of this hidden directory can be changed at mount
  time with ``-o snapdirname=.somethingelse`` (Linux) or the config
  option ``client_snapdir`` (libcephfs, ceph-fuse).
