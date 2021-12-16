:orphan:

======================================
 mount.ceph -- 用于挂载 Ceph 文件系统
======================================

.. program:: mount.ceph

提纲
====

| **mount.ceph** *name*@*fsid*.*fs_name*=/[*subdir*] *dir* [-o *options* ]


描述
====

**mount.ceph** 是在 Linux 主机上挂载 Ceph 文件系统的辅助程序。
它只负责把监视器主机名解析为 IP 地址、从硬盘读取认证密钥，
大多数实际工作由 Linux 内核客户端组件完成。
挂载一个 Ceph 文件系统的命令： ::

  mount.ceph name@07fe3187-00d9-42a3-814b-72a4d5e7d5be.fs_name=/ /mnt/mycephfs -o mon_addr=1.2.3.4

挂载辅助程序可以从 ceph 配置文件读取集群的 FSID 。
我们建议通过 mount(8) 调用挂载辅助程序，像这样： ::

  mount -t ceph name@.fs_name=/ /mnt/mycephfs -o mon_addr=1.2.3.4

注意，这里的点 ``.`` 仍然是必要的，它是设备字符串的一部分。

mount 命令的第一个参数是设备部分，
它包括用来认证的 RADOS 用户名、文件系统名和
CephFS 内要挂载到挂载点的路径。

监视器地址可以通过 ``mon_addr`` 挂载选项传入，
可以传入多个监视器地址，用斜杠（ `/` ）分隔开即可。
只要有一个监视器地址就能成功挂载，
客户端可以从任意响应的监视器了解到所有监视器。
即便如此，最好指定多个监视器，免得正挂载的时候遇上那个挂了。
监视器地址的格式是 ip_address[:port] ，
如果端口没指定，就用默认端口 6789 。

如果没指定监视器地址，那么 **mount.ceph** 就会尝试用\
本地配置文件、和/或 DNS SRV 记录来获取监视器地址。
类似地，如果 Ceph 集群开启了认证（用的是 CephX ），
但命令却没指定 ``secret`` 和 ``secretfile`` ，
挂载辅助程序就会派生出一个子进程，
它会用标准的 Ceph 库例程去寻找密钥环、并从中获取密钥
（如果没指定监视器地址和 FSID ，也会一并寻找）。

可以挂载文件系统的子目录， mount 时，在设备部分的 "=" 后面、
紧跟着指定子目录的（绝对）路径即可。

mount 辅助程序的惯例是前两个选项分别为要挂载的设备和目标路径，
其它选项必须位于这些固定参数之后。


选项
====

基础的
------
.. Basic

:command:`conf`
    ceph.cong 文件的路径。这是用于初始化 Ceph 上下文的，
    之后就能自动发现监视器和认证密钥了。
    默认会在标准路径内搜寻 ceph.conf 文件。

:command:`mount_timeout`
    整数（秒）。默认：60

:command:`ms_mode=<legacy|crc|secure|prefer-crc|prefer-secure>`
    设置客户端用来传输的连接模式，
    可用模式有：

    - ``legacy``: 使用 v1 信使协议和集群通讯；

    - ``crc``: 使用 v2 信使协议，没有线路加密；

    - ``secure``: 使用 v2 信使协议，有线路加密；

    - ``prefer-crc``: crc 模式，如果拒绝就是用 secure 模式；

    - ``prefer-secure``: secure 模式，如果拒绝就是用 crc 模式；

:command:`mon_addr`
    集群监视器的地址，格式是 ip_address[:port] 。

:command:`fsid`
    集群的 FSID ，可以用 `ceph fsid` 命令找到。

:command:`secret`
    用于 CephX 的密钥。这个选项不安全，因为它把密钥暴露在了命令行，
    用 secretfile 选项可避免此问题。

:command:`secretfile`
    包含密钥的文件路径， CephX 要用到。

:command:`recover_session=<no|clean>`
    设置重连模式，以防客户端被屏蔽。
    可用的模式有 ``no`` 和 ``clean`` ，默认是 ``no`` 。

    - ``no``: 客户端探测到自己被屏蔽后永远不要尝试重连。
      被屏蔽的客户端不会尝试重连，并且它们的操作也会失败。

    - ``clean``: 客户端们探测到自己被屏蔽后会自动重连到 Ceph 集群。
      在重连期间，客户端会丢弃脏数据、元数据，使得页缓存和可写的文件句柄失效。
      重连后，各文件锁会落伍，因为 MDS 失去了对它们的追踪。
      如果一个 inode 内有落伍的文件锁，所有落伍文件锁释放之前，
      这个 inode 不允许读写。


