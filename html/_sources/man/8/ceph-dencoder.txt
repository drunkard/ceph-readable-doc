:orphan:

=========================================
 ceph-dencoder -- ceph 编码器/解码器工具
=========================================

.. program:: ceph-dencoder

提纲
====

| **ceph-dencoder** [commands...]


描述
====

**ceph-dencoder** 工具用来编码、解码和转储 Ceph 数据结构。常用于调试或测试版\
本间的兼容性。

**ceph-dencoder** 只是简单地读入命令列表并依次执行。


命令
====

.. option:: version

   打印 **ceph-dencoder** 二进制程序的版本字符串。

.. option:: import <file>

   从指定文件读入已编码的二进制数据块。它将被放入内存驻留缓冲。

.. option:: export <file>

   把当前内存驻留缓冲的内容写入指定文件。

.. option:: list_types

   列出此 **ceph-dencoder** 程序已知的所有数据类型。

.. option:: type <name>

   为即将进行的 ``encode`` 或 ``decode`` 操作指定类型。

.. option:: skip <bytes>

   导入的文件先找到 <bytes> 字节处再开始读数据结构，当对象中感兴趣的部分之前\
   有前同步码或头部时可加此选项。

.. option:: decode

   把内存驻留缓冲中的内容解码为之前选定类型的例程。若遇到错误，则只报告。

.. option:: encode

   把之前选定类型的驻留内存例程编码为驻留内存缓冲。

.. option:: dump_json

   打印内存驻留对象 JSON 格式的描述。

.. option:: count_tests

   打印出之前选定类型、且 **ceph-dencoder** 支持的内建测试例程数量。

.. option:: select_test <n>

   用指定的内建测试例程作为同类型的内存驻留例程。

.. option:: get_features

   打印此版本 **ceph-dencoder** 所支持功能集的十进制值。每一位表示一个功能，\
   它们对应于 src/include/ceph_features.h 中定义的 CEPH_FEATURE_* 。

.. option:: set_features <f>

   把提供给 ``encode`` 的功能位设置为 *f* 。设置了此选项你就能编码出旧版软件\
   可理解的对象（它所支持的类型）。


实例
====

比如你想检查 ``ceph-osd`` 存储的一对象的一个属性，可以这样：

::

    $ cd /mnt/osd.12/current/2.b_head
    $ attr -l foo_bar_head_EFE6384B
    Attribute "ceph.snapset" has a 31 byte value for foo_bar_head_EFE6384B
    Attribute "ceph._" has a 195 byte value for foo_bar_head_EFE6384B
    $ attr foo_bar_head_EFE6384B -g ceph._ -q > /tmp/a
    $ ceph-dencoder type object_info_t import /tmp/a decode dump_json
    { "oid": { "oid": "foo",
          "key": "bar",
          "snapid": -2,
          "hash": 4024842315,
          "max": 0},
      "locator": { "pool": 2,
          "preferred": -1,
          "key": "bar"},
      "category": "",
      "version": "9'1",
      "prior_version": "0'0",
      "last_reqid": "client.4116.0:1",
      "size": 1681,
      "mtime": "2012-02-21 08:58:23.666639",
      "lost": 0,
      "wrlock_by": "unknown.0.0:0",
      "snaps": [],
      "truncate_seq": 0,
      "truncate_size": 0,
      "watchers": {}}

或者，你也许想转储一个内部 CephFS 元数据对象，可以这样：

::

   $ rados -p metadata get mds_snaptable mds_snaptable.bin
   $ ceph-dencoder type SnapServer skip 8 import mds_snaptable.bin decode dump_json
   { "snapserver": { "last_snap": 1,
      "pending_noop": [],
      "snaps": [],
      "need_to_purge": {},
      "pending_create": [],
      "pending_destroy": []}} 


使用范围
========

**ceph-dencoder** 是 Ceph 的一部分，这是个伸缩力强、开源、分布式的存储系统，\
更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph <ceph>`\(8)
