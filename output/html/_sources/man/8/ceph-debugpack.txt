:orphan:

=========================================
 ceph-debugpack -- ceph 调试信息打包工具
=========================================

.. program:: ceph-debugpack

提纲
====

| **ceph-debugpack** [ *options* ] *filename.tar.gz*


描述
====

**ceph-debugpack** 会打包各种用于崩溃调试的信息。当调试某问题时，可把此压缩\
包共享给 Ceph 开发者。

此压缩包会包含 ceph-mds 、 ceph-osd 、 ceph-mon 、 radosgw 的二进制文件，所\
有日志文件， ceph.conf 配置文件，能找到的核心转储文件，以及（若集群在运行） \
'ceph report' 生成的当前集群状态转储。


选项
====

.. option:: -c ceph.conf, --conf=ceph.conf

   用 *ceph.conf* 配置文件而非默认的 ``/etc/ceph/ceph.conf`` 来确定启动时所\
   需的监视器地址。


使用范围
========

:program:`ceph-debugpack` 是 Ceph 的一部分，这是个伸缩力强、开源、分布式的存储系统，\
更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph <ceph>`\(8)
:doc:`ceph-post-file <ceph-post-file>`\(8)