高级的
------
.. Advanced

:command:`cap_release_safety`
    整数。默认：自行计算

:command:`caps_wanted_delay_max`
    整数，能力释放延迟时间。默认：60

:command:`caps_wanted_delay_min`
    整数，能力释放延迟时间。默认：5

:command:`dirstat`
    用 `cat dirname` 读取文件信息。默认： off

:command:`nodirstat`
    不用 `cat dirname` 读取文件信息

:command:`ip`
    本机 IP

:command:`noasyncreaddir`
    读目录时不经过 dcache

:command:`nocrc`
    写入时不做 crc 校验

:command:`noshare`
    创建新客户端例程，而不是和挂载同一集群的例程共享资源。

:command:`osdkeepalive`
    整数。默认：5

:command:`osd_idle_ttl`
    整数（秒）。默认：60

:command:`rasize`
    整数（字节数），最大预读尺寸，默认： 8388608 (8192*1024)

:command:`rbytes`
    目录的 st_size 报告产生于目录内容的递归尺寸。默认： on

:command:`norbytes`
    目录的 st_size 无需通过递归目录内容来获取。

:command:`readdir_max_bytes`
    整数。默认： 524288 （ 512*1024 ）

:command:`readdir_max_entries`
    整数。默认： 1024

:command:`rsize`
    整数（字节数），最大读尺寸。默认： 16777216 (16*1024*1024)

:command:`snapdirname`
    字符串，为快照的隐藏目录设置个名字。默认： .snap

:command:`write_congestion_kb`
    整数（ kb ），运行中的最大回写量，随可用内存变化。\
    默认：根据可用内存计算

:command:`wsize`
    整数（字节数），最大写尺寸。默认： 16777216 (16*1024*1024)
    （回写用较小的 wsize 和条带单元）

:command:`wsync`
    同步地执行所有命名空间操作。
    这能确保只有收到 MDS 的回复才算命名空间操作完成。这是默认的。

:command:`nowsync`
    允许客户端异步执行命名空间操作。
    启用此选项后，命名空间操作在收到 MDS 的回复前就可以完成，
    如果它有足够的能力这样做。


实例
====

挂载整个文件系统： ::

    mount -t ceph fs_user@.mycephfs2=/ /mnt/mycephfs

只挂载命名空间、文件系统的一部分： ::

    mount.ceph fs_user@.mycephfs2=/some/directory/in/cephfs /mnt/mycephfs

传入监视器主机的 IP 地址，可选的： ::

    mount.ceph fs_user@.mycephfs2=/ /mnt/mycephfs -o mon_addr=192.168.0.1

如果端口不是标准的，随 IP 传入端口： ::

    mount.ceph fs_user@.mycephfs2=/ /mnt/mycephfs -o mon_addr=192.168.0.1:7000

如果有多个监视器，传入时用 `/` 分隔开： ::

    mount.ceph fs_user@.mycephfs2=/ /mnt/mycephfs -o mon_addr=192.168.0.1/192.168.0.2/192.168.0.3

传入 CephX 用户的密钥，可选的： ::

    mount.ceph fs_user@.mycephfs2=/ /mnt/mycephfs -o secret=AQATSKdNGBnwLhAAnNDKnH65FmVKpXZJVasUeQ==

传入包含密钥的文件，以免把密钥留在 shell 命令历史里： ::

    mount.ceph fs_user@.mycephfs2=/ /mnt/mycephfs -o secretfile=/etc/ceph/fs_username.secret

如果 Ceph 集群关闭了认证，忽略与凭证相关的选项： ::

    mount.ceph fs_user@.mycephfs2=/ /mnt/mycephfs


使用范围
========

**mount.ceph** 是 Ceph 的一部分，这是个伸缩力强、开源、\
分布式的存储系统，更多信息参见 https://docs.ceph.com 。


功能适用范围
============
.. Feature Availability

``recover_session=`` 选项是在 v5.4 加进主线内核的。
``wsync`` 和 ``nowsync`` 是在 v5.7 加入的。


参考
====

:doc:`ceph-fuse <ceph-fuse>`\(8),
:doc:`ceph <ceph>`\(8)
