:orphan:

====================================================
 ceph-rbdnamer -- 用于命名 RBD 设备的 udev 辅助程序
====================================================

.. program:: ceph-rbdnamer


提纲
====

| **ceph-rbdnamer** *num*


描述
====

**ceph-rbdnamer** 会把指定 RBD 设备的存储池、
命名空间、映像名和快照名打印到标准输出。
`udev` 设备管理器用这些信息来配置 RBD 设备的符号链接。
合理的 `udev` 规则已经写入了名为 `50-rbd.rules` 的文件里。


使用范围
========

**ceph-rbdnamer** 是 Ceph 的一部分，这是个伸缩力强、开源、
分布式的存储系统，更多信息参见 https://docs.ceph.com 。


参考
====

:doc:`rbd <rbd>`\(8),
:doc:`ceph <ceph>`\(8)
