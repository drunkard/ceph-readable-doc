:orphan:

==================================
 cephadm -- 管理本地 cephadm 主机
==================================

.. program:: cephadm

提纲
====

| **cephadm**** [-h] [--image IMAGE] [--docker] [--data-dir DATA_DIR]
|               [--log-dir LOG_DIR] [--logrotate-dir LOGROTATE_DIR]
|               [--unit-dir UNIT_DIR] [--verbose] [--timeout TIMEOUT]
|               [--retry RETRY] [--no-container-init]
|               {version,pull,inspect-image,ls,list-networks,adopt,rm-daemon,rm-cluster,run,shell,enter,ceph-volume,unit,logs,bootstrap,deploy,check-host,prepare-host,add-repo,rm-repo,install}
|               ...


| **cephadm** **pull**

| **cephadm** **inspect-image**

| **cephadm** **ls** [-h] [--no-detail] [--legacy-dir LEGACY_DIR]

| **cephadm** **list-networks**

| **cephadm** **adopt** [-h] --name NAME --style STYLE [--cluster CLUSTER]
|                       [--legacy-dir LEGACY_DIR] [--config-json CONFIG_JSON]
|                       [--skip-firewalld] [--skip-pull]

| **cephadm** **rm-daemon** [-h] --name NAME --fsid FSID [--force]
|                           [--force-delete-data]

| **cephadm** **rm-cluster** [-h] --fsid FSID [--force]

| **cephadm** **run** [-h] --name NAME --fsid FSID

| **cephadm** **shell** [-h] [--fsid FSID] [--name NAME] [--config CONFIG]
                        [--keyring KEYRING] --mount [MOUNT [MOUNT ...]] [--env ENV]
                        [--] [command [command ...]]

| **cephadm** **enter** [-h] [--fsid FSID] --name NAME [command [command ...]]

| **cephadm** **ceph-volume** [-h] [--fsid FSID] [--config-json CONFIG_JSON]
                              [--config CONFIG] [--keyring KEYRING]
                              command [command ...]

| **cephadm** **unit**  [-h] [--fsid FSID] --name NAME command

| **cephadm** **logs** [-h] [--fsid FSID] --name NAME [command [command ...]]

| **cephadm** **bootstrap** [-h] [--config CONFIG] [--mon-id MON_ID]
|                           [--mon-addrv MON_ADDRV] [--mon-ip MON_IP]
|                           [--mgr-id MGR_ID] [--fsid FSID]
|                           [--log-to-file] [--single-host-defaults]
|                           [--output-dir OUTPUT_DIR]
|                           [--output-keyring OUTPUT_KEYRING]
|                           [--output-config OUTPUT_CONFIG]
|                           [--output-pub-ssh-key OUTPUT_PUB_SSH_KEY]
|                           [--skip-ssh]
|                           [--initial-dashboard-user INITIAL_DASHBOARD_USER]
|                           [--initial-dashboard-password INITIAL_DASHBOARD_PASSWORD]
|                           [--ssl-dashboard-port SSL_DASHBOARD_PORT]
|                           [--dashboard-key DASHBOARD_KEY]
|                           [--dashboard-crt DASHBOARD_CRT]
|                           [--ssh-config SSH_CONFIG]
|                           [--ssh-private-key SSH_PRIVATE_KEY]
|                           [--ssh-public-key SSH_PUBLIC_KEY]
|                           [--ssh-user SSH_USER] [--skip-mon-network]
|                           [--skip-dashboard] [--dashboard-password-noupdate]
|                           [--no-minimize-config] [--skip-ping-check]
|                           [--skip-pull] [--skip-firewalld] [--allow-overwrite]
|                           [--allow-fqdn-hostname] [--skip-prepare-host]
|                           [--orphan-initial-daemons] [--skip-monitoring-stack]
|                           [--apply-spec APPLY_SPEC]
|                           [--registry-url REGISTRY_URL]
|                           [--registry-username REGISTRY_USERNAME]
|                           [--registry-password REGISTRY_PASSWORD]
|                           [--registry-json REGISTRY_JSON]



