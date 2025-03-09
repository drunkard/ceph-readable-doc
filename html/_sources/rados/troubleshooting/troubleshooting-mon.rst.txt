.. _rados-troubleshooting-mon:

================
 监视器故障排除
================
.. Troubleshooting Monitors

.. index:: monitor, high availability

即使集群出现与监视器相关的问题，也不一定会有瘫痪的危险。
如果集群失去了多个监视器，只要有足够多活着的监视器们能形成法定人数，
集群仍能保持正常运行。

如果您的集群出现监视器相关问题，建议您参考以下故障排除信息。


开始排障
========
.. Initial Troubleshooting

在排除 Ceph 监视器故障的过程中，首先要确保监视器正在运行，
并且能够与网络通信、并且在网络上能看到它。
按照本节中的步骤排除监视器故障的最简单起因。

#. **确保监视器在运行。**

   确保监视器（ *mon* ）守护进程的各个进程
   （ ``ceph-mon`` ）在运行。
   有可能是升级后没有重启监视器进程。
   查验这一简单的疏忽可以省去数小时艰苦的故障排除工作。

   确保管理器守护进程（ ``ceph-mgr`` ）在运行也很重要。
   记住，典型的集群配置为每个监视器（ ``ceph-mon`` ）
   配备了一个管理器（ ``ceph-mgr`` ）。

   .. note:: 在 v1.12.5 之前的版本中，
      Rook 不会运行两个以上管理器。

#. **确保到各个监视器节点可达。**

   在某些罕见的情况下， ``iptables`` 规则可能会拦截\
   到监视器节点或 TCP 端口的访问。
   这些规则可能是以前压力测试或开发规则时遗留下来的。
   要检查是否存在这样的规则，可通过 SSH 进入每个监视器节点，
   并用 ``telnet`` 或 ``nc`` 或类似工具尝试连接\
   其他监视器节点的 ``tcp/3300`` 和 ``tcp/6789`` 端口。

#. **确保 "ceph status" 命令能运行，并收到集群回复。**

   如果 ``ceph status`` 命令收到了来自集群的回复，
   那就说明集群已启动并正在运行。
   监视器们只有形成法定人数后才会响应 ``status`` 请求。
   确认（ ``status`` 命令）报告了一个或多个 ``mgr`` 守护进程正在运行。
   在没有缺陷的集群中， ``ceph status`` 应该报告所有 ``mgr`` 守护进程都在运行。

   如果 ``ceph status`` 命令没有收到来自集群的回复，
   则可能没有足够的监视器状态为 ``up`` 来形成法定人数。
   如果运行 ``ceph -s`` 命令时没有加更多选项，
   它将连接到任意选择的监视器。但在某些情况下，
   在命令中加上 ``-m`` 标志连接到指定监视器
   （或依次连接到多个指定监视器）：
   例如， ``ceph status -m mymon1`` 。

#. **这些都不行，怎么办？**

   如果上述解决方法没能解决你的问题，
   挨个检查监视器们也许有用。即使没有达成法定人数，
   也可以单独连线各监视器去询问其状态，
   用 ``ceph tell mon.ID mon_status`` 命令
   （其中 ID 是监视器的标识符）。

   执行 ``ceph tell mon.ID mon_status`` 命令查询集群内的各个监视器。
   关于此命令的输出详情，参阅\ :ref:`深入理解 mon_status
   <rados_troubleshoting_troubleshooting_mon_understanding_mon_status>` 。

   还有另外一种方法单独连接各个监视器：
   SSH 登录到各监视器节点、并查询那个守护进程的
   admin socket （管理套接字）。参阅 :ref:`使用监视器的管理套接字
   <rados_troubleshoting_troubleshooting_mon_using_admin_socket>` 。


.. _rados_troubleshoting_troubleshooting_mon_using_admin_socket:

使用监视器的管理套接字
======================
.. Using the monitor's admin socket

通过管理套接字（ admin socket ），你可以用 Unix 套接字文件\
直接与指定守护进程交互。这个文件位于你监视器的 ``run`` 目录下。

