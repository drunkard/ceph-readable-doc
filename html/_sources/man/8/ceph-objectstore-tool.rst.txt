:orphan:

==================================================
ceph-objectstore-tool -- 修改或检查一个 OSD 的状态
==================================================

提纲
====

| **ceph-objectstore-tool** --data-path *path to osd* [--op *list* ]

支持的对象操作：

* (get|set)-bytes [file]
* set-(attr|omap) [file]
* (get|rm)-attr|omap)
* get-omaphdr
* set-omaphdr [file]
* list-attrs
* list-omap
* remove|removeall
* dump
* set-size
* clear-data-digest
* remove-clone-metadata 

描述
====

**ceph-objectstore-tool** 工具是用于修改 OSD 状态的。它能够修改对象内容、删除对象、罗列 omap 、修改 omap 头部、修改 omap 键、罗列对象属性、修改对象属性键。

**ceph-objectstore-tool** 有两个主要用法： (1) 指定了 "--op" 参数的模式（例如 **ceph-objectstore-tool** --data-path $PATH_TO_OSD --op $SELECT_OPERATION [--pgid $PGID] [--dry-run]），还有 (2) 定位对象操作，在此模式下，对象可以按 ID 或 ``--op list`` 的 JSON 格式输出来指定。

| **ceph-objectstore-tool** --data-path *path to osd* [--pgid *$PG_ID* ][--op *command*]
| **ceph-objectstore-tool** --data-path *path to osd* [ --op *list $OBJECT_ID*]

支持的 --op 命令： ::

* info
* log
* remove
* mkfs
* fsck
* repair
* fuse
* dup
* export
* export-remove
* import
* list
* list-slow-omap
* fix-lost
* list-pgs
* dump-super
* meta-list
* get-osdmap
* set-osdmap
* get-superblock
* set-superblock
* get-inc-osdmap
* set-inc-osdmap
* mark-complete
* reset-last-complete
* update-mon-db
* dump-export
* trim-pg-log

安装
====

**ceph-objectstore-tool** 在 `ceph-osd` 软件包里。

实例
====

对象的修改
----------
.. Modifying Objects

这些命令可修改一个 OSD 的状态，使用 ceph-objectstore-tool 时这个 OSD 一定不能运行。

罗列对象和归置组
----------------
.. Listing Objects and Placement Groups

确保目标 OSD 处于停机状态::

   systemctl status ceph-osd@$OSD_NUMBER

找出一个 OSD 内的所有对象::

   ceph-objectstore-tool --data-path $PATH_TO_OSD --op list

找出一个归置组内的所有对象::

   ceph-objectstore-tool --data-path $PATH_TO_OSD --pgid $PG_ID --op list

找出一个对象所属的归置组（ PG ）::

   ceph-objectstore-tool --data-path $PATH_TO_OSD --op list $OBJECT_ID

丢失对象的修正
--------------
.. Fixing Lost Objects   

确保此 OSD 处于停机状态::

   systemctl status ceph-osd@OSD_NUMBER

修正所有丢失的对象::

   ceph-objectstore-tool --data-path $PATH_TO_OSD --op fix-lost

修正指定归置组内、所有丢失的对象::

   ceph-objectstore-tool --data-path $PATH_TO_OSD --pgid $PG_ID --op fix-lost

根据标识符修正一个丢失的对象::

   ceph-objectstore-tool --data-path $PATH_TO_OSD --op fix-lost $OBJECT_ID

修正以前丢失的对象::

   ceph-objectstore-tool --data-path $PATH_TO_OSD --op fix-lost

修改一个对象的内容
------------------
.. Manipulating an object's content

1. 确保目标 OSD 处于停机状态::

    systemctl status ceph-osd@$OSD_NUMBER

2. 通过罗列此 OSD 或归置组内的对象找到要修改的对象。