| **cephadm** **deploy** [-h] --name NAME --fsid FSID [--config CONFIG]
|                        [--config-json CONFIG_JSON] [--keyring KEYRING]
|                        [--key KEY] [--osd-fsid OSD_FSID] [--skip-firewalld]
|                        [--tcp-ports TCP_PORTS] [--reconfig] [--allow-ptrace]

| **cephadm** **check-host** [-h] [--expect-hostname EXPECT_HOSTNAME]

| **cephadm** **prepare-host**

| **cephadm** **add-repo** [-h] [--release RELEASE] [--version VERSION]
|                          [--dev DEV] [--dev-commit DEV_COMMIT]
|                          [--gpg-url GPG_URL] [--repo-url REPO_URL]


| **cephadm** **rm-repo**

| **cephadm** **install** [-h] [packages [packages ...]]

| **cephadm** **registry-login** [-h] [--registry-url REGISTRY_URL]
|                                [--registry-username REGISTRY_USERNAME]
|                                [--registry-password REGISTRY_PASSWORD]
|                                [--registry-json REGISTRY_JSON] [--fsid FSID]

| **cephadm** **list-images**

| **cephadm** **update-osd-service** [-h] [--fsid FSID] --osd-ids OSD_IDS --service-name SERVICE_NAME


描述
====

:program:`cephadm` 是个命令行工具，用于为 cephadm 编排器管理本地主机。

它有很多命令，可以用于调查和修改当前主机的状态。

:program:`cephadm` 不用装在所有主机上，只是在调查某个特定的守护进程时有用。


选项
====

.. option:: --image IMAGE

   容器映像。也可以通过 CEPHADM_IMAGE 环境变量设置
   （默认：无）。

.. option:: --docker

   采用 docker 而非 podman (默认: False)

.. option:: --data-dir DATA_DIR

   守护进程数据的顶级目录（默认: /var/lib/ceph ）

.. option:: --log-dir LOG_DIR

   守护进程日志的顶级目录（默认： /var/log/ceph ）

.. option:: --logrotate-dir LOGROTATE_DIR

   logrotate 配置文件的位置（默认： /etc/logrotate.d ）

.. option:: --unit-dir UNIT_DIR

   systemd 单元的顶级目录（默认： /etc/systemd/system ）

.. option:: --verbose, -v

   显示调试级的日志消息（默认： False ）

.. option:: --timeout TIMEOUT

   超时秒数（默认：无）

.. option:: --retry RETRY

   重试的最大次数（默认： 10 ）

.. option:: --no-container-init

   运行 podman/docker 时不要加 `--init` (默认: False)


命令
====
.. Commands

add-repo
--------

配置本地软件包仓库，以包含 ceph 软件库。

参数：

* [--release RELEASE]       使用指定大版本的最新版（如 octopus ）
* [--version VERSION]       采用具体的某个上游版本（ x.y.z ）
* [--dev DEV]               采用指定的最前沿构建，来自 git 分支或标签
* [--dev-commit DEV_COMMIT] 采用指定的最前沿构建，来自某个 git 提交
* [--gpg-url GPG_URL]       指定替代的 GPG 密钥位置
* [--repo-url REPO_URL]     指定替代的 repo 位置


adopt
-----

采用不同的部署工具部署好的守护进程。

参数：

* [--name NAME, -n NAME]       守护进程的名字 (type.id)
* [--style STYLE]              部署风格 (legacy, ...)
* [--cluster CLUSTER]          集群的名字
* [--legacy-dir LEGACY_DIR]    遗留守护进程数据的顶级目录
* [--config-json CONFIG_JSON]  JSON 格式的额外配置信息
* [--skip-firewalld]           不要配置 firewalld
* [--skip-pull]                采用前不要拉取最新映像

配置：

启动 shell 时， cephadm 按如下顺序查找配置信息。只采用第一个找到的值：

