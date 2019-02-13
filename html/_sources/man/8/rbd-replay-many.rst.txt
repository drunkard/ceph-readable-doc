:orphan:

====================================================
 rbd-replay-many -- 在几个客户端上重放 RBD 工作负荷
====================================================

.. program:: rbd-replay-many

提纲
====

| **rbd-replay-many** [ *options* ] --original-image *name* *host1* [ *host2* [ ... ] ] -- *rbd_replay_args*


描述
====

**rbd-replay-many** 工具用于在几个客户端上重放 RBD 载荷。虽然所有客户端使用\
相同的载荷，但它们对单独的映像重放。正像 librbd 的常规用法，其中各原始客户端\
都是有各自映像的虚拟机。

配置和重放文件不会自动复制到客户端。重放映像必须存在才能进行重放。


选项
====

.. option:: --original-image name

   指定最初被追踪映像的名字（和快照），对正确地映射名字是必要的。

.. option:: --image-prefix prefix

   要进行重放的映像名前缀。指定 --image-prefix=foo 可使客户端重放 foo-0 、 \
   foo-1 等等。默认为最初映像名。

.. option:: --exec program

   rbd-replay 可执行文件的路径。

.. option:: --delay seconds

   启动各客户端的延时，默认为 0 。


实例
====

典型用法： ::

       rbd-replay-many host-0 host-1 --original-image=image -- -c ceph.conf replay.bin

实际上将执行下列的命令： ::

       ssh host-0 'rbd-replay' --map-image 'image=image-0' -c ceph.conf replay.bin
       ssh host-1 'rbd-replay' --map-image 'image=image-1' -c ceph.conf replay.bin


使用范围
========

**rbd-replay-many** 是 Ceph 的一部分，这是个伸缩力强、开源、分布式的存储系统，\
更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`rbd-replay <rbd-replay>`\(8),
:doc:`rbd <rbd>`\(8)
