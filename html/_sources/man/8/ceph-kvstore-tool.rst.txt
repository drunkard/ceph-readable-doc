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

:program:`ceph-kvstore-tool` 是个 kvstore （键值存储）\
操作工具，可以用来离线操作 leveldb/rocksdb 的数据（比如 OSD 的
omap ）。


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
    Get the value of the KV pair stored with the URL encoded prefix and key.
    If file is also specified, write the value to the file.

:command:`crc <prefix> <key>`
    Get the CRC of the KV pair stored with the URL encoded prefix and key. 

:command:`get-size [<prefix> <key>]`
    Get estimated store size or size of value specified by prefix and key.

:command:`set <prefix> <key> [ver <N>|in <file>]`
    Set the value of the KV pair stored with the URL encoded prefix and key. 
    The value could be *version_t* or text.

:command:`rm <prefix> <key>`
    Remove the KV pair stored with the URL encoded prefix and key.

:command:`rm-prefix <prefix>`
    Remove all KV pairs stored with the URL encoded prefix.

:command:`store-copy <path> [num-keys-per-tx]`
    Copy all KV pairs to another directory specified by ``path``. 
    [num-keys-per-tx] is the number of KV pairs copied for a transaction.

:command:`store-crc <path>`
    Store CRC of all KV pairs to a file specified by ``path``.

:command:`compact`
    Subcommand ``compact`` is used to compact all data of kvstore. It will open
    the database, and trigger a database's compaction. After compaction, some 
    disk space may be released.

:command:`compact-prefix <prefix>`
    Compact all entries specified by the URL encoded prefix. 
   
:command:`compact-range <prefix> <start> <end>`
    Compact some entries specified by the URL encoded prefix and range.

:command:`destructive-repair`
    Make a (potentially destructive) effort to recover a corrupted database.
    Note that in the case of rocksdb this may corrupt an otherwise uncorrupted
    database--use this only as a last resort!

:command:`stats`
    Prints statistics from underlying key-value database. This is only for informative purposes.
    Format and information content may vary between releases. For RocksDB information includes
    compactions stats, performance counters, memory usage and internal RocksDB stats. 


使用范围
========

:program:`ceph-kvstore-tool` 是 Ceph 的一部分，这是个伸缩力强、\
开源、分布式的存储系统，更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph <ceph>`\(8)
