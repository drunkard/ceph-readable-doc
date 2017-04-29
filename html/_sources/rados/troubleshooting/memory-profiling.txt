==========
 内存剖析
==========

Ceph 监视器、 OSD 、和元数据服务器可利用 ``tcmalloc`` 生成堆栈\
剖析，此功能依赖 ``google-perftools`` ： ::

	sudo apt-get install google-perftools

剖析器会把输出保存到 ``log file`` 目录（如 ``/var/log/ceph`` ），\
详情见\ `日志记录和调试`_\ 。剖析器日志可用 Google 性能工具来查\
看，执行如下命令： ::

    google-pprof --text {path-to-daemon}  {log-path/filename}

例如： ::

    $ ceph tell osd.0 heap start_profiler
    $ ceph tell osd.0 heap dump
    osd.0 tcmalloc heap stats:------------------------------------------------
    MALLOC:        2632288 (    2.5 MiB) Bytes in use by application
    MALLOC: +       499712 (    0.5 MiB) Bytes in page heap freelist
    MALLOC: +       543800 (    0.5 MiB) Bytes in central cache freelist
    MALLOC: +       327680 (    0.3 MiB) Bytes in transfer cache freelist
    MALLOC: +      1239400 (    1.2 MiB) Bytes in thread cache freelists
    MALLOC: +      1142936 (    1.1 MiB) Bytes in malloc metadata
    MALLOC:   ------------
    MALLOC: =      6385816 (    6.1 MiB) Actual memory used (physical + swap)
    MALLOC: +            0 (    0.0 MiB) Bytes released to OS (aka unmapped)
    MALLOC:   ------------
    MALLOC: =      6385816 (    6.1 MiB) Virtual address space used
    MALLOC:
    MALLOC:            231              Spans in use
    MALLOC:             56              Thread heaps in use
    MALLOC:           8192              Tcmalloc page size
    ------------------------------------------------
    Call ReleaseFreeMemory() to release freelist memory to the OS (via madvise()).
    Bytes released to the OS take up virtual address space but no physical memory.
    $ google-pprof --text \
                   /usr/bin/ceph-osd  \
                   /var/log/ceph/ceph-osd.0.profile.0001.heap
     Total: 3.7 MB
     1.9  51.1%  51.1%      1.9  51.1% ceph::log::Log::create_entry
     1.8  47.3%  98.4%      1.8  47.3% std::string::_Rep::_S_create
     0.0   0.4%  98.9%      0.0   0.6% SimpleMessenger::add_accept_pipe
     0.0   0.4%  99.2%      0.0   0.6% decode_message
     ...

再次转储堆栈会生成另外一个文件，这样便于和前面的堆栈转储相比较，\
看看这段时间发生了什么。例如： ::

    $ google-pprof --text --base out/osd.0.profile.0001.heap \
          ceph-osd out/osd.0.profile.0003.heap
     Total: 0.2 MB
     0.1  50.3%  50.3%      0.1  50.3% ceph::log::Log::create_entry
     0.1  46.6%  96.8%      0.1  46.6% std::string::_Rep::_S_create
     0.0   0.9%  97.7%      0.0  26.1% ReplicatedPG::do_op
     0.0   0.8%  98.5%      0.0   0.8% __gnu_cxx::new_allocator::allocate

详情见 `Google 堆栈剖析器`_ 。

安装堆栈剖析器后，启动集群就可以开始使用了。你可以在运行时启用、\
或禁用堆栈剖析器，或确保它在持续运行。在随后的几个命令行用法中，\
用 ``mon`` 、 ``osd`` 或 ``mds`` 替换掉 ``{daemon-type}`` ，用 \
OSD 号、监视器或元数据服务器的 ID 替换掉 ``{daemon-id}`` 。


启动剖析器
----------

要启动堆栈剖析器，执行如下命令： ::

	ceph tell {daemon-type}.{daemon-id} heap start_profiler

例如： ::

	ceph tell osd.1 heap start_profiler

另外，在启动守护进程时若设置了 ``CEPH_HEAP_PROFILER_INIT=true`` \
环境变量，剖析器也会启动。


打印统计信息
------------

用下列命令打印统计状态： ::

	ceph  tell {daemon-type}.{daemon-id} heap stats

例如： ::

	ceph tell osd.0 heap stats

.. note:: 打印状态不要求剖析器在运行，也不会把堆栈分配信息转储\
   到文件。


转储堆栈信息
------------

用下列命令转储堆栈信息： ::

	ceph tell {daemon-type}.{daemon-id} heap dump

例如： ::

	ceph tell mds.a heap dump

.. note:: 只能在剖析器运行的时候转储堆栈信息。


释放内存
--------

要释放由 ``tcmalloc`` 分配、但不是被 Ceph 守护进程使用的内存，\
用下列命令： ::

	ceph tell {daemon-type}{daemon-id} heap release

例如： ::

	ceph tell osd.2 heap release


停止剖析器
----------

要停止堆栈剖析器，执行下列命令： ::

	ceph tell {daemon-type}.{daemon-id} heap stop_profiler

例如： ::

	ceph tell osd.0 heap stop_profiler

.. _日志记录和调试: ../log-and-debug
.. _Google 堆栈剖析器: http://google-perftools.googlecode.com/svn/trunk/doc/heapprofile.html
