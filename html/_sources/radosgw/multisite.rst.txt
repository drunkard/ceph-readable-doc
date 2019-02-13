.. _Multi-Site:

======
 多站
======

.. versionadded:: Jewel

单个域的配置通常包括一个域组，其内有一个域，域内有一或多个（可\
以对客户端的网关请求做负载均衡） `ceph-radosgw` 例程。在单域配\
置中，多个网关例程通常指向一个 Ceph 存储集群，而 Kraken 版的 \
Ceph 对象网关支持几种多站配置方案：

- **多域（ Multi-zone ）：** 比较高级的配置包括一个域组和多个\
  域，每个域里有一或多个 `ceph-radosgw` 例程，每个域背后都配置\
  了自己的 Ceph 存储集群。如果某个域有重大损伤，域组内的多个域\
  还能实现灾难恢复。在 Kraken 版中，每个域都处于活跃状态，可处\
  理写操作。除灾难恢复外，多个活跃的域也可以作内容分发网络的底\
  层。

- **多域组（ Multi-zone-group ）：** 以前称为“region”。 Ceph \
  对象网关也支持多域组，每个域组内可以有一或多个域。只要确保对\
  象 ID 在域组和域中唯一，那么在同一 realm 内，可以跨域组和域\
  共享同一个全局对象命名空间。

- **多个 realm ：** Kraken 版的 Ceph 对象网关支持 realm 概念，\
  它是单个或多个域组、和 realm 的一个全局唯一的命名空间。多
  realm 功能使之有能力支持多种配置方案和命名空间。

同一域组内、两个域之间的数据复制情形如下：

.. image:: ../images/zone-sync2.png
   :align: center

网关集群的配置细节请参考\ `产品级的 Ceph 对象网关
<https://access.redhat.com/documentation/en-us/red_hat_ceph_storage/2/html/ceph_object_gateway_for_production/>`__\ 。




.. Functional Changes from Infernalis

从 Infernalis 以来功能上的变化
==============================

在 Kraken 里，你可以让各 Ceph 对象网关使用多活域配置，这样就可\
以写入非主域。

多站配置信息存储在名为 realm 的容器内， realm 内存储着域组、\
域、和带有多个时间结的 peroid ，用于跟踪配置变更。在 Kraken \
里，同步由 ``ceph-radosgw`` 守护进程处理，无需再单独配置同步代\
理。另外，新的同步方法允许 Ceph 对象网关以多活（ active-active
）模式运行，不再是主从（ active-passive ）模式。




.. Requirements and Assumptions

环境需求和假设
==============

一套多站配置需要至少两个 Ceph 存储集群，最好有不同的集群名。至\
少两个 Ceph 对象网关例程，分别连接两个 Ceph 存储集群。

本指南中，我们假设至少有两个地理位置不同的 Ceph 存储集群，其实\
都在本地也能跑通；还假设有两个 Ceph 对象网关服务器，分别名为
``rgw1`` 和 ``rgw2`` 。

一套多站配置必须有一个主域组和一个主域，另外，各域组都得有自己\
的主域；域组可以有一或多个副域组、副域。

在本指南中， ``rgw1`` 主机将在主域组的主域中提供服务；而
``rgw2`` 主机将在主域组的副域中提供服务。




.. Pools

存储池
======

我们建议用 `Ceph 归置组的单存储池计算器 <http://ceph.com/pgcalc/>`__\
给这些存储池计算个合适的归置组数量， ``ceph-radosgw`` 守护进程\
创建存储池时会用到。把算出的值写入 Ceph 配置文件，作为默认值，\
例如： ::

    osd pool default pg num = 50
    osd pool default pgp num = 50

.. note:: 把这些变更写入你的 Ceph 配置文件，然后使之生效，这样\
   网关例程在创建存储池时就会使用这些默认值了。

或者，手动创建存储池，创建细节请参考\ `存储池
<http://docs.ceph.com/docs/master/rados/operations/pools/#pools>`__\ 。

一个域的存储池名称应该遵循命名惯例 ``{zone-name}.pool-name`` ，\
例如，名为 ``us-east`` 的域需要创建如下存储池：

-  ``.rgw.root``

-  ``us-east.rgw.control``

-  ``us-east.rgw.data.root``

-  ``us-east.rgw.gc``

-  ``us-east.rgw.log``

-  ``us-east.rgw.intent-log``

-  ``us-east.rgw.usage``

-  ``us-east.rgw.users.keys``

-  ``us-east.rgw.users.email``

-  ``us-east.rgw.users.swift``

-  ``us-east.rgw.users.uid``

-  ``us-east.rgw.buckets.index``

-  ``us-east.rgw.buckets.data``




.. _Configuring a Master Zone:

配置主域
========

在多站配置中，所有网关都会向主域组中、主域内的 ``ceph-radosgw``
守护进程索取配置。所以在多站配置中，要配置各网关，需要挑一个
``ceph-radosgw`` 例程，在它上面配置主域组和主域。


.. Create a Realm

创建 realm
----------

realm 包含域组和域的多站配置信息，同时在 realm 内强制施行全局\
唯一的命名空间。

在为主域组和域提供服务的主机上，打开命令行，执行下列命令创建多\
站配置所需的新 realm::

    # radosgw-admin realm create --rgw-realm={realm-name} [--default]

例如： ::

    # radosgw-admin realm create --rgw-realm=movies --default

如果此集群上只会配置一个 realm ，可加上 ``--default`` 标记，指\
定 ``--default`` 以后， ``radosgw-admin`` 就会默认使用这个
realm ；如果没指定 ``--default`` ，新增域组和域时就必须指定
``--rgw-realm`` 或 ``--realm-id`` 标记，以确定在哪个 realm 新\
建。