3. 在对象中写入字节串之前，先做此对象的备份和工作副本。下面是此命令的语法格式::
   
    ceph-objectstore-tool --data-path $PATH_TO_OSD --pgid $PG_ID $OBJECT get-bytes > $OBJECT_FILE_NAME

例如::

   [root@osd ~]# ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-0 --pgid 0.1c '{"oid":"zone_info.default","key":"","snapid":-2,"hash":235010478,"max":0,"pool":11,"namespace":""}' get-bytes > zone_info.default.backup

   [root@osd ~]# ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-0 --pgid 0.1c '{"oid":"zone_info.default","key":"","snapid":-2,"hash":235010478,"max":0,"pool":11,"namespace":""}' get-bytes > zone_info.default.working-copy

第一个命令创建了备份副本，而第二个命令创建的是工作副本。

4. 编辑工作副本那个对象文件。

5. 填入此对象的变更字节::
     
     ceph-objectstore-tool --data-path $PATH_TO_OSD --pgid $PG_ID $OBJECT set-bytes < $OBJECT_FILE_NAME

例如::

   [root@osd ~]# ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-0 --pgid 0.1c '{"oid":"zone_info.default","key":"","snapid":-2,"hash":235010478,"max":0,"pool":11,"namespace":""}' set-bytes < zone_info.default.working-copy


对象的删除
----------
.. Removing an Object

用 **ceph-objectstore-tool** 删除对象。对象被删除后，其内容以及引用都会从归置组（ PG ）删除。

删除一个对象（语法）::

   ceph-objectstore-tool --data-path $PATH_TO_OSD --pgid $PG_ID $OBJECT remove

删除一个对象（实例）::

    [root@osd ~]# ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-0 --pgid 0.1c '{"oid":"zone_info.default","key":"","snapid":-2,"hash":235010478,"max":0,"pool":11,"namespace":""}' remove


罗列对象图
----------
.. Listing the Object Map

用 ceph-objectstore-tool 罗列对象图（ OMAP ）的内容。其输出是一系列键名。

1. 确认此 OSD 处于停机状态：

   语法::

    systemctl status ceph-osd@$OSD_NUMBER

   实例::

    [root@osd ~]# systemctl status ceph-osd@1

2. 罗列其对象图：

   语法::

    ceph-objectstore-tool --data-path $PATH_TO_OSD --pgid $PG_ID $OBJECT list-omap

   实例::

    [root@osd ~]# ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-0 --pgid 0.1c '{"oid":"zone_info.default","key":"","snapid":-2,"hash":235010478,"max":0,"pool":11,"namespace":""}' list-omap

修改 OMAP 的头部
----------------
.. Manipulating the Object Map Header

**ceph-objectstore-tool** 工具可以按键值对输出 OMAP 头部。

必备条件
^^^^^^^^

    * 有 Ceph OSD 节点的 root 权限
    * 停掉 ceph-osd 守护进程

流程
^^^^

确保目标 OSD 处于停机状态：

  语法::

    systemctl status ceph-osd@$OSD_NUMBER

  实例::

    [root@osd ~]# systemctl status ceph-osd@1

取出 omap 头：

  语法::

        ceph-objectstore-tool --data-path $PATH_TO_OSD --pgid $PG_ID $OBJECT get-omaphdr > $OBJECT_MAP_FILE_NAME

  实例::

    [root@osd ~]# ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-0 --pgid 0.1c '{"oid":"zone_info.default","key":"","snapid":-2,"hash":235010478,"max":0,"pool":11,"namespace":""}'  get-omaphdr > zone_info.default.omaphdr.txt

设置 omap 头：

  语法::

    ceph-objectstore-tool --data-path $PATH_TO_OSD --pgid $PG_ID $OBJECT get-omaphdr < $OBJECT_MAP_FILE_NAME

  实例::

    [root@osd ~]# ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-0 --pgid 0.1c '{"oid":"zone_info.default","key":"","snapid":-2,"hash":235010478,"max":0,"pool":11,"namespace":""}'  set-omaphdr < zone_info.default.omaphdr.txt


