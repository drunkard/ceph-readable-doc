cephfs-journal-tool
===================


目的
----

如果 CephFS 的日志损坏了，就需要专家介入，把文件系统恢复到工作状态。

``cephfs-journal-tool`` 工具提供的功能可帮助专家们检查、修改、或从日志中提取\
数据。

.. warning::

    此工具很\ **危险**\ ，因为它会直接修改文件系统的内部数据结构。做好备份、\
    多加小心、多寻求专家建议，如果不确定，就别运行此工具。

语法
----

::

    cephfs-journal-tool journal <inspect|import|export|reset>
    cephfs-journal-tool header <get|set>
    cephfs-journal-tool event <get|splice|apply> [filter] <list|json|summary>

此工具有三种模式： ``journal`` 、 ``header`` 和 ``event`` ，分别操作日志内的\
整个日志、头部、和事件。


Journal 模式
------------

这应该是你访问日志状态的起点。

* ``inspect`` 报告日志的健康状况，它将鉴定存储日志中丢失或被篡改的对象。要注\
  意的是，此过程不能鉴定事件本身的不一致，它仅鉴定存在且可解码的。

* ``import`` 和 ``export`` 以一个稀疏文件格式读取和写入日志的二进制转储，用\
  文件名作为最后一个参数。导出操作在日志损坏时可能不会可靠地运行，因为丢失了\
  对象。

* ``reset`` 删减日志，丢弃其内的任意信息。


实例： journal inspect
~~~~~~~~~~~~~~~~~~~~~~

::

    # cephfs-journal-tool journal inspect
    Overall journal integrity: DAMAGED
    Objects missing:
      0x1
    Corrupt regions:
      0x400000-ffffffffffffffff


实例：日志的导入和导出
~~~~~~~~~~~~~~~~~~~~~~

::

    # cephfs-journal-tool journal export myjournal.bin
    journal is 4194304~80643
    read 80643 bytes at offset 4194304
    wrote 80643 bytes at offset 4194304 to myjournal.bin

注：这是个\ **稀疏**\ 文件，你可以在保留其稀疏特性的前提下，用 \
``tar cSzf myjournal.bin.tgz myjournal.bin`` 进行压缩。

::

    # cephfs-journal-tool journal import myjournal.bin
    undump myjournal.bin
    start 4194304 len 80643
    writing header 200.00000000
     writing 4194304~80643
    done.

.. note::

    明智的做法是，在开始修改前先用 ``journal export <backup file>`` 命令备\
    份日志。


头模式
------

* ``get`` 输出日志头部的当前内容。

* ``set`` 修改头部的一个属性。可修改属性有 ``trimmed_pos`` 、 ``expire_pos`` \
  和 ``write_pos`` 。


实例： header get/set
~~~~~~~~~~~~~~~~~~~~~

::

    # cephfs-journal-tool header get
    { "magic": "ceph fs volume v011",
      "write_pos": 4274947,
      "expire_pos": 4194304,
      "trimmed_pos": 4194303,
      "layout": { "stripe_unit": 4194304,
          "stripe_count": 4194304,
          "object_size": 4194304,
          "cas_hash": 4194304,
          "object_stripe_unit": 4194304,
          "pg_pool": 4194304}}

    # cephfs-journal-tool header set trimmed_pos 4194303
    Updating trimmed_pos 0x400000 -> 0x3fffff
    Successfully updated header.


事件模式
--------

事件模式下可对日志内容进行详细的检查和操作。事件模式可操作日志中的所有或过滤\
出的事件。

``cephfs-journal-tool event`` 的参数由动作、可选过滤器参数、和输出模式组成。

::

    cephfs-journal-tool event <action> [filter] <output>

动作：

* ``get`` 从日志读出事件；
* ``splice`` 擦除日志中的某些事件或区域；
* ``apply`` 从事件中提取文件系统元数据、并试着应用到元数据存储中。

过滤器选项：

* ``--range <int begin>..[int end]`` 过滤从 begin （包含）到 end （不包含）\
  之间的事件；
* ``--path <path substring>`` 按字符串过滤相关事件，此字符串包含在与这些事件\
  相关的元数据中；
* ``--inode <int>`` 按字符串过滤事件，此字符串包含在与这些事件相关的元数据中；
* ``--type <type string>`` 过滤此类型的事件；
* ``--frag <ino>[.frag id]`` 过滤与此目录片段相关的事件；
* ``--dname <string>`` 过滤与目录片段内的指定 dentry 相关的事件，只能和 \
  ``--frag`` 一起使用；
* ``--client <int>`` 过滤来自此客户端会话 ID 的事件。

过滤器可用“与”操作组合使用，也就是最终结果为各过滤器的交集。

输出模式：

* ``binary``: 把各事件写入一个二进制文件，放入 ``--path`` 指定的目录；
* ``json``: 把所有事件组织为序列化的 JSON 对象列表，并写入单个文件；
* ``summary``: 把人类可读的事件汇总写到标准输出；
* ``list``: 写出一个人类可读的摘要列表，其中包含各事件的类型、以及此事件所影\
  响的文件路径。


实例：事件模式
~~~~~~~~~~~~~~

::

    # cephfs-journal-tool event get json --path output.json
    Wrote output to JSON file 'output.json'

    # cephfs-journal-tool event get summary
    Events by type:
      NOOP: 2
      OPEN: 2
      SESSION: 2
      SUBTREEMAP: 1
      UPDATE: 43

    # cephfs-journal-tool event get list
    0x400000 SUBTREEMAP:  ()
    0x400308 SESSION:  ()
    0x4003de UPDATE:  (setattr)
      /
    0x40068b UPDATE:  (mkdir)
      diralpha
    0x400d1b UPDATE:  (mkdir)
      diralpha/filealpha1
    0x401666 UPDATE:  (unlink_local)
      stray0/10000000001
      diralpha/filealpha1
    0x40228d UPDATE:  (unlink_local)
      diralpha
      stray0/10000000000
    0x402bf9 UPDATE:  (scatter_writebehind)
      stray0
    0x403150 UPDATE:  (mkdir)
      dirbravo
    0x4037e0 UPDATE:  (openc)
      dirbravo/.filebravo1.swp
    0x404032 UPDATE:  (openc)
      dirbravo/.filebravo1.swpx

    # cephfs-journal-tool event get --path /filebravo1 list
    0x40785a UPDATE:  (openc)
      dirbravo/filebravo1
    0x4103ee UPDATE:  (cap update)
      dirbravo/filebravo1

    # cephfs-journal-tool event splice --range 0x40f754..0x410bf1 summary
    Events by type:
      OPEN: 1
      UPDATE: 2

    # cephfs-journal-tool event apply --range 0x410bf1.. summary
    Events by type:
      NOOP: 1
      SESSION: 1
      UPDATE: 9

    # cephfs-journal-tool event get --inode=1099511627776 list
    0x40068b UPDATE:  (mkdir)
      diralpha
    0x400d1b UPDATE:  (mkdir)
      diralpha/filealpha1
    0x401666 UPDATE:  (unlink_local)
      stray0/10000000001
      diralpha/filealpha1
    0x40228d UPDATE:  (unlink_local)
      diralpha
      stray0/10000000000

    # cephfs-journal-tool event get --frag=1099511627776 --dname=filealpha1 list
    0x400d1b UPDATE:  (mkdir)
      diralpha/filealpha1
    0x401666 UPDATE:  (unlink_local)
      stray0/10000000001
      diralpha/filealpha1

    # cephfs-journal-tool event get binary --path bin_events
    Wrote output to binary files in directory 'bin_events'

