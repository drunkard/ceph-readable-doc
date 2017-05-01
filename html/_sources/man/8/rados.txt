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

**rados** 工具可操作 Ceph 对象存储集群（ RADOS ），是 Ceph 分\
布式存储系统的一部分。


选项
====

.. option:: -p pool, --pool pool

   操作指定的存储池。大多数命令都得指定此参数。

.. option:: -s snap, --snap snap

   从指定的存储池快照读出。适用于所有与存储池相关的读操作。

.. option:: -i infile

   指定输入文件，其内容将作为此命令的载荷发送给监视器集群。仅\
   适用于部分监视器命令。

.. option:: -o outfile

   把监视器集群返回的载荷写入 outfile 。仅适用于某些会返回载荷\
   的监视器命令（如 osd getmap ）。

.. option:: -c ceph.conf, --conf=ceph.conf

   用指定的 ceph.conf 配置文件而非默认的 /etc/ceph/ceph.conf
   来确定监视器的初始地址。

.. option:: -m monaddress[:port]

   连接指定监视器（而非通过 ceph.conf 查找）。

.. option:: -b block_size

   设置块尺寸，适用于 put/get/append 操作、及写入压力测试。

.. option:: --striper

   使用 rados 的条带化 API 而非默认的，支持的操作有 stat 、
   get 、 put 、 append 、 truncate 、 rm 、 ls 以及所有与
   xattr 相关的操作。


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
  从集群读出名为 name 的对象、并把它写入 outfile 。

:command:`put` *name* *infile*
  把 infile 的内容写入成集群内名为 name 的对象。

:command:`append` *name* *infile*
  把 infile 的内容追加给集群内名为 name 的对象。

:command:`rm` *name*
  删除名为 name 的对象。

:command:`listwatchers` *name*
  罗列此对象名的关注者。

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
  压力测试 *seconds* 秒。 *mode* 可以是 *write* 、 *seq* 或 \
  *rand* 。 *seq* 和 *rand* 分别是顺序读、随机读压力测试，要想\
  做读压力测试，先得加 *--no-cleanup* 选项做一次写压力测试。默\
  认对象尺寸是 4 MB ，默认的模拟线程数（并行写操作）为 16 。\
  *--run-name <label>* 选项适用于多个客户端并行测试以评估最大\
  载荷。 *<label>* 表示任意对象名，默认为 \
  "benchmark_last_metadata" ，且作为“读”和“写”操作的底层对象名。
  注： -b *objsize* 仅适用于 *write* 模式。
  注： *write* 和 *seq* 必须运行在相同的主机上，否则 *write* \
  所创建对象的名字不能被 *seq* 所接受。

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