创建 realm 后， ``radosgw-admin`` 会顺便显示此 realm 的配置信\
息，例如： ::

    {
        "id": "0956b174-fe14-4f97-8b50-bb7ec5e1cf62",
        "name": "movies",
        "current_period": "1950b710-3e63-4c41-a19e-46a715000980",
        "epoch": 1
    }

.. note:: Ceph 会给这个 realm 生成唯一的 ID ，这样，需要的话可\
   以随时改名。


.. _Create a Master Zone Group:

创建主域组
----------

一个 realm 必须至少有一个域组，只有一个的话它将作为此 realm 的\
主域组。

在为主域组和域提供服务的主机上，打开命令行，执行下列命令新建多\
站配置所需的主域组： ::

    # radosgw-admin zonegroup create --rgw-zonegroup={name} --endpoints={url} [--rgw-realm={realm-name}|--realm-id={realm-id}] --master --default

例如： ::

    # radosgw-admin zonegroup create --rgw-zonegroup=us --endpoints=http://rgw1:80 --rgw-realm=movies --master --default

如果这个 realm 只有一个域组，创建时可加上 ``--default`` 标记，\
指定 ``--default`` 后， ``radosgw-admin`` 在创建新域时将默认使\
用此域组；如果未指定 ``--default`` 标记，新增或修改域时就得指定
``--rgw-zonegroup`` 或 ``--zonegroup-id`` 标记，以确定在哪个域\
组内操作。

创建主域组后， ``radosgw-admin`` 会显示这个域组的配置，例如： ::

    {
        "id": "f1a233f5-c354-4107-b36c-df66126475a6",
        "name": "us",
        "api_name": "us",
        "is_master": "true",
        "endpoints": [
            "http:\/\/rgw1:80"
        ],
        "hostnames": [],
        "hostnames_s3webzone": [],
        "master_zone": "",
        "zones": [],
        "placement_targets": [],
        "default_placement": "",
        "realm_id": "0956b174-fe14-4f97-8b50-bb7ec5e1cf62"
    }


.. _Create a Master Zone:

创建主域
--------

.. important:: 域必须在此域内的 Ceph 对象网关所在节点上创建。

