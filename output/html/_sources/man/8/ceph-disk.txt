===================================================
 ceph-disk -- Ceph 上的 OSD 数据硬盘准备和激活工具
===================================================

.. program:: ceph-disk

提纲
====

| **ceph-disk** **prepare** [--cluster *clustername*] [--cluster-uuid *uuid*]
	[--fs-type *xfs|ext4|btrfs*] [*data-path*] [*journal-path*]

| **ceph-disk** **activate** [*data-path*] [--activate-key *path*]

| **ceph-disk** **activate-all**

| **ceph-disk** **list**


描述
====

**ceph-disk** 工具可把硬盘、分区或目录预处理并激活为 OSD 。可单独使用，也可\
由 **ceph-deploy** 或 udev 调用。

其实它是把手动创建并启动 OSD 的步骤自动化成了两步：预处理和激活，分别对应两\
个子命令 **prepare** 和 **activate** 。


子命令
======

**prepare**\ ：预处理用作 Ceph OSD 的目录、硬盘或驱动器。它会创建 GPT 分区、\
给分区打上 Ceph 风格的 UUID 标记、创建文件系统、把此文件系统标记为已就绪、使\
用日志磁盘的整个分区并新增一分区。可单独使用，也可由 **ceph-deploy** 或 udev \
调用。

用法： ceph-disk prepare --cluster [cluster-name] --cluster-uuid [uuid]
--fs-type [ext4|xfs|btrfs] [data-path] [journal-path]

此子命令也支持这些选项： --osd-uuid 、 --journal-uuid 、 --zap-disk 、 \
--data-dir 、 --data-dev 、 --journal-file 、 --journal-dev 、 --dmcrypt 和 \
--dmcrypt-key-dir 。

**activate**\ ：激活 Ceph OSD 。先把此卷挂载到一临时位置，分配 OSD 惟一标识\
符（若有必要），重挂载到正确位置 /var/lib/ceph/osd/$cluster-$id 并启动 \
ceph-osd 。当 udev 看到有 OSD 标记的 GPT 分区时，或者用 \
``ceph disk activate-all`` 启动 Ceph 服务时会触发此命令。可单独使用，也可由 \
**ceph-deploy** 调用。

用法： ceph-disk activate [PATH]

这里的 [PATH] 是块设备或目录的路径。

若 OSD 节点上没有 /var/lib/ceph/bootstrap-osd/{cluster}.keyring 副本，必须给\
此子命令额外指定 [--activate-key PATH] 选项。

用法： ceph-disk activate [PATH] [--activate-key PATH]

此子命令支持的其他选项： --mark-init 。

**activate-journal**\ ：通过其日志设备激活一 OSD ， udev 会基于分区类型触发 \
``ceph-disk activate-journal <dev>`` 命令。

用法： ceph-disk activate-journal [DEV]

这里的 [DEV] 是日志块设备的路径。

此子命令支持的其他选项： --activate-key 、 --mark-init 。

用法： ceph-disk activate-journal [--activate-key PATH]
[--mark-init INITSYSTEM] [DEV]

**activate-all**\ ：激活所有标记的 OSD 分区。 activate-all 靠 \
/dev/disk/by-parttype-uuid/$typeuuid.$uuid 发现所有分区， Ceph 安装了专用的 \
udev 规则来创建这些链接。此命令可在 Ceph 服务启动时触发、或直接运行。

用法： ceph-disk activate-all

此子命令支持的其他选项： --activate-key 、 --mark-init 。

用法： ceph-disk activate-all [--activate-key PATH] [--mark-init INITSYSTEM]

**list**\ ：列出硬盘分区和 OSD 。可单独使用，也可由 **ceph-deploy** 调用。

用法： ceph-disk list

**suppress-activate**\ ：禁止一设备（前缀）激活。用类似 \
/var/lib/ceph/tmp/suppress-activate.sdb 的文件禁止某些设备激活，此文件的最后\
一位是禁止的设备名（ /dev/X 去掉 /dev/ 前缀）。函数 is_suppressed() 会检查并\
匹配前缀（不含 /dev/ ），也就是说禁止 sdb 的同时也禁止了 sdb1 、 sdb2 等设备。

用法： ceph-disk suppress-activate [PATH]

这里的 [PATH] 是块设备或目录的路径。

**unsuppress-activate**\ ：停止某设备（前缀）的禁止激活配置。

用法： ceph-disk unsuppress-activate [PATH]

这里的 [PATH] 是块设备或目录的路径。

**zap**\ ：杀死、擦除、销毁一设备的分区表和内容。实际上它用 ``sgdisk`` 加 \
``--zap-all`` 选项来销毁 GPT 和 MBR 数据结构，这样才能重新分区；然后用 \
``--mbrtogpt`` 选项把 MBR 或 BSD 格式的分区转换为 GPT 格式。现在就可以执行 \
**prepare** 子命令来新建 GPT 分区了。可单独使用，也可由 **ceph-deploy** 调用。

用法： ceph-disk zap [DEV]

这里的 [DEV] 是块设备路径。


选项
====

.. option:: --prepend-to-path PATH

   为保持向后兼容性，把 PATH （默认为 /usr/bin ）加到 $PATH 之前。

.. option:: --statedir PATH

   Ceph 配置所在目录（默认为 /usr/lib/ceph ）。

.. option:: --sysconfdir PATH

   Ceph 配置文件所在目录（默认为 /etc/ceph ）。

.. option:: --cluster

   为正在预处理的 OSD 指定所在集群的名字。

.. option:: --cluster-uuid

   为正在预处理的 OSD 指定所在集群的 UUID 。

.. option:: --fs-type

   为 OSD 指定文件系统类型，如 'xfs/ext4/btrfs' 。

.. option:: --osd-uuid

   给此硬盘分配的全局惟一 OSD UUID 。

.. option:: --journal-uuid

   给日志分配全局惟一的 UUID 。

.. option:: --zap-disk

   销毁分区表和磁盘内容。

.. option:: --data-dir

   验证 [data-path] 确实是目录。

.. option:: --data-dev

   验证 [data-path] 确实是块设备。

.. option:: --journal-file

   验证日志是个文件。

.. option:: --journal-dev

   验证日志是个块设备。

.. option:: --dmcrypt

   用 dm-crypt 加密 [data-path] 和/或日志设备。

.. option:: --dmcrypt-key-dir

   保存 dm-crypt 密钥的目录。

.. option:: --activate-key

   若 OSD 节点上没有 /var/lib/ceph/bootstrap-osd/{cluster}.keyring 副本，可\
   以用此选项追加密钥环路径。

.. option:: --mark-init

   指定用 init 系统管理此 OSD 目录。


使用范围
========

**ceph-disk** 是 Ceph 分布式文件系统的一部分，更多信息参见 http://ceph.com/docs 。