修改 OMAP 的某个键
------------------
.. Manipulating the Object Map Key

使用 **ceph-objectstore-tool** 工具更改 OMAP 键，
你得提供数据路径、归置组标识符（ PG ID ）、对象、和 OMAP 的键名。

必备条件
^^^^^^^^

    * 有 Ceph OSD 节点的 root 权限
    * 停掉 ceph-osd 守护进程

命令、流程
^^^^^^^^^^

在 OSD 节点上以 ``root`` 身份执行命令。

* **获取 OMAP 键**

   语法：

   .. code-block:: ini 

      ceph-objectstore-tool --data-path $PATH_TO_OSD --pgid $PG_ID $OBJECT get-omap $KEY > $OBJECT_MAP_FILE_NAME

   实例::

    ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-0 --pgid 0.1c '{"oid":"zone_info.default","key":"","snapid":-2,"hash":235010478,"max":0,"pool":11,"namespace":""}'  get-omap "" > zone_info.default.omap.txt

* **设置此 OMAP 键**

   语法：

   .. code-block:: ini 

      ceph-objectstore-tool --data-path $PATH_TO_OSD --pgid $PG_ID $OBJECT set-omap $KEY < $OBJECT_MAP_FILE_NAME

   实例： ::

    ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-0 --pgid 0.1c '{"oid":"zone_info.default","key":"","snapid":-2,"hash":235010478,"max":0,"pool":11,"namespace":""}' set-omap "" < zone_info.default.omap.txt

* **删除这个 OMAP 键**

   语法：

   .. code-block:: ini 

      ceph-objectstore-tool --data-path $PATH_TO_OSD --pgid $PG_ID $OBJECT rm-omap $KEY

   实例::

    ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-0 --pgid 0.1c '{"oid":"zone_info.default","key":"","snapid":-2,"hash":235010478,"max":0,"pool":11,"namespace":""}' rm-omap ""


罗列一个对象的属性
------------------
.. Listing an Object's Attributes

用 **ceph-objectstore-tool** 工具罗列某一对象的属性。其输出是此对象的键名和值。

必备条件
^^^^^^^^

    * 有 Ceph OSD 节点的 root 权限
    * 停掉 ceph-osd 守护进程

流程
^^^^

   确保目标 OSD 处于停机状态：

   语法::

    systemctl status ceph-osd@$OSD_NUMBER

   实例::

    [root@osd ~]# systemctl status ceph-osd@1

   罗列此对象的属性：

   语法::

    ceph-objectstore-tool --data-path $PATH_TO_OSD --pgid $PG_ID $OBJECT list-attrs

   实例::

    [root@osd ~]# ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-0 --pgid 0.1c '{"oid":"zone_info.default","key":"","snapid":-2,"hash":235010478,"max":0,"pool":11,"namespace":""}' list-attrs


修改对象的属性键
----------------
.. MANIPULATING THE OBJECT ATTRIBUTE KEY

用 ceph-objectstore-tool 工具更改一个对象的属性。要修改此对象的属性，你得有数据和日志路径、归置组标识符（ PG ID ）、对象、还有对象属性的键名。

必备条件

    * 有 Ceph OSD 节点的 root 权限
    * 停掉 ceph-osd 守护进程

流程

确保目标 OSD 处于停机状态：

 语法::

    systemctl status ceph-osd@$OSD_NUMBER

 实例::

    [root@osd ~]# systemctl status ceph-osd@1

获取此对象的属性：

 语法::

   ceph-objectstore-tool --data-path $PATH_TO_OSD --pgid $PG_ID $OBJECT get-attrs $KEY > $OBJECT_ATTRS_FILE_NAME

 实例::

   [root@osd ~]# ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-0  --pgid 0.1c '{"oid":"zone_info.default","key":"","snapid":-2,"hash":235010478,"max":0,"pool":11,"namespace":""}' get-attrs "oid" > zone_info.default.attr.txt

