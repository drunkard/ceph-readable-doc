.. _snap-schedule:

==============
 快照计划模块
==============
.. Snapshot Scheduling Module

该模块为 CephFS 实现了按计划做快照。它提供了一个用户界面，可以添加、查询和删除\
快照计划和保留策略，还提供了一个调度程序，用于拍下快照并相应修剪现有快照。


如何启用
========
.. How to enable

*snap_schedule* 模块可以这样启用： ::

  ceph mgr module enable snap_schedule


用法
====
.. Usage

此模块用到 :doc:`/dev/cephfs-snapshots` ，最好也读一下这篇文档。

此模块的子命令位于 ``ceph fs snap-schedule`` 命名空间下。
参数可以是位置参数或关键字参数。一旦遇到关键字参数，
后面的所有参数也会被认为是关键字参数。

快照计划由路径、重复时间间隔和开始时间确定。
重复间隔定义了两个连续快照之间的时间间隔。
它由一个数字和一个周期乘数指定，是 `h(our)` 、 `d(ay)` 、
`w(eek)` 、 `M(onth)` 和 `y(ear)` 中的一个。
例如， `12h` 的重复间隔表示每 12 小时拍一次快照。
开始时间是一个时间字符串
（下文将详细介绍传递时间）。
默认情况下，开始时间是前一天半夜。
因此，如果在 13:50 添加重复间隔为 `1h` 的快照计划，
并采用默认开始时间，那么第一个快照将在 14:00 拍下。
如果字符串里没有明确指定时区，那就假定时区是 UTC 。
明确指定的时区将在执行时映射成 UTC 时间。
开始时间必须按 ISO8601 格式。示例如下：

- UTC：2022-08-08T05:30:00 即上午 5:30 UTC ，没指定时区偏移
- IDT：2022-08-08T09:00:00+03:00 即上午 6:00 UTC
- EDT：2022-08-08T05:30:00-04:00 即上午 9:30 UTC

保留规范由路径和保留规范本身标识。
保留规范由一个数字和一个时间段组成，
中间用空格隔开，或者由 `<数字><时间段>` 对拼接而成。
其含义是，该规范将确保保留至少相隔 `<时间段>` 的 `<数字>` 个快照。
例如， `7d` 表示用户希望保留 7 个快照，
这些快照之间至少相隔一天（也可能更长）。
支持的时间段有： `h(our)` 、 `d(ay)` 、 `w(eek)` 、 `M(onth)` 、
`y(ear)` 和 `n` ，最后一个是个特殊的修饰符，
例如 `10n` 意思是保留最后 10 个快照，而不是按时间算、

所有子命令都接受可选的 `fs` 参数，用于指定\
多文件系统配置和 :doc:`/cephfs/fs-volumes` 管理配置中的路径。
如果没有传递 `fs` 参数，则默认使用 fs_map 中列出的第一个文件系统。
使用 :doc:`/cephfs/fs-volumes` 时，参数 `fs` 等同于 `volume` 。

当传入时间戳时（ `add` 、 `remove` 、 `activate` 和 `deactivate` 子命令中的 `start` 参数），
ISO 格式 `%Y-%m-%dT%H:%M:%S` 最通用。
如果用的是 python3.7 或更高版本，
或安装了 https://github.com/movermeyer/backports.datetime_fromisoformat ，
则任何能被 python 的 `datetime.fromisoformat` 解析的有效 ISO 时间戳都是可以用的。

没有提供子命令时，将打印出命令提纲： ::

  #> ceph fs snap-schedule
  no valid command found; 8 closest matches:
  fs snap-schedule status [<path>] [<fs>] [<format>]
  fs snap-schedule list <path> [--recursive] [<fs>] [<format>]
  fs snap-schedule add <path> <snap_schedule> [<start>] [<fs>]
  fs snap-schedule remove <path> [<repeat>] [<start>] [<fs>]
  fs snap-schedule retention add <path> <retention_spec_or_period> [<retention_count>] [<fs>]
  fs snap-schedule retention remove <path> <retention_spec_or_period> [<retention_count>] [<fs>]
  fs snap-schedule activate <path> [<repeat>] [<start>] [<fs>]
  fs snap-schedule deactivate <path> [<repeat>] [<start>] [<fs>]
  Error EINVAL: invalid command

注意
^^^^
.. Note:

此命令不再支持 `subvolume` 参数。


查验快照计划
------------
.. Inspect snapshot schedules

此模块有两个子命令来检查当前的计划表： `list` 和 `status` 。
用可选的 `format` 参数选择纯文本或 json 格式的输出，
默认为纯文本。`list` 子命令会以简短的单行格式列出路径上的所有计划表。
它提供了一个 `recursive` （递归）参数，
可以列出指定目录和及其包含目录中的所有计划表。
`status` 子命令会打印一个路径的所有可用计划表和保留规格。

实例： ::

  ceph fs snap-schedule status /
  ceph fs snap-schedule status /foo/bar --format=json
  ceph fs snap-schedule list /
  ceph fs snap-schedule list / --recursive=true # list all schedules in the tree


增加或删除计划
--------------
.. Add and remove schedules

