==========
 CPU 剖析
==========

如果你从源码编译时启用了 `oprofile`_ ，那就可以剖析 Ceph 的 CPU 使用情况，详情见\ \
`安装 Oprofile`_ 。


初始化 oprofile
===============

你首次使用 ``oprofile`` 时要初始化，找到对应于当前运行内核的 ``vmlinux`` 映像： ::

	ls /boot
	sudo opcontrol --init
	sudo opcontrol --setup --vmlinux={path-to-image} --separate=library --callgraph=6


启动 oprofile
=============

执行下面的命令启动 ``oprofile`` ： ::

	opcontrol --start

启动 ``oprofile`` 后，你可以运行一些 Ceph 测试。


停止 oprofile
=============

执行下面的命令停止 ``oprofile`` ： ::

	opcontrol --stop


查看 oprofile 运行结果
======================

要查看 ``cmon`` 最近的结果，执行下面的命令： ::

	opreport -gal ./cmon | less

要检索 ``cmon`` 最近的调用图结果，执行下面的命令： ::

	opreport -cal ./cmon | less

.. important:: 回顾结果后，重新剖析前应该先重置，重置 ``oprofile`` 动作从会话目录\
   里删除了数据。


重置 oprofile
=============

要重置 ``oprofile`` ，执行下面的命令： ::

	sudo opcontrol --reset   

.. important:: 你应该分析后再重置，以免混合不同的剖析结果。


.. _oprofile: http://oprofile.sourceforge.net/about/
.. _安装 Oprofile: ../../../dev/cpu-profiler
