:orphan:

==================================
 ceph-disk -- Ceph 的硬盘管理工具
==================================

.. program:: ceph-disk

提纲
====

| **ceph-disk** **prepare** [--cluster *clustername*] [--cluster-uuid *uuid*]
	[--fs-type *xfs|ext4|btrfs*] [*data-path*] [*journal-path*]

| **ceph-disk** **activate** [*data-path*] [--activate-key *path*]
	[--mark-init *sysvinit|upstart|systemd|auto|none*]
	[--no-start-daemon] [--reactivate]

| **ceph-disk** **activate-all**

| **ceph-disk** **list**

| **ceph-disk** **deactivate** [--cluster *clustername*] [*device-path*]
        [--deactivate-by-id *id*] [--mark-out]

| **ceph-disk** **destroy** [--cluster *clustername*] [*device-path*]
        [--destroy-by-id *id*] [--dmcrypt-key-dir *KEYDIR*] [--zap]


描述
====

:program:`ceph-disk` 工具可把硬盘、分区或目录预处理并激活为 OSD 。\
可单独使用，也可由 :program:`ceph-deploy` 或 ``udev`` 调用，还可\
由其他部署工具，如 ``Chef`` 、 ``Juju`` 、 ``Puppet`` 调用。

其实它是把手动创建并启动 OSD 的步骤自动化成了两步：预处理和激活，\
分别对应两个子命令 ``prepare`` 和 ``activate`` 。

:program:`ceph-disk` 也把手动停止和销毁某一 OSD 的多个步骤自动化\
成了两步：去激活和销毁，分别对应 ``deactivate`` 和 ``destroy`` \
子命令。


子命令
======


prepare
-------

预处理用作 Ceph OSD 的目录、磁盘。它会创建 GPT 分区、给分区打上 \
Ceph 风格的 ``uuid`` 标记、创建文件系统、把此文件系统标记为已就\
绪、使用日志磁盘的整个分区并新增一分区。可单独使用，也可由 \
:program:`ceph-deploy` 用。

用法： ::

	ceph-disk prepare --cluster [cluster-name] --cluster-uuid [uuid]
	--fs-type [ext4|xfs|btrfs] [data-path] [journal-path]

此子命令也支持这些选项： :option:`--osd-uuid` 、 \
:option:`--journal-uuid` 、 :option:`--zap-disk` 、 \
:option:`--data-dir` 、 :option:`--data-dev` 、 \
:option:`--journal-file` 、 :option:`--journal-dev` 、 \
:option:`--dmcrypt` 和 :option:`--dmcrypt-key-dir` 。


activate
--------

激活 Ceph OSD 。先把此卷挂载到一临时位置，分配 OSD 惟一标识符（若\
有必要），重挂载到正确位置 ``/var/lib/ceph/osd/$cluster-$id`` 并\
启动 ceph-osd 。当 udev 看到有 OSD 标记的 GPT 分区时，或者用 \
``ceph disk activate-all`` 启动 Ceph 服务时会触发此命令。可单独使\
用，也可由 :program:`ceph-deploy` 调用。

用法： ::

	ceph-disk activate [PATH]

这里的 [PATH] 是某一块设备或目录的路径。

若 OSD 节点上没有 ``/var/lib/ceph/bootstrap-osd/{cluster}.keyring`` \
副本，必须给此子命令额外指定 :option:`--activate-key` 选项。

用法： ::

	ceph-disk activate [PATH] [--activate-key PATH]

此子命令还支持 :option:`--mark-init` 选项。 ``--mark-init`` 选项\
赋予了 init 系统管理 OSD 目录的能力。默认值是 ``auto`` ，也就是\
它会自动探测适合 ceph 的 init 系统（可以是 ``sysvinit`` 、 \
``systemd`` 或者 ``upstart`` ），指定参数后就可以忽略 init 系统。\
当操作系统支持多种 init 系统时，有这个选项就方便多了，比如 \
Debian GNU/Linux jessie 就同时支持 ``systemd`` 和 ``sysvinit`` 。\
如果参数是 ``none`` ，那就不会给此 OSD 标记任何 init 系统，而且\
每次重启之后必须显式地调用 ``ceph-disk activate`` 。

用法： ::

	ceph-disk activate [PATH] [--mark-init *sysvinit|upstart|systemd|auto|none*]

如果加了 :option:`--no-start-daemon` 选项，就只进行激活，而不启\
动 OSD 守护进程。

用 :option:`--reactivate` 选项可以重新激活已经被 ``deactivate`` \
子命令去激活的 OSD 。

用法： ::

	ceph-disk activate [PATH] [--reactivate]


activate-journal
----------------

通过其日志设备激活一 OSD ， ``udev`` 会基于分区类型触发 \
``ceph-disk activate-journal <dev>`` 命令。

用法： ::

	ceph-disk activate-journal [DEV]

这里的 [DEV] 是日志块设备的路径。

此子命令支持的其他选项： :option:`--activate-key` 、 \
:option:`--mark-init` 。

``--mark-init`` 选项赋予了 init 系统管理 OSD 目录的能力。

用法： ::

	ceph-disk activate-journal [--activate-key PATH] [--mark-init INITSYSTEM] [DEV]


activate-all
------------

激活所有标记的 OSD 分区。 ``activate-all`` 靠 \
``/dev/disk/by-parttype-uuid/$typeuuid.$uuid`` 发现所有分区， Ceph 安装了专\
用的 ``udev`` 规则来创建这些链接。此命令可在 Ceph 服务启动时触发、或直接运行。

用法： ::

	ceph-disk activate-all

