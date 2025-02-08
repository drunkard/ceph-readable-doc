.. _adding-and-removing-monitors:

=================
 增加/删除监视器
=================
.. Adding/Removing Monitors

你的集群启动并运行后，可以在运行时增加、或删除监视器。请参考\
`手动部署`_\ 或\ `监视器自举启动`_\ 完成初始设置。


.. _adding-monitors:

增加监视器
==========
.. Adding Monitors

Ceph 监视器是轻量级进程，它维护着集群运行图的主副本。\
一个集群可以只有一个监视器，
我们推荐生产环境至少 3 个监视器。\
Ceph 使用 `Paxos`_ 算法的一个变种对各种图、\
以及其它对集群来说至关重要的信息达成共识。\
由于 Paxos 算法的天性， Ceph 要求大部分监视器在运行，\
以形成法定人数（并因此达成共识）。

建议跑奇数个监视器。
奇数个监视器比偶数个更具弹性。
例如，如果部署了两个监视器，
就不能容忍失败并维持法定人数；
如果有三个监视器，就可以容忍一个失败；
如果有四个监视器，可以容忍一个失败；
如果有五个监视器，可以容忍两个失败；
这是为了避免可怕的 *脑裂* 现象，也同时说明了为什么奇数个最好。
简而言之， Ceph 要求大多数监视器都活着（并且能够互相通信），
但是这个大多数可以用单个监视器实现，或者 2 个监视器中的 2 个，
3 个中的 2 个， 4 个中的 3 个，以此类推。

对于比较小的、或者非关键 Ceph 多节点集群，
建议部署三个监视器；对于较大的集群，
可以把监视器增加到 5 个，以免遇到双重失效。
极少需要 7 个或更多。

正因为监视器是轻量级的，所以有可能在作为 OSD 的主机上同时运行它；\
然而，我们建议在独立的主机上运行它们，\
因为与内核的 `fsync` 问题会影响性能。\
专用的监视器节点们也能最小化停机时间，\
因为某一个节点崩溃或关机维护时，
监视器和 OSD 守护进程不会同时失活。

专用的监视器节点还能使维护工作干净利落，\
避免了在节点重启、离线或崩溃时造成 OSD 和监视器同时离线。

.. note:: 这里的\ *大多数*\ 监视器之间必须能互通，
   这样才能形成法定人数。


部署硬件
--------
.. Deploy your Hardware

如果你增加新监视器时要新增一台主机，关于其最低硬件配置请参见\
`硬件推荐`_\ 。要增加一个监视器主机，首先要安装最新版的 Linux
（如 Ubuntu 12.04 或者 RHEL 7 ）。

把监视器主机安装上架，连通网络。

.. _硬件推荐: ../../../start/hardware-recommendations


安装必要软件
------------
.. Install the Required Software

手动部署的集群， Ceph 软件包必须手动装，详情参见\
`安装软件包`_\ 。应该配置一个用户，使之可以无密码登录 SSH 、\
且有 root 权限。

.. _安装软件包: ../../../install/install-storage-cluster


.. _增加监视器（手动）:

增加监视器（手动）
------------------
.. Adding a Monitor (Manual)

本步骤创建 ``ceph-mon`` 数据目录、获取监视器运行图和监视器密钥环、
增加一个 ``ceph-mon`` 守护进程。如果这导致只有 2 个监视器守护进程，
你可以重演此步骤来增加一或多个监视器，
直到你拥有足够多 ``ceph-mon`` 达到法定人数。

现在该指定监视器的标识号了。传统上，
监视器曾用单个字母（ ``a`` 、 ``b`` 、 ``c`` ……）命名，
但你可以指定任何形式。在本文档里，
要记住 ``{mon-id}`` 应该是你所选的标识号，
不包含 ``mon.`` 前缀，如在 ``mon.a`` 中，
其 ``{mon-id}`` 是 ``a`` 。


#. 在新监视器主机上创建默认目录：

   .. prompt:: bash $

    ssh {new-mon-host}
    sudo mkdir /var/lib/ceph/mon/ceph-{mon-id}

