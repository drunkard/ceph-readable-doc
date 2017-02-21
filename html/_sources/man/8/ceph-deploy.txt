:orphan:

==============================
 ceph-deploy -- Ceph 部署工具
==============================

.. program:: ceph-deploy

提纲
====

| **ceph-deploy** **new** [*initial-monitor-node(s)*]

| **ceph-deploy** **install** [*ceph-node*] [*ceph-node*...]

| **ceph-deploy** **mon** *create-initial*

| **ceph-deploy** **osd** *prepare* [*ceph-node*]:[*dir-path*]

| **ceph-deploy** **osd** *activate* [*ceph-node*]:[*dir-path*]

| **ceph-deploy** **osd** *create* [*ceph-node*]:[*dir-path*]

| **ceph-deploy** **admin** [*admin-node*][*ceph-node*...]

| **ceph-deploy** **purgedata** [*ceph-node*][*ceph-node*...]

| **ceph-deploy** **forgetkeys**


描述
====

:program:`ceph-deploy` 工具可用于简单、快速地部署 Ceph 集群，而无需涉及繁杂\
的手动配置。它在管理节点上通过 ssh 获取其它 Ceph 节点的访问权、通过 sudo 获\
取其上的管理权限、通过底层 Python 脚本自动化各节点上的 Ceph 安装进程。它简单\
到可以运行在工作站上，不需要服务器、数据库或任何其它的自动化工具。 有了 \
:program:`ceph-deploy` ，安装和拆除集群非常简单。然而它不是通用部署工具，是\
专为想快速安装、运行 Ceph 的人们设计的专用工具，这样的集群只包含必要的的初始\
配置选项，就没必要安装像 ``Chef`` 、 ``Puppet`` 或 ``Juju`` 这样的部署工具。\
如果你想定制安全选项、分区或目录位置，并按照详细的手动步骤设置集群，应该选用\
其它工具，即 ``Chef`` 、 ``Puppet`` 、 ``Juju`` 或 ``Crowbar`` 。

用 :program:`ceph-deploy` 工具可以在远程节点上安装 Ceph 软件包、创建集群、增\
加监视器、收集或忘记密钥、增加 OSD 和元数据服务器、配置管理主机或拆除集群。


命令
====

new
---

开始部署新集群，并写好配置文件和密钥环。它会尝试把管理节点上的 SSH 密钥复制\
到监视器节点以获得无密码访问权限，验证主机 IP ，新建一或多个监视器节点以组成\
监视器法定人数，生成新 Ceph 集群所需的配置文件、监视器密钥环和日志文件。然后\
把新建的集群 ``fsid`` 、主机名和初始监视器成员的 IP 地址组装成 Ceph 配置文件。

用法： ::

	ceph-deploy new [MON][MON...]

这里的 [MON] 是初始监视器主机名（短主机名，即 ``hostname -s`` ）。

此命令还可加这些选项： :option:`--no-ssh-copykey` 、 :option:`--fsid` 、 \
:option:`--cluster-network` 和 :option:`--public-network` 。

如果使用了多个网卡，那么必须在 Ceph 配置文件的 ``[global]`` 段下加 \
``public network`` 选项。指定公共网子网后， ``new`` 命令将选用在此子网范围\
内的远程主机 IP 。公共网也可以在运行时用 :option:`--public-network` 选项加\
给前述命令。


install
-------

在远程主机上安装 Ceph 软件包。首先，它会用无密码 ssh 和 sudo 在管理节点和其\
它节点上安装 ``yum-plugin-priorities`` ，这样来自上流软件库的 Ceph 软件包就\
可获得较高优先级。之后，它会探测这些主机的平台和发行版，并且，在软件库准备充\
分时安装兼容此发行版的软件包。加 ``--release`` 选项后它会安装最新版。在安装\
前的平台和发行版探测中，如果发现 ``distro.init`` 是 ``sysvinit`` （如 \
Fedora 、 CentOS/RHEL 等），那么安装时就不能定制集群名，且自动采用默认名 \
``ceph`` 。

如果用户用 :option:`--repo-url` 选项显式地指定了软件库 URL 作为软件源，那么\
它会覆盖探测结果，并从定制软件库安装 Ceph 软件包。若有必要，也会检测并安装有\
效的定制存储池。从定制软件库安装时，需输入一个布尔值确定所需逻辑，然后才能继\
续定制软件库的安装。它所用的定制软件库安装辅助程序会查验配置、下载软件库（及\
其它附加软件库）并安装它。 ``cd_conf`` 是 ``argparse`` 构建的对象，它所存储\
的标识和信息决定了会用到配置里的哪些元数据。

