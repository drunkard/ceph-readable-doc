======
LazyIO
======

LazyIO 放宽了 POSIX 语义。它允许缓冲读、写，即使\
一个文件同时被多个客户端上的多个应用打开时也允许。
缓存一致性由应用程序自己负责。

Libcephfs 从 nautilus 版起支持 LazyIO 功能。

启用 LazyIO
===========
.. Enable LazyIO

LazyIO 的启用方法有下面几种。

- ``client_force_lazyio`` 选项可以在全局范围内给 libcephfs 和 ceph-fuse
  启用 LAZY_IO 。

- ``ceph_lazyio(...)`` 和 ``ceph_ll_lazyio(...)`` 可以在 libcephfs 里的\
  文件句柄上启用 LAZY_IO 。

使用 LazyIO
===========
.. Using LazyIO

LazyIO 包含两个方法： ``lazyio_propagate()`` 和 ``lazyio_synchronize()`` 。
启用了 LazyIO 时，写操作结果在调用 ``lazyio_propagate()`` 之后才对其它客户端可见。
一直到调用 ``lazyio_synchronize()`` 之前，读取的内容还是来自本地缓存
（暂且不管其它客户端对文件的更改）。

- ``lazyio_propagate(int fd, loff_t offset, size_t count)`` - 确保\
  客户端的写操作缓冲，位于某个特定范围（位于 offset+count 范围内的）内的，
  已经散播给了共享的文件。如果 offset 和 count 都是 0 ，
  就对整个文件执行此操作。当前还只支持这样操作。

- ``lazyio_synchronize(int fd, loff_t offset, size_t count)`` - 确保\
  客户端在一次顺序读调用时，能够读取到更新过的文件，也就是包含了\
  所有其它客户端散播来的写操作。在 CephFS 里是这样实现的，
  让相关 inode 下的文件缓存失效、也就意味着强制此客户端重新获取、
  重新缓存已更新文件的数据。同时，如果发起调用的客户端它的写缓存不干净
  （没有散播出去）的话， lazyio_synchronize() 也会刷回它。

下面的实例（使用了 libcephfs ），是在一个并行应用程序里，
某个特定的客户端、文件描述符的 I/O 循环样本： ::

        /* Client a (ca) opens the shared file file.txt */
        int fda = ceph_open(ca, "shared_file.txt", O_CREAT|O_RDWR, 0644); 

        /* Enable LazyIO for fda */
        ceph_lazyio(ca, fda, 1));

        for(i = 0; i < num_iters; i++) {
            char out_buf[] = "fooooooooo";
            
            ceph_write(ca, fda, out_buf, sizeof(out_buf), i);
            /* Propagate the writes associated with fda to the backing storage*/
            ceph_propagate(ca, fda, 0, 0);
            
            /* The barrier makes sure changes associated with all file descriptors
            are propagated so that there is certainty that the backing file 
            is upto date */
            application_specific_barrier();

            char in_buf[40];
            /* Calling ceph_lazyio_synchronize here will ascertain that ca will
            read the updated file with the propagated changes and not read
            stale cached data */
            ceph_lazyio_synchronize(ca, fda, 0, 0);
            ceph_read(ca, fda, in_buf, sizeof(in_buf), 0);

            /* A barrier is required here before returning to the next write
            phase so as to avoid overwriting the portion of the shared file still
            being read by another file descriptor */
            application_specific_barrier();
        }
