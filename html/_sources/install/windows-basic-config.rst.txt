:orphan:

==================
 Windows 基本配置
==================

本页描述的是在 Windows 上使用客户端组件需要的最小化 Ceph 配置。


ceph.conf
=========

Windows 上 ``ceph.conf`` 文件的默认位置是
``%ProgramData%\ceph\ceph.conf`` ，通常展开成
``C:\ProgramData\ceph\ceph.conf`` 。

下面是个样本，请填入\
你自己的监视器地址。

.. code:: ini

    [global]
        log to stderr = true
        ; Uncomment the following in order to use the Windows Event Log
        ; log to syslog = true

        run dir = C:/ProgramData/ceph/out
        crash dir = C:/ProgramData/ceph/out

        ; Use the following to change the cephfs client log level
        ; debug client = 2
    [client]
        keyring = C:/ProgramData/ceph/keyring
        ; log file = C:/ProgramData/ceph/out/$name.$pid.log
        admin socket = C:/ProgramData/ceph/out/$name.$pid.asok

        ; client_permissions = true
        ; client_mount_uid = 1000
        ; client_mount_gid = 1000
    [global]
        mon host = <ceph_monitor_addresses>

别忘了，还得把你的密钥环文件复制到指定位置，
并确保配置的目录存在（例如 ``C:\ProgramData\ceph\out`` ）。

在 ``ceph.conf`` 里面请用斜杠 ``/`` 而不是反斜杠 ``\``
作为路径的分隔符。