管理套接字默认位于 ``/var/run/ceph/ceph-mon.ID.asok`` 。
管理套接字的默认位置可以覆盖掉。
如果其默认位置被覆盖掉了，那么管理套接字就在别处。
集群守护进程部署在容器内时，常常出现这种情况。

要找出管理套接字的目录，看一下 ``ceph.conf`` 里是否配置了其它路径、
或者执行下列命令：

.. prompt:: bash $

   ceph-conf --name mon.ID --show-config-value admin_socket

只有在监视器运行时管理套接字才可用。
每次监视器正常关闭时，管理套接字会被删除；
如果监视器不运行了、但管理套接字还存在，
就说明监视器不是正常关闭的。如果监视器没在运行，
你就不能使用管理套接字，而且 ``ceph`` 命令会返回类似 \
``Error 111: Connection Refused`` 的消息。

要访问管理员套接字，执行 ``ceph tell`` 命令
（指定你感兴趣的守护进程）：

.. prompt:: bash $

   ceph tell mon.<id> mon_status

此命令通过管理套接字向指定的、运行中监视器守护进程 ``<id>`` 传入 ``help`` 命令。
如果知道管理员套接字文件的完整路径，
可以更直接地操作，执行下列命令：

.. prompt:: bash $

   ceph --admin-daemon <full_path_to_asok_file> <command>

执行 ``ceph help`` 可显示管理套接字支持的所有命令。
特别关注一下 ``config get`` 、 ``config show`` 、 ``mon stat`` \
和 ``quorum_status`` 命令。


.. _rados_troubleshoting_troubleshooting_mon_understanding_mon_status:

理解 mon_status
===============
.. Understanding mon_status

监视器的状态（由 ``ceph tell mon.X mon_status`` 命令报告的）
可通过管理套接字获取。 ``ceph tell mon.X mon_status``
命令会输出大量有关监视器的信息
（包括在 ``quorum_status`` 命令输出中找到的信息）。

.. note:: ``ceph tell mon.X mon_status`` 命令并不意味着要按字面输入。
   运行该命令时， ``mon.X`` 的 ``X`` 部分应替换为 Ceph 集群的特定值。

要了解此命令的输出，参考下面的示例，
我们将看到 ``ceph tell mon.c mon_status`` 的输出： ::
  
  { "name": "c",
    "rank": 2,
    "state": "peon",
    "election_epoch": 38,
    "quorum": [
          1,
          2],
    "outside_quorum": [],
    "extra_probe_peers": [],
    "sync_provider": [],
    "monmap": { "epoch": 3,
        "fsid": "5c4e9d53-e2e1-478a-8061-f543f8be4cf8",
        "modified": "2013-10-30 04:12:01.945629",
        "created": "2013-10-29 14:14:41.914786",
        "mons": [
              { "rank": 0,
                "name": "a",
                "addr": "127.0.0.1:6789\/0"},
              { "rank": 1,
                "name": "b",
                "addr": "127.0.0.1:6790\/0"},
              { "rank": 2,
                "name": "c",
                "addr": "127.0.0.1:6795\/0"}]}}

输出结果显示， monmap 中有三个监视器
（ ``a`` 、 ``b`` 和 ``c`` ），
法定人数由两个监视器组成，并且 ``c`` 是 ``peon`` 。

**哪个监视器在法定人数之外？**

  答案是 ``a`` （即 ``mon.a`` ）。 ``mon.a`` 在法定人数之外。

**在本例中，我们如何知道 mon.a 不在法定人数之列？**

  我们之所以知道 ``mon.a`` 不在法定人数之列，是因为它的 rank 为 ``0`` ，
  而根据定义， rank 为 ``0`` 的监视器不在法定人数之列。

  如果我们检查一下 ``quorum`` 集合，就会清楚地发现此集合里有两个监视器：
  ``1`` 和 ``2`` ，但这些不是监视器名称。
  它们是监视器的 rank ，正如当前 ``monmap`` 所确定的那样。
  法定人数集合不包括 rank 为 ``0`` 的监视器，
  而根据 ``monmap`` ，该监视器就是 ``mon.a`` 。

