:orphan:

====================================
rgw-policy-check -- 校验桶策略的语法
====================================

.. program:: rgw-policy-check

提纲
====

| **rgw-policy-check**
   -t *tenant* [ *filename* ... ]


描述
====

本程序读入一个或多个包含桶策略的文件，
并判断它的语法是否正确。
它不会检查策略是否合理，
只检查 :program:`radsogw` 内部的\
策略解析逻辑是否会接受该文件。

可以指定多个文件名。
如果没有指定文件，程序将从 stdin 读取。

成功的话，程序什么也不会说。
如果失败，程序将发出错误消息，指出问题所在。
如果一个或多个策略无法读取或解析，
程序将以非零状态码终止。

选项
====

.. option:: -t *tenant*

   指定租户为 *tenant* 。
   这是策略解析逻辑要求必备的，
   将用于构建策略的内部状态表达式。


使用范围
========

:program:`rgw-policy-check` 是 Ceph 的一部分，这是个伸缩力强、\
开源、分布式的存储系统，更多信息参见 https://docs.ceph.com 。

参考
====

:doc:`radosgw <radosgw>`\(8)

.. _桶策略: ../../radosgw/bucketpolicy.rst
