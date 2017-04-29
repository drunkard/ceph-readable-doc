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

**ceph-rbdnamer** 把指定 RBD 设备所属的存储池和映像名打印到标准输出，以便 \
udev （使用类似如下的规则）设置设备的符号链接： ::

        KERNEL=="rbd[0-9]*", PROGRAM="/usr/bin/ceph-rbdnamer %n", SYMLINK+="rbd/%c{1}/%c{2}"


使用范围
========

**ceph-rbdnamer** 是 Ceph 的一部分，这是个伸缩力强、开源、分布式的存储系统，\
更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`rbd <rbd>`\(8),
:doc:`ceph <ceph>`\(8)
