:orphan:

======================================
 mount.ceph -- 用于挂载 Ceph 文件系统
======================================

.. program:: mount.ceph

提纲
====

| **mount.ceph** *monaddr1*\ [,\ *monaddr2*\ ,...]:/[*subdir*] *dir* [
  -o *options* ]


描述
====

**mount.ceph** 是在 Linux 主机上挂载 Ceph 文件系统的简单助手。它只负责把监视\
器主机名解析为 IP 地址、从硬盘读取认证密钥，大多数实际工作由 Linux 内核客户\
端组件完成。事实上，无需认证的 Ceph 文件系统无需 mount.ceph 也能挂载，只要指\
定监视器 IP 地址即可： ::

	mount -t ceph 1.2.3.4:/ mountpoint

各监视器地址 monaddr 的格式为 host[:port] ，如果端口未指定，就用默认的 6789 。

多个监视器地址用逗号分隔。要成功地挂载只需一个监视器即可，客户端将从某个能响\
应的监视器获知其它监视器。然而最好指定多个监视器，以免挂载时正好赶上那个监视\
器挂了。

如果要挂载文件系统的一子集，可指定一个子目录 subdir 。

mount 助手程序的惯例是前两个选项分别为要挂载的设备和目标路径，其它选项必须位\
于这些固定参数之后。


选项
====

:command:`wsize`
  整数，最大写尺寸。默认：无（回写用较小的 wsize 和条带单元）

:command:`rsize`
  整数（字节），最大预读， 1024 的倍数。默认： 524288 （ 512*1024 ）

:command:`osdtimeout`
  整数（秒）。默认：60

:command:`osdkeepalivetimeout`
  整数。默认：5

:command:`mount_timeout`
  整数（秒）。默认：60

:command:`osd_idle_ttl`
  整数（秒）。默认：60

:command:`caps_wanted_delay_min`
  整数，能力释放延迟时间。默认：5

:command:`caps_wanted_delay_max`
  整数，能力释放延迟时间。默认：60

:command:`cap_release_safety`
  整数。默认：自行计算

:command:`readdir_max_entries`
  整数。默认： 1024

:command:`readdir_max_bytes`
  整数。默认： 524288 （ 512*1024 ）

:command:`write_congestion_kb`
  整数（ kb ），运行中的最大回写量，随可用内存变化。默认：根据可用内存计算

:command:`snapdirname`
  字符串，为快照的隐藏目录设置个名字。默认： .snap

:command:`name`
  使用 cephx 认证时的 RADOS 用户名。默认： guest

:command:`secret`
  用于 cephx 的密钥。这个选项不安全，因为它把密钥暴露在了命令行，用 \
  secretfile 选项可避免此问题。

:command:`secretfile`
  用于 cephx 的密钥文件路径。

:command:`ip`
  本机 IP

:command:`noshare`
  创建新客户端例程，而不是和挂载同一集群的例程共享资源。

:command:`dirstat`
  用 `cat dirname` 读取文件信息。默认： off

:command:`nodirstat`
  不用 `cat dirname` 读取文件信息

:command:`rbytes`
  目录的 st_size 报告产生于目录内容的递归尺寸。默认： on

:command:`norbytes`
  目录的 st_size 无需通过递归目录内容来获取。

:command:`nocrc`
  写入时不做 crc 校验

:command:`noasyncreaddir`
  读目录时不经过 dcache


实例
====

挂载整个文件系统： ::

        mount.ceph monhost:/ /mnt/foo

如果有多个监视器： ::

        mount.ceph monhost1,monhost2,monhost3:/ /mnt/foo

如果 :doc:`ceph-mon <ceph-mon>`\(8) 运行于非默认端口： ::

        mount.ceph monhost1:7000,monhost2:7000,monhost3:7000:/ /mnt/foo

只挂载文件系统命名空间的一部分： ::

        mount.ceph monhost1:/some/small/thing /mnt/thing

假设 mount.ceph(8) 安装正确， mount(8) 应该能自动调用它： ::

        mount -t ceph monhost:/ /mnt/foo


使用范围
========

**mount.ceph** 是 Ceph 的一部分，这是个伸缩力强、开源、分布式的存储系统，\
更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph-fuse <ceph-fuse>`\(8),
:doc:`ceph <ceph>`\(8)
