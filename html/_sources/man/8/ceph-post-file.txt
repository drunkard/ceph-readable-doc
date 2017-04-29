:orphan:

==================================================
 ceph-post-file -- 把文件上传给 ceph 开发者
==================================================

.. program:: ceph-post-file

提纲
====

| **ceph-post-file** [-d *description] [-u *user*] *file or dir* ...


描述
====

**ceph-post-file** 可把文件或目录上传到 ceph.com 让 Ceph 开发者稍后分析。

每次调用都会把文件或目录上传到一个带惟一标签的独立目录。此标签可共享给开发者\
或在缺陷报告（ http://tracker.ceph.com/ ）里引用。一旦上传完成，目录就被标记\
为不可读且不可写，以防其它用户访问或修改。


警告
====

上传的数据做了基本的防护措施，只对有权限访问 ceph.com 基础设施的开发者可见。\
然而，用户自己也要再三考虑并采取恰当的预防措施，然后再上传（潜在的）敏感数据\
（例如包含 Ceph 密钥的日志或数据目录）。


选项
====

.. option:: -d *description*, --description *description*

   给此次上传加个简短的描述，这是引用缺陷号码的最佳位置。无默认值。

.. option:: -u *user*

   设置此上传的用户元数据。默认为 `whoami`@`hostname -f` 。


实例
====

上传单个日志文件： ::

   ceph-post-file /var/log/ceph/ceph-mon.`hostname`.log

上传几个目录： ::

   ceph-post-file -d 'mon data directories' /var/log/ceph/mon/*


使用范围
========

**ceph-post-file** 是 Ceph 的一部分，这是个伸缩力强、开源、分布式的存储系统，\
更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph <ceph>`\(8),
:doc:`ceph-debugpack <ceph-debugpack>`\(8),
