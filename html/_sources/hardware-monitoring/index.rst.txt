.. _hardware-monitoring:

硬件监控
========
.. Hardware monitoring

`node-proxy` 是一个内部名称，用于指定运行中的代理，该代理负责清点机器的硬件、
报告各种状态、还能让操作员执行某些操作。它通过 RedFish API 收集详细信息、
处理数据并将数据推送到 Ceph 管理器守护进程中的代理终结点。

.. graphviz::

     digraph G {
         node [shape=record];
         mgr [label="{<mgr> ceph manager}"];
         dashboard [label="<dashboard> ceph dashboard"];
         agent [label="<agent> agent"];
         redfish [label="<redfish> redfish"];
     
         agent -> redfish [label=" 1." color=green];
         agent -> mgr [label=" 2." color=orange];
         dashboard:dashboard -> mgr [label=" 3."color=lightgreen];
         node [shape=plaintext];
         legend [label=<<table border="0" cellborder="1" cellspacing="0">
             <tr><td bgcolor="lightgrey">图例</td></tr>
             <tr><td align="center">1. 通过 redfish API 收集数据</td></tr>
             <tr><td align="left">2. 把数据推送到 ceph mgr</td></tr>
             <tr><td align="left">3. 查询 ceph mgr</td></tr>
         </table>>];
     }

必备条件
--------
.. Limitations

目前， `node-proxy` 代理依赖于 RedFish API 。这意味着
`node-proxy` 代理和 `ceph-mgr` 守护进程都需要接入带外网络才能工作。

代理的部署
----------
.. Deploying the agent

| 第一步就是提供带外管理工具的凭证。
| 在向服务规范文件里添加主机的时候可以写上凭证。

.. code-block:: bash

  # cat host.yml
  ---
  service_type: host
  hostname: node-10
  addr: 10.10.10.10
  oob:
    addr: 20.20.20.10
    username: admin
    password: p@ssword

应用此规范：

.. code-block:: bash

  # ceph orch apply -i host.yml
  Added host 'node-10' with addr '10.10.10.10'

部署此代理：

.. code-block:: bash

  # ceph config set mgr mgr/cephadm/hw_monitoring true

CLI
---

| **orch** **hardware** **status** [hostname] [--category CATEGORY] [--format plain | json]

支持的类别（ category ）有：

* summary (default)
* memory
* storage
* processors
* network
* power
* fans
* firmwares
* criticals

实例
****

硬件健康状态概况
++++++++++++++++
.. hardware health statuses summary 

.. code-block:: bash

  # ceph orch hardware status
  +------------+---------+-----+-----+--------+-------+------+
  |    HOST    | STORAGE | CPU | NET | MEMORY | POWER | FANS |
  +------------+---------+-----+-----+--------+-------+------+
  |   node-10  |    ok   |  ok |  ok |   ok   |   ok  |  ok  |
  +------------+---------+-----+-----+--------+-------+------+

存储设备报告
++++++++++++
.. storage devices report