**监视器 rank 是如何确定的？**

  每当有监视器加入或移出集群时，都会计算（或重新计算）
  监视器的 rank 。 rank 的计算遵循一个简单的规则： ``IP:PORT`` 组合\ **越大**\ ，
  其 rank **就越低**\ 。在本例中，由于 ``127.0.0.1:6789`` (``mon.a``) 
  在数值上小于其他两个 ``IP:PORT`` 组合（“监视器 b”的组合是 ``127.0.0.1:6790``
  而“监视器 c”的是 ``127.0.0.1:6795`` ），
  因此 ``mon.a`` 的 rank 最高：即 rank ``0`` 。


最常见的监视器问题
==================
.. Most Common Monitor Issues

集群存在法定人数但是挂了不止一个监视器
--------------------------------------
.. The Cluster Has Quorum but at Least One Monitor is Down

集群达成了法定人数，但至少有一个监视器挂掉时，
``ceph health detail`` 会返回类似下面的消息： ::

      $ ceph health detail
      [snip]
      mon.a (rank 0) addr 127.0.0.1:6789/0 is down (out of quorum)

**Ceph 集群存在法定人数却至少有一个监视器挂掉时，如何排查？**

  #. 确认一下 ``mon.a`` 在运行。

  #. 确保可以从其他监视器节点连接到 ``mon.a`` 所在的节点。
     同时还要检查 TCP 端口。检查所有节点上的 ``iptables`` 和
     ``nf_conntrack`` ，确保没有丢弃/拒绝连接。

  如果这些初步的故障排除没有解决问题，
  则需要进一步调查。

  首先，通过管理套接字检查问题监视器的 ``mon_status`` ，
  方法在 `使用监视器的管理套接字`_ 和
  `理解 mon_status`_ 里介绍过了。

  如果有个监视器不在法定人数内，那么它的状态将是以下状态之一：
  ``probing`` 、 ``electing`` 或 ``synchronizing`` 。
  如果监视器的状态是 ``leader`` 或 ``peon`` ，那么这个监视器认为\
  自己在法定人数中，但集群的其余部分认为它不在法定人数中。
  处于 ``probing`` 、 ``electing`` 或 ``synchronizing`` 状态的监视器\
  有可能在故障排除过程中就已经进入了法定人数。
  在故障排除期间，再次检查 ``ceph status`` ，
  以确定监视器是否已进入了法定人数。
  如果监视器仍然在法定人数之外，
  那就继续进行本节文档所述的排查。


**监视器状态为 ``probing`` 时，是什么意思？**

  如果 ``ceph health detail`` 显示监视器的状态为 ``probing`` ，
  表示这个监视器仍在寻找其他监视器。
  每个监视器在启动时都会保持这种状态一段时间。
  当监视器连接到 ``monmap`` 中指定的其他监视器后，
  就不再处于 ``probing`` 状态。监视器处于
  ``probing`` 状态的时间长短取决于其所在集群的参数。
  例如，当监视器是单监视器集群的一部分时
  （在生产环境中切勿这样做），监视器几乎会瞬间通过 probing 状态。
  在多监视器集群中，监视器会一直处于 ``probing`` 状态，
  直到找到足够的监视器能形成法定人数为止，
  这意味着如果集群的三个监视器中有两个 ``down`` 掉了，
  剩下的一个监视器将无限期地处于 ``probing`` 状态，
  直到其他监视器中的一个启动。

  如果法定人数已经建立，只要它们能够被联系到，
  那么监视器守护进程应该能快速找到其他监视器。
  如果监视器卡在 ``probing`` 状态，
  并且您已经用尽了前述排查监视器之间通信故障的步骤，
  那么有可能是问题监视器试图用错误的地址联系其他监视器。
  ``mon_status`` 会输出这个监视器已知的 ``monmap`` ：
  确定 ``monmap`` 中指定的其他监视器的位置\
  是否与网络中监视器的位置相匹配。
  如果不匹配，参阅 :ref:`修复监视器损坏的 monmap
  <rados_troubleshooting_troubleshooting_mon_recovering_broken_monmap>` 。
  如果 ``monmap`` 中指定的监视器位置与监视器在网络中的位置一致，
  那么持续的 ``probing`` 状态可能与\
  监视器节点之间严重的时钟偏差有关。
  参阅\ `时钟偏差`_\ 。如果\ `时钟偏差`_\
  中的信息无法使监视器摆脱 ``probing`` 状态，
  则请准备好系统日志并向 Ceph 社区寻求帮助。
  有关正确准备日志的信息，参阅\ `收集所需日志`_\ 。


