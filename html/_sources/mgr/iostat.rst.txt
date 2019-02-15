.. _mgr-iostat-overview:

iostat
======

此插件可显示 Ceph 集群当前的吞吐量和完成的 IOPS 。

启用
----

为验证 *iostat* 模块是否已启用，用命令： ::

  ceph mgr module ls

此模块可用如下命令启用： ::

  ceph mgr module enable iostat

执行此模块，用命令： ::

  ceph iostat

要更改统计信息打印的频率，可加 ``-p`` 选项： ::

  ceph iostat -p <period in seconds>

例如，下列命令每 5 秒打印一次统计信息： ::

  ceph iostat -p 5

要停止，按下 Ctrl-C 。