此子命令支持的其他选项： :option:`--activate-key` 、 :option:`--mark-init` 。

``--mark-init`` 选项赋予了 init 系统管理 OSD 目录的能力。

用法： ::

	ceph-disk activate-all [--activate-key PATH] [--mark-init INITSYSTEM]


list
----

列出硬盘分区和 OSD 。可单独使用，也可由 :program:`ceph-deploy` 调用。

用法： ::

	ceph-disk list


suppress-activate
-----------------

禁止一设备（前缀）激活。用类似 ``/var/lib/ceph/tmp/suppress-activate.sdb`` \
的文件标记不想激活的设备，此文件的最后一位是禁止的设备名（ /dev/X 去掉 /dev/ \
前缀）。函数 ``is_suppressed()`` 会检查并匹配前缀（不含 /dev/ ），也就是说禁\
止 sdb 的同时也禁止了 sdb1 、 sdb2 等设备。

用法： ::

	ceph-disk suppress-activate [PATH]

这里的 [PATH] 是某一块设备或目录的路径。


unsuppress-activate
-------------------

取消某设备（前缀）的禁止激活配置。可用于激活之前用 ``suppress-activate`` 禁\
止的设备。

用法： ::

	ceph-disk unsuppress-activate [PATH]

这里的 [PATH] 是某一块设备或目录的路径。


deactivate
----------

去激活 Ceph OSD 。它会停止 OSD 守护进程、还可以同时标记为 out 。\
此 OSD 的内容还保留着，但是 *ready* 、 *active* 、\ *初始化相关的*\
文件都删除了（这样它就不会被 ``udev`` 规则自动重激活了），并且创\
建了 deactive 文件以示此 OSD 被去激活了；如果此 OSD 是 dmcrypt ，\
还会删除数据的 dmcrypt 映射图。去激活完成后， OSD 状态为 \
``down`` 。已经去激活的 OSD 还可以用 ``activate`` 子命令加 \
:option:`--reactivate` 选项重新激活。

用法： ::

	ceph-disk deactivate [PATH]

这里的 [PATH] 是块设备或目录的路径。

这个子命令还支持 :option:`--mark-out` 选项。 ``--mark-out`` 会把 \
OSD 标记为 out ，其内的对象会被重映射。如果你还不确定要不要销毁这\
个 OSD ，最好别用这个选项。

去激活一个 OSD 时你也可以指定一个 OSD 的 ID ，要用到 \
:option:`--deactivate-by-id` 选项。

用法： ::

	ceph-disk deactivate --deactivate-by-id [OSD-ID]

destroy
-------

销毁 Ceph OSD 。它会从集群、 crushmap 删除 OSD ，并回收 OSD ID 。\
它只能销毁处于 *down* 状态的 OSD 。

用法： ::

	ceph-disk destroy [PATH]

这里的 [PATH] 是块设备或目录的路径。

这个子命令还支持 :option:`--zap` 选项。 ``--zap`` 选项会销毁分\
区表和硬盘内容。

用法： ::

	ceph-disk destroy [PATH] [--zap]

除了路径，你也可以指定 OSD 的 ID ，用 :option:`--destroy-by-id` \
选项。

用法： ::

	ceph-disk destroy --destroy-by-id [OSD-ID]


zap
---

杀死、擦除、销毁一设备的分区表和内容。实际上它用 ``sgdisk`` 加 \
``--zap-all`` 选项来销毁 GPT 和 MBR 数据结构，这样才能重新分区；\
然后用 ``--mbrtogpt`` 选项把 MBR 或 BSD 格式的分区转换为 GPT 格\
式。现在就可以执行 ``prepare`` 子命令来新建 GPT 分区了。可单独使\
用，也可由 :program:`ceph-deploy` 调用。

用法： ::

	ceph-disk zap [DEV]

这里的 [DEV] 是块设备路径。


选项
====

.. option:: --prepend-to-path PATH

   为保持向后兼容性，把 PATH （默认为 ``/usr/bin`` ）加到 $PATH \
   之前。

.. option:: --statedir PATH

   Ceph 配置所在目录（默认为 ``/usr/lib/ceph`` ）。

.. option:: --sysconfdir PATH

   Ceph 配置文件所在目录（默认为 ``/etc/ceph`` ）。

.. option:: --cluster

   为正在预处理的 OSD 指定所在集群的名字。

.. option:: --cluster-uuid

   为正在预处理的 OSD 指定所在集群的 UUID 。

.. option:: --fs-type

   为 OSD 指定文件系统类型，如 ``xfs/ext4/btrfs`` 。

.. option:: --osd-uuid

   给此硬盘分配的全局惟一 OSD UUID 。

.. option:: --journal-uuid

   给日志分配全局惟一的 UUID 。

.. option:: --zap-disk

   销毁分区表和磁盘内容。

.. option:: --data-dir

   验证 ``[data-path]`` 确实是目录。

.. option:: --data-dev

   验证 ``[data-path]`` 确实是块设备。

.. option:: --journal-file

   验证日志是个文件。

.. option:: --journal-dev

   验证日志是个块设备。

.. option:: --dmcrypt

   用 ``dm-crypt`` 加密 ``[data-path]`` 和/或日志设备。

.. option:: --dmcrypt-key-dir

   保存 ``dm-crypt`` 密钥的目录。


使用范围
========

:program:`ceph-disk` 是 Ceph 的一部分，这是个伸缩力强、开源、\
分布式的存储系统，更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph-osd <ceph-osd>`\(8),
:doc:`ceph-deploy <ceph-deploy>`\(8)