.. code-block:: bash

  # ceph orch hardware status IBM-Ceph-1 --category storage
  +------------+--------------------------------------------------------+------------------+----------------+----------+----------------+--------+---------+
  |    HOST    |                          NAME                          |      MODEL       |      SIZE      | PROTOCOL |       SN       | STATUS |  STATE  |
  +------------+--------------------------------------------------------+------------------+----------------+----------+----------------+--------+---------+
  |   node-10  | Disk 8 in Backplane 1 of Storage Controller in Slot 2  | ST20000NM008D-3D | 20000588955136 |   SATA   |    ZVT99QLL    |   OK   | Enabled |
  |   node-10  | Disk 10 in Backplane 1 of Storage Controller in Slot 2 | ST20000NM008D-3D | 20000588955136 |   SATA   |    ZVT98ZYX    |   OK   | Enabled |
  |   node-10  | Disk 11 in Backplane 1 of Storage Controller in Slot 2 | ST20000NM008D-3D | 20000588955136 |   SATA   |    ZVT98ZWB    |   OK   | Enabled |
  |   node-10  | Disk 9 in Backplane 1 of Storage Controller in Slot 2  | ST20000NM008D-3D | 20000588955136 |   SATA   |    ZVT98ZC9    |   OK   | Enabled |
  |   node-10  | Disk 3 in Backplane 1 of Storage Controller in Slot 2  | ST20000NM008D-3D | 20000588955136 |   SATA   |    ZVT9903Y    |   OK   | Enabled |
  |   node-10  | Disk 1 in Backplane 1 of Storage Controller in Slot 2  | ST20000NM008D-3D | 20000588955136 |   SATA   |    ZVT9901E    |   OK   | Enabled |
  |   node-10  | Disk 7 in Backplane 1 of Storage Controller in Slot 2  | ST20000NM008D-3D | 20000588955136 |   SATA   |    ZVT98ZQJ    |   OK   | Enabled |
  |   node-10  | Disk 2 in Backplane 1 of Storage Controller in Slot 2  | ST20000NM008D-3D | 20000588955136 |   SATA   |    ZVT99PA2    |   OK   | Enabled |
  |   node-10  | Disk 4 in Backplane 1 of Storage Controller in Slot 2  | ST20000NM008D-3D | 20000588955136 |   SATA   |    ZVT99PFG    |   OK   | Enabled |
  |   node-10  | Disk 0 in Backplane 0 of Storage Controller in Slot 2  | MZ7L33T8HBNAAD3  | 3840755981824  |   SATA   | S6M5NE0T800539 |   OK   | Enabled |
  |   node-10  | Disk 1 in Backplane 0 of Storage Controller in Slot 2  | MZ7L33T8HBNAAD3  | 3840755981824  |   SATA   | S6M5NE0T800554 |   OK   | Enabled |
  |   node-10  | Disk 6 in Backplane 1 of Storage Controller in Slot 2  | ST20000NM008D-3D | 20000588955136 |   SATA   |    ZVT98ZER    |   OK   | Enabled |
  |   node-10  | Disk 0 in Backplane 1 of Storage Controller in Slot 2  | ST20000NM008D-3D | 20000588955136 |   SATA   |    ZVT98ZEJ    |   OK   | Enabled |
  |   node-10  | Disk 5 in Backplane 1 of Storage Controller in Slot 2  | ST20000NM008D-3D | 20000588955136 |   SATA   |    ZVT99QMH    |   OK   | Enabled |
  |   node-10  |           Disk 0 on AHCI Controller in SL 6            |  MTFDDAV240TDU   |  240057409536  |   SATA   |  22373BB1E0F8  |   OK   | Enabled |
  |   node-10  |           Disk 1 on AHCI Controller in SL 6            |  MTFDDAV240TDU   |  240057409536  |   SATA   |  22373BB1E0D5  |   OK   | Enabled |
  +------------+--------------------------------------------------------+------------------+----------------+----------+----------------+--------+---------+

固件详情
++++++++
.. firmwares details

.. code-block:: bash

  # ceph orch hardware status node-10 --category firmwares
  +------------+----------------------------------------------------------------------------+--------------------------------------------------------------+----------------------+-------------+--------+
  |    HOST    |                                 COMPONENT                                  |                             NAME                             |         DATE         |   VERSION   | STATUS |
  +------------+----------------------------------------------------------------------------+--------------------------------------------------------------+----------------------+-------------+--------+
  |   node-10  |               current-107649-7.03__raid.backplane.firmware.0               |                         Backplane 0                          | 2022-12-05T00:00:00Z |     7.03    |   OK   |
  
  
  ... omitted output ...
  
  
  |   node-10  |               previous-25227-6.10.30.20__idrac.embedded.1-1                |             Integrated Remote Access Controller              |      00:00:00Z       |  6.10.30.20 |   OK   |
  +------------+----------------------------------------------------------------------------+--------------------------------------------------------------+----------------------+-------------+--------+

硬件严重警告报告
++++++++++++++++
.. hardware critical warnings report

.. code-block:: bash

  # ceph orch hardware status --category criticals
  +------------+-----------+------------+----------+-----------------+
  |    HOST    | COMPONENT |    NAME    |  STATUS  |      STATE      |
  +------------+-----------+------------+----------+-----------------+
  |   node-10  |   power   | PS2 Status | critical |    unplugged    |
  +------------+-----------+------------+----------+-----------------+

开发资料
--------
.. Developpers

.. py:currentmodule:: cephadm.agent
.. autoclass:: NodeProxyEndpoint
.. automethod:: NodeProxyEndpoint.__init__
.. automethod:: NodeProxyEndpoint.oob
.. automethod:: NodeProxyEndpoint.data
.. automethod:: NodeProxyEndpoint.fullreport
.. automethod:: NodeProxyEndpoint.summary
.. automethod:: NodeProxyEndpoint.criticals
.. automethod:: NodeProxyEndpoint.memory
.. automethod:: NodeProxyEndpoint.storage
.. automethod:: NodeProxyEndpoint.network
.. automethod:: NodeProxyEndpoint.power
.. automethod:: NodeProxyEndpoint.processors
.. automethod:: NodeProxyEndpoint.fans
.. automethod:: NodeProxyEndpoint.firmwares
.. automethod:: NodeProxyEndpoint.led

