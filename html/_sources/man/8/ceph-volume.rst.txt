:orphan:

========================================
 ceph-volume -- Ceph OSD 部署和检查工具
========================================

.. program:: ceph-volume

提纲
====

| **ceph-volume** [-h] [--cluster CLUSTER] [--log-level LOG_LEVEL]
|                 [--log-path LOG_PATH]

| **ceph-volume** **inventory**

| **ceph-volume** **lvm** [ *trigger* | *create* | *activate* | *prepare*
| *zap* | *list* | *batch*]

| **ceph-volume** **simple** [ *trigger* | *scan* | *activate* ]


描述
====

:program:`ceph-volume` 是一个单用途命令行工具，用于把逻辑卷\
部署为 OSD ，其准备、激活和创建 OSD 的 API 和 ``ceph-disk``
相似。

它与  ``ceph-disk`` 不同的地方是，它不支持交互、或依赖于随同
Ceph 一起安装的 udev 规则。通过这些规则，系统可以自动探测之前\
配置好的各个设备，随后传入 ``ceph-disk`` 以激活它们。


子命令
======

inventory
---------

这个子命令可搜集到主机的物理磁盘清单，并报告它们的元数据。在\
这些元数据中，有与磁盘相关的数据（像型号、尺寸、是机械磁盘还\
是固态的）；还有与 Ceph 相关的，像是否可用于 Ceph 、或是否有\
逻辑卷。

实例： ::

    ceph-volume inventory
    ceph-volume inventory /dev/sda
    ceph-volume inventory --format json-pretty

可选参数：

* [-h, --help]       打印帮助消息、然后退出
* [--format]         输出格式，可用值有 ``plain`` （默认的）、
                     ``json`` 和 ``json-pretty``

lvm
---

通过 LVM 标签， ``lvm`` 子命令可以存储标记，且在稍后重新发现并\
查询与 OSD 有关的各个设备，以便稍后激活它们。

可用子命令：

**batch**
用一串设备创建基于 ``filestore`` 或 ``bluestore`` （默认的）的
OSD 。它会创建 OSD 正常运行所必需的卷组和逻辑卷。

加三个设备的用法实例： ::

    ceph-volume lvm batch --bluestore /dev/sda /dev/sdb /dev/sdc

可选参数：

* [-h, --help]          打印帮助消息、然后退出
* [--bluestore]         使用 bluestore 对象存储器（默认）
* [--filestore]         使用 filestore 对象存储器
* [--yes]               跳过报告和提示，径直开通服务
* [--prepare]           仅仅准备 OSD ，不激活
* [--dmcrypt]           为底层 OSD 设备启用加密功能
* [--crush-device-class] 指定分配给这个 OSD 的 CRUSH 设备类
* [--no-systemd]        不要启用或创建任何 systemd 单元
* [--report]            报告当前输入可能产生的潜在结果（需要\
                        传入设备）
* [--format]            报告时（和 --report 一起使用）的输出格\
                        式，可以是 pretty 或 json 之一
* [--block-db-size]     设置（或覆盖） bluestore_block_db_size
                        的值，单位是字节
* [--journal-size]      覆盖 osd_journal_size 的值，单位是 MB

必需的位置参数：

* <DEVICE>    原始设备的完整路径，如 ``/dev/sda`` 。可以指定\
              多个 ``<DEVICE>`` 设备路径。


**activate**
启用写死了 OSD ID 及其 UUID （在 Ceph CLI 工具里也叫 ``fsid``
）的 systemd 单元，这样，在引导时它就能知道哪个 OSD 被启用、\
且需挂载。

用法： ::

    ceph-volume lvm activate --bluestore <osd id> <osd fsid>

可选参数：

* [-h, --help]    打印帮助消息、然后退出
* [--auto-detect-objectstore]    通过检查 OSD 来自动探测对象存储器
* [--bluestore]   对象存储器是 bluestore （默认的）
* [--filestore]   对象存储器是 filestore
* [--all]         激活系统内找到的所有 OSD
* [--no-systemd]  不要创建、启用 systemd 单元、和启动 OSD 服务

用（ idempotent ） ``--all`` 标记可以一次激活多个 OSD ： ::

    ceph-volume lvm activate --all


**prepare**
准备一个用作 OSD 及其日志（配置为 ``filestore`` 或默认的
``bluestore`` ）的逻辑卷。除了额外增加元数据之外，它不会创建\
或修改逻辑卷。

用法： ::

    ceph-volume lvm prepare --filestore --data <data lv> --journal <journal device>

可选参数：

* [-h, --help]          打印帮助消息、然后退出
* [--journal JOURNAL]   一个逻辑组名字、逻辑卷路径、或设备路径
* [--bluestore]         使用 bluestore 对象存储器（默认的）
* [--block.wal]         bluestore block.wal 的逻辑卷或分区路径
* [--block.db]          bluestore block.db 的逻辑卷或分区路径
* [--filestore]         使用 filestore 对象存储器
* [--dmcrypt]           为底层 OSD 设备启用加密功能
* [--osd-id OSD_ID]     重用已有的 OSD id
* [--osd-fsid OSD_FSID] 重用已有的 OSD fsid
* [--crush-device-class] 指定分配给这个 OSD 的 CRUSH 设备类

必需参数：

* --data                一个逻辑组名字、或一个逻辑卷路径

要加密 OSD 的话，在准备时必须加上 ``--dmcrypt`` 标志（
``create`` 子命令里也支持）。


