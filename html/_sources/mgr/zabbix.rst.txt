Zabbix 模块
===========

Zabbix 模块可向 Zabbix 服务器持续发送信息，像这些：

- Ceph 状态
- I/O 操作
- I/O 带宽
- OSD 状态
- 存储空间利用率


必需条件
--------
.. Requirements

此模块要求 *zabbix_sender* 可执行文件在\ *所有*\ 运行 ceph-mgr
的主机上都存在。大多数发行版都可以用软件包管理器安装它。


依赖
^^^^
.. Dependencies

在 Ubuntu 或 CentOS 上安装 zabbix_sender 可以用 apt 或 dnf
命令。

在 Ubuntu Xenial上： ::

    apt install zabbix-agent

在 Fedora 上： ::

    dnf install zabbix-sender


如何启用
--------
.. Enabling

你可以用以下命令启用 *zabbix* 模块： ::

    ceph mgr module enable zabbix


配置
----
.. Configuration

有两个配置选项对于模块是否能运行至关重要：

- zabbix_host
- identifier （可选项）

The parameter *zabbix_host* controls the hostname of the Zabbix server to which
*zabbix_sender* will send the items. This can be a IP-Address if required by
your installation.

The *identifier* parameter controls the identifier/hostname to use as source
when sending items to Zabbix. This should match the name of the *Host* in
your Zabbix server.

When the *identifier* parameter is not configured the ceph-<fsid> of the cluster
will be used when sending data to Zabbix.

This would for example be *ceph-c4d32a99-9e80-490f-bd3a-1d22d8a7d354*

Additional configuration keys which can be configured and their default values:

- zabbix_port: 10051
- zabbix_sender: /usr/bin/zabbix_sender
- interval: 60
- discovery_interval: 100


配置键
^^^^^^
.. Configuration keys

可以在任何有合适 cephx 凭据的主机上配置键，通常是在持有
*client.admin* 密钥的监视器主机上。\ ::

    ceph zabbix config-set <key> <value>

例如： ::

    ceph zabbix config-set zabbix_host zabbix.localdomain
    ceph zabbix config-set identifier ceph.eu-ams02.local

此模块的当前配置可以这样查看： ::

   ceph zabbix config-show


模板
^^^^
.. Template

A `template <https://raw.githubusercontent.com/ceph/ceph/master/src/pybind/mgr/zabbix/zabbix_template.xml>`_. 
(XML) to be used on the Zabbix server can be found in the source directory of the module.

This template contains all items and a few triggers. You can customize the triggers afterwards to fit your needs.


多个 Zabbix 服务器
^^^^^^^^^^^^^^^^^^
.. Multiple Zabbix servers

It is possible to instruct zabbix module to send data to multiple Zabbix servers.

Parameter *zabbix_host* can be set with multiple hostnames separated by commas.
Hostnames (or IP addresses) can be followed by colon and port number. If a port
number is not present module will use the port number defined in *zabbix_port*.

例如： ::

    ceph zabbix config-set zabbix_host "zabbix1,zabbix2:2222,zabbix3:3333"


手动发送数据
------------
.. Manually sending data

如有必要，此模块可以立即发送数据，无须间隔时间。

用如下命令即可： ::

    ceph zabbix send

此模块就会立即把它的最新数据发送给 Zabbix 服务器。

Items discovery is accomplished also via zabbix_sender, and runs every `discovery_interval * interval` seconds. If you wish to launch discovery 
manually, this can be done with this command:

::

    ceph zabbix discovery


调试
----
.. Debugging

你要是想调试 Zabbix 模块，增大 ceph-mgr 的日志记录级别、并检查\
日志即可。 ::

    [mgr]
        debug mgr = 20

Should you want to debug the Zabbix module increase the logging level for
ceph-mgr and check the logs.
