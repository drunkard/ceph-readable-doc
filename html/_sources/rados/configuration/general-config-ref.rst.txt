==============
 常规配置参考
==============


``fsid``

:描述: 文件系统 ID ，每集群一个。
:类型: UUID
:是否必需: No.
:默认值: 无。通常由部署工具生成。


``admin socket``

:描述: 在某个守护进程上执行管理命令的套接字，不管 Ceph 监视器团体是否已建立。
:类型: String
:是否必需: No
:默认值: ``/var/run/ceph/$cluster-$name.asok``


``pid file``

:描述: mon 、 osd 、 mds 将把它们的 PID 写入此文件，其名为 \
       ``/var/run/$cluster/$type.$id.pid`` 。如名为 Ceph 的集群，其 ID 为 \
       ``a`` 的 ``mon`` 进程将创建 ``/var/run/ceph/mon.a.pid`` 。如果是正常\
       停止的， ``pid file`` 就会被守护进程删除；如果进程未进入后台运行（即\
       启动时加了 ``-f`` 或 ``-d`` 选项），它就不会创建 ``pid file`` 。

:类型: String
:是否必需: No
:默认值: No


``chdir``

:描述: Ceph 进程一旦启动、运行就进入这个目录，默认推荐 ``/`` 。
:类型: String
:是否必需: No
:默认值: ``/``


``max open files``

:描述: 如果设置了， :term:`Ceph 存储集群`\ 启动时会设置操作系统级的 \
       ``max open fds`` （即文件描述符最大允许值），这有助于防止耗尽文件描述符。

:类型: 64-bit Integer
:是否必需: No
:默认值: ``0``


``fatal signal handlers``

:描述: 如果设置了，将安装 SEGV 、 ABRT 、 BUS 、 ILL 、 FPE 、 XCPU 、 \
       XFSZ 、 SYS 信号处理器，用于产生有用的日志信息。

:类型: Boolean
:默认值: ``true``