#. 创建临时目录 ``{tmp}`` ，用以保存此过程中用到的文件。
   此目录要不同于前面步骤创建的监视器数据目录，
   且完成后可删除。

   .. prompt:: bash $

    mkdir {tmp}

#. 获取监视器密钥环， ``{tmp}`` 是密钥环文件保存路径、
   ``{filename}`` 是包含密钥的文件名。

   .. prompt:: bash $

    ceph auth get mon. -o {tmp}/{key-filename}

#. 获取监视器运行图， ``{tmp}`` 是获取到的监视器运行图、
   ``{filename}`` 是包含监视器运行图的文件名。

   .. prompt:: bash $

    ceph mon getmap -o {tmp}/{map-filename}

#. 准备第一步创建的监视器数据目录。
   必须指定监视器运行图路径，
   这样才能获得监视器法定人数和它们 ``fsid`` 的信息；
   还要指定监视器密钥环路径。

   .. prompt:: bash $

    sudo ceph-mon -i {mon-id} --mkfs --monmap {tmp}/{map-filename} --keyring {tmp}/{key-filename}

#. 启动新监视器，它会自动加入集群。
   此守护进程需知道绑定到哪个地址，
   通过 ``--public-addr {ip:port}`` 或
   ``--public-network {network}`` 参数指定，例如：

   .. prompt:: bash $

    ceph-mon -i {mon-id} --public-addr {ip:port}


.. _removing-monitors:

删除监视器
==========
.. Removing Monitors

从集群删除监视器时，必须认识到，
Ceph 监视器们用 PASOX 算法关于主集群运行图达成共识。\
必须有足够多的监视器才能对集群运行图达成共识。


.. _删除监视器（手动）:

删除监视器（手动）
------------------
.. Removing a Monitor (Manual)

本步骤从集群删除 ``ceph-mon`` 守护进程，\
如果此步骤导致只剩 2 个监视器了，你得增加或删除一个监视器，\
直到凑足法定人数所必需的 ``ceph-mon`` 数。

#. 停止监视器。

   .. prompt:: bash $

      service ceph -a stop mon.{mon-id}

#. 从集群删除监视器。

   .. prompt:: bash $

      ceph mon remove {mon-id}

#. 删除 ``ceph.conf`` 对应条目。


.. _rados-mon-remove-from-unhealthy: 

从不健康集群删除监视器
----------------------
.. Removing Monitors from an Unhealthy Cluster

本步骤从不健康集群删除 ``ceph-mon`` ，
例如集群内的监视器不能形成法定人数。

#. 停止所有监视器主机上的所有 ``ceph-mon`` 守护进程。

   .. prompt:: bash $

      ssh {mon-host}
      systemctl stop ceph-mon.target
      # 要在所有监视器主机上执行

#. 找出一个活着的监视器并登录其所在主机。

   .. prompt:: bash $

      ssh {mon-host}

#. 提取 ``monmap`` 副本，执行下列命令：

   .. prompt:: bash $

      ceph-mon -i {mon-id} --extract-monmap {map-path}

   这是个更具体的例子，在本例中， ``hostname`` 是 ``{mon-id}`` ，
   而 ``/tmp/monpap`` 是 ``{map-path}`` ：

   .. prompt:: bash $

      ceph-mon -i `hostname` --extract-monmap /tmp/monmap

#. 删除不保留或有问题的监视器：

   .. prompt:: bash $

      monmaptool {map-path} --rm {mon-id}

   例如，假设你有 3 个监视器 |---| ``mon.a`` 、 ``mon.b`` 和
   ``mon.c`` |---| ，其中仅保留 ``mon.a`` ，按如下步骤：

   .. prompt:: bash $

      monmaptool /tmp/monmap --rm b
      monmaptool /tmp/monmap --rm c

#. 把去除过监视器后剩下的 monmap 注入存活的监视器。

   .. prompt:: bash $

      ceph-mon -i {mon-id} --inject-monmap {map-path}

   比如，用下列命令把一张运行图注入 ``mon.a`` 监视器，
   执行下列命令：

   .. prompt:: bash $

      ceph-mon -i a --inject-monmap /tmp/monmap

#. 只启动保留下来的监视器。

#. 确认这些监视器形成了法定人数，执行命令 ``ceph -s`` 。