**监视器状态为 ``electing`` 时，是什么意思？**

  如果 ``ceph health detail`` 显示监视器的状态为 ``electing`` ，
  则表示监视器们正在进行选举。选举通常会很快完成，
  但有时监视器会陷入所谓的\ *选举风暴（ election storm ）*\ 。
  有关监视器选举的更多信息，参阅
  :ref:`监视器选举 <dev_mon_elections>` 。

  选举风暴的出现可能表明监视器节点之间存在时钟偏差。
  详情见\ `时钟偏差`_\ 。

  如果您的时钟已正确同步，请在邮件列表和 bug tracker 中搜索\
  与您的问题相似的问题。 ``electing`` 状态不太可能一直持续。
  在 Cuttlefish 版之后的 Ceph 版本中，
  除了时钟偏差之外，没有其他已知原因可以解释为何
  ``electing`` 状态会持续存在。

  如果在调查时把问题监视器置于 ``down`` 状态，
  有可能查出它持续处于 ``electing`` 状态的起因。
  这只有在监视器数量足以形成法定人数时才有可能。

**监视器状态为 ``synchronizing`` 时，是什么意思？**

  如果 ``ceph health detail`` 显示监视器状态为 ``synchronizing`` （正在同步），
  则表示监视器正在赶上集群的其他部分，以便加入法定人数。
  监视器与其他法定人数同步所需的时间\
  取决于集群监视器存储的大小、
  集群的规模以及集群的状态。
  与较小的、新集群相比，较大的和降级的集群，
  它们的监视器处于 ``synchronizing`` 状态的时间更长。

  监视器的状态从 ``synchronizing`` 变为 ``electing`` ，
  然后又变回 ``synchronizing`` ，这说明了一个问题：
  集群状态的变化（即生成新的映射图）可能太快了，
  同步过程跟不上创建新映射图的产生速度。
  这个问题在 Cuttlefish 版之前比在最近的版本中出现得更频繁，
  因为从那时起，为了避免这种动态变化，同步过程经过了重构和增强。
  如果您在之后的版本中遇到此问题，请在
  `Ceph bug 跟踪器 <https://tracker.ceph.com>`_ 中报告此问题。
  准备并提供日志以证实您提出的 bug 。
  有关正确准备日志的信息，参阅\ `收集所需日志`_\ 。


**监视器状态为 ``leader`` 或 ``peon`` 时，是什么意思？**

  集群处于 ``HEALTH_OK`` 状态下的常规 Ceph 操作期间，
  Ceph 集群中的一个监视器处于 ``leader`` 状态，
  其余监视器处于 ``peon`` 状态。可以检查
  ``ceph tell <mon_name> mon_status`` 命令\
  返回的、状态键的值来确定指定监视器的状态。

  如果 ``ceph health detail`` 显示监视器处于 ``leader`` 状态或
  ``peon`` 状态，那么很可能存在时钟偏差。
  按照\ `时钟偏差`_\ 中的指导进行操作。如果已经做过了那些操作，
  但 ``ceph health detail`` 仍然显示监视器处于 ``leader`` 状态或
  ``peon`` 状态，请在 `Ceph bug 跟踪器
  <https://tracker.ceph.com>`_ 中报告该问题。
  如果您提出了问题，请提供日志以证实该问题。
  有关正确准备日志的信息，参阅\ `收集所需日志`_\ 。


.. _rados_troubleshooting_troubleshooting_mon_recovering_broken_monmap:

修复监视器损坏的 monmap
-----------------------
.. Recovering a Monitor's Broken "monmap"

如\ :ref:`深入理解 mon_status
<rados_troubleshoting_troubleshooting_mon_understanding_mon_status>` 所述，
可以用 ``ceph tell mon.c mon_status`` 命令来获取 monmap 。

