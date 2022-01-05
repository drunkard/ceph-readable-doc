:orphan:

===============================================
 ceph-kvstore-tool -- ceph 的 kvstore 操作工具
===============================================

.. program:: ceph-kvstore-tool

提纲
====

| **ceph-kvstore-tool** <leveldb|rocksdb|bluestore-kv> <store path> *command* [args...]


描述
====

:program:`ceph-kvstore-tool` 是个 kvstore （键值存储）操作工具，\
可以用来离线操作 leveldb/rocksdb 的数据（比如 OSD 的 omap ）。


命令
====

:program:`ceph-kvstore-tool` 工具有很多命令，可用于调试，如下：

:command:`list [prefix]`
    打印所有键值对的键名，有 URL 编码的前缀时只打印匹配的。

:command:`list-crc [prefix]`
    打印所有键值对的 CRC 校验码，有 URL 编码的前缀时只打印匹配的。

:command:`dump [prefix]`
    打印所有键值对的键名及其值，有 URL 编码的前缀时只打印匹配的。

:command:`exists <prefix> [key]`
    查验是否存在以此 URL 为前缀的键值对；如果指定了键名，就只\
    查验带有此前缀的键名。

:command:`get <prefix> <key> [out <file>]`
    通过 URL 编码存储的 prefix 和 key 获取 KV 对的值。
    如果指定了文件，把此值写入该文件。

:command:`crc <prefix> <key>`
    通过 URL 编码存储的 prefix 和 key 获取 KV 对的 CRC 。

:command:`get-size [<prefix> <key>]`
    获取 prefix 和 key 所指内容的大概尺寸、或是值的尺寸。

:command:`set <prefix> <key> [ver <N>|in <file>]`
    通过 URL 编码存储的 prefix 和 key 设置 KV 对的值。
    这个值可以是 *version_t* 或文本。

:command:`rm <prefix> <key>`
    删除 URL 编码的 prefix 和 key 的 KV 对。

:command:`rm-prefix <prefix>`
    删除 URL 编码存储的 prefix 之下的所有 KV 对。

:command:`store-copy <path> [num-keys-per-tx]`
    把所有 KV 对都复制到 ``path`` 目录里。
    [num-keys-per-tx] 是一个事务可以复制的 KV 对数量。

:command:`store-crc <path>`
    把所有 KV 对的 CRC 都存储到 ``path`` 内的一个文件里。

:command:`compact`
    子命令 ``compact`` 用于压缩 kvstore 的所有数据。
    它会打开数据库、触发数据库的压缩功能。压缩后，
    会释放一些磁盘空间。

:command:`compact-prefix <prefix>`
    压缩 URL 编码的 prefix 相关的所有条目。
   
:command:`compact-range <prefix> <start> <end>`
    压缩由 URL 编码的 prefix 和范围确定的那些条目。

:command:`destructive-repair`
    尽力（有可能是破坏性地）恢复损坏的数据库。
    注意，底层是 rocksdb 时，此操作可能会损坏本来正常的数据库——
    此命令只能作为终极手段！

:command:`stats`
    打印底层键值数据库的统计信息，这只是个信息来源。
    格式和信息内容随版本变化频繁，对于 RocksDB ，
    包含的信息有压缩统计信息、性能计数器、内存使用率和 RocksDB 内部统计信息。

:command:`histogram`
    呈现出底层 KV 数据库的键/值尺寸分布情况统计。

使用范围
========

**ceph-kvstore-tool** 是 Ceph 的一部分，这是个伸缩力强、\
开源、分布式的存储系统，更多信息参见 https://docs.ceph.com 。


参考
====

:doc:`ceph <ceph>`\(8)