`add` 和 `remove` 子命令分别用于添加和删除快照计划。二者都需要\
至少一个 `path` 参数， `add` 还需要一个 `schedule` 参数，如 USAGE 章节所述。

一个路径可以添加多个不同的计划表。
如果两个计划表的重复时间间隔和开始时间不同，则认为它们彼此不同。

如果在路径上设置了多个计划表，指定精确的重复间隔和开始时间后，
`remove` 可以删除一个路径的单个计划，或者在只指定 `path` （路径）时，
这个子命令会删除此路径的所有计划。

实例： ::

  ceph fs snap-schedule add / 1h
  ceph fs snap-schedule add / 1h 11:55
  ceph fs snap-schedule add / 2h 11:55
  ceph fs snap-schedule remove / 1h 11:55 # removes one single schedule
  ceph fs snap-schedule remove / 1h # removes all schedules with --repeat=1h
  ceph fs snap-schedule remove / # removes all schedules on path /

增加或删除保留策略
------------------
.. Add and remove retention policies

`retention add` 和 `retention remove` 子命令允许管理保留策略。
一个路径只有一个保留策略，
但一个策略可以包含多个计时对，以便指定复杂的保留策略。
可以通过 `ceph fs snap-schedule retention add <path> <time period> <count>`
和 `ceph fs snap-schedule retention add <path> <countTime period>[countTime period]`
单独地或批量地添加、删除保留策略。

实例： ::

  ceph fs snap-schedule retention add / h 24 # keep 24 snapshots at least an hour apart
  ceph fs snap-schedule retention add / d 7 # and 7 snapshots at least a day apart
  ceph fs snap-schedule retention remove / h 24 # remove retention for 24 hourlies
  ceph fs snap-schedule retention add / 24h4w # add 24 hourly and 4 weekly to retention
  ceph fs snap-schedule retention remove / 7d4w # remove 7 daily and 4 weekly, leaves 24 hourly

.. note:: 向 snap-schedule 添加路径时，记住去掉挂载点的路径前缀。
   给 snap-schedule 指定的路径应该是从相应的 CephFS 文件系统根目录开始，
   而不是从客户端主机的文件系统根目录开始。例如，
   假设 Ceph 文件系统挂载在 ``/mnt`` ，而需要拍快照的路径是 ``/mnt/some/path`` ，
   那么 snap-schedule 所需的实际路径就只有 ``/some/path`` 。

.. note:: 需要注意的是， snap-schedule status 命令输出中\
   的“ created ”字段是创建计划时的时间戳。
   created 时间戳与实际快照的创建无关。
   实际快照创建情况在 created_count 字段中说明，
   该字段是迄今为止创建的快照总数的累积计数。

.. note:: 每个目录保留快照的最大数量由配置选项
   `mds_max_snaps_per_dir` 限制，其默认值为 100 。
   为确保能创建新快照，将保留比该数量少一个的快照。
   因此，默认情况下最多保留 99 个快照。

.. note:: 如果有多个文件系统，还得加上 --fs 参数。


有效和无效的计划
----------------
.. Active and inactive schedules

可以为目录树中还不存在的路径添加快照计划。
同样，删除路径也不会影响该路径的快照计划。
如果按计划拍快照时目录不存在，
此计划将被设置为无效（ inactive ），
并将被排除在计划调度之外，直到再次激活为止。
可以手动把计划设置为无效，以暂停按计划创建快照。
本模块为此功能提供了 `activate` （激活）\
和 `deactivate` （停用）子命令。

实例： ::

  ceph fs snap-schedule activate / # activate all schedules on the root directory
  ceph fs snap-schedule deactivate / 1d # deactivates daily snapshots on the root directory


局限性
------
.. Limitations

快照使用 python Timer （计时器）进行调度。
在正常情况下，指定 1h 的计划会让拍快照相当精确地间隔 1 小时。
但是，如果 mgr 守护进程负载过重，定时器线程可能无法立即调度，
导致快照稍有延迟。如果出现这种情况，
下一个快照仍将照旧，就像前一个快照没延迟过一样，
即一个或多个延迟的快照不会导致计划表的整体偏移。

如果在卷上快照计划处于活动状态时删除了卷，那么在这些卷上执行命令时，
可能会在日志文件或命令行中看到 Python Tracebacks （回溯）。
虽然我们已采取措施考虑 fs_map 的变化、并删除活跃的计时器、
关闭数据库连接，以避免 Python 回溯，但由于问题的固有性质，不可能完全消除回溯。
在出现此类回溯的情况下，要使系统达到稳定状态，
唯一的解决办法就是禁用并重新启用 snap_schedule 管理器模块。

为了在一定程度上限制文件系统中快照的总体数量，
该模块只为每个目录保留最多 50 个快照。
如果保留策略想要保留的快照超过 50 个，保留列表将缩短成最新的 50 个快照。

数据存储
--------
.. Data storage

快照计划数据存储在 cephfs 元数据存储池里的一个 rados 对象中。运行时，
所有数据都放在一个 sqlite 数据库中；存储时，就被序列化成一个 rados 对象。
