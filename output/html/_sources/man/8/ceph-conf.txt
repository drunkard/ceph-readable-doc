================================
 ceph-conf -- ceph 配置文件工具
================================

.. program:: ceph-conf

提纲
====

| **ceph-conf** -c *conffile* --list-all-sections
| **ceph-conf** -c *conffile* -L
| **ceph-conf** -c *conffile* -l *prefix*
| **ceph-conf** *key* -s *section1* ...
| **ceph-conf** [-s *section* ] --lookup *key*
| **ceph-conf** [-s *section* ] *key*


描述
====

**ceph-conf** 是用来获取 Ceph 配置文件信息的工具。像大多数 Ceph 程序一样，你\
可以用 ``-c`` 选项指定 Ceph 配置文件。


功能
====

.. TODO 把这一段格式化为规范的手册页

**ceph-conf** 可执行以下功能之一：

.. option:: --list-all-sections, -L

   列举配置文件中所有的段落名字。

.. option:: --list-sections, -l

   列举包含指定前缀的所有段落。例如， ``--list-sections mon`` 会罗列出所有以 \
   ``mon`` 打头的段落。

.. option:: --lookup

   搜寻指定名字的配置信息。默认情况下，所搜寻段落由你所指定的 Ceph 名字（默\
   认为 ``client.admin`` ）决定，此名字可用 ``--name`` 指定。

   例如，若指定了 ``--name osd.0`` ，将会搜寻这些段落： [osd.0] 、 [osd] 、 \
   [global] 。

.. option:: --section, -s

   额外指定要搜寻的段落，这些段落会先于正常搜索范围。同样，它会返回先匹配到\
   的条目。

注： ``--lookup`` 是默认动作。如果没在命令行上指定其它动作，那就默认为查找。


实例
====

要查明 osd 0 的 ``osd data`` 选项会用什么值： ::

        ceph-conf -c foo.conf --name osd.0 --lookup "osd data"

要查明 mds a 的 ``log file`` 选项会用什么值： ::

        ceph-conf -c foo.conf --name mds.a "log file"

要罗列以 osd 打头的所有段落： ::

        ceph-conf -c foo.conf -l osd

要罗列所有段落： ::

        ceph-conf -c foo.conf -L


使用范围
========

**ceph-conf** 是 Ceph 的一部分，这是个伸缩力强、开源、分布式的存储系统，\
更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph <ceph>`\(8),