#. 已删除监视器的数据目录位于 ``/var/lib/ceph/mon`` ：
   可以把这个目录备份到安全位置、或者删掉此数据目录。
   然而，除非你能确定留下的监视器很健康、且冗余性足够，
   否则就不要删掉它。
   确保有足够的空间，用于在线的 DB 扩展和压缩，
   还要确保有空间存放归档的 DB 副本，归档的副本可以压缩。


.. _更改监视器的 IP 地址:

更改监视器的 IP 地址
====================
.. Changing a Monitor's IP Address

.. important:: 现有监视器不应该更改其 IP 地址。

监视器是 Ceph 集群的关键组件。
只有监视器们维持着法定人数，整个系统才能正常工作；而且，
只有监视器们通过它们的 IP 地址能互相发现对方，才能建立法定人数。
Ceph 对监视器的发现要求严格。

Ceph 客户端及其它 Ceph 守护进程用 ``ceph.conf`` 文件发现监视器，
然而，监视器之间用监视器运行图发现对方，而非 ``ceph.conf`` 。
这就是为什么创建新监视器时得获取当前集群的 ``monmap`` ：
就像前面\ `增加监视器（手动）`_\ 里说过的，\
``monmap`` 是 ``ceph-mon -i {mon-id} --mkfs`` 命令的必要参数之一。
下面几段解释了 Ceph 监视器的一致性要求，
和几种改 IP 的安全方法。


一致性要求
----------
.. Consistency Requirements

监视器发现集群内的其它监视器时总是参照 monmap 的本地副本，
用 monmap 而非 ``ceph.conf`` 可避免因配置错误
（例如在 ``ceph.conf`` 指定监视器地址或端口时拼写错误）
而损坏集群。正因为监视器用 ``monmaps`` 相互发现、
且共享于客户端和其它 Ceph 守护进程间，
所以 monmap 给监视器提供了苛刻的一致性保证。

苛刻的一致性要求也适用于 monmap 的更新，
因为任何有关监视器的更新、 monmap 的更改\
都通过名为 `Paxos`_ 的分布式一致性算法运行。
为保证法定人数里的所有监视器都持有同版本 monmap ，
所有监视器都要赞成 monmap 的每一次更新，
像增加、删除监视器。 monmap 的更新是增量的，
这样监视器都有最近商定的版本以及一系列之前版本，
这样可使一个有较老 monmap 的监视器赶上集群当前的状态。

如果监视器通过 Ceph 配置文件而非 monmap 相互发现，
就会引进额外风险，因为
Ceph 配置文件不会自动更新和发布。
监视器有可能用了较老的 ``ceph.conf`` 而导致不能识别某监视器、
掉出法定人数、或者发展为一种 `Paxos`_ 不能精确确定当前系统状态的情形。
总之，更改现有监视器的 IP 地址必须慎之又慎。


.. _operations_add_or_rm_mons_changing_mon_ip:

更改监视器 IP 地址（正确方法）
------------------------------
.. Changing a Monitor's IP address (The Right Way)

仅仅在 ``ceph.conf`` 里更改监视器的 IP 不足以\
让集群内的其它监视器接受更新。
要更改一个监视器的 IP 地址，
你必须以先以想用的 IP 地址增加一个监视器
（见\ `增加监视器（手动）`_\ ），
确保新监视器成功加入法定人数，然后删除用旧 IP 的监视器，
最后更新 ``ceph.conf`` 以确保客户端和其它守护进程得知新监视器的 IP 地址。

例如，我们假设有 3 个监视器，如下： ::

    [mon.a]
        host = host01
        addr = 10.0.0.1:6789
    [mon.b]
        host = host02
        addr = 10.0.0.2:6789
    [mon.c]
        host = host03
        addr = 10.0.0.3:6789

要把 ``host04`` 上 ``mon.c`` 的 IP 改为 ``10.0.0.4`` ，
按照\ `增加监视器（手动）`_\ 里的步骤增加一个新监视器 ``mon.d`` ，
确认它运行正常后再删除 ``mon.c`` ，
否则会破坏法定人数；
最后依照\ `删除监视器（手动）`_\ 删除 ``mon.c`` 。
3 个监视器都要更改的话，每次都要重复一次。


