:orphan:

======================================
 mount.ceph -- 用于挂载 Ceph 文件系统
======================================

.. program:: mount.ceph

提纲
====

| **mount.ceph** [*mon1_socket*\ ,\ *mon2_socket*\ ,...]:/[*subdir*] *dir* [
  -o *options* ]


描述
====

**mount.ceph** 是在 Linux 主机上挂载 Ceph 文件系统的辅助程序。\
它只负责把监视器主机名解析为 IP 地址、从硬盘读取认证密钥，\
大多数实际工作由 Linux 内核客户端组件完成。其实，不需要认证的
Ceph 文件系统无需 mount.ceph 也能挂载，只要指定监视器 IP 地址\
即可： ::

	mount -t ceph 1.2.3.4:/ mountpoint

mount 命令的第一个参数是设备部分，它包括主机的套接字和 CephFS
内要挂载到挂载点的路径。套接字，格式显然是 ip_address[:port] ，\
如果没指定端口，就假设是 Ceph 默认的 6789 。多个监视器地址用\
逗号分隔。要成功地挂载只需一个监视器即可，客户端将从某个能\
响应的监视器了解到其它的监视器。然而最好指定多个监视器，以免\
挂载时正好赶上那个监视器挂了。

If the host portion of the device is left blank, then **mount.ceph** will
attempt to determine monitor addresses using local configuration files
and/or DNS SRV records. In similar way, if authentication is enabled on Ceph
cluster (which is done using CephX) and options ``secret`` and ``secretfile``
are not specified in the command, the mount helper will spawn a child process
that will use the standard Ceph library routines to find a keyring and fetch
the secret from it.

A sub-directory of the file system can be mounted by specifying the (absolute)
path to the sub-directory right after ":" after the socket in the device part
of the mount command.

mount 辅助程序的惯例是前两个选项分别为要挂载的设备和目标路径，\
其它选项必须位于这些固定参数之后。


选项
====

.. Basic

基础的
------
:command:`conf`
    Path to a ceph.conf file. This is used to initialize the Ceph context
    for autodiscovery of monitor addresses and auth secrets. The default is
    to use the standard search path for ceph.conf files.

:command: `fs=<fs-name>`
    Specify the non-default file system to be mounted. Not passing this
    option mounts the default file system.

:command: `mds_namespace=<fs-name>`
    A synonym of "fs=" and its use is deprecated.

:command:`mount_timeout`
    整数（秒）。默认：60

:command:`name`
    使用 cephx 认证时的 RADOS 用户名。默认： guest

:command:`secret`
    secret key for use with CephX. This option is insecure because it exposes
    the secret on the command line. To avoid this, use the secretfile option.

:command:`secret`
    用于 cephx 的密钥。这个选项不安全，因为它把密钥暴露在了命令行，用 \
    secretfile 选项可避免此问题。

:command:`secretfile`
    path to file containing the secret key to use with CephX

:command:`recover_session=<no|clean>`
    Set auto reconnect mode in the case where the client is blocklisted. The
    available modes are ``no`` and ``clean``. The default is ``no``.

    - ``no``: never attempt to reconnect when client detects that it has been
       blocklisted. Blacklisted clients will not attempt to reconnect and
       their operations will fail too.

    - ``clean``: client reconnects to the Ceph cluster automatically when it
      detects that it has been blocklisted. During reconnect, client drops
      dirty data/metadata, invalidates page caches and writable file handles.
      After reconnect, file locks become stale because the MDS loses track of
      them. If an inode contains any stale file locks, read/write on the inode
      is not allowed until applications release all stale file locks.


.. Advanced

高级的
------
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

:command:`osdtimeout`
    整数（秒）。默认：60

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
    整数（ kb ），运行中的最大回写量，随可用内存变化。默认：根据可用内存计算

:command:`wsize`
    整数（字节数），最大写尺寸。默认： 16777216 (16*1024*1024)
    （回写用较小的 wsize 和条带单元）

:command:`wsync`
    Execute all namespace operations synchronously. This ensures that the
    namespace operation will only complete after receiving a reply from
    the MDS. This is the default.

:command:`nowsync`
    Allow the client to do namespace operations asynchronously. When this
    option is enabled, a namespace operation may complete before the MDS
    replies, if it has sufficient capabilities to do so.


实例
====

挂载整个文件系统： ::

        mount.ceph :/ /mnt/mycephfs

假设 mount.ceph 安装得没问题， mount(8) 应该能自动调用它： ::

    mount -t ceph :/ /mnt/mycephfs

Mount only part of the namespace/file system::

    mount.ceph :/some/directory/in/cephfs /mnt/mycephfs

Mount non-default FS, in case cluster has multiple FSs::
    mount -t ceph :/ /mnt/mycephfs2 -o fs=mycephfs2
    
    or
    
    mount -t ceph :/ /mnt/mycephfs2 -o mds_namespace=mycephfs2 # This option name is deprecated.

Pass the monitor host's IP address, optionally::

    mount.ceph 192.168.0.1:/ /mnt/mycephfs

Pass the port along with IP address if it's running on a non-standard port::

    mount.ceph 192.168.0.1:7000:/ /mnt/mycephfs

If there are multiple monitors, passes addresses separated by a comma::

   mount.ceph 192.168.0.1,192.168.0.2,192.168.0.3:/ /mnt/mycephfs

If authentication is enabled on Ceph cluster::

    mount.ceph :/ /mnt/mycephfs -o name=fs_username

Pass secret key for CephX user optionally::

    mount.ceph :/ /mnt/mycephfs -o name=fs_username,secret=AQATSKdNGBnwLhAAnNDKnH65FmVKpXZJVasUeQ==

Pass file containing secret key to avoid leaving secret key in shell's command
history::

    mount.ceph :/ /mnt/mycephfs -o name=fs_username,secretfile=/etc/ceph/fs_username.secret


使用范围
========

**mount.ceph** 是 Ceph 的一部分，这是个伸缩力强、开源、\
分布式的存储系统，更多信息参见 http://ceph.com/docs 。


.. Feature Availability

功能适用范围
============

The ``recover_session=`` option was added to mainline Linux kernels in v5.4.
``wsync`` and ``nowsync`` were added in v5.7.


参考
====

:doc:`ceph-fuse <ceph-fuse>`\(8),
:doc:`ceph <ceph>`\(8)