用户也可以用 :option:`--repo` 选项做到只装软件库，而不装 Ceph 及其依赖软件包。

用法： ::

	ceph-deploy install [HOST][HOST...]

这里的 [HOST] 是将被安装 Ceph 软件的主机节点。

``--release`` 选项可用于指定安装的版本，参数为 CODENAME ，默认为 firefly 。

此命令支持的其它选项： :option:`--testing` 、 :option:`--dev` 、 \
:option:`--adjust-repos` 、 :option:`--no-adjust-repos` 、 \
:option:`--repo` 、 :option:`--local-mirror` 、 :option:`--repo-url` 和 \
:option:`--gpg-url` 。


mds
---

在远程主机上部署 Ceph 元数据服务器。元数据服务器对 CephFS 来说是必需的， \
``mds`` 命令可用于在指定节点上创建它。 ``create`` 子命令就是做这个的，它首先\
获取目标 mds 主机的主机名和发行版信息，之后尝试读取集群的 ``bootstrap-mds`` \
密钥并部署到目标主机。密钥格式通常是 ``{cluster}.bootstrap-mds.keyring`` ，\
如果它没找到此密钥环，就用 ``gatherkeys`` 来获取此密钥环；然后在目标主机上创\
建 mds （在 ``/var/lib/ceph/mds/`` 路径下、按 \
``/var/lib/ceph/mds/{cluster}-{name}`` 格式）和自举引导密钥环（在 \
``/var/lib/ceph/bootstrap-mds/`` 下、按 \
``/var/lib/ceph/bootstrap-mds/{cluster}.keyring`` 格式）；然后根据 \
``distro.init`` 运行相应命令来启动 ``mds`` 。要删除 mds ，可用 ``destroy`` \
子命令。

用法： ::

	ceph-deploy mds create [HOST[:DAEMON-NAME]] [HOST[:DAEMON-NAME]...]

	ceph-deploy mds destroy [HOST[:DAEMON-NAME]] [HOST[:DAEMON-NAME]...]

[DAEMON-NAME] 是可选项。


mon
---

在远程主机上部署 Ceph 监视器。 ``mon`` 用特定子命令把 Ceph 监视器部署到其它\
节点。

``create-initial`` 子命令会部署 Ceph 配置文件中 ``[global]`` 段下、 \
``mon initial members`` 定义的监视器，然后等它们形成法定人数后收集密钥，并一\
直报告此期间的监视器状态。如果监视器未能形成法定人数，此命令最终会超时。

用法： ::

	ceph-deploy mon create-initial

``create`` 子命令用于部署 Ceph 监视器，需明确指定想作监视器的主机。如果没指\
定，它将默认采用 Ceph 配置文件中 ``[global]`` 段下 ``mon initial members`` \
定义的。 ``create`` 首先会探测目标主机的平台和发行版、并检查主机名是否能兼容\
此部署；然后采用 ``new`` 命令创建的监视器密钥环、并在目标主机上部署监视器。\
如果运行 ``new`` 命令时指定了多个主机，即 ``mon initial members`` 内含多个主\
机、且创建了多个密钥环，那么部署监视器时将使用串连过的密钥环，在此期间将用到\
密钥环解析器，它会在各密钥环中搜索并返回一连串 ``[entity]`` 段落；然后用一个\
辅助程序把所有密钥环集中到一个单体二进制数据块中，此数据块将随 \
:option:`--mkfs` 选项被注入远程主机上的监视器。所有要被串连的密钥环应该位于\
同一目录、且以 ``.keyring`` 结尾，在此过程中，此辅助程序用密钥环解析器返回的\
段落列表来检查实体是否已经存在于密钥环中，没有的话加上。串连起的密钥环被用于\
往多个主机上部署监视器。

用法： ::

	ceph-deploy mon create [HOST] [HOST...]

这里的 [HOST] 是目标监视器主机的主机名。

``add`` 子命令用于向已有集群增加一监视器。它首先探测目标主机的平台和发行版、\
并检查主机名是否能兼容此部署；然后使用监视器密钥环、确认新监视器主机的配置、\
并把它加入集群。如果此监视器配置段已存在且定义了 mon addr ，那就采用此地址，\
否则就得先把主机名解析为 IP 地址；如果加了 :option:`--address` 选项，它将覆\
盖所有其它选项。加完监视器后会等它启动，然后检查有什么监视器错误、并检查监视\
器状态。监视器错误可能有：它未加入 ``mon initial members`` 、没在 ``monmap`` \
里面、 ``public_addr`` 和 ``public_network`` 关键字都没定义，这时，监视器们\
不能组建法定人数。监视器状态能说明此监视器是否启动且正常运行着。此状态是通过\
在远端运行 ``ceph daemon mon.hostname mon_status`` 获取的，此命令的输出和布\
尔值状态能说明当前状况； ``False`` 意味着此监视器有问题，即使它已启动且运行\
着， ``True`` 意味着此监视器已启动且运行正常。