在为主域组和域提供服务的主机上，打开命令行，执行下列命令新建多\
站配置所需的主域： ::

    # radosgw-admin zone create --rgw-zonegroup={zone-group-name} \
                                --rgw-zone={zone-name} \
                                --master --default \
                                --endpoints={http://fqdn}[,{http://fqdn}]

例如： ::

    # radosgw-admin zone create --rgw-zonegroup=us --rgw-zone=us-east \
                                --master --default \
                                --endpoints={http://fqdn}[,{http://fqdn}]

.. note:: 这里没有指定 ``--access-key`` 和 ``--secret`` ，在后\
   面的章节中创建用户后，再把这些配置写入域。

.. important:: 后续步骤假设是在新安装好的系统上实施多站配置，\
   上面还没有数据。如果你已经用它存储了一些数据，\ **不要删除**
   ``default`` 域及其存储池，否则数据会被删除且不可恢复。


.. _Delete Default Zone Group and Zone:

删除默认域组与域
----------------

如果有 ``default`` 域，要先从域组里删除，然后再删掉它。 ::

    # radosgw-admin zonegroup remove --rgw-zonegroup=default --rgw-zone=default
    # radosgw-admin period update --commit
    # radosgw-admin zone delete --rgw-zone=default
    # radosgw-admin period update --commit
    # radosgw-admin zonegroup delete --rgw-zonegroup=default
    # radosgw-admin period update --commit

最后，如果这个 Ceph 存储集群里还有 ``default`` 存储池，也需一\
并删除。

.. important:: 后续步骤假设是在新安装好的系统上实施多站配置，\
   上面还没有数据。如果你已经用它存储了一些数据，\ **不要删除**
   ``default`` 域及其存储池，否则数据会被删除且不可恢复。

::

    # rados rmpool default.rgw.control default.rgw.control --yes-i-really-really-mean-it
    # rados rmpool default.rgw.data.root default.rgw.data.root --yes-i-really-really-mean-it
    # rados rmpool default.rgw.gc default.rgw.gc --yes-i-really-really-mean-it
    # rados rmpool default.rgw.log default.rgw.log --yes-i-really-really-mean-it
    # rados rmpool default.rgw.users.uid default.rgw.users.uid --yes-i-really-really-mean-it


.. Create a System User

创建系统用户
------------

``ceph-radosgw`` 守护进程在拉取 realm 和 peroid 信息前必须先通\
过认证。在主域里，创建一个系统用户，用于守护进程之间的认证。 ::

    # radosgw-admin user create --uid="{user-name}" --display-name="{Display Name}" --system

例如： ::

    # radosgw-admin user create --uid="synchronization-user" --display-name="Synchronization User" --system

记下 ``access_key`` 和 ``secret_key`` 的内容，因为副域需要用它\
们与主域认证。

最后，把系统用户加入主域。 ::

    # radosgw-admin zone modify --rgw-zone=us-east --access-key={access-key} --secret={secret}
    # radosgw-admin period update --commit


.. Update the Period

更新 period
-----------

更新主域配置信息后，再更新 peroid::

    # radosgw-admin period update --commit

.. note:: 更新 period 会更改 epoch ，还需确保其它的域会收到更\
   新过的配置信息。


.. Update the Ceph Configuration File

更新 Ceph 配置文件
------------------

更新主域所在主机上的 Ceph 配置文件，把 ``rgw_zone`` 配置选项和\
主域的名字写在例程配置段下： ::

    [client.rgw.{instance-name}]
    ...
    rgw_zone={zone-name}

例如： ::

    [client.rgw.rgw1]
    host = rgw1
    rgw frontends = "civetweb port=80"
    rgw_zone=us-east


.. Start the Gateway

启动网关
--------

在对象网关所在的主机上，启动 Ceph 对象网关、并启用服务： ::

    # systemctl start ceph-radosgw@rgw.`hostname -s`
    # systemctl enable ceph-radosgw@rgw.`hostname -s`




.. _Configure Secondary Zones:

配置副域
========

一个域组内的域们会复制所有数据，以确保各个域都有相同的数据。创\
建副域需在作为副域的主机上执行下面的所有操作。

.. note:: 增加第三个域和增加副域的过程相同，必须用不同的域名称。

.. important:: 你必须在主域内的主机上执行元数据操作，如用户创\
   建。主域和副域都可以处理桶操作，但是副域会把桶操作重定向到\
   主域；如果主域倒下了，桶操作会失败。


.. Pull the Realm

拉取 realm
----------

用主域组中主域的 URL 、访问密钥和私钥可以把 realm 拉到本主机。\
如果要拉取的不是默认 realm ，还需用 ``--rgw-realm`` 或
``--realm-id`` 选项指定 realm 。 ::

    # radosgw-admin realm pull --url={url-to-master-zone-gateway} --access-key={access-key} --secret={secret}

如果这是默认 realm 或仅有的一个 realm ，可以让它成为默认 realm::

    # radosgw-admin realm default --rgw-realm={realm-name}


.. Pull the Period

拉取 period
-----------

用主域组中主域的 URL 、访问密钥和私钥可以把 period 拉到本主机。\
如果要从非默认 realm 拉取 period ，还需用 ``--rgw-realm`` 或
``--realm-id`` 选项指定 realm ::

    # radosgw-admin period pull --url={url-to-master-zone-gateway} --access-key={access-key} --secret={secret}


.. note:: 拉取 period 会获取到此 realm 最新版本的域组和域配置\
   信息。


.. Create a Secondary Zone

创建副域
--------

.. important:: 域必须在此域内的 Ceph 对象网关所在节点上创建。

在副域内提供服务的主机上，打开命令行新建多站配置所需的副域，需\
指定域组 ID 、新的域名和这个域内配置的终结点；\ **不要加**
``--master`` 或 ``--default`` 标记。在 Kraken 里，所有域都按多\
活配置运行，也就是说，网关客户端可写入任意一个域，这个域会把数\
据复制到同一域组内、除此之外的其它域上。如果不想让副域处理写操\
作，创建时可以加 ``--read-only`` 标记，这样主域和副域就会按主\
从方式配置。另外，还需提供系统用户的 ``access_key`` 和
``secret_key`` ，它存储在主域组的主域内。命令如下： ::

    # radosgw-admin zone create --rgw-zonegroup={zone-group-name}\
                                --rgw-zone={zone-name} --endpoints={url} \
                                --access-key={system-key} --secret={secret}\
                                --endpoints=http://{fqdn}:80 \
                                [--read-only]

例如： ::

    # radosgw-admin zone create --rgw-zonegroup=us --rgw-zone=us-west \
                                --access-key={system-key} --secret={secret} \
                                --endpoints=http://rgw2:80


.. important:: 后续步骤假设是在新安装好的系统上实施多站配置，\
   上面还没有数据。如果你已经用它存储了一些数据，\ **不要删除**
   ``default`` 域及其存储池，否则数据会被删除且不可恢复。

如有必要，删除默认域： ::

    # radosgw-admin zone delete --rgw-zone=default

最后，删除 Ceph 存储集群内的默认存储池。 ::

    # rados rmpool default.rgw.control default.rgw.control --yes-i-really-really-mean-it
    # rados rmpool default.rgw.data.root default.rgw.data.root --yes-i-really-really-mean-it
    # rados rmpool default.rgw.gc default.rgw.gc --yes-i-really-really-mean-it
    # rados rmpool default.rgw.log default.rgw.log --yes-i-really-really-mean-it
    # rados rmpool default.rgw.users.uid default.rgw.users.uid --yes-i-really-really-mean-it


.. Update the Ceph Configuration File

更新 Ceph 配置文件
------------------

更新副域所在主机上的 Ceph 配置文件，把 ``rgw_zone`` 配置选项和\
副域的名字写在例程配置段下： ::

    [client.rgw.{instance-name}]
    ...
    rgw_zone={zone-name}

例如： ::

    [client.rgw.rgw2]
    host = rgw2
    rgw frontends = "civetweb port=80"
    rgw_zone=us-west


.. Update the Period

更新 period
-----------

更新完主域配置信息后，更新 period::

    # radosgw-admin period update --commit

.. note:: 更新 period 会更改 epoch ，还需确保其它的域会收到更\
   新过的配置信息。


.. Start the Gateway

启动网关
--------

在对象网关所在的主机上，启动 Ceph 对象网关、并启用服务： ::

    # systemctl start ceph-radosgw@rgw.`hostname -s`
    # systemctl enable ceph-radosgw@rgw.`hostname -s`


.. Check Synchronization Status

检查同步状态
------------

副域起来并正常运行后，检查一下同步状态。同步就是把主域中创建的\
用户和桶都复制到副域。

::

    # radosgw-admin sync status

此命令的输出会显示同步操作的状态，例如： ::

    realm f3239bc5-e1a8-4206-a81d-e1576480804d (earth)
        zonegroup c50dbb7e-d9ce-47cc-a8bb-97d9b399d388 (us)
             zone 4c453b70-4a16-4ce8-8185-1893b05d346e (us-west)
    metadata sync syncing
                  full sync: 0/64 shards
                  metadata is caught up with master
                  incremental sync: 64/64 shards
        data sync source: 1ee9da3e-114d-4ae3-a8a4-056e8a17f532 (us-east)
                          syncing
                          full sync: 0/128 shards
                          incremental sync: 128/128 shards
                          data is caught up with source

.. note:: 副域可以接受桶操作，然而它们会把桶操作重定向到主域，\
   然后再与主域同步，获取桶操作的结果。如果主域倒下了，副域上\
   的桶操作会失败，但是对象操作仍会成功。




.. Maintenance

维护
====


.. Checking the Sync Status

检查同步状态
------------

某个域的复制状态可以这样查询： ::

    $ radosgw-admin sync status
            realm b3bc1c37-9c44-4b89-a03b-04c269bea5da (earth)
        zonegroup f54f9b22-b4b6-4a0e-9211-fa6ac1693f49 (us)
             zone adce11c9-b8ed-4a90-8bc5-3fc029ff0816 (us-2)
            metadata sync syncing
                  full sync: 0/64 shards
                  incremental sync: 64/64 shards
                  metadata is behind on 1 shards
                  oldest incremental change not applied: 2017-03-22 10:20:00.0.881361s
        data sync source: 341c2d81-4574-4d08-ab0f-5a2a7b168028 (us-1)
                          syncing
                          full sync: 0/128 shards
                          incremental sync: 128/128 shards
                          data is caught up with source
                  source: 3b5d1a3f-3f27-4e4a-8f34-6072d4bb1275 (us-3)
                          syncing
                          full sync: 0/128 shards
                          incremental sync: 128/128 shards
                          data is caught up with source


.. _Changing the Metadata Master Zone:

更改元数据主域
--------------

.. important::
   把某个域改为元数据主域时要格外小心。如果一个域还没与当前的\
   主域同步完元数据，那么它晋级成为主域后，不能为尚未同步完的\
   条目提供服务，而且这些变更将丢失。有鉴于此，我们建议先等这\
   个域的元数据同步 ``radosgw-admin sync status`` 赶上后再把它\
   晋级为主域。

   同样，如果元数据变更是由当前的主域处理的，此时另一个域却被\
   晋级成了主域，那么这些变更会也丢失。为避免出现此类情况，建\
   议关闭先前主域内的所有 ``radosgw`` 例程；等晋级完另一个域之\
   后，可以用 ``radosgw-admin period pull`` 拉取新的 period ，\
   并启动先前停掉的网关。

要想把一个域（例如 ``us`` 域组内的 ``us-2`` 域）晋级为元数据主\
域，在这个域上做如下操作： ::

    $ radosgw-admin zone modify --rgw-zone=us-2 --master
    $ radosgw-admin zonegroup modify --rgw-zonegroup=us --master
    $ radosgw-admin period update --commit

这样就会生成一个新 period ，而且 ``us-2`` 域内的 radosgw 例程\
会把这个 period 发给其它域。




.. Failover and Disaster Recovery

故障切换和灾难恢复
==================

如果主域失败，则切换到副域以作灾难恢复。

#. 让副域成为默认的主域，例如： ::

       # radosgw-admin zone modify --rgw-zone={zone-name} --master --default

   默认情况下， Ceph 对象网关运行在多活模式下。如果集群被配置\
   成了主从模式，那么副域是个只读域，需要去除 ``--read-only``
   状态，以允许这个域处理写操作。例如： ::

       # radosgw-admin zone modify --rgw-zone={zone-name} --master --default \
                                   --read-only=False

#. 更新 period 以使变更生效。 ::

       # radosgw-admin period update --commit

#. 最后，重启 Ceph 对象网关。 ::

       # systemctl restart ceph-radosgw@rgw.`hostname -s`


如果前任主域恢复了，还原上述操作。

#. 在已恢复的域里，从当前的主域拉取 period::

       # radosgw-admin period pull --url={url-to-master-zone-gateway} \
                                   --access-key={access-key} --secret={secret}

#. 让恢复的域成为默认的主域，例如： ::

       # radosgw-admin zone modify --rgw-zone={zone-name} --master --default

#. 更新 period 以使变更生效： ::

       # radosgw-admin period update --commit

#. 然后，在恢复好的域里重启 Ceph 对象网关。 ::

       # systemctl restart ceph-radosgw@rgw.`hostname -s`

#. 如果副域还要恢复为只读配置，更新一下副域。 ::

       # radosgw-admin zone modify --rgw-zone={zone-name} --read-only

#. 更新 period 以使变更生效。 ::

       # radosgw-admin period update --commit

#. 最后，重启次域里的 Ceph 对象网关。 ::

       # systemctl restart ceph-radosgw@rgw.`hostname -s`




.. _Migrating a Single Site System to Multi-Site:

从单站迁移到多站配置
====================

要想从只有一个 ``default`` 域组和域的单站系统迁移到多站系统，\
可以按如下步骤实施：

#. 创建一个 realm ，把下面命令中的 ``<name>`` 换成 realm 名字。 ::

       # radosgw-admin realm create --rgw-realm=<name> --default

#. 重命名默认域和域组，把 ``<name>`` 替换成域组和域名字。 ::

       # radosgw-admin zonegroup rename --rgw-zonegroup default --zonegroup-new-name=<name>
       # radosgw-admin zone rename --rgw-zone default --zone-new-name us-east-1 --rgw-zonegroup=<name>

#. 配置主域组。把 ``<name>`` 替换成 realm 或域组的名字；
   ``<fqdn>`` 替换成域组内配置的全资域名。 ::

       # radosgw-admin zonegroup modify --rgw-realm=<name> --rgw-zonegroup=<name> --endpoints http://<fqdn>:80 --master --default

#. 配置主域。把 ``<name>`` 替换成 realm 、域组或域的名字；
   ``<fqdn>`` 替换成域组内配置的全资域名。 ::

       # radosgw-admin zone modify --rgw-realm=<name> --rgw-zonegroup=<name> \
                                   --rgw-zone=<name> --endpoints http://<fqdn>:80 \
                                   --access-key=<access-key> --secret=<secret-key> \
                                   --master --default

#. 创建一个系统用户。把 ``<user-id>`` 替换成用户名；
   ``<display-name>`` 替换成显示名称，它可以包含空格。 ::

       # radosgw-admin user create --uid=<user-id> --display-name="<display-name>"\
                                   --access-key=<access-key> --secret=<secret-key> --system

#. 提交更新过的配置： ::

       # radosgw-admin period update --commit

#. 最后，重启 Ceph 对象网关： ::

       # systemctl restart ceph-radosgw@rgw.`hostname -s`

完成这一步以后，可以继续在主域组中\
`创建和配置副域 <#configure-secondary-zones>`__\ 。




.. _Multi-Site Configuration Reference:

多站配置参考
============

以下是附上细节信息，以及与 realm 、 period 、 zone group 、 zone
相关的命令行用法。




Realms
------

一个 realm 代表一个全局唯一的命名空间，其内可包含一或多个域组、\
域组有可能包含了一或多个域、域包含桶、桶内是对象。 realm 概念\
可以让 Ceph 对象网关在同一套硬件上配置多个命名空间。

realm 暗含了 period 概念，每个 period 表示了域组和域在当时的状\
态。每次更改域组或域后都需要更新 period 并提交它。

考虑到与 Infernalis 以及更早版本的向后兼容问题，默认情况下，
Ceph 对象网关不会创建 realm 。然而，我们建议您最好在新集群上创\
建 realm 。


.. Create a Realm

创建 realm
~~~~~~~~~~

创建 realm 可用 ``realm create`` 命令，并加上 realm 名字。如果\
要创建默认的 realm ，需指定 ``--default`` 参数。 ::

    # radosgw-admin realm create --rgw-realm={realm-name} [--default]

例如： ::

    # radosgw-admin realm create --rgw-realm=movies --default

指定 ``--default`` 以后，每次调用 ``radosgw-admin`` 都会默认指\
向这个 realm ，除非另外指定了 ``--rgw-realm`` 和 realm 名字。


.. Make a Realm the Default

让 realm 成为默认
~~~~~~~~~~~~~~~~~

一堆 realm 里应该有一个默认的，而且只能有一个默认的。如果只有\
一个 realm ，而且创建时没把它设置为默认，也可以稍后设置成默认\
的。或者，要把某个 realm 改成默认的，用命令： ::

    # radosgw-admin realm default --rgw-realm=movies

.. note:: 有默认 realm 后，命令每次运行就会默认加
   ``--rgw-realm=<realm-name>`` 参数。


.. Delete a Realm

删除 realm
~~~~~~~~~~

删除 realm 可用 ``realm delete`` 并加上其名字。 ::

    # radosgw-admin realm delete --rgw-realm={realm-name}

例如： ::

    # radosgw-admin realm delete --rgw-realm=movies


.. Get a Realm

查看 realm
~~~~~~~~~~

查看 realm 可用 ``realm get`` 并加上其名字。 ::

    #radosgw-admin realm get --rgw-realm=<name>

例如： ::

    # radosgw-admin realm get --rgw-realm=movies [> filename.json]

这个命令行会显示一个 JSON 对象，其内是 realm 属性： ::

    {
        "id": "0a68d52e-a19c-4e8e-b012-a8f831cb3ebc",
        "name": "movies",
        "current_period": "b0c5bbef-4337-4edd-8184-5aeab2ec413b",
        "epoch": 1
    }

命令后面再加上 ``>`` 和输出文件名字即可把 JSON 对象写入文件。


.. Set a Realm

配置 realm
~~~~~~~~~~

配置 realm 用 ``realm set`` 并指定其名字、和 ``--infile=`` 与\
输入文件名。 ::

    #radosgw-admin realm set --rgw-realm=<name> --infile=<infilename>

例如： ::

    # radosgw-admin realm set --rgw-realm=movies --infile=filename.json


.. List Realms

罗列 realm
~~~~~~~~~~

罗列 realm 可用 ``realm list`` ： ::

    # radosgw-admin realm list


.. List Realm Periods

罗列 realm 的 period
~~~~~~~~~~~~~~~~~~~~

罗列 realm 的 period 可用 ``realm list-periods`` 。 ::

    # radosgw-admin realm list-periods


.. Pull a Realm

拉取 realm 配置
~~~~~~~~~~~~~~~

要把 realm 配置从包含主域组和主域的节点拉取到包含副域组或副域\
的节点，在接收 realm 配置的节点上执行 ``realm pull`` ： ::

    # radosgw-admin realm pull --url={url-to-master-zone-gateway} --access-key={access-key} --secret={secret}


.. Rename a Realm

重命名 realm
~~~~~~~~~~~~

realm 并非 period 的一部分，所以，对 realm 的重命名只在本地生\
效，不会随 ``realm pull`` 拉过去。重命名一个包含多个域的 realm
时，需要在各个域上分别执行这个命令。命令如下： ::

    # radosgw-admin realm rename --rgw-realm=<current-name> --realm-new-name=<new-realm-name>

.. note:: **不要**\ 用 ``realm set`` 更改 ``name`` 参数，这样\
   只能更改内部名字，指定 ``--rgw-realm`` 时还会用老的 realm \
   名。




.. Zone Groups

域组
----

通过域组概念（也就是 Infernalis 版之前的 region ）， Ceph 对象\
网关可支持多站部署和全局命名空间。域组定义了各个域内一或多个 \
Ceph 对象网关例程的地理位置。

域组的配置与典型的配置过程有所不同，因为不是所有配置都在 Ceph \
配置文件里。你可以罗列域组、查看或更改域组配置。


.. Create a Zone Group

创建域组
~~~~~~~~

创建域组时需指定：一个域组名； ``--rgw-realm=<realm-name>`` ，\
否则就在默认 realm 里创建；加 ``--default`` 参数则创建为默认域\
组；加 ``--master`` 参数则创建为主域组。例如： ::

    # radosgw-admin zonegroup create --rgw-zonegroup=<name> [--rgw-realm=<name>][--master] [--default]

.. note:: 已存在域组的配置可用 \
   ``zonegroup modify --rgw-zonegroup=<zonegroup-name>`` 更改。


.. Make a Zone Group the Default

让域组成为默认
~~~~~~~~~~~~~~

一堆域组应该有一个默认的，且只能有一个默认域组。如果只有一个域\
组，且创建时没指定为默认，可让它成为默认域组。用命令： ::

    # radosgw-admin zonegroup default --rgw-zonegroup=comedy

.. note:: 有默认域组时，每次执行命令会默认为加了
   ``--rgw-zonegroup=<zonegroup-name>`` 参数。

然后，更新 period ： ::

    # radosgw-admin period update --commit


.. Add a Zone to a Zone Group

把域加进域组
~~~~~~~~~~~~

把域加入域组可以用： ::

    # radosgw-admin zonegroup add --rgw-zonegroup=<name> --rgw-zone=<name>

然后，更新 period ： ::

    # radosgw-admin period update --commit


.. Remove a Zone from a Zone Group

删除域组中的域
~~~~~~~~~~~~~~

从域组删除域可以用下列命令： ::

    # radosgw-admin zonegroup remove --rgw-zonegroup=<name> --rgw-zone=<name>

然后，更新 period ： ::

    # radosgw-admin period update --commit


.. Rename a Zone Group

重命名域组
~~~~~~~~~~

重命名一个域组可以用： ::

    # radosgw-admin zonegroup rename --rgw-zonegroup=<name> --zonegroup-new-name=<name>

然后，更新 period ： ::

    # radosgw-admin period update --commit


.. Delete a Zone Group

删除域组
~~~~~~~~

删除域组可以用： ::

    # radosgw-admin zonegroup delete --rgw-zonegroup=<name>

然后，更新 period ： ::

    # radosgw-admin period update --commit


.. List Zone Groups

罗列域组
~~~~~~~~

一个 Ceph 集群可以创建很多域组，用以下命令可以罗列出来： ::

    # radosgw-admin zonegroup list

``radosgw-admin`` 会返回 JSON 格式的域组列表： ::

    {
        "default_info": "90b28698-e7c3-462c-a42d-4aa780d24eda",
        "zonegroups": [
            "us"
        ]
    }


.. Get a Zone Group Map

查看域组映射图
~~~~~~~~~~~~~~

查看各域组的详情可执行： ::

    # radosgw-admin zonegroup-map get

.. note:: 如果你遇到了 ``failed to read zonegroup map`` 错误，\
   首先试一下以 root 身份运行 ``radosgw-admin zonegroup-map update`` 。


.. Get a Zone Group

查看域组
~~~~~~~~

查看域组配置可以用命令： ::

    radosgw-admin zonegroup get [--rgw-zonegroup=<zonegroup>]

域组配置的长相如下：

.. code-block:: json

    {
        "id": "90b28698-e7c3-462c-a42d-4aa780d24eda",
        "name": "us",
        "api_name": "us",
        "is_master": "true",
        "endpoints": [
            "http:\/\/rgw1:80"
        ],
        "hostnames": [],
        "hostnames_s3website": [],
        "master_zone": "9248cab2-afe7-43d8-a661-a40bf316665e",
        "zones": [
            {
                "id": "9248cab2-afe7-43d8-a661-a40bf316665e",
                "name": "us-east",
                "endpoints": [
                    "http:\/\/rgw1"
                ],
                "log_meta": "true",
                "log_data": "true",
                "bucket_index_max_shards": 0,
                "read_only": "false"
            },
            {
                "id": "d1024e59-7d28-49d1-8222-af101965a939",
                "name": "us-west",
                "endpoints": [
                    "http:\/\/rgw2:80"
                ],
                "log_meta": "false",
                "log_data": "true",
                "bucket_index_max_shards": 0,
                "read_only": "false"
            }
        ],
        "placement_targets": [
            {
                "name": "default-placement",
                "tags": []
            }
        ],
        "default_placement": "default-placement",
        "realm_id": "ae031368-8715-4e27-9a99-0c9468852cfe"
    }


.. Set a Zone Group

配置域组
~~~~~~~~

定义域组需创建一个 JSON 对象，至少得指定必需选项：

#. ``name``: 域组的名字，必需。

#. ``api_name``: 域组的 API 名字，可选。

#. ``is_master``: 决定此域组是否为主域组，必需。\ **注意：**\
   一套系统只能有一个主域组。

#. ``endpoints``: 此域组可服务的终结点列表，例如，你可以让多个\
   域名指向同一域组。记得转义正斜线（ ``\/`` ）。每个终结点都\
   可以分别指定端口（ ``fqdn:port`` ）。可选参数。

#. ``hostnames``: 域组内所有主机名的列表，例如，你可以让多个域\
   名指向同一域组。可选参数。 ``rgw dns name`` 选项会自动包含\
   在这个列表内，更改此选项后需重启网关进程。

#. ``master_zone``: 域组的主域，不指定则为默认域，可选参数。\
   **注意：**\ 每个域组只能有一个主域。

#. ``zones``: 域组内所有域的列表，每个域需包含其名字（必需）、\
   终结点列表（可选）、以及网关是否需记录元数据和数据操作（默\
   认为否）。

#. ``placement_targets``: 归置靶列表（可选），每个归置靶需包含\
   其名字（必需）、和一个标签列表（可选），只有打了这些标签的\
   用户才可以使用这个归置靶（即用户信息里的 ``placement_tags``
   字段）。

#. ``default_placement``: 对象索引和对象数据的默认归置靶，默认\
   为 ``default-placement`` 。你也可以为每个用户分别设置它们自\
   己的默认归置靶，设置在用户信息里。

要配置域组，需创建一个包含必需字段的 JSON 对象，并存入文件（例\
如 ``zonegroup.json`` ），然后执行下列命令： ::

    # radosgw-admin zonegroup set --infile zonegroup.json

其中 ``zonegroup.json`` 是刚刚创建的 JSON 文件。

.. important:: 名为 ``default`` 的域组其 ``is_master`` 选项的\
   值默认是 ``true`` 。如果你要新建域组并让它成为主域组，必须\
   把域组 ``default`` 的 ``is_master`` 选项设置为 ``false`` ，\
   或者删除域组 ``default`` 。

最后，更新 period::

    # radosgw-admin period update --commit


.. Set a Zone Group Map

配置域组映射图
~~~~~~~~~~~~~~

要配置域组映射图，需创建一个包含一或多个域组的 JSON 对象，并设\
置集群的 ``master_zonegroup`` 。域组映射图里的每个域组都包含一\
个键值对，其中 ``key`` 选项相当于单个域组配置里的 ``name`` 选\
项， ``val`` 是包含着整个域组配置的 JSON 对象。

你只能有一个 ``is_master`` 为 ``true`` 的域组，而且它必须是域\
组映射图尾部 ``master_zonegroup`` 选项的值。下面是默认域组映射\
图的一个实例：

.. code-block:: json

    {
        "zonegroups": [
            {
                "key": "90b28698-e7c3-462c-a42d-4aa780d24eda",
                "val": {
                    "id": "90b28698-e7c3-462c-a42d-4aa780d24eda",
                    "name": "us",
                    "api_name": "us",
                    "is_master": "true",
                    "endpoints": [
                        "http:\/\/rgw1:80"
                    ],
                    "hostnames": [],
                    "hostnames_s3website": [],
                    "master_zone": "9248cab2-afe7-43d8-a661-a40bf316665e",
                    "zones": [
                        {
                            "id": "9248cab2-afe7-43d8-a661-a40bf316665e",
                            "name": "us-east",
                            "endpoints": [
                                "http:\/\/rgw1"
                            ],
                            "log_meta": "true",
                            "log_data": "true",
                            "bucket_index_max_shards": 0,
                            "read_only": "false"
                        },
                        {
                            "id": "d1024e59-7d28-49d1-8222-af101965a939",
                            "name": "us-west",
                            "endpoints": [
                                "http:\/\/rgw2:80"
                            ],
                            "log_meta": "false",
                            "log_data": "true",
                            "bucket_index_max_shards": 0,
                            "read_only": "false"
                        }
                    ],
                    "placement_targets": [
                        {
                            "name": "default-placement",
                            "tags": []
                        }
                    ],
                    "default_placement": "default-placement",
                    "realm_id": "ae031368-8715-4e27-9a99-0c9468852cfe"
                }
            }
        ],
        "master_zonegroup": "90b28698-e7c3-462c-a42d-4aa780d24eda",
        "bucket_quota": {
            "enabled": false,
            "max_size_kb": -1,
            "max_objects": -1
        },
        "user_quota": {
            "enabled": false,
            "max_size_kb": -1,
            "max_objects": -1
        }
    }

更改域组映射图的命令如下： ::

    # radosgw-admin zonegroup-map set --infile zonegroupmap.json

其中 ``zonegroupmap.json`` 是你创建的 JSON 文件，需确保域组映\
射图里的域都已创建。最后，更新 period::

    # radosgw-admin period update --commit




.. Zones

域
--

Ceph 对象网关支持域概念，域是一或多个 Ceph 对象网关例程的逻辑\
分组。

域的配置不同于典型配置过程，因为有些配置不在 Ceph 配置文件里。\
你可以罗列域、查看或修改域配置。


.. Create a Zone

创建域
~~~~~~

创建域时，需指定其名字。如果创建的是主域，得加上 ``--master`` \
选项，一个域组只能有一个主域；若要把域加入域组，需加上
``--rgw-zonegroup`` 选项和域组名字。 ::

    # radosgw-admin zone create --rgw-zone=<name> \
                    [--zonegroup=<zonegroup-name]\
                    [--endpoints=<endpoint>[,<endpoint>] \
                    [--master] [--default] \
                    --access-key $SYSTEM_ACCESS_KEY --secret $SYSTEM_SECRET_KEY

然后，更新 period::

    # radosgw-admin period update --commit


.. Delete a Zone

删除域
~~~~~~

删除域前，要先从域组删掉： ::

    # radosgw-admin zonegroup remove --zonegroup=<name>\
                                     --zone=<name>

然后，更新 period::

    # radosgw-admin period update --commit

接下来删除域，用此命令： ::

    # radosgw-admin zone delete --rgw-zone<name>

最后，更新 period::

    # radosgw-admin period update --commit

.. important:: 从域组删掉域之前先不要删除这个域，否则更新 period
   时会失败。

域被删除后，如果其它地方也不需要与之相关的存储池，可以考虑删除\
掉，把下面实例中的 ``<del-zone>`` 替换成已删除域的名字即可。

.. important:: 只能删除以域名打头的存储池。若删除根存储池（如
   ``.rgw.root`` ），会删除整个系统的配置。

.. important:: 一旦删除存储池，其内的数据也会被删除，且不可恢\
   复。所以，确定存储池内容不需要了再删除。

::

    # rados rmpool <del-zone>.rgw.control <del-zone>.rgw.control --yes-i-really-really-mean-it
    # rados rmpool <del-zone>.rgw.data.root <del-zone>.rgw.data.root --yes-i-really-really-mean-it
    # rados rmpool <del-zone>.rgw.gc <del-zone>.rgw.gc --yes-i-really-really-mean-it
    # rados rmpool <del-zone>.rgw.log <del-zone>.rgw.log --yes-i-really-really-mean-it
    # rados rmpool <del-zone>.rgw.users.uid <del-zone>.rgw.users.uid --yes-i-really-really-mean-it


.. Modify a Zone

修改域配置
~~~~~~~~~~

修改域配置需指定域名、以及你想更改的参数。 ::

    # radosgw-admin zone modify [options]

其中 ``[options]`` 可以是： 

- ``--access-key=<key>``
- ``--secret/--secret-key=<key>``
- ``--master``
- ``--default``
- ``--endpoints=<list>``

然后，更新 period::

    # radosgw-admin period update --commit


.. List Zones

罗列域
~~~~~~

以 ``root`` 身份罗列集群中的域： ::

    # radosgw-admin zone list


.. Get a Zone

查看域
~~~~~~

以 ``root`` 身份查看某个域的配置： ::

    # radosgw-admin zone get [--rgw-zone=<zone>]

``default`` 这个域的配置长相如下：

.. code-block:: json

    { "domain_root": ".rgw",
      "control_pool": ".rgw.control",
      "gc_pool": ".rgw.gc",
      "log_pool": ".log",
      "intent_log_pool": ".intent-log",
      "usage_log_pool": ".usage",
      "user_keys_pool": ".users",
      "user_email_pool": ".users.email",
      "user_swift_pool": ".users.swift",
      "user_uid_pool": ".users.uid",
      "system_key": { "access_key": "", "secret_key": ""},
      "placement_pools": [
          {  "key": "default-placement",
             "val": { "index_pool": ".rgw.buckets.index",
                      "data_pool": ".rgw.buckets"}
          }
        ]
      }


.. Set a Zone

配置域
~~~~~~

配置域时需指定一系列 Ceph 对象网关例程的存储池，我们建议用域的\
名字作为存储池前缀，存储池配置细节见\ `存储池 \
<http://docs.ceph.com/docs/master/rados/operations/pools/#pools>`__\ 。

要配置域，需创建一个包含存储池的 JSON 对象，并存入一个文件（如
``zone.json`` ），然后执行下列命令（把 ``{zone-name}`` 替换为\
域的名字）： ::

    # radosgw-admin zone set --rgw-zone={zone-name} --infile zone.json

其中 ``zone.json`` 是你创建的 JSON 文件。

然后，以 ``root`` 用户身份更新 period::

    # radosgw-admin period update --commit


.. Rename a Zone

重命名域
~~~~~~~~

要重命名域，需指定域的名字和新的域名。\ ::

    # radosgw-admin zone rename --rgw-zone=<name> --zone-new-name=<name>

然后，更新 period::

    # radosgw-admin period update --commit




.. Zone Group and Zone Settings

域组和域选项
------------

配置默认的域组和域时，存储池名字里包含域的名字，例如：

-  ``default.rgw.control``

要更改默认值，把下列选项写入 Ceph 配置文件里 \
``[client.radosgw.{instance-name}]`` 例程配置段下面。

+-------------------------------------+------------------------------+---------+-----------------------+
| 名字                                | 描述                         | 类型    | 默认值                |
+=====================================+==============================+=========+=======================+
| ``rgw_zone``                        | 配置在网关例程上的域的名字。 | String  | None                  |
+-------------------------------------+------------------------------+---------+-----------------------+
| ``rgw_zonegroup``                   | 配置在网关例程上的域组名。   | String  | None                  |
+-------------------------------------+------------------------------+---------+-----------------------+
| ``rgw_zonegroup_root_pool``         | 域组的根存储池。             | String  | ``.rgw.root``         |
+-------------------------------------+------------------------------+---------+-----------------------+
| ``rgw_zone_root_pool``              | 域的根存储池。               | String  | ``.rgw.root``         |
+-------------------------------------+------------------------------+---------+-----------------------+
| ``rgw_default_zone_group_info_oid`` | 用于存储默认域组的 OID 。    | String  | ``default.zonegroup`` |
|                                     | 我们不建更改此选项。         |         |                       |
+-------------------------------------+------------------------------+---------+-----------------------+
| ``rgw_num_zone_opstate_shards``     | 用于暂存域组之间同步进度的   | Integer | ``128``               |
|                                     | 最大片段数。                 |         |                       |
+-------------------------------------+------------------------------+---------+-----------------------+