更改监视器 IP 地址（高级方法）
------------------------------
.. Changing a Monitor's IP address (Advanced Method)

在某些情况下，无法使用 :ref:`operations_add_or_rm_mons_changing_mon_ip`
里概述过的方法。比如，
可能有时候集群监视器不得不挪到不同的网络、
数据中心的不同位置、甚至完全不同的数据中心，
此时，仍然能更改监视器的 IP 地址，
但必须用不同的方法。

在这种情形下，必须用集群里每个监视器的新 IP 地址\
生成新的监视器运行图，并注入进每个监视器。
虽然这个方法并不简单，
好在这样的迁移不常见。再次重申，
现有监视器不应该更改 IP 地址。

继续按照 :ref:`operations_add_or_rm_mons_changing_mon_ip` 里的监视器配置实例，
假设打算把所有监视器
从 ``10.0.0.x`` 段迁移到 ``10.1.0.x`` 段，
并且两个网络互不相通，步骤如下：


#. 获取监视器运行图，其中
   ``{tmp}`` 是所获取的运行图路径，
   ``{filename}`` 是包含监视器运行图的文件名：

   .. prompt:: bash $

      ceph mon getmap -o {tmp}/{filename}

#. 查看监视器运行图内容：

   .. prompt:: bash $

      monmaptool --print {tmp}/{filename}

   ::

    monmaptool: monmap file {tmp}/{filename}
    epoch 1
    fsid 224e376d-c5fe-4504-96bb-ea6332a19e61
    last_changed 2012-12-17 02:46:41.591248
    created 2012-12-17 02:46:41.591248
    0: 10.0.0.1:6789/0 mon.a
    1: 10.0.0.2:6789/0 mon.b
    2: 10.0.0.3:6789/0 mon.c

#. 从监视器运行图删除现有监视器：

   .. prompt:: bash $

      monmaptool --rm a --rm b --rm c {tmp}/{filename}

   ::

    monmaptool: monmap file {tmp}/{filename}
    monmaptool: removing a
    monmaptool: removing b
    monmaptool: removing c
    monmaptool: writing epoch 1 to {tmp}/{filename} (0 monitors)

#. 把新监视器位置加进监视器运行图：

   .. prompt:: bash $

      monmaptool --add a 10.1.0.1:6789 --add b 10.1.0.2:6789 --add c 10.1.0.3:6789 {tmp}/{filename}

   ::

      monmaptool: monmap file {tmp}/{filename}
      monmaptool: writing epoch 1 to {tmp}/{filename} (3 monitors)

#. 检查监视器运行图的新内容：

   .. prompt:: bash $

       monmaptool --print {tmp}/{filename}

   ::

    monmaptool: monmap file {tmp}/{filename}
    epoch 1
    fsid 224e376d-c5fe-4504-96bb-ea6332a19e61
    last_changed 2012-12-17 02:46:41.591248
    created 2012-12-17 02:46:41.591248
    0: 10.1.0.1:6789/0 mon.a
    1: 10.1.0.2:6789/0 mon.b
    2: 10.1.0.3:6789/0 mon.c


此时，我们假设监视器（及存储）已经被安装到了新位置。
下一步，把修改过的监视器运行图散播到新监视器，
并且把它们注入每个新监视器。

#. 确认已经关停所有监视器。
   一定不能在监视器守护进程运行时注入。

#. 注入监视器运行图：

   .. prompt:: bash $

      ceph-mon -i {mon-id} --inject-monmap {tmp}/{filename}

#. 重启所有监视器。

迁移到新位置的操作现在算是完成了，监视器们应该可以正常运行了。


用 cephadm 更改公网（ public network ）
=======================================
.. Using cephadm to change the public network

概述
----
.. Overview

本概览里只是用 ``cephadm`` 更改公网的大致步骤。

#. 备份所有的密钥环、配置文件、和当前 monmap 。

#. 关停集群并禁用 ``ceph.target`` ，以防守护进程启动。

#. 搬运服务器，而后通电开机。

#. 更改所需的网络配置。


步骤实例
--------
.. Example Procedure 

