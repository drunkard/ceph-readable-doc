:orphan:

===========================================================
 rgw-gap-list -- 列出已损坏 RADOS 后端对象的存储桶索引条目
===========================================================

.. program:: rgw-gap-list

提纲
====

| **rgw-gap-list**


描述
====

:program:`rgw-gap-list` 是一个\ *实验性*\ 的 RADOS 网关用户管理工具。
它会列出缺少后端 RADOS 对象的桶索引条目。
该工具会把结果和中间文件保存在本地文件系统上，
而不是放到 Ceph 集群本身，
因此不会额外占用集群的存储空间。

理论上，这样的缺口本不应存在。然而，由于 Ceph 发展迅速、
程序缺陷确实会不时出现，它们可能导致桶索引条目有丢失的 RADOS 对象，
比如删除操作未能全部完成时。

在幕后，它会运行 `rados ls` 和 `radosgw-admin bucket radoslist ...` ，
并生成一个列表，列出那些出现在后者中但未出现在前者的条目。
这些条目被假定为缺口（ gap ）。

注意：根据所涉及存储池的大小，
此工具生成结果的速度可能会非常慢。


警告
====

此工具尚处于\ *实验阶段*\ 。


选项
====

.. option:: -p pool

   RGW 桶数据存储池的名字。如果忽略此选项，
   在执行时会提示需要存储池名字。
   多个存储池可以用空格分隔、并用双引号括起来。

.. option:: -t temp_directory

   此工具会产生海量中间文件。默认会用 ``/tmp`` ，
   但如果 ``/tmp`` 所在文件系统没有足够的空闲空间，
   可以指定其它目录（位于有足够空闲空间的文件系统上）。

.. option:: -m

   使用两个（多个）线程来加速运行。


实例
====

启动此工具：\ ::

        $ rgw-gap-list -p default.rgw.buckets.data -t /home/super_admin/temp_files


适用范围
========

:program:`rgw-gap-list` 是 Ceph 的一部分，这是个伸缩力强、\
开源、分布式的存储系统，更多信息参见 https://docs.ceph.com 。


参考
====

:doc:`radosgw-admin <radosgw-admin>`\(8)
:doc:`rgw-orphan-list <rgw-orphan-list>`\(8)