1. 明确指定的、用户提供的配置文件路径（ ``-c/--config`` 选项）
2. 用 ``--name`` 参数（ ``/var/lib/ceph/<fsid>/<daemon-name>/config`` ）指定的守护进程配置文件。
3. ``/var/lib/ceph/<fsid>/config/ceph.conf`` ，如果它存在的话
4. 如果有的话，采用 ``mon`` 守护进程（ ``/var/lib/ceph/<fsid>/mon.<mon-id>/config`` ）的配置文件
5. 最后：回退到默认文件 ``/etc/ceph/ceph.conf``


bootstrap
---------

在本地主机上自举引导一个集群。它会部署一个 MON 和一个 MGR 、
而后还会在此主机上自动部署监控栈（见 --skip-monitoring-stack ）、
并调用 ``ceph orch host add $(hostname)`` （见 --skip-ssh ）。

参数：

* [--config CONFIG, -c CONFIG]    要合并的 ceph 配置文件
* [--mon-id MON_ID]               mon id （默认：本地主机名）
* [--mon-addrv MON_ADDRV]         mon IPs (例如： [v2:localipaddr:3300,v1:localipaddr:6789])
* [--mon-ip MON_IP]               mon IP 地址
* [--mgr-id MGR_ID]               mgr id （默认：随机生成）
* [--fsid FSID]                   集群的 FSID
* [--log-to-file]                 让集群把日志记录到传统的日志文件中
* [--single-host-defaults]        让集群在单台主机上运行
* [--output-dir OUTPUT_DIR]       写出配置、密钥环和公钥文件的目录
* [--output-keyring OUTPUT_KEYRING] 写出装载新集群 admin 和 mon 密钥的密钥环文件的路径
* [--output-config OUTPUT_CONFIG] 写出用于连接新集群的配置文件的路径
* [--output-pub-ssh-key OUTPUT_PUB_SSH_KEY] 写出集群 SSH 公钥的路径
* [--skip-ssh                     跳过本机上的 ssh 密钥配置
* [--initial-dashboard-user INITIAL_DASHBOARD_USER] dashboard 的初始用户
* [--initial-dashboard-password INITIAL_DASHBOARD_PASSWORD] 初始 dashboard 用户的初始密码
* [--ssl-dashboard-port SSL_DASHBOARD_PORT] 通过 SSL 连接 dashboard 的端口号
* [--dashboard-key DASHBOARD_KEY] Dashboard 密钥
* [--dashboard-crt DASHBOARD_CRT] Dashboard 证书
* [--ssh-config SSH_CONFIG]       SSH 配置
* [--ssh-private-key SSH_PRIVATE_KEY] SSH 私钥
* [--ssh-public-key SSH_PUBLIC_KEY] SSH 公钥
* [--ssh-user SSH_USER]           通过 SSH 登陆集群主机的用户名，对于非 root 用户需要预先配置好无密码 sudo
* [--skip-mon-network]            根据 mon IP 配置 public_network
* [--skip-dashboard]              不要启用 Ceph Dashboard
* [--dashboard-password-noupdate] 不要强制让用户更改 dashboard 密码
* [--no-minimize-config]          不要同化并最小化配置文件
* [--skip-ping-check]             不要检验 mon IP 地址是否可以 ping
* [--skip-pull]                   自举引导前不要拉取最新映像
* [--skip-firewalld]              不要配置 firewalld
* [--allow-overwrite]             允许覆盖已有的 --output-* config/keyring/ssh 文件
* [--allow-fqdn-hostname]         允许全资主机名（包含点 "." 的）
* [--skip-prepare-host]           不要准备主机
* [--orphan-initial-daemons]      不要创建初始的 mon 、 mgr 和崩溃服务规范
* [--skip-monitoring-stack]       不要自动提供监控栈(prometheus, grafana, alertmanager, node-exporter)
* [--apply-spec APPLY_SPEC]       自举引导之后按规范配置集群（复制 ssh 密钥、增加主机并应用服务）
* [--registry-url REGISTRY_URL]   要登录的自制注册处的 URL ，例如 docker.io 、 quay.io
* [--registry-username REGISTRY_USERNAME] 登录自制注册处的帐户用户名
* [--registry-password REGISTRY_PASSWORD] 登录自制注册处的帐户密码
* [--registry-json REGISTRY_JSON] 包含注册处登录信息的 JSON 文件（见 registry-login 命令的文档）


ceph-volume
-----------

在容器内运行 ceph-volume::

    cephadm ceph-volume inventory

位置参数：
* [command]               命令

参数：

* [--fsid FSID]                    集群的 FSID
* [--config-json CONFIG_JSON]      有配置和（ client.bootrap-osd ）密钥的 JSON 文件
* [--config CONFIG, -c CONFIG]     ceph 配置文件
* [--keyring KEYRING, -k KEYRING]  要传进容器的 ceph.keyring


check-host
----------

检查主机配置是否适合 Ceph 集群。

参数：

* [--expect-hostname EXPECT_HOSTNAME] 检查主机名与期望的值是否匹配


deploy
------

在本地主机上部署一个守护进程。编排器 CLI 会用到： ::

    cephadm shell -- ceph orch apply <type> ...

参数：

* [--name NAME]               守护进程的名字 (type.id)
* [--fsid FSID]               集群的 FSID
* [--config CONFIG, -c CONFIG] 新守护进程的配置文件
* [--config-json CONFIG_JSON] JSON 格式的附加配置信息
* [--keyring KEYRING]         新守护进程的密钥环
* [--key KEY]                 新守护进程的密钥
* [--osd-fsid OSD_FSID]       OSD uuid ，如果在创建 OSD 容器
* [--skip-firewalld]          不要配置 firewalld
* [--tcp-ports                要在主机防火墙上开放的 TCP 端口列表
* [--reconfig]                重新配置一个之前部署的守护进程
* [--allow-ptrace]            在守护进程容器中打开 SYS_PTRACE


enter
-----

在运行着的守护进程容器内打开一个交互式 shell::

    cephadm enter --name mgr.myhost.ysubfo

位置参数：
* [command]               命令

参数：

* [--fsid FSID]           集群的 FSID
* [--name NAME, -n NAME]  守护进程的名字 (type.id)


install
-------

安装 ceph 软件包。

位置参数：

* [packages]    软件包


inspect-image
-------------

检查本地的 ceph 容器映像。从 Reef 版起，必须用 ``--image`` 指定要检查的映像： ::

    cephadm --image IMAGE_NAME inspect-image


list-networks
-------------

罗列 IP 网络段。


ls
--

罗列 **这台** 主机上 cephadm 知道的守护进程例程： ::

    $ cephadm ls
    [
        {
            "style": "cephadm:v1",
            "name": "mgr.storage-14b-1.ysubfo",
            "fsid": "5110cb22-8332-11ea-9148-0894ef7e8bdc",
            "enabled": true,
            "state": "running",
            "container_id": "8562de72370a3836473ecfff8a22c9ccdd99815386b4692a2b30924fb5493c44",
            "container_image_name": "docker.io/ceph/ceph:v15",
            "container_image_id": "bc83a388465f0568dab4501fb7684398dca8b50ca12a342a57f21815721723c2",
            "version": "15.2.1",
            "started": "2020-04-21T01:16:41.831456",
            "created": "2020-04-21T01:16:41.775024",
            "deployed": "2020-04-21T01:16:41.415021",
            "configured": "2020-04-21T01:16:41.775024"
        },
    ...

参数：

* [--no-detail]             不要显示守护进程状态
* [--legacy-dir LEGACY_DIR] 遗留守护进程数据的顶级目录


logs
----

打印一个守护进程容器的 journald 日志： ::

    cephadm logs --name mgr.myhost.ysubfo

和这个相似： ::

    journalctl -u mgr.myhost.ysubfo

还能指定额外的日志参数： ::

    cephadm logs --name mgr.myhost.ysubfo -- -n 20 # last 20 lines
    cephadm logs --name mgr.myhost.ysubfo -- -f # follow the log

位置参数：

* [command]               命令（可选的）

参数：

* [--fsid FSID]           集群的 FSID
* [--name NAME, -n NAME]  守护进程的名字 (type.id)


prepare-host
------------

准备一台主机给 cephadm 用。

参数：

* [--expect-hostname EXPECT_HOSTNAME] 设置主机名


pull
----

拉取 ceph 映像： ::

    cephadm pull

registry-login
--------------

向 cephadm 提供登录信息，以便与注册处认证（ URL 、用户名和密码）。
Cephadm 将尝试让调用主机登录进那个注册处： ::

      cephadm registry-login --registry-url [REGISTRY_URL] --registry-username [USERNAME]
                             --registry-password [PASSWORD]

也可以用如下格式的 JSON 文件装载登录信息： ::

      {
       "url":"REGISTRY_URL",
       "username":"REGISTRY_USERNAME",
       "password":"REGISTRY_PASSWORD"
      }

然后用命令代入它： ::

      cephadm registry-login --registry-json [JSON FILE]

参数：

* [--registry-url REGISTRY_URL]   要登录的注册处的 URL ，例如 docker.io 、 quay.io
* [--registry-username REGISTRY_USERNAME] 登录注册处的帐户用户名
* [--registry-password REGISTRY_PASSWORD] 登录注册处的帐户密码
* [--registry-json REGISTRY_JSON] 包含注册处登录信息的 JSON 文件
* [--fsid FSID]                   集群的 FSID

rm-daemon
---------

删除某个特定的守护进程例程。

参数：

* [--name NAME, -n NAME]  守护进程的名字 (type.id)
* [--fsid FSID]           集群的 FSID
* [--force]               继续进行，即使这样可能损坏有价值的数据
* [--force-delete-data]   不做备份，直接删除有用的守护进程数据


rm-cluster
----------

删除一个集群的所有守护进程。

参数：

* [--fsid FSID]  集群的 FSID
* [--force]      继续进行，即使这样可能损坏有价值的数据。

rm-repo
-------

删除软件仓库配置。

run
---

在容器内运行一个 ceph 守护进程，在前台。

参数：

* [--name NAME, -n NAME]  守护进程的名字 (type.id)
* [--fsid FSID]           集群的 FSID


shell
-----

启动一个交互式 shell::

    cephadm shell

或者容器内一个特定的命令： ::

    cephadm shell -- ceph orch ls


位置参数：

* [command]               命令（可选的）

参数：

* [--fsid FSID]                   集群的 FSID
* [--name NAME, -n NAME]          守护进程的名字 (type.id)
* [--config CONFIG, -c CONFIG]    要传给容器的 ceph.conf
* [--keyring KEYRING, -k KEYRING] 要传给容器的 ceph.keyring
* [--mount MOUNT, -m MOUNT]       在容器内的 /mnt 下挂载一个文件或目录
* [--env ENV, -e ENV]             设置环境变量


unit
----

操作守护进程的 systemd 单元。

位置参数：

* [command]               systemd 命令 (start, stop, restart, enable, disable, ...)

参数：

* [--fsid FSID]           集群的 FSID
* [--name NAME, -n NAME]  守护进程的名字 (type.id)


list-images
-----------

按 ini 格式列出所有服务的默认容器映像。可以用自定义镜像修改输出，并在引导过程中传递给 --config 标志。


update-osd-service
------------------

更新指定 OSD 的 OSD 服务。

参数：

* [--fsid FSID]                 集群 FSID
* --osd-ids OSD_IDS             逗号分隔的 OSD IDs
* --service-name SERVICE_NAME   OSD 服务名字


使用范围
========

**cephadm** 是 Ceph 的一部分，这是个伸缩力强、开源、
分布式的存储系统，更多信息参见 https://docs.ceph.com 。


参考
====

:doc:`ceph-volume <ceph-volume>`\(8),