.. note:: 在此步骤中，“旧网络”的地址在 ``10.10.10.0/24`` 网段内，
   而“新网络”的地址在 ``192.168.160.0/24`` 网段内。

#. 进入第一个监视器的 shell ：

   .. prompt:: bash #

      cephadm shell --name mon.reef1

#. 从 ``mon.reef1`` 提取出当前的 monmap ：

   .. prompt:: bash #
      
      ceph-mon -i reef1 --extract-monmap monmap

#. 打印 monmap 的内容：

   .. prompt:: bash #

      monmaptool --print monmap

   ::

      monmaptool: monmap file monmap
      epoch 5
      fsid 2851404a-d09a-11ee-9aaa-fa163e2de51a
      last_changed 2024-02-21T09:32:18.292040+0000
      created 2024-02-21T09:18:27.136371+0000
      min_mon_release 18 (reef)
      election_strategy: 1
      0: [v2:10.10.10.11:3300/0,v1:10.10.10.11:6789/0] mon.reef1
      1: [v2:10.10.10.12:3300/0,v1:10.10.10.12:6789/0] mon.reef2
      2: [v2:10.10.10.13:3300/0,v1:10.10.10.13:6789/0] mon.reef3

#. 删除用着旧地址的监视器：

   .. prompt:: bash #

      monmaptool --rm reef1 --rm reef2 --rm reef3 monmap

#. 增加用新地址的监视器：

   .. prompt:: bash #

      monmaptool --addv reef1 [v2:192.168.160.11:3300/0,v1:192.168.160.11:6789/0] --addv reef2 [v2:192.168.160.12:3300/0,v1:192.168.160.12:6789/0] --addv reef3 [v2:192.168.160.13:3300/0,v1:192.168.160.13:6789/0] monmap

#. 检验一下，对 monmap 做的更改是否成功：

   .. prompt:: bash #

      monmaptool --print monmap 

   ::

      monmaptool: monmap file monmap
      epoch 4
      fsid 2851404a-d09a-11ee-9aaa-fa163e2de51a
      last_changed 2024-02-21T09:32:18.292040+0000
      created 2024-02-21T09:18:27.136371+0000
      min_mon_release 18 (reef)
      election_strategy: 1
      0: [v2:192.168.160.11:3300/0,v1:192.168.160.11:6789/0] mon.reef1
      1: [v2:192.168.160.12:3300/0,v1:192.168.160.12:6789/0] mon.reef2
      2: [v2:192.168.160.13:3300/0,v1:192.168.160.13:6789/0] mon.reef3

#. 把新的 monmap 注入 Ceph 集群：

   .. prompt:: bash #

      ceph-mon -i reef1 --inject-monmap monmap

#. 在集群内的所有监视器上重复上述步骤。

#. 更新 ``/var/lib/ceph/{FSID}/mon.{MON}/config`` 。

#. 启动监视器。

#. 更新 ceph ``public_network`` ：

   .. prompt:: bash #

      ceph config set mon public_network 192.168.160.0/24

#. 更新各个管理器的配置文件（ ``/var/lib/ceph/{FSID}/mgr.{mgr}/config`` ）
   并启动它们。 Orchestrator 现在应该可用了，
   但它会尝试连接旧网络，
   因为它的主机列表包含的是旧地址。

#. 更新各主机地址，执行下列命令：

   .. prompt:: bash #

      ceph orch host set-addr reef1 192.168.160.11
      ceph orch host set-addr reef2 192.168.160.12
      ceph orch host set-addr reef3 192.168.160.13

#. 等几分钟，让 orchestrator 连接到各主机。

#. 重新配置 OSD ，这样才能自动更新掉它们的配置文件：
   
   .. prompt:: bash #
    
      ceph orch reconfig osd

*上述步骤由 Eugen Block 开发，并于 2024 年 2 月\
在 Ceph 18.2.1 (Reef) 版上测试成功。*

.. _手动部署: ../../../install/manual-deployment
.. _监视器自举启动: ../../../dev/mon-bootstrap
.. _Paxos: https://en.wikipedia.org/wiki/Paxos_(computer_science)

.. |---|   unicode:: U+2014 .. EM DASH
   :trim:
