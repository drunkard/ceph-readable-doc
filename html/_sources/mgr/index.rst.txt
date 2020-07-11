.. Ceph Manager Daemon
.. _ceph-manager-daemon:

=====================
 Ceph 管理器守护进程
=====================

:term:`Ceph 管理器`\ 守护进程（ ceph-mgr ）附着在监视器守护进\
程上，为外部的监控和管理系统提供了额外的监控和接口。

Ceph 从 12.x (*luminous*) 版起，日常操作也离不开 ceph-mgr 守\
护进程了。 ceph-mgr 守护进程在 11.x (*kraken*) 版是可选组件。

默认情况下，管理器守护进程不需要过多配置，只要确保它在运行即\
可。如果没有运行着的管理器守护进程，你就会看到一条相关的健康\
告警，并且 `ceph status` 输出里的某些信息会缺失或过时，除非
mgr 启动。

用你常用的部署工具，如 ceph-ansible 或 ceph-deploy ，在各监视\
器节点上配置好 ceph-mgr 守护进程。把 mgr 和监视器部署到同一节\
点不是强制的，但差不多是最合理的。


.. toctree::
    :maxdepth: 1

    安装和配置 <administrator>
    模块编程 <modules>
    编写 orchestrator 插件 <orchestrator_modules>
    仪表盘模块 <dashboard>
    Alerts 模块 <alerts>
    DiskPrediction 模块 <diskprediction>
    localpool 模块 <localpool>
    REST 风格模块 <restful>
    Zabbix 模块 <zabbix>
    Prometheus 模块 <prometheus>
    Influx 模块 <influx>
    Hello 模块 <hello>
    Telegraf 模块 <telegraf>
    Telemetry 模块 <telemetry>
    Iostat 模块 <iostat>
    OSD Support 模块 <osd_support>
    Crash 模块 <crash>
    Insights 模块 <insights>
    Orchestrator 模块 <orchestrator>
    Rook 模块 <rook>