下面是一个 ``monmap`` 的示例： ::

      epoch 3
      fsid 5c4e9d53-e2e1-478a-8061-f543f8be4cf8
      last_changed 2013-10-30 04:12:01.945629
      created 2013-10-29 14:14:41.914786
      0: 127.0.0.1:6789/0 mon.a
      1: 127.0.0.1:6790/0 mon.b
      2: 127.0.0.1:6795/0 mon.c

此 ``monmap`` 正常，但您的 ``monmap`` 可能不正常。
某个节点中的 ``monmap`` 可能已经过时，因为该节点宕机了很长时间，
在此期间集群的监视器发生了变化。

更新监视器过时的 ``monmap`` 有两种方法： 

A. **废弃这些监视器并重新部署。**

    只有在确定不会丢失已报废监视器所保存信息的情况下，
    才可以这样做。确保其他监视器状态良好，
    这样新监视器才能与活着的监视器们同步。记住，
    如果没有监视器的其他内容副本，销毁监视器可能会导致数据丢失。

B. **把一份 monmap 注入监视器。**

    可以这样修复监视器：从集群中活着的监视器中提取出最新的
    ``monmap`` ，并将其注入损坏或丢失了 ``monmap`` 的监视器，
    来修复 ``monmap`` 过时了的监视器。

    执行以下步骤，施行此解决方案： 

    #. 通过以下两种方式之一提取 ``monmap`` ：

       a. **如果监视器们达成了法定人数：**

          从法定人数提取 ``monmap`` ：

             .. prompt:: bash

                ceph mon getmap -o /tmp/monmap

       b. **如果监视器们没有达成法定人数：**

          直接从已经停机的监视器提取 ``monmap`` ：

             .. prompt:: bash

                ceph-mon -i ID-FOO --extract-monmap /tmp/monmap

          在本例中，已停机的监视器的 ID 是 ``ID-FOO`` 。

    #. 停掉要注入 ``monmap`` 的那个监视器：

       .. prompt:: bash 

          service ceph -a stop mon.{mon-id}

    #. 把 monmap 注入已停掉的监视器：

       .. prompt:: bash

          ceph-mon -i ID --inject-monmap /tmp/monmap

    #. 启动这个监视器。

       .. warning:: 向监视器注入 ``monmap`` 可能会导致严重问题。
          注入 ``monmap`` 会覆盖监视器上存储的最新 ``monmap`` 。
          务必小心！


时钟偏差
--------
.. Clock Skews

Paxos 共识算法需要严格的时间同步，这意味着法定人数内部、
监视器之间的时钟偏差会对监视器的运行产生严重影响。
由此产生的行为可能会令人费解。为避免这一问题，
需要在监视器节点上运行时钟同步工具：例如，
用 ``Chrony`` 或传统的 ``ntpd`` 工具。配置每个监视器节点，
使 `iburst` 选项生效，这样每个监视器就有多个对等节点，
包括以下内容： 

* 相互之间
* 内部 ``NTP`` 服务器
* 多个外部的、公用的 pool 服务器

.. note:: ``iburst`` 选项会一次发送八个数据包组成的一组数据包，
   而不是通常的单个数据包，
   在让两个节点进行初始同步时会这样运行。

此外，最好将集群中的 *所有* 节点与内部和外部服务器同步，
甚至可能与监视器同步。要在物理裸机上运行 ``NTP`` 服务器：
VM 的虚拟化时钟不适合用于稳定的计时。
有关网络时间协议 (NTP) 的更多信息，
参阅 `https://www.ntp.org <https://www.ntp.org>`_ 。
您的组织可能已经有了高质量的内部 ``NTP`` 服务器。
``NTP`` 服务器设备的来源有以下几种：

* Microsemi (之前叫 Symmetricom) `https://microsemi.com <https://www.microsemi.com/product-directory/3425-timing-synchronization>`_
* EndRun `https://endruntechnologies.com <https://endruntechnologies.com/products/ntp-time-servers>`_
* Netburner `https://www.netburner.com <https://www.netburner.com/products/network-time-server/pk70-ex-ntp-network-time-server>`_

时钟偏差问答
~~~~~~~~~~~~
.. Clock Skew Questions and Answers

