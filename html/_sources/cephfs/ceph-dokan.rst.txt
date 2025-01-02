.. _ceph-dokan:

==========================
 在 Windows 上挂载 CephFS
==========================
.. Mount CephFS on Windows

``ceph-dokan`` 可用于在 Windows 上挂载 CephFS 文件系统。它利用了
Windows 驱动程序 Dokany ，该驱动程序允许在用户空间实现文件系统，很像 FUSE 。

请查看\ `安装指南`_\ 开始使用。

用法
====
.. Usage

挂载文件系统
------------
.. Mounting filesystems

要挂载 ceph 文件系统，可以用以下命令： ::

    ceph-dokan.exe -c c:\ceph.conf -l x

此命令会把默认的 ceph 文件系统挂载到驱动盘符 ``x`` 下。如果 ``ceph.conf``
放在默认位置（即 ``%ProgramData%\ceph\ceph.conf`` ），则此参数变为可选参数。

``-l`` 参数还允许用空文件夹作为挂载点，而不是驱动盘符。

用于挂载文件系统的 uid 和 gid 默认为 0 ，
可以用以下 ``ceph.conf`` 选项进行更改： ::

    [client]
    # client_permissions = true
    client_mount_uid = 1000
    client_mount_gid = 1000

如果 Ceph 集群上有多个 FS ，
用选项 ``--client_fs`` 挂载非默认 FS ： ::

    mkdir -Force C:\mnt\mycephfs2
    ceph-dokan.exe --mountpoint C:\mnt\mycephfs2 --client_fs mycephfs2

CephFS 子目录可以用 ``--root-path`` 参数指定挂载： ::

    ceph-dokan -l y --root-path /a

如果加了 ``-o --removable`` 选项，
挂载后可以显示在 ``Get-Volume`` 的结果中： ::

    PS C:\> Get-Volume -FriendlyName "Ceph*" | `
            Select-Object -Property @("DriveLetter", "Filesystem", "FilesystemLabel")

    DriveLetter Filesystem FilesystemLabel
    ----------- ---------- ---------------
              Z Ceph       Ceph
              W Ceph       Ceph - new_fs

用 ``ceph-dokan --help`` 查看所有的参数。

凭证
----
.. Credentials

``--id`` 选项传入 CephX 用户名，我们要用该用户的密钥环挂载 CephFS 。
下列命令是等效的： ::

    ceph-dokan --id foo -l x
    ceph-dokan --name client.foo -l x

卸载文件系统
------------
.. Unmounting filesystems

发出 ctrl-c 或者用 unmap 命令，就可以卸载： ::

    ceph-dokan.exe unmap -l x

注意，在取消 Ceph 文件系统映射时，必须用与创建映射时完全相同的挂载点参数。

局限性
------
.. Limitations

注意， Windows ACL 会被忽略。支持 Posix ACL ，但是不能用当前的 CLI 修改。
将来，我们可能会添加一些命令操作来更改文件所有权或权限。

还有一点需要注意， cephfs 不支持强制文件锁，
而 Windows 则非常依赖。目前，我们让 Dokan 来处理文件锁，
它只能在本地执行。

与 ``rbd-wnbd`` 不同， ``ceph-dokan`` 目前没有 ``service`` 命令。
为了让 cephfs 挂载在主机重启后继续运行，考虑用 ``NSSM`` 。

故障排除
========
.. Troubleshooting

请参考 `Windows 故障排除`_\ 。

.. _Windows 故障排除: ../../install/windows-troubleshooting
.. _安装指南: ../../install/windows-install