用法： ::

	ceph-deploy mon add [HOST]

	ceph-deploy mon add [HOST] --address [IP]

这里的 [HOST] 是主机名、 [IP] 是目标监视器节点的 IP 地址。要注意，不像其他\
的 ``mon`` 子命令，这里一次只能指定一个节点。

``destroy`` 子命令用于从远程主机上完全删除监视器，其参数为主机名。它会停止\
监视器、确认 ``ceph-mon`` 是否确实停止了、在 ``/var/lib/ceph/`` 下创建存档\
目录 ``mon-remove`` 、把监视器的旧目录按 ``{cluster}-{hostname}-{stamp}`` \
格式归档进去、并运行 ``ceph remove...`` 命令从集群删除它。

用法： ::

	ceph-deploy mon destroy [HOST] [HOST...]

这里的 [HOST] 是要删除的监视器的主机名。


gatherkeys
----------

用于收集新节点所需的认证密钥，以主机名作参数。它会到监视器主机检查并取来 \
``client.admin`` 密钥环、监视器密钥环和 ``bootstrap-mds/bootstrap-osd`` 密\
钥环，向集群新增 ``monitors/OSDs/MDS`` 时会用到这些认证密钥。

用法： ::

	ceph-deploy gatherkeys [HOST] [HOST...]

这里的 [HOST] 是监视器主机名，密钥将从这里拉取。


disk
----

管理远程主机上的硬盘，实际上它会调用 ``ceph-disk`` 工具及其子命令来管理硬盘。

``list`` 子命令罗列硬盘分区和 Ceph OSD 。

用法： ::

	ceph-deploy disk list [HOST:[DISK]]

这里的 [HOST] 是节点的主机名、 [DISK] 是硬盘名称或路径。

``prepare`` 子命令用于预处理 Ceph OSD 所需的目录、硬盘或驱动器。它会创建 GPT \
分区、用 Ceph 的类型 UUID 标记分区、创建文件系统、把文件系统标记为可接收 \
Ceph 数据、使用日志硬盘的整个分区并新增一分区。

用法： ::

	ceph-deploy disk prepare [HOST:[DISK]]

这里的 [HOST] 是节点主机名， [DISK] 是硬盘名或路径。

``activate`` 子命令用于激活 Ceph OSD 。它会把卷挂载到一个临时位置、分配 OSD \
惟一标识符（如有必要）、重新挂载到正确位置 ``/var/lib/ceph/osd/$cluster-$id`` \
并启动 ``ceph-osd`` 。此 OSD 可由 ``udev`` （当它探测到 OSD GPT 分区类型时）\
或 ceph 服务启动命令 ``ceph disk activate-all`` 激活。

用法： ::

	ceph-deploy disk activate [HOST:[DISK]]

这里的 [HOST] 是节点主机名， [DISK] 是硬盘名或路径。

``zap`` 子命令可用于杀死、擦除、销毁一设备的分区表和内容。实际上它用 \
``sgdisk`` 加 ``--zap-all`` 选项来销毁 GPT 和 MBR 数据结构，这样才能重新分\
区；然后用 ``--mbrtogpt`` 选项把 MBR 或 BSD 格式的分区转换为 GPT 格式。现\
在就可以执行 ``prepare`` 子命令来新建 GPT 分区了。

用法： ::

	ceph-deploy disk zap [HOST:[DISK]]

这里的 [HOST] 是节点主机名， [DISK] 是硬盘名或路径。


osd
---

用于管理 OSD ，预处理远程主机上的数据盘。 ``osd`` 用几个子命令管理 OSD 。

``prepare`` 子命令用于预处理用作 Ceph OSD 的目录、硬盘或驱动器。它会先检查将\
要创建的多个 OSD 、并且在可能会超过建议值时发出警告，超过系统允许的最大 PID \
数时会产生问题；然后读取集群的 bootstrap-osd 密钥，或没找到时生成一个自举引\
导密钥；然后调用 :program:`ceph-disk` 工具的 ``prepare`` 子命令预处理硬盘、\
日志并在目标主机上部署 OSD 。预处理完成后，它会给 OSD 一些时间处理自身事务，\
并检查任何可能错误，发现的话报告给用户。