设置一个对象的属性：

 语法::

   ceph-objectstore-tool --data-path $PATH_TO_OSD --pgid $PG_ID $OBJECT  set-attrs $KEY < $OBJECT_ATTRS_FILE_NAME

 实例::

   [root@osd ~]# ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-0 --pgid 0.1c '{"oid":"zone_info.default","key":"","snapid":-2,"hash":235010478,"max":0,"pool":11,"namespace":""}' set-attrs "oid" < zone_info.default.attr.txt

删除对象属性：

 语法::

   ceph-objectstore-tool --data-path $PATH_TO_OSD --pgid $PG_ID $OBJECT rm-attrs $KEY

 实例::

   [root@osd ~]# ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-0 --pgid 0.1c '{"oid":"zone_info.default","key":"","snapid":-2,"hash":235010478,"max":0,"pool":11,"namespace":""}' rm-attrs "oid"


选项
====

.. option:: --help          

   输出帮助消息

.. option:: --type arg        

   参数 arg 是 [bluestore (默认的), filestore, memstore] 其中之一。此工具不能确定 --data-path 的类型时需要加此选项。
 
.. option:: --data-path arg

   对象存储器的路径，必备参数；
   
.. option:: --journal-path arg

   日志路径，此工具找不到时需要加。
   
.. option:: --pgid arg

   PG id，对 info, log, remove, export, export-remove, mark-complete, trim-pg-log 命令是必备。

.. option:: --pool arg

   存储池名字

.. option:: --op arg

   参数 arg 是 [info, log, remove, mkfs, fsck, repair, fuse, dup, export, export-remove, import, list, fix-lost, list-pgs, dump-super, meta-list, get-osdmap, set-osdmap, get-superblock, set-superblock, get-inc-osdmap, set-inc-osdmap, mark-complete, reset-last-complete, update-mon-db, dump-export, trim-pg-log] 其中之一。

.. option:: --epoch arg

   为 get-osdmap 和 get-inc-osdmap 指定 epoch 号，如果没指定就用当前的 epoch 号。

.. option:: --file arg             
   
   export, export-remove, import, get-osdmap, set-osdmap, get-inc-osdmap 或  set-inc-osdmap 操作所需的文件路径。

.. option:: --mon-store-path arg

   update-mon-db 所需的 monstore 路径。

.. option:: --fsid arg

   mkfs 新建存储的 fsid 。

.. option:: --target-data-path arg

   目标对象存储器的路径（ --op dup 需要）。
   
.. option:: --mountpoint arg

   fuse 挂载点。

.. option:: --format arg (=json-pretty) 

   输出格式，可以是 json, json-pretty, xml, xml-pretty

.. option:: --debug

   让诊断信息输出到 stderr 。

.. option:: --force

   忽略某些类型的错误、并继续操作 - **慎用：可能损坏数据，现在或将来都是！**

.. option:: --skip-journal-replay

   禁用日志重放。

.. option:: --skip-mount-omap

   禁用 omap 的挂载。

.. option:: --head

   按名字搜索对象时也去 head 、 snapdir 里找。

.. option:: --dry-run

   不要真的修改 objectstore

.. option:: --namespace arg

   搜索对象时指定命名空间。

.. option:: --rmtype arg      

   已损坏对象删除时指定 'snapmap' 或是 'nosnapmap' - **仅用于测试**

错误码
======
"Mount failed with '(11) Resource temporarily unavailable" - 可能是\
你试图在一个运行着的 OSD 上运行 **ceph-objectstore-tool** 。

使用范围
========

**rgw-orphan-list** 是 Ceph 的一部分，这是个伸缩力强、开源、
分布式的存储系统，更多信息参见 https://docs.ceph.com 。
