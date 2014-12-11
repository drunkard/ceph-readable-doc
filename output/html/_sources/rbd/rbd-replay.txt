==========
 RBD 重放
==========

.. index:: Ceph Block Device; RBD Replay

RBD 重放是个用于捕捉和重放 Rados 块设备（ RBD ）载荷的工具集。要捕捉 RBD 载\
荷，必须在客户端安装 ``lttng-tools`` ，且客户端上的 ``librbd`` 必须在 Giant \
版以上。

捕捉、重放需三步：

#. 捕捉一个追踪过程，确保捕捉的是 ``pthread_id`` 上下文： ::

	mkdir -p traces
	lttng create -o traces librbd
	lttng enable-event -u 'librbd:*'
	lttng add-context -u -t pthread_id
	lttng start
	# run RBD workload here
	lttng stop

#. 用 `rbd-replay-prep`_ 处理捕捉到的追踪结果： ::

	rbd-replay-prep traces/ust/uid/*/* replay.bin

#. 用 `rbd-replay`_ 重放此追踪过程。熟知它的行为前请用只读模式： ::

	rbd-replay --read-only replay.bin

.. important:: ``rbd-replay`` 默认会销毁数据。不要在想保留的映像上使用，除非\
   指定了 ``--read-only`` 选项。

重放载荷并非只能在与捕获时相同的 RBD 映像或相同的集群上进行。为说明差异，你\
也许得在执行 ``rbd-replay`` 命令时指定 ``--pool`` 和 ``--map-image`` 选项。


.. _rbd-replay: ../../man/8/rbd-replay
.. _rbd-replay-prep: ../../man/8/rbd-replay-prep
