:orphan:

==========================================
 ceph-disk -- Ceph 的磁盘工具，适用于 OSD
==========================================

.. program:: ceph-disk

提纲
====

| **ceph-disk** [-h] [-v] [--log-stdout] [--prepend-to-path PATH]
|               [--statedir PATH] [--sysconfdir PATH]
|               [--setuser USER] [--setgroup GROUP]
|               ...

可选参数
--------

-h, --help          显示本帮助消息后退出
-v, --verbose       输出得更详细些
--log-stdout        日志输出到 stdout
--prepend-to-path PATH
                    把 PATH 安插到 $PATH 前面，为保持向后兼容性（默认为 /usr/bin ）
--statedir PATH     用于存放 Ceph 状态的目录（默认为 /var/lib/ceph ）
--sysconfdir PATH   用于存放 Ceph 配置文件的目录（默认为 /etc/ceph ）
--setuser USER      指定子进程的所属用户，取代默认的 ceph 或 root
--setgroup GROUP    指定子进程的所属用户组，取代默认的 ceph 或 root

子命令
------

prepare
    为 Ceph OSD 准备一个目录或磁盘
activate
    激活一个 Ceph OSD
activate-lockbox
    激活一个 Ceph lockbox
activate-block
    通过其块设备激活一个 OSD
activate-journal
    通过其日志设备激活一个 OSD
activate-all
    激活所有标记过的 OSD 分区
list
    罗列磁盘、分区、和 Ceph OSD
suppress-activate
    抑制设备，防止被激活（前缀）
unsuppress-activate
    取消对设备的抑制（前缀）
deactivate
    弄死一个 Ceph OSD
destroy
    拆除一个 Ceph OSD
zap
    干掉、擦除、破坏某一设备的分区表（及其内容）
trigger
    触发一个事件（底层为 udev ）
fix
    修正 SELinux 标签和、或文件权限位。


描述
====

:program:`ceph-disk` 工具可把硬盘、分区或目录预处理并激活为
Ceph OSD 。可单独使用，也可由 :program:`ceph-deploy` 或
``udev`` 调用，还可由其他部署工具，如 ``Chef`` 、 ``Juju`` 、
``Puppet`` 等调用。

其实它是把手动创建并启动 OSD 的很多步骤自动化成了两步：预处\
理和激活，分别对应两个子命令 ``prepare`` 和 ``activate`` 。

:program:`ceph-disk` 也把手动停止和销毁某一 OSD 的多个步骤\
自动化成了两步：停用和销毁，分别对应 ``deactivate`` 和
``destroy`` 子命令。

各个子命令的文档（ prepare 、 activate 等等）可用它的
``--help`` 选项显示，例如 ``ceph-disk prepare --help`` 。


已知缺陷
========

请参考 :doc:`ceph-detect-init <ceph-detect-init>`\(8) 里的\
``已知缺陷``\ 一节。


使用范围
========

:program:`ceph-disk` 是 Ceph 的一部分，这是个伸缩力强、开源、\
分布式的存储系统，更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph-osd <ceph-osd>`\(8),
:doc:`ceph-deploy <ceph-deploy>`\(8)