**容许的最大时钟偏差量是多少？**

  默认情况下，监视器允许时钟偏差最多 0.05 秒（ 50 毫秒）。

**我可以增大容许的最大时钟偏差吗？**

  可以，但我们强烈建议不要这样做。允许的最大时钟偏差可通过
  ``mon-clock-drift-allowed`` （允许的时钟偏差）
  选项进行配置，但更改此选项差不多肯定是个馊主意。
  之所以设置最大时钟偏差，是因为发生了时钟偏差的监视器不再可靠。
  目前的默认值已经证明了其价值，
  就是在监视器出现严重问题之前向用户发出警报。
  更改此值可能会对监视器的稳定性、
  和整个集群的健康状况造成不可预见的影响。

**我如何判断是否出现了时钟偏差？**

  监视器会通过集群状态 ``HEALTH_WARN`` 警告你。
  出现时钟偏差时， ``ceph health detail`` 和 ``ceph status``
  命令会返回类似下面的输出： ::

      mon.c addr 10.10.0.1:6789/0 clock skew 0.08235s > max 0.05s (latency 0.0045s)

  在本例中，监视器 ``mon.c`` 被标记为遭遇了时钟偏差。

  在 Luminous 及其后续版本中，可以执行
  ``ceph time-sync-status`` 命令来检查时钟偏差。
  注意， lead 监视器通常拥有数值最小的 IP 地址。
  它将始终显示 ``0`` ：其他监视器报告的偏移量是相对于 lead 监视器的，
  而不是相对于哪一个外部源的。

**如果出现了时钟偏差，我该怎么办？**

  同步时钟。可能要靠 NTP 客户端。不过，
  如果您已经在使用 NTP 客户端，但仍遇到时钟偏差问题，
  请确定您使用的 NTP 服务器是位于远程网络的\
  还是托管在您自己的网络上。
  搭建自己的 NTP 服务器往往能减轻时钟偏差问题。


客户端不能连接或挂载
--------------------
.. Client Can't Connect or Mount

如果一个客户端不能连接到集群或不能挂载，检查防火墙配置。
有些操作系统安装工具把 ``REJECT`` 规则加入了 ``iptables`` ，
它会拒绝除 ``ssh`` 以外的所有入栈连接。
如果你的监视器主机的 iptables 有这样的 ``REJECT`` 规则，
别的客户端进来的连接就会失败，进而导致超时错误。
得先找到拒绝客户端连接 Ceph 守护进程的 ``iptables`` 规则。
例如： ::

   REJECT all -- anywhere anywhere reject-with icmp-host-prohibited

你也许还要在 Ceph 主机上增加 iptables 规则\
来放通 Ceph 监视器的 TCP 端口（默认是 6789 端口）、
和 Ceph OSD 端口（默认从 6800 到 7568 ）。例如： ::

   iptables -A INPUT -m multiport -p tcp -s {ip-address}/{netmask} --dports 6789,6800:7300 -j ACCEPT


监视器存储故障
==============
.. Monitor Store Failures

存储损坏的症状
--------------
.. Symptoms of store corruption

Ceph 监视器把\ :term:`集群运行图`\ 存储在键值数据库里。如果某个监视器\
由于键值存储损坏而发生故障，监视器日志里可能出现如下错误消息： ::

  Corruption: error in middle of record

或者： ::

  Corruption: 1 missing files; e.g.: /var/lib/ceph/mon/mon.foo/store.db/1234567.ldb


用健康的监视器恢复
------------------
.. Recovery using healthy monitor(s)

如果集群内还有幸存的监视器，就可以用新监视器\
:ref:`替换掉 <adding-and-removing-monitors>`\ 损坏的。新监视器启动后，
会与健康节点同步。新监视器完全同步后，就可以服务客户端了。


.. _mon-store-recovery-using-osds:

用 OSD 恢复
-----------
.. Recovery using OSDs

即使所有监视器同时发生故障，仍然有可能通过存储在 OSD 中的信息来恢复监视器的存储。
我们鼓励您在一个 Ceph 集群中至少部署三个（最好是五个）监视器。
在这样的部署中，监视器不太可能全部发生故障。但是，
数据中心意外断电，同时磁盘配备或文件系统选项配置不当，
可能会导致底层文件系统故障，并导致所有监视器瘫痪。
在这种情况下，可以用 OSD 中的数据来恢复监视器。
下面是个脚本，可用于在这种情况下恢复监视器：

