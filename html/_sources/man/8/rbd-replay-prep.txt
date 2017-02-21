:orphan:

=========================================================
 rbd-replay-prep -- 预处理捕捉到的用于重放工作负荷的 RBD
=========================================================

.. program:: rbd-replay-prep

提纲
====

| **rbd-replay-prep** [ --window *seconds* ] [ --anonymize ] *trace_dir* *replay_file*


描述
====

**rbd-replay-prep** 可处理原始 RBD 映像跟踪文件，以便用于 **rbd-replay** 。


选项
====

.. option:: --window seconds

   请求多于 'seconds' 秒之后被认为是独立的。

.. option:: --anonymize

   匿名化映像和快照名。

.. option:: --verbose

   把所有处理过的事件打印到控制台。


实例
====

要预处理 workload1-trace 以便重放： ::

       rbd-replay-prep workload1-trace/ust/uid/1000/64-bit workload1


使用范围
========

**rbd-replay-prep** 是 Ceph 的一部分，这是个伸缩力强、开源、分布式的存储系统，\
更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`rbd-replay <rbd-replay>`\(8),
:doc:`rbd <rbd>`\(8)
