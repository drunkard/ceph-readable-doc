CephFS 配额管理
===============
.. CephFS Quotas

CephFS 允许给系统内的任意目录设置配额，
这个配额可以限制目录树中这一点以下的\
*字节*\ 数或者\ *文件*\ 数。

就像 CephFS 里的其它配置一样，
配额也是用虚拟扩展属性来配置的：

 * ``ceph.quota.max_files`` -- 文件限额
 * ``ceph.quota.max_bytes`` -- 字节数限额

如果这些属性出现在目录索引节点里，就意味着那里配置了配额；
如果不存在，那么那个目录就没设置配额
（然而父目录仍然有可能配置了）。

要设置配额，给 CephFS 目录的扩展属性设置数值即可： ::

    setfattr -n ceph.quota.max_bytes -v 100000000 /some/dir     # 100 MB
    setfattr -n ceph.quota.max_files -v 10000 /some/dir         # 10,000 files

设置 ``ceph.quota.max_bytes`` 数值时也可以加上人性化单位： ::

  setfattr -n ceph.quota.max_bytes -v 100K /some/dir          # 100 KiB
  setfattr -n ceph.quota.max_bytes -v 5Gi /some/dir           # 5 GiB

.. note:: 即使输入的是 SI （国际）单位，数值也将严格转换为 IEC 单位，
   例如 1K 转换为 1024 字节。

要查看设置的配额： ::

  $ getfattr -n ceph.quota.max_bytes /some/dir
  # file: dir1/
  ceph.quota.max_bytes="100000000"
  $
  $ getfattr -n ceph.quota.max_files /some/dir
  # file: dir1/
  ceph.quota.max_files="10000"

.. note:: 在一个 CephFS 目录上执行 ``getfattr /some/dir -d -m -``
   命令将不会打印任何 CephFS 扩展属性。这是因为 CephFS 核心和
   FUSE 客户端隐藏了 ``listxattr(2)`` 系统调用返回的这些信息。
   相反，可以通过执行 ``getfattr /some/dir -n ceph.<some-xattr>``
   查看特定的 CephFS 扩展属性。

要删除或禁用配额，可以删除相应的扩展属性或\
把它的数值设置为 ``0`` 。

这样删除： ::

  $ setfattr -x ceph.quota.max_bytes /some/dir
  $ getfattr /some/dir -n ceph.quota.max_bytes
  /some/dir/: ceph.quota.max_bytes: No such attribute
  $
  $ setfattr -x ceph.quota.max_files /some/dir
  $ getfattr /some/dir/ -n ceph.quota.max_files
  /some/dir/: ceph.quota.max_files: No such attribute

通过把数值设置为零来删除： ::

  $ setfattr -n ceph.quota.max_bytes -v 0 /some/dir
  $ getfattr /some/dir -n ceph.quota.max_bytes
  /some/dir/: ceph.quota.max_bytes: No such attribute
  $
  $ setfattr -n ceph.quota.max_files -v 0 /some/dir
  $ getfattr /some/dir/ -n ceph.quota.max_files
  /some/dir/: ceph.quota.max_files: No such attribute

空间利用率报告和 CephFS 配额
----------------------------
.. Space Usage Reporting and CephFS Quotas

当 CephFS 挂载的根目录设置了配额时，空间利用率报告工具（如 ``df`` ）报告的
CephFS 可用空间就是基于配额限制的。也就是说， ``可用空间 = 配额限制 - 已用空间`` ，
而不是 ``可用空间 = 总空间 - 已用空间`` 。

这种行为可以在 ``ceph.conf`` 的客户端部分设置下列选项来禁用： ::

    client quota df = false

局限性
------
.. Limitations

#. *配额是合作性的、非对抗性的。* CephFS 的配额功能\
   依赖于挂载它的客户端的合作，
   在达到上限时要停止写入；
   无法阻止篡改过的或者对抗性的客户端，
   它们可以想写多少就写多少。
   在客户端完全不可信时，用配额防止多占空间是靠不住的。

#. *配额是不准确的。* 在达到配额限制一小段时间后，
   正在写入文件系统的进程才会被停止。
   很难避免它们超过配置的限额、
   多写入一些数据。会超过配额多大幅度主要取决于时间长短，
   而非数据量。一般来说，
   超出配置的限额之后 10 秒内，
   写入会被停掉。

#. *内核客户端在 4.17 及更高版才实现配额功能。*
   用户空间客户端（ libcephfs 、 ceph-fuse ）早已支持配额了；
   >=4.17 的 Linux 内核客户端也支持配额，但仅在
   mimic 及更高版本的集群上支持。内核客户端（即使是最近的版本），
   即使它们能设置配额扩展属性，也不能处理老集群上的配额。

#. *基于路径限制挂载时必须谨慎地配置配额。*
   客户端必须能够访问配置了配额的那个目录的索引节点，
   这样才能执行配额管理。如果某一客户端被 MDS 能力\
   限制成了只能访问一个特定路径（如 ``/home/user`` ），
   并且它们无权访问配置了配额的父目录（如 ``/home`` ），
   这个客户端就不会按配额执行。所以，
   基于路径做访问控制时，
   最好在限制了客户端的那个目录
   （如 ``/home/user`` ）、
   或者它下面的子目录上配置配额。

   如果是内核客户端，在一个目录 inode 上设置了配额后，
   还需要有其父目录的访问权限，才能执行配额。
   如果配额设置在一个目录路径上（如 ``/home/volumes/group`` ），
   则 kclient 需要有访问其父节点
   （如 ``/home/volumes`` ）的权限。

   创建这样的用户命令示例如下： ::

     $ ceph auth get-or-create client.guest mds 'allow r path=/home/volumes, allow rw path=/home/volumes/group' mgr 'allow rw' osd 'allow rw tag cephfs metadata=*' mon 'allow r'

   参阅: https://tracker.ceph.com/issues/55090

#. *此后删除或更改的快照文件数据不计入配额。*
   另见 http://tracker.ceph.com/issues/24284 。
