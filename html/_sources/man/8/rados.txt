:orphan:

=============================
 rados -- rados 对象存储工具
=============================

.. program:: rados

提纲
====

| **rados** [ -m *monaddr* ] [ mkpool | rmpool *foo* ] [ -p | --pool
  *pool* ] [ -s | --snap *snap* ] [ -i *infile* ] [ -o *outfile* ]
  *command* ...


描述
====

**rados** is a utility for interacting with a Ceph object storage
cluster (RADOS), part of the Ceph distributed storage system.


选项
====

.. option:: -p pool, --pool pool

   Interact with the given pool. Required by most commands.

.. option:: -s snap, --snap snap

   Read from the given pool snapshot. Valid for all pool-specific read operations.

.. option:: -i infile

   will specify an input file to be passed along as a payload with the
   command to the monitor cluster. This is only used for specific
   monitor commands.

.. option:: -o outfile

   will write any payload returned by the monitor cluster with its
   reply to outfile. Only specific monitor commands (e.g. osd getmap)
   return a payload.

.. option:: -c ceph.conf, --conf=ceph.conf

   Use ceph.conf configuration file instead of the default
   /etc/ceph/ceph.conf to determine monitor addresses during startup.

.. option:: -m monaddress[:port]

   连接指定监视器（而非通过 ceph.conf 查找）。

.. option:: -b block_size

   设置块尺寸，适用于 put/get 操作、及写入压力测试。

.. option:: --striper

   使用 rados 的条带化 API 而非默认的，支持的操作有 stat 、 get 、 \
   put 、 truncate 、 rm 、 ls 以及所有与 xattr 相关的操作。


全局命令
========

:command:`lspools`
  罗列对象存储池

:command:`df`
  Show utilization statistics, including disk usage (bytes) and object
  counts, over the entire system and broken down by pool.

:command:`mkpool` *foo*
  Create a pool with name foo.

:command:`rmpool` *foo* [ *foo* --yes-i-really-really-mean-it ]
  Delete the pool foo (and all its data)


特定于存储池的命令
==================

:command:`get` *name* *outfile*
  Read object name from the cluster and write it to outfile.

:command:`put` *name* *infile*
  Write object name to the cluster with contents from infile.

:command:`rm` *name*
  Remove object name.

:command:`ls` *outfile*
  List objects in given pool and write to outfile.

:command:`lssnap`
  List snapshots for given pool.

:command:`clonedata` *srcname* *dstname* --object-locator *key*
  Clone object byte data from *srcname* to *dstname*.  Both objects must be stored with the locator key *key* (usually either *srcname* or *dstname*).  Object attributes and omap keys are not copied or cloned.

:command:`mksnap` *foo*
  Create pool snapshot named *foo*.

:command:`rmsnap` *foo*
  Remove pool snapshot named *foo*.

:command:`bench` *seconds* *mode* [ -b *objsize* ] [ -t *threads* ]
  压力测试 *seconds* 秒。 mode 可以是 *write* 、 *seq* 或 *rand* 。 *seq* \
  和 *rand* 分别是顺序读、随机读压力测试，要想做读压力测试，先得加 \
  *--no-cleanup* 选项做一次写压力测试。默认对象尺寸是 4 MB ，默认模拟线程\
  数为 16 。
  注： -b *objsize* 仅适用于 *write* 模式。

:command:`cleanup`

:command:`listomapkeys` *name*
  List all the keys stored in the object map of object name.

:command:`listomapvals` *name*
  List all key/value pairs stored in the object map of object name.
  The values are dumped in hexadecimal.

:command:`getomapval` *name* *key*
  Dump the hexadecimal value of key in the object map of object name.

:command:`setomapval` *name* *key* *value*
  Set the value of key in the object map of object name.

:command:`rmomapkey` *name* *key*
  Remove key from the object map of object name.

:command:`getomapheader` *name*
  Dump the hexadecimal value of the object map header of object name.

:command:`setomapheader` *name* *value*
  Set the value of the object map header of object name.


实例
====

To view cluster utilization::

       rados df

To get a list object in pool foo sent to stdout::

       rados -p foo ls -

To write an object::

       rados -p foo put myobject blah.txt

To create a snapshot::

       rados -p foo mksnap mysnap

To delete the object::

       rados -p foo rm myobject

To read a previously snapshotted version of an object::

       rados -p foo -s mysnap get myobject blah.txt.old


使用范围
========

**rados** 是 Ceph 的一部分，这是个伸缩力强、开源、分布式的存储系统，\
更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph <ceph>`\(8)
