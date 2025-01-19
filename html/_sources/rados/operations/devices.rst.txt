.. _devices:

设备管理
========
.. Device Management

设备管理是 Ceph 用来定位硬件故障的功能。
Ceph 会跟踪哪些守护进程管理着哪些硬件存储设备（如 HDD、SSD），
并收集这些设备的健康指标，以便用专门的工具来预测和/或自动应对硬件故障。


设备跟踪
--------
.. Device tracking

要查看在用存储设备的列表，执行下列命令：

.. prompt:: bash $

   ceph device ls


或者，要按守护进程或按主机列出设备，执行下列命令之一：

.. prompt:: bash $

   ceph device ls-by-daemon <daemon>
   ceph device ls-by-host <host>

要查看指定设备的位置信息以及设备的使用情况，
执行下列命令：

.. prompt:: bash $

   ceph device info <devid>


找到物理设备
------------
.. Identifying physical devices

为使更换故障磁盘更容易、更不容易出错，可以（有的机箱支持）
“闪烁（ blink ）”机箱上的 LED 硬盘指示灯： ::

  device light on|off <devid> [ident|fault] [--force]

.. note:: 使用该命令闪烁指示灯可能没用。
   是否有效取决于内核版本、 SES 固件或 HBA 设置等因素。

``<devid>`` 参数是设备标识。
要找出此信息，执行下列命令：

.. prompt:: bash $

   ceph device ls

``[ident|fault]`` 参数决定闪烁哪种灯。
默认闪烁 `identification` （辨识）灯。

.. note:: 只有启用了 Cephadm 或 Rook `orchestrator
   <https://docs.ceph.com/docs/master/mgr/orchestrator/#orchestrator-cli-module>`_ 模块，
   才能用这个命令。要查看已启用了哪个 orchestrator 模块，
   执行下列命令：

   .. prompt:: bash $

      ceph orch status

可以让硬盘 LED 灯闪烁的命令是 `lsmcli` 。
要自定义该命令，可通过 Jinja2 模板配置，
执行下列命令： ::

   ceph config-key set mgr/cephadm/blink_device_light_cmd "<template>"
   ceph config-key set mgr/cephadm/<host>/blink_device_light_cmd "lsmcli local-disk-{{ ident_fault }}-led-{{'on' if on else 'off'}} --path '{{ path or dev }}'"

下列参数可用于自定义 Jinja2 模板：

* ``on``
    一个布尔值。
* ``ident_fault``
    包含 `ident` 或 `fault` 的字符串。
* ``dev``
    设备 ID 字符串，例如 `SanDisk_X400_M.2_2280_512GB_162924424784` 。
* ``path``
    设备路径字符串，例如 `/dev/sda` 。


.. _enabling-monitoring:

打开监控
--------
.. Enabling monitoring

Ceph 还可以监控设备的健康指标。例如， SATA 硬盘执行一种名为 SMART 的标准，
可提供有关设备使用情况和健康状况的各种内部指标（例如：通电开机时长、
电源循环次数、无法恢复的读取错误次数）。其他设备类型，如 SAS 和 NVMe
也有一套类似的指标（标准略有不同）。
Ceph 可以通过 ``smartctl`` 工具收集所有这些指标。

您可以启用或禁用健康监控，执行下列命令：

.. prompt:: bash $

   ceph device monitoring on
   ceph device monitoring off


扫描
----
.. Scraping

如果打开了监控，系统会定期自动扫描设备指标。可以配置这个间隔时间，
执行下列命令：

.. prompt:: bash $

   ceph config set mgr mgr/devicehealth/scrape_frequency <seconds>

默认情况下，每 24 小时扫描一次设备指标。

要手动扫描所有设备，执行下列命令：
   
.. prompt:: bash $

   ceph device scrape-health-metrics

要扫描单个设备，执行下列命令：

.. prompt:: bash $

   ceph device scrape-health-metrics <device-id>

要扫描单个守护进程的设备，执行下列命令：

.. prompt:: bash $

   ceph device scrape-daemon-health-metrics <who>

要检出存储的某个设备的健康指标（可选特定时间戳），执行下列命令：

.. prompt:: bash $

   ceph device get-health-metrics <devid> [sample-timestamp]


故障预测
--------
.. Failure prediction

Ceph 可通过分析收集到的健康指标来预测驱动器的预期寿命和设备故障。
预测模式如下：

* *none*: 禁用设备故障预测。
* *local*: 使用 ``ceph-mgr`` 守护进程的预训练预测模型。

要配置预测模式，执行下列命令：

.. prompt:: bash $

   ceph config set global device_failure_prediction_mode <mode>

正常情况下，故障预测会在后台定期运行。因此，
预期寿命值可能只有在经过相当长的时间后才能产生。
所有设备的预期寿命可用下列命令查看：

.. prompt:: bash $

   ceph device ls

要查看指定设备的元数据，执行下列命令：

.. prompt:: bash $

   ceph device info <devid>

要明确地强行预测一个指定设备的预期寿命，
执行下列命令：

.. prompt:: bash $

   ceph device predict-life-expectancy <devid>

除了 Ceph 内部的设备故障预测外，你还可以配备设备故障的外部信息源。
要告知 Ceph 一个指定设备的预期寿命，执行下列命令：

.. prompt:: bash $

   ceph device set-life-expectancy <devid> <from> [<to>]

预期寿命是以时间段来表示的。
意思是预期寿命的不确定性是用时间范围的形式表示的，
而且可能是一个很大的时间范围。这个时间段的终点可以是空的。


健康警报
--------
.. Health alerts

``mgr/devicehealth/warn_threshold`` 配置选项控制着\
一个预期设备故障的健康检查。
如果这个设备预计将在指定的时间段内发生故障，则发出警报。

要检查存储着的所有设备的预期寿命、并生成适用的健康警报，
执行下列命令：

.. prompt:: bash $

   ceph device check-health


自动化的迁移
------------
.. Automatic Mitigation

``mgr/devicehealth/self_heal`` 选项（默认已启用）\
会自动从预计即将发生故障的设备上迁移数据。如果启用此选项，
本模块就会把此类设备标记为 ``out`` ，以便触发自动迁移。

.. note:: ``mon_osd_min_up_ratio`` 配置选项有助于防止这一过程演变为全面故障。
   如果 "self heal （自愈）" 模块标记为 ``out`` 的 OSD 数量过多，
   超过了 ``mon_osd_min_up_ratio`` 的比率值，
   集群就会启动 ``DEVICE_HEALTH_TOOMANY`` 健康检查。
   有关在这种情况下该如何操作的说明，参阅
   :ref:`DEVICE_HEALTH_TOOMANY<rados_health_checks_device_health_toomany>` 。

``mgr/devicehealth/mark_out_threshold`` 配置选项指定了自动迁移的时间段。
如果设备预计将在指定的时间段内发生故障，就会自动被标记为 ``out`` 。