.. code-block:: bash

  ms=/root/mon-store
  mkdir $ms

  # 从已关停的 OSD 收集集群运行图
  for host in $hosts; do
    rsync -avz $ms/. user@$host:$ms.remote
    rm -rf $ms
    ssh user@$host <<EOF
      for osd in /var/lib/ceph/osd/ceph-*; do
        ceph-objectstore-tool --data-path \$osd --no-mon-config --op update-mon-db --mon-store-path $ms.remote
      done
  EOF
    rsync -avz user@$host:$ms.remote/. $ms
  done

  # 用收集来的运行图重建监视器存储，如果集群没用 cephx 认证，\
  # 我们可以跳过更新密钥环的步骤，也不用加 --keyring 选项了，\
  # 就是说可以直接运行 ``ceph-monstore-tool $ms rebuild``
  ceph-authtool /path/to/admin.keyring -n mon. \
    --cap mon 'allow *'
  ceph-authtool /path/to/admin.keyring -n client.admin \
    --cap mon 'allow *' --cap osd 'allow *' --cap mds 'allow *'
  # add one or more ceph-mgr's key to the keyring. in this case, an encoded key
  # for mgr.x is added, you can find the encoded key in
  # /etc/ceph/${cluster}.${mgr_name}.keyring on the machine where ceph-mgr is
  # deployed
  ceph-authtool /path/to/admin.keyring --add-key 'AQDN8kBe9PLWARAAZwxXMr+n85SBYbSlLcZnMA==' -n mgr.x \
    --cap mon 'allow profile mgr' --cap osd 'allow *' --cap mds 'allow *'
  # If your monitors' ids are not sorted by ip address, please specify them in order.
  # For example. if mon 'a' is 10.0.0.3, mon 'b' is 10.0.0.2, and mon 'c' is  10.0.0.4,
  # please passing "--mon-ids b a c".
  # In addition, if your monitors' ids are not single characters like 'a', 'b', 'c', please
  # specify them in the command line by passing them as arguments of the "--mon-ids"
  # option. if you are not sure, please check your ceph.conf to see if there is any
  # sections named like '[mon.foo]'. don't pass the "--mon-ids" option, if you are
  # using DNS SRV for looking up monitors.
  ceph-monstore-tool $ms rebuild -- --keyring /path/to/admin.keyring --mon-ids alpha beta gamma

  # 备份一下损坏的 store.db 以防万一！
  # 所有监视器上都要备份一下。
  mv /var/lib/ceph/mon/mon.foo/store.db /var/lib/ceph/mon/mon.foo/store.db.corrupted

  # move rebuild store.db into place.  repeat for all monitors.
  mv $ms/store.db /var/lib/ceph/mon/mon.foo/store.db
  chown -R ceph:ceph /var/lib/ceph/mon/mon.foo/store.db

此脚本会执行下列步骤：

#. 从所有 OSD 收集映射图
#. 然后重建监视器存储
#. 把各项目加进密钥环文件，并分配相应的能力
#. 用恢复好的副本替换 ``mon.foo`` 上损坏的存储。


已知的局限性
~~~~~~~~~~~~
.. Known limitations

上述恢复工具无法恢复以下信息：

- **某些加过的密钥环**\ ：所有用 ``ceph auth add`` 命令加上的
  OSD 密钥环都从 OSD 副本中恢复了； ``client.admin`` 密钥环也用
  ``ceph-monstore-tool`` 导入了。但是，在已恢复的监视器存储中，
  MDS 密钥环和其它所有密钥环都会丢失，你也许得手动重加。

- **正在创建的存储池**: 如果有过正在创建的 RADOS 存储池，
  那些状态会丢失。恢复工具操作时假定：所有存储池都已创建。
  如果在没创建完的存储池恢复后，有 PG 卡在 ``unknown`` 状态，
  可以运行 ``ceph osd force-create-pg`` 命令强制创建 *空* PG 。
  此命令会创建一个 *空* PG ，因此确定存储池为空时才可以执行此操作。
  （译者：否则可能清空此 PG ）