用法： ::

	ceph-deploy osd prepare HOST:DISK[:JOURNAL] [HOST:DISK[:JOURNAL]...]

``activate`` 子命令可激活用 ``prepare`` 子命令预处理过的 OSD 。实际上它是基\
于发行版的 init 类型执行 :program:`ceph-disk` 工具的 ``activate`` 子命令来激\
活 OSD 的。激活后，它会给 OSD 一些时间让它启动，然后检查任何可能错误，发现的\
话报告给用户。它会检查预处理过的 OSD 状态、检查 OSD 树、确保这些 OSD 已启动\
且在运行。

用法： ::

	ceph-deploy osd activate HOST:DISK[:JOURNAL] [HOST:DISK[:JOURNAL]...]

``create`` 子命令用 ``prepare`` 和 ``activate`` 子命令创建 OSD 。

用法： ::

	ceph-deploy osd create HOST:DISK[:JOURNAL] [HOST:DISK[:JOURNAL]...]

``list`` 子命令可罗列磁盘分区、 Ceph OSD 且打印 OSD 元数据。它要先从监视器主\
机获取 OSD 树、根据 ``ceph-disk-list`` 的输出匹配 OSD 名字以获取挂载点、从文\
件读取元数据、检查是否存在日志路径、此 OSD 是否在 OSD 树里并打印 OSD 元数据。

用法： ::

	ceph-deploy osd list HOST:DISK[:JOURNAL] [HOST:DISK[:JOURNAL]...]


admin
-----

把配置和 ``client.admin`` 密钥推送到远程主机，它把管理节点上的 \
``{cluster}.client.admin.keyring`` 复制到目标节点的 ``/etc/ceph`` 目录下。

用法： ::

	ceph-deploy admin [HOST] [HOST...]

这里的 [HOST] 是目标主机，它将被配置为 Ceph 管理主机。


config
------

把配置文件推送到远程主机、或从远程主机拉取。它用 ``push`` 子命令把管理主机上\
的配置文件写入远程主机的 ``/etc/ceph`` 目录下； ``pull`` 子命令则相反，也就\
是把远程主机 ``/etc/ceph`` 目录下的配置文件拉取到管理节点上。

用法： ::

	ceph-deploy push [HOST] [HOST...]

	ceph-deploy pull [HOST] [HOST...]

这里的 [HOST] 是节点主机名，将到这里推送或拉取配置文件。


uninstall
---------

删除远程主机上的 Ceph 软件包。它会探测指定主机的平台及发行版、并卸载其上的 \
Ceph 软件包。然而像 ``librbd1`` 和 ``librados2`` 这样的依赖包不会被删除，因\
为 ``qemu-kvm`` 还需要它们。

用法： ::

	ceph-deploy uninstall [HOST] [HOST...]

这里的 [HOST] 是要卸载 Ceph 的节点主机名。


purge
-----

删除远程主机上的 Ceph 软件包、并清除所有数据。它会探测指定主机的平台及发行\
版、卸载 Ceph 软件包并清除所有数据。然而像 ``librbd1`` 和 ``librados2`` 这\
样的依赖包不会被删除，因为 ``qemu-kvm`` 还需要它们。

用法： ::

	ceph-deploy purge [HOST] [HOST...]

这里的 [HOST] 是将被清除 Ceph 痕迹的节点主机名。


purgedata
---------

清除（删除、销毁、丢弃、粉碎） ``/var/lib/ceph`` 之下的所有 Ceph 数据。探测\
到目标主机的平台及发行版后，它会先检查指定主机上是否仍装着 Ceph ，若安装了就\
不会清除数据；若 Ceph 已被卸载，它就会尝试卸载 ``/var/lib/ceph`` 下的内容。\
如果失败了，说明 OSD 数据盘可能还挂载着，要先卸载才能继续。它会卸载各 OSD 、\
再次尝试删除 ``/var/lib/ceph`` 下的内容、并检查报错；它也会删除 ``/etc/ceph`` \
下的内容。所有步骤都成功完成后，指定主机上的所有 Ceph 数据就清除完了。

用法： ::

	ceph-deploy purgedata [HOST] [HOST...]

这里的 [HOST] 是要清除数据的节点主机名。


forgetkeys
----------

删除本地目录中的认证密钥。它会删除此节点上的所有认证密钥，即监视器密钥环、 \
client.admin 密钥环、 bootstrap-osd 和 bootstrap-mds 密钥环。