**create**
把开通新 OSD 的两步过程（先调用 ``prepare`` 之后 ``activate``
）包装成一步。倾向于使用 ``prepare`` 再 ``activate`` 的原因是\
为了把新 OSD 们缓慢地加入集群，以避免大量数据被重新均衡。

这个单步调用过程统一了 ``prepare`` 和 ``activate`` 所做的事\
情，为简便起见，它一次完成。选项和常规用法与 ``prepare`` 和
``activate`` 子命令的基本一样。


**trigger**
这个子命令不是给用户直接使用的，是给 systemd 用的，它会分析
systemd 发来的输入、探测与 OSD 关联的 UUID 和 ID ，然后代理给
``ceph-volume lvm activate`` 。

用法： ::

    ceph-volume lvm trigger <SYSTEMD-DATA>

systemd “数据”应该按如下格式： ::

    <OSD ID>-<OSD UUID>

与 OSD 关联过的逻辑卷应该预先准备好，也就是所需的标签和元数据\
必须已备好。

位置参数：

* <SYSTEMD_DATA>  来自 systemd 单元的数据包含 OSD 的 ID 和 UUID 。


**list**
罗列与 Ceph 关联的设备或逻辑卷，即设备是否有与 OSD 相关的\
信息。通过查询 LVM 的元数据，建立 OSD 与设备的关系。

与 OSD 关联的逻辑卷必须是经过 ceph-volume 准备过的，这样它才会\
有所需的标签和元数据。

用法： ::

    ceph-volume lvm list

罗列一个特定的设备，报告与之相关的所有元数据： ::

    ceph-volume lvm list /dev/sda1

罗列一个逻辑卷、以及它的所有元数据（ vg 是卷组、 lv 是逻辑卷\
名字）： ::

    ceph-volume lvm list {vg/lv}

位置参数：

* <DEVICE>  逻辑卷的话要按格式 ``vg/lv`` ；常规设备为路径
  ``/path/to/sda1`` 或 ``/path/to/sda`` 。


**zap**
删除指定的逻辑卷或分区。如果指定的是逻辑卷路径，必须按 vg/lv
格式。指定逻辑卷或分区上的文件系统会被删除、所有数据都会被\
清除。

不过，逻辑卷或分区还会保持原样。

对于逻辑卷，用法是： ::

      ceph-volume lvm zap {vg/lv}

对于分区，用法是： ::

      ceph-volume lvm zap /dev/sdc1

要完全删除设备，需加 ``--destroy`` 选项（适用于所有设备类型）： ::

      ceph-volume lvm zap --destroy /dev/sdc1

要删除多个设备，可指定 OSD ID 和/或 OSD FSID ： ::

      ceph-volume lvm zap --destroy --osd-id 1
      ceph-volume lvm zap --destroy --osd-id 1 --osd-fsid C9605912-8395-4D76-AFC0-7DFDAC315D59

位置参数：

* <DEVICE>  逻辑卷的话要按格式 ``vg/lv`` ；常规设备为路径
  ``/path/to/sda1`` 或 ``/path/to/sda`` 。


simple
------

扫描旧 OSD 目录或数据设备，它们可能是由 ceph-disk 创建、或手动\
创建的。

子命令：

**activate**
启用写死了 OSD ID 及其 UUID （在 Ceph CLI 工具里也叫 ``fsid``
）的 systemd 单元，这样，在系统引导时，通过读取之前创建并保存在
``/etc/ceph/osd/`` 内的 JSON 数据，它就能知道哪个 OSD 被启用\
了、且需挂载。

用法： ::

    ceph-volume simple activate --bluestore <osd id> <osd fsid>

可选参数：

* [-h, --help]  打印帮助消息，然后退出
* [--bluestore] 使用 bluestore 对象存储器（默认）
* [--filestore] 使用 filestore 对象存储器

注意： JSON 文件名格式必须是下面这样： ::

    /etc/ceph/osd/<osd id>-<osd fsid>.json


**scan**
扫描一个运行着的 OSD 或数据设备，以收集其元数据，稍后可用于
ceph-volume 激活和管理这个 OSD 。这个扫描命令会创建一个 JSON
文件，其内是必需的信息、还有在 OSD 目录内搜集到的其它信息。

另外， JSON 数据块也可以发到标准输出，以便进一步检查。

扫描所有运行着的 OSD ： ::

    ceph-voume simple scan

扫描数据设备： ::

    ceph-volume simple scan <data device>

扫描运行着的 OSD 的目录： ::

    ceph-volume simple scan <path to osd dir>

可选参数：

* [-h, --help]          打印帮助消息，然后退出
* [--stdout]            把 JSON 数据块发到标准输出
* [--force]             如果目标 JSON 文件已存在，直接覆盖它

必需的位置参数：

* <DATA DEVICE or OSD DIR>  实际的数据分区或指向在运行 OSD 的路径


**trigger**
这个子命令不是给用户直接使用的，是给 systemd 用的，它会分析
systemd 发来的输入、探测与 OSD 关联的 UUID 和 ID ，然后代理给
``ceph-volume simple activate`` 。

用法： ::

    ceph-volume simple trigger <SYSTEMD-DATA>

systemd “数据”应该按如下格式： ::

    <OSD ID>-<OSD UUID>

与 OSD 关联的 JSON 文件应该提前保存到位，通过扫描（或手写），\
以使所需元数据随时可用。

位置参数：

* <SYSTEMD_DATA>  systemd 单元发来的数据，内含 OSD 的 ID 和 UUID


使用范围
========

:program:`ceph-volume` 是 Ceph 的一部分，这是个伸缩力强、\
开源、分布式的存储系统，更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph-osd <ceph-osd>`\(8),