- **MDS 映射图**\ ： MDS 的各种映射图会丢失。


所有尝试都失败了，怎么办？
==========================
.. Everything Failed! Now What?

到外面寻求帮助
--------------
.. Reaching out for help

您可以在 OFTC（服务器 irc.oftc.net）上的 #ceph 和 #ceph-devel IRC 频道中，
或在 ``dev@ceph.io`` 和 ``ceph-users@lists.ceph.com`` 中寻求帮助。
先准备好日志，并在发出请求时将其准备就绪。

可以通过这个地址加入上游 Ceph Slack 工作区：
https://ceph-storage.slack.com/ 

与上游 Ceph 社区取得联系的最新信息（截至 2023 年 12 月），
参见 https://ceph.io/en/community/connect/ 。


收集所需日志
------------
.. Preparing your logs

监视器日志的默认位置是 ``/var/log/ceph/ceph-mon.FOO.log*`` 。
监视器日志的位置可能已经改了，不是默认位置。
如果监视器日志不在默认位置，可执行以下命令查找监视器日志的位置：

.. prompt:: bash

   ceph-conf --name mon.FOO --show-config-value log_file

日志中的信息量由集群配置文件中的调试级别决定。
如果 Ceph 在用的是默认调试级别，
那么您的日志可能会错过重要信息，
这些信息有助于上游 Ceph 社区定位问题。

提高调试级别以确保监视器日志包含相关信息。
在此，我们对来自监视器的信息感兴趣。
与其他组件一样，监视器也有不同的部分，
输出不同子系统的调试信息。

如果您是一位经验丰富的 Ceph 故障排除者，我们建议您提高最相关子系统的调试级别。
这种方法对初学者来说可能不太容易。但在大多数情况下，
如果输入以下调试级别，就能记录足够的信息来定位问题： ::

      debug_mon = 10
      debug_ms = 1

有时，这些调试级别没能产生足够的信息。在这种情况下，
上游 Ceph 社区成员会让您对这些或其他调试级别进行额外更改。
无论如何，对我们来说，收到一些有用的信息总比收到空日志要好。


我需要重启监视器来更改调试级别吗？
----------------------------------
.. Do I need to restart a monitor to adjust debug levels?

不需要。更改监视器的调试级别时没必要重启它。

更改调试级别有两种方法。一种方法是在有法定人数时使用。
另一种方法是在没有法定人数时使用。

**有法定人数时更改调试级别** 

  把调试选项注入需要调试的指定监视器::

        ceph tell mon.FOO config set debug_mon 10/10

  或者一次性注入所有监视器： ::

        ceph tell mon.* config set debug_mon 10/10


**没有法定人数时更改调试级别**

  使用需要调试的指定监视器的管理套接字，
  直接调整这个监视器的配置选项： ::

      ceph daemon mon.FOO config set debug_mon 10/10


**把调试级别恢复成它的默认值**

要把调试级别恢复成默认值，应该用调试级别 ``1/10``
而不是调试级别 ``10/10`` 运行上述命令。
要检查监视器的当前值，用管理套接字并运行\
下列任一命令：

  .. prompt:: bash

     ceph daemon mon.FOO config show

或者：

  .. prompt:: bash

     ceph daemon mon.FOO config get 'OPTION_NAME'



我在某个调试级别下重现了问题，然后呢？
--------------------------------------
.. I Reproduced the problem with appropriate debug levels. Now what?

只需向上游 Ceph 社区发送日志中\
与监视器问题相关的部分即可。
由于确定哪些部分是相关的并不容易，
因此上游 Ceph 社区接受完整且未经删节的日志。
但不要发送包含成千上万行且没有额外说明的日志。
有助于让 Ceph 社区帮助您的一个常识性方法是，
记下您重现问题当时的时间和日期，
然后根据此信息提取日志对应部分的内容。

联系上游 Ceph 社区，可以在邮件列表、 IRC 或 Slack 上、
或在 `tracker`_ 上提交新问题。


.. _tracker: http://tracker.ceph.com/projects/ceph/issues/new
