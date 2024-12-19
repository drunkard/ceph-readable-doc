:orphan:

==============================================
 ceph-monstore-tool -- ceph monstore 操作工具
==============================================

.. program:: ceph-monstore-tool

提纲
====

| **ceph-monstore-tool** <store path> <cmd> [args|options]


描述
====

:program:`ceph-monstore-tool` 用于离线操作 MonitorDBStore 的数据
（ monmap 、 osdmap 等）。它与 `ceph-kvstore-tool` 类似。

注意：
    与 Ceph 相关的选项的格式为 `--option-name=VAL` ，
    **不要忘了等号（ '=' ）。**
    例如， `dump-keys --debug-rocksdb=0`

    命令相关的选项必须在 `--` 之后传递，
    例如， `get monmap -- --version 10 --out /tmp/foo`

命令
====

:program:`ceph-monstore-tool` 有很多命令用于调试：

:command:`store-copy <path>`
    把存储库复制到 PATH 。

:command:`get monmap [-- options]`
    提取 monmap （如果指定了，就提取这个版本 VER ）（默认：最后提交的）

:command:`get osdmap [-- options]`
    提取 osdmap （如果指定了，就提取这个版本 VER ） （默认：最后提交的）.

:command:`get msdmap [-- options]`
    提取 msdmap （如果指定了，就提取这个版本 VER ） （默认：最后提交的）.

:command:`get mgr [-- options]`
    提取 mgrmap （如果指定了，就提取这个版本 VER ） （默认：最后提交的）.

:command:`get crushmap [-- options]`
    提取 crushmap （如果指定了，就提取这个版本 VER ） （默认：最后提交的）.

:command:`get-key <prefix> <key> [-- options]`
    把密钥提取到文件 FILE （默认： stdout ）。

:command:`remove-key <prefix> <key> [-- options]`
    删除密钥。

:command:`dump-keys`
    把存储的密钥转储到文件 FILE （默认： stdout ）。

:command:`dump-paxos [-- options]`
    转储 Paxos 事务（ -- --help 更多帮助信息）。

:command:`dump-trace FILE  [-- options]`
    把追踪内容转储到文件 FILE （ -- --help 更多帮助信息）。

:command:`replay-trace FILE  [-- options]`
    重放 FILE 里的追踪信息（ -- --help 更多帮助信息）。

:command:`random-gen [-- options]`
    向存储库里增加一个随机生成的操作（ -- --help 更多帮助信息）。

:command:`rewrite-crush [-- options]`
    向存储库里增加一个重写提交。

:command:`rebuild`
    重建存储


使用范围
========

:program:`ceph-monstore-tool` 是 Ceph 的一部分，这是个伸缩力强、\
开源、分布式的存储系统，更多信息参见 https://docs.ceph.com 。


参考
====

:doc:`ceph <ceph>`\(8)