用法： ::

	ceph-deploy forgetkeys


pkg
---

管理远程主机上的软件包，可用于安装或删除软件包。要安装或删除的软件包名字必须\
加在命令行之后，对应选项分别为 :option:`--install` 和 :option:`--remove` 。

用法： ::

	ceph-deploy pkg --install [PKGs] [HOST] [HOST...]

	ceph-deploy pkg --remove [PKGs] [HOST] [HOST...]

这里的 [PKGs] 是逗号分隔的软件包名字， [HOST] 是远程节点的主机名，将在此主机\
安装或删除软件包。


calamari
--------

安装并配置 Calamari 节点。它首先检查 ceph-deploy 是否支持在此发行版上安装 \
Calamari 。 ``connect`` 参数用于安装和配置，它会检查 ``ceph-deploy`` 配置文\
件（ cd_conf ）和 Calamari 或 ``calamari-minion`` 软件库，它靠默认值安装软\
件库，除非另外指定，否则它不会安装 Ceph 。 ``options`` 字典也需定义，因为 \
``ceph-deploy`` 内部会弹出各条目，如果各主机都必须有这些条目时就可能产生问\
题。若发行版是 Debian/Ubuntu ，它会确保对 ``calamari-minion`` 禁用了代理，\
之后就会安装 ``calamari-minion`` 包、并添加定制软件库文件。安装时会优先放置 \
minion 配置，这样在 minion 首次启动时就能使用它了；然后创建配置目录、 \
calamari 的初始配置、安装 salt-minion 软件包。如果是 Redhat/CentOS 发行版，\
还需启动 salt-minion 服务。

用法： ::

	ceph-deploy calamari {connect} [HOST] [HOST...]

这里的 [HOST] 是要安装 Calamari 的主机名。

``--release`` 选项可用于选定 :program:`ceph-deploy` 配置里指定的版本，默认\
为 ``calamari-minion`` 。

此命令也支持 :option:`--master` 选项。


选项
====

.. option:: --version

   当前所安装的 :program:`ceph-deploy` 版本。

.. option:: --username

   连接远程主机所有的用户名。

.. option:: --overwrite-conf

   覆盖远程主机上的已有配置文件（若存在）。

.. option:: --cluster

   集群名字。

.. option:: --ceph-conf

   采用（或重用）指定的 ``ceph.conf`` 文件。

.. option:: --no-ssh-copykey

   不要复制 ssh 密钥。

.. option:: --fsid

   生成 ``ceph.conf`` 时另外指定 FSID 。

.. option:: --cluster-network

   指定（内部的）集群网。

.. option:: --public-network

   指定集群的公共网。

.. option:: --testing

   安装最新开发版。

.. option:: --dev

   安装最前沿版本，从 Git 分支或某标签（默认为 master ）编译而来。

.. option:: --adjust-repos

   安装会修改源软件库的软件包。

.. option:: --no-adjust-repos

   安装不会修改源软件库的软件包。

.. option:: --repo

   只安装软件库文件（跳过软件包安装）。

.. option:: --local-mirror

   抓取软件包并推送到某些主机上作为本地软件库镜像。

.. option:: --repo-url

   指定一个镜像或包含 Ceph 软件包的软件库 url 。

.. option:: --gpg-url

   指定用于定制软件库的 GPG 密钥 URL （默认为 ceph.com ）。

.. option:: --address

   将被加入集群的主机节点的 IP 地址。

.. option:: --keyrings

   串连要放置到新监视器的多个密钥环。

.. option:: --zap-disk

   销毁分区表和硬盘内容。

.. option:: --fs-type

   格式化硬盘时指定的文件系统 ``(xfs, btrfs or ext4)`` 。

.. option:: --dmcrypt

   用 ``dm-crypt`` 加密 [data-path] 和/或日志设备。

.. option:: --dmcrypt-key-dir

   ``dm-crypt`` 的密钥所在目录。

.. option:: --install

   要安装到远程主机的软件包，以逗号分隔。

.. option:: --remove

   要从远程主机删除的软件包，以逗号分隔。

.. option:: --master

   Calamari 主服务器的域。


使用范围
========

:program:`ceph-deploy` 是 Ceph 的一部分，这是个伸缩力强、开源、分布式的存储\
系统，更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph-mon <ceph-mon>`\(8),
:doc:`ceph-osd <ceph-osd>`\(8),
:doc:`ceph-disk <ceph-disk>`\(8),
:doc:`ceph-mds <ceph-mds>`\(8)
