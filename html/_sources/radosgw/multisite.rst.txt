.. _multisite:

======
 多站
======
.. Multi-Site

单域配置和多站配置
==================
.. Single-zone Configurations and Multi-site Configurations

单域配置
--------
.. Single-zone Configurations

单个域的配置通常包括两个东西：

#. 一个域组（ zonegroup ），其内有一个域。
#. 一或多个 `ceph-radosgw` 例程，可以对客户端请求做负载均衡。

在典型的单域配置中，会有多个
`ceph-radosgw` 例程使用同一个 Ceph 存储集群。

多站配置的种类
--------------
.. Varieties of Multi-site Configuration

.. versionadded:: Jewel

从 Kraken 版起， Ceph 逐步能够支持好几种对象网关的多站配置方案：

- **多域（ Multi-zone ）：** “多域”配置具有复杂的拓扑结构。
  一套多域配置由一个域组和多个域组成。每个域由一或多个 `ceph-radosgw` 例程组成，
  **每个域都由它自己的 Ceph 存储集群支撑。**

  一个域组内存在多个域是为了让这个域组具备灾难恢复能力，
  在其中一个域出现重大故障时有能力恢复。
  每个域都处于活跃状态，可处理写操作。
  一套包含多个活跃域的多域配置可增强灾难恢复能力，
  还可以用作内容分发网络的底层。

- **多个域组（ Multi-zonegroups ）：** Ceph 对象网关也支持多个域组，
  以前称为“region”。每个域组内可以有一或多个域。
  如果两个域处于同一域组，而且这个域组与第二个域组处于同一个 realm ，
  那么这两个域中存储的对象共享一个全局对象命名空间。
  这个全局对象名称空间可确保\
  跨域组和跨域的对象 ID 具有唯一性。

  每个桶都归它创建时所在的域组所有
  （除非在创建桶时被 :ref:`LocationConstraint<s3_bucket_placement>` 改写），
  其对象数据只能复制到该域组中的其他域。
  所有发送到其他域组的、目标是该桶中数据的请求，
  都会重定向到这个桶所在的域组。

  当你想跨越多个域组共享用户和桶的命名空间，
  并把对象数据隔离在少数几个域时，
  创建多个域组就很有用。假设你有几个连接在一起的站点，
  它们共享存储空间，但出于灾难恢复的目的，只需要一个备份。
  在这种情况下，可以创建几个域组，每个域组只有两个域，
  以免把所有对象都复制到所有区域。

  在其他情况下，你可能会将数据隔离在不同的 realm 中，
  每个 realm 只有一个域组。通过分别控制数据和元数据的隔离，
  域组实现了灵活性。

- **多个 realm ：** 从 Kraken 版起， Ceph 对象网关支持
  realm 概念，它是域组的容器。
  有了 realm ，就可以实现为多个域组设置统一的策略。
  realm 有个全局唯一的命名空间，可以包含单个域组或多个域组。
  如果你决定使用多个 realm ，
  就可以定义多个命名空间和多套配置
  （这意味着每个 realm 都可以有不同于其他 realm 的一套配置）。


示意图 - 域之间对象数据的复制
-----------------------------
.. Diagram - Replication of Object Data Between Zones

同一域组内、两个域之间的数据复制\
情形如下：

.. image:: ../images/zone-sync.svg
   :align: center

在本图的顶部，我们可以看到两个应用程序（也称为“客户端”）。
右边的应用程序通过 RADOS 网关 (RGW) 从 Ceph 集群写入和读出数据。
左边的应用程序只通过 RADOS 网关 (RGW) 实例从 Ceph 集群读出数据。
在这两种情况下（读写和只读），
数据传输都是按照 REST 规范处理的。

在图表的中间，有两个域，每个域都包含一个
RADOS 网关 (RGW) 实例。这些 RGW 实例负责处理\
从应用程序到域组的数据传输。
从主域（ US-EAST ）到辅域（ US-WEST ）的箭头表示\
数据同步行为。

在本图的底部，我们看到，数据分发进了 Ceph 存储集群。

网关集群的配置细节请参考\ `生产级的 Ceph 对象网关
<https://access.redhat.com/documentation/en-us/red_hat_ceph_storage/3/html/ceph_object_gateway_for_production/index/>`_ 。


从 Infernalis 以来功能上的变化
==============================
.. Functional Changes from Infernalis

从 Kraken 版起，各个 Ceph 对象网关可以配置成处于多活域模式\
（ active-active zone mode ）下，这样就可以写入非主域。

多站配置信息存储在名为 realm 的容器内，\
realm 内存储着域组、域、和带有多个时间结的 peroid ，\
时间结用于跟踪配置变更。

从 Kraken 版起，由 ``ceph-radosgw`` 守护进程处理数据的跨域同步，\
无需再单独配置同步代理。
新同步方法允许 Ceph 对象网关以多活（ active-active ）模式运行，\
不再是主从（ active-passive ）模式。


环境需求和假设
==============
.. Requirements and Assumptions

一套多站配置需要至少两个 Ceph 存储集群，
至少两个 Ceph 对象网关例程，分别连接两个 Ceph 存储集群。

本指南中，我们假设至少有两个地理位置不同的 Ceph 存储集群，
其实都在本地也能跑通；
还假设有两个 Ceph 对象网关服务器，
分别名为 ``rgw1`` 和 ``rgw2`` 。

.. important:: **不建议**\ 运营单个跨地理区域的 Ceph 存储集群，\
   除非你的 WAN 连接延时够低。

一套多站配置必须有一个主域组和一个主域，
各域组都得有自己的主域；
域组可以有一或多个副域组、副域。

在本指南中， ``rgw1`` 主机将在主域组的主域中提供服务；
而 ``rgw2`` 主机将在主域组的副域中提供服务。

为 Ceph 对象存储服务创建和调整存储池的文档请参考\ `存储池`_\ 。

`同步策略配置`_ 里会指引你如何定义粒度合适的桶同步策略的规则。


.. _master-zone-label:

配置主域
========
.. Configuring a Master Zone

在多站配置中，所有网关都会向主域组中、主域内的
``ceph-radosgw`` 守护进程索取配置。
所以在多站配置中，要配置各网关，需要挑一个 ``ceph-radosgw`` 例程，
在它上面配置主域组和主域。


创建 realm
----------
.. Create a Realm

realm 包含域组和域的多站配置信息，同时在 realm 内\
强制施行全局唯一的命名空间。

#. 在为主域组和域提供服务的主机上，打开命令行，
   执行下列命令创建多站配置所需的新 realm:

   .. prompt:: bash #

      radosgw-admin realm create --rgw-realm={realm-name} [--default]

   例如：

   .. prompt:: bash #

      radosgw-admin realm create --rgw-realm=movies --default

   .. note:: 如果此集群上只会配置一个 realm ，可加上 ``--default`` 标记。

      指定 ``--default`` 以后， ``radosgw-admin`` 就会默认使用这个 realm ；

      如果没指定 ``--default`` ，新增域组和域时就必须指定
      ``--rgw-realm`` 或 ``--realm-id`` 标记，以确定在哪个 realm 新建。

#. 创建 realm 后， ``radosgw-admin`` 会顺便显示此 realm 的配置信息，例如：

   ::

       {
           "id": "0956b174-fe14-4f97-8b50-bb7ec5e1cf62",
           "name": "movies",
           "current_period": "1950b710-3e63-4c41-a19e-46a715000980",
           "epoch": 1
       }

   .. note:: Ceph 会给这个 realm 生成唯一的 ID ，这样，需要的话可以随时改名。


创建主域组
----------
.. Create a Master Zone Group

一个 realm 必须至少有一个域组，只有一个的话它将作为此 realm 的主域组。

#. 在为主域组和域提供服务的主机上，打开命令行，
   执行下列命令新建多站配置所需的主域组：

   .. prompt:: bash #

      radosgw-admin zonegroup create --rgw-zonegroup={name} --endpoints={url} [--rgw-realm={realm-name}|--realm-id={realm-id}] --master --default

   例如：

   .. prompt:: bash #

      radosgw-admin zonegroup create --rgw-zonegroup=us --endpoints=http://rgw1:80 --rgw-realm=movies --master --default

   .. note:: 如果这个 realm 只有一个域组，创建时可加上 ``--default`` 标记。

      指定 ``--default`` 后， ``radosgw-admin`` 在创建新域时将默认使用此域组。

      如果未指定 ``--default`` 标记，新增或修改域时就得指定
      ``--rgw-zonegroup`` 或 ``--zonegroup-id`` 标记，以确定在哪个域组内操作。

#. 创建主域组后， ``radosgw-admin`` 会显示这个域组的配置，例如：

   ::

       {
           "id": "f1a233f5-c354-4107-b36c-df66126475a6",
           "name": "us",
           "api_name": "us",
           "is_master": "true",
           "endpoints": [
               "http:\/\/rgw1:80"
           ],
           "hostnames": [],
           "hostnames_s3website": [],
           "master_zone": "",
           "zones": [],
           "placement_targets": [],
           "default_placement": "",
           "realm_id": "0956b174-fe14-4f97-8b50-bb7ec5e1cf62"
       }


创建主域
--------
.. Create a Master Zone

.. important:: 域必须在此域内的 Ceph 对象网关所在节点上创建。

在为主域组和域提供服务的主机上，
打开命令行，执行下列命令新建\
多站配置所需的主域：

.. prompt:: bash #

   # radosgw-admin zone create --rgw-zonegroup={zone-group-name} \
                                --rgw-zone={zone-name} \
                                --master --default \
                                --endpoints={http://fqdn}[,{http://fqdn}]

例如：

.. prompt:: bash #

   # radosgw-admin zone create --rgw-zonegroup=us --rgw-zone=us-east \
                                --master --default \
                                --endpoints={http://fqdn}[,{http://fqdn}]


.. note:: 这里没有指定 ``--access-key`` 和 ``--secret`` ，
   在后面的章节中创建用户后，再把这些配置写入域。

.. important:: 后续步骤假设是在新安装好的系统上实施多站配置，
   上面还没有数据。如果你已经用它存储了一些数据，
   **不要删除** ``default`` 域及其存储池，
   否则数据会被删除且不可恢复。


删除默认域组和域
----------------
.. Delete Default Zonegroup and Zone

#. 如果有 ``default`` 域，要先从域组里删除，然后再删掉它。

   .. prompt:: bash #

      radosgw-admin zonegroup delete --rgw-zonegroup=default --rgw-zone=default
      radosgw-admin period update --commit
      radosgw-admin zone delete --rgw-zone=default
      radosgw-admin period update --commit
      radosgw-admin zonegroup delete --rgw-zonegroup=default
      radosgw-admin period update --commit

#. 如果这个 Ceph 存储集群里还有 ``default`` 存储池，也需一并删除。

   .. important:: 后续步骤假设是在新安装好的系统上实施多站配置，
      上面还没有数据。如果你已经用它存储了一些数据，
      **不要删除** ``default`` 域及其存储池，否则数据会被删除且不可恢复。

   .. prompt:: bash #

      ceph osd pool rm default.rgw.control default.rgw.control --yes-i-really-really-mean-it
      ceph osd pool rm default.rgw.data.root default.rgw.data.root --yes-i-really-really-mean-it
      ceph osd pool rm default.rgw.gc default.rgw.gc --yes-i-really-really-mean-it
      ceph osd pool rm default.rgw.log default.rgw.log --yes-i-really-really-mean-it
      ceph osd pool rm default.rgw.users.uid default.rgw.users.uid --yes-i-really-really-mean-it


创建系统用户
------------
.. Create a System User

#. ``ceph-radosgw`` 守护进程在拉取 realm 和 peroid 信息前必须先通过认证。
   在主域里，创建一个系统用户，用于守护进程之间的认证。

   .. prompt:: bash #

      radosgw-admin user create --uid="{user-name}" --display-name="{Display Name}" --system

   例如：

   .. prompt:: bash #

      radosgw-admin user create --uid="synchronization-user" --display-name="Synchronization User" --system

#. 记下 ``access_key`` 和 ``secret_key`` 的内容，因为副域需要用它们与主域认证。

#. 把系统用户加入主域。

   .. prompt:: bash #

      radosgw-admin zone modify --rgw-zone={zone-name} --access-key={access-key} --secret={secret}
      radosgw-admin period update --commit


更新 period
-----------
.. Update the Period

更新主域配置信息后，再更新 peroid ：

.. prompt:: bash #

   radosgw-admin period update --commit

.. note:: 更新 period 会更改 epoch ，还需确保其它的域会收到更新过的配置信息。


更新 Ceph 配置文件
------------------
.. Update the Ceph Configuration File

更新主域所在主机上的 Ceph 配置文件，把 ``rgw_zone`` 配置选项和\
主域的名字写在例程配置段下。 ::

    [client.rgw.{instance-name}]
    ...
    rgw_zone={zone-name}

例如： ::

    [client.rgw.rgw1]
    host = rgw1
    rgw frontends = "civetweb port=80"
    rgw_zone=us-east


启动网关
--------
.. Start the Gateway

在对象网关所在的主机上，启动 Ceph 对象网关、并启用服务：

.. prompt:: bash #

   systemctl start ceph-radosgw@rgw.`hostname -s`
   systemctl enable ceph-radosgw@rgw.`hostname -s`


.. _secondary-zone-label:

配置副域
========
.. Configure Secondary Zones

一个域组内的域们会复制所有数据，以确保各个域都有相同的数据。
创建副域需在作为副域的主机上执行下面的所有操作。

.. note:: 增加第二个副域（就是在一个域组内，已经有一个副域了，又建第二个副域）和
   :ref:`增加副域的过程相同 <radosgw-multisite-secondary-zone-creating>`\ 。
   必须用不同的域名称，不能和第一个副域一样。

.. important:: 必须在主域内的主机上执行元数据操作，
   如用户创建。主域和副域都可以接受桶操作，
   但是副域会把桶操作重定向到主域；
   如果主域倒下了，桶操作会失败。


拉取 realm
----------
.. Pulling the Realm Configuration

用主域组中主域的 URL 、访问密钥和私钥可以把 realm 拉到本主机。\
如果要拉取的不是默认 realm ，还需用 ``--rgw-realm`` 或
``--realm-id`` 选项指定 realm 。

.. prompt:: bash #

   radosgw-admin realm pull --url={url-to-master-zone-gateway}
   --access-key={access-key} --secret={secret}

.. note:: 拉取 realm 时也会检出远端的当前 period 、
   并使之成为本机的当前 period 。

如果这个 realm 是唯一的一个，可以让它成为默认 realm ，执行下列命令：

.. prompt:: bash #

   radosgw-admin realm default --rgw-realm={realm-name}


.. _radosgw-multisite-secondary-zone-creating:

创建副域
--------
.. Creating a Secondary Zone

.. important:: 域必须在此域内的 Ceph 对象网关所在节点上创建。

在副域内提供服务的主机上，打开命令行新建多站配置所需的副域，
需指定域组 ID 、新的域名和这个域内配置的终结点；
**不要加** ``--master`` 或 ``--default`` 标记。
在 Kraken 里，所有域都按多活配置运行，
也就是说，网关客户端可写入任意一个域，
这个域会把数据复制到同一域组内、除此之外的其它域上。
如果不想让副域处理写操作，创建时可以加 ``--read-only`` 标记，
这样主域和副域就会按主从方式配置。另外，
还需提供系统用户的 ``access_key`` 和 ``secret_key`` ，
它存储在主域组的主域内。
命令如下：

.. prompt:: bash #

   radosgw-admin zone create --rgw-zonegroup={zone-group-name} \
                                --rgw-zone={zone-name} \
                                --access-key={system-key} --secret={secret} \
                                --endpoints=http://{fqdn}:80 \
                                [--read-only]

例如：

.. prompt:: bash #

   radosgw-admin zone create --rgw-zonegroup=us --rgw-zone=us-west \
                                --access-key={system-key} --secret={secret} \
                                --endpoints=http://rgw2:80

.. important:: 后续步骤假设是在新安装好的系统上实施多站配置，
   上面还没有数据。
   如果你已经用它存储了一些数据，\ **不要删除** ``default`` **域\
   及其存储池**\ ，否则数据会被删除且不可恢复。

如有必要，删除默认域：

.. prompt:: bash #

   radosgw-admin zone delete --rgw-zone=default

最后，如果有必要，删除 Ceph 存储集群内的默认存储池。

.. prompt:: bash #

   ceph osd pool rm default.rgw.control default.rgw.control --yes-i-really-really-mean-it
   ceph osd pool rm default.rgw.data.root default.rgw.data.root --yes-i-really-really-mean-it
   ceph osd pool rm default.rgw.gc default.rgw.gc --yes-i-really-really-mean-it
   ceph osd pool rm default.rgw.log default.rgw.log --yes-i-really-really-mean-it
   ceph osd pool rm default.rgw.users.uid default.rgw.users.uid --yes-i-really-really-mean-it


更新 Ceph 配置文件
------------------
.. Updating the Ceph Configuration File

更新副域所在主机上的 Ceph 配置文件，把 ``rgw_zone`` 配置选项和\
副域的名字写在例程配置段下：

::

    [client.rgw.{instance-name}]
    ...
    rgw_zone={zone-name}

例如：

::

    [client.rgw.rgw2]
    host = rgw2
    rgw frontends = "civetweb port=80"
    rgw_zone=us-west

更新 period
-----------
.. Updating the Period

更新完主域配置信息后，更新 period 。

.. prompt:: bash #

   radosgw-admin period update --commit

.. note:: 更新 period 会更改 epoch ，还需确保其它的域会收到更\
   新过的配置信息。

启动网关
--------
.. Starting the Gateway

在对象网关所在的主机上，启动 Ceph 对象网关、并启用服务：

.. prompt:: bash #

   systemctl start ceph-radosgw@rgw.`hostname -s`
   systemctl enable ceph-radosgw@rgw.`hostname -s`

如果使用 ``cephadm`` 命令部署集群，则无法使用 ``systemctl`` 启动网关，
因为不存在 ``systemctl`` 可以运行的服务。这是由于 ``cephadm`` 部署的
Ceph 集群是容器化的。如果用了 ``cephadm`` 命令并拥有一个容器化集群，
可以运行以下命令来启动网关：

.. prompt:: bash #

   ceph orch apply rgw <name> --realm=<realm> --zone=<zone> --placement --port


检查同步状态
------------
.. Checking Synchronization Status

副域起来并正常运行后，检查一下同步状态。
同步过程就是把主域中创建的用户和桶都复制到副域。

.. prompt:: bash #

   radosgw-admin sync status

此命令的输出会显示同步操作的状态，例如：

::

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

.. note:: 副域可以接受桶操作，
   然而它们会把桶操作重定向到主域，
   然后再与主域同步，获取桶操作的结果。
   如果主域倒下了，副域上的桶操作会失败，
   但是对象操作仍会成功。


对象的校验
----------
.. Verifying an Object

默认情况下，一个对象成功同步后不会再次校验。
要启用此功能，你可以把 :confval:`rgw_sync_obj_etag_verify` 设置为 ``true`` 。
启用这个可选功能后，对象还会继续同步，
在同步源和目的地都会额外计算对象的 MD5 校验和并核对。
在经过 HTTP 远程传输后，包括多站同步，这样做可以保证对象的整体性。
这个选项会降低 RGW 的性能，因为需要的计算量更大。


维护
====
.. Maintenance

检查同步状态
------------
.. Checking the Sync Status

某个域的复制状态可以这样查询：

.. prompt:: bash $

   radosgw-admin sync status

::

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

输出结果不同由同步状态决定。同步时，分片有两种类型：

- **Behind shards （落伍分片）** 是那些需要完整的数据同步、
  或者需要增量数据同步的分片们，因为它们不是最新版本。

- **Recovery shards （恢复分片）** 是那些在同步时遇到错误、
  并被标记为需要重试的分片们。这种错误大多是小问题，像获取桶的锁。
  一般它都能自己解决。


检查日志
--------
.. Check the logs

仅限于多站配置而言，你可以检查元数据日志（ ``mdlog`` ）、
桶索引日志（ ``bilog`` ）、和数据日志（ ``mdlog`` ）。
你可以罗列出它们、还能清理它们。大多数情况下都不需要清理这些日志，
因为 :confval:`rgw_sync_log_trim_interval` 默认设置成了 20 分钟。
如果把 :confval:`rgw_sync_log_trim_interval` 手动设置成 0 ，就需要清理日志。


更改元数据主域
--------------
.. Changing the Metadata Master Zone

.. important:: 要把一个域提升成主的，从而更改元数据主域时要格外小心。
   如果一个域还没与当前的主域同步完元数据，那么它晋级成为主域后，
   不能为尚未同步完的条目提供服务，而且这些变更将丢失。
   有鉴于此，我们建议先等这个域的元数据同步状态
   ``radosgw-admin sync status`` 完成，然后再把它晋级为主域。

类似地，如果当前的主域正在处理元数据变更，
此时另一个域却被晋级成了主域，那么这些变更很可能会丢失。
为避免出现此类情况，建议关闭先前主域内的所有 ``radosgw`` 例程；
等晋级完另一个域之后，可以用 ``radosgw-admin period pull``
拉取先前主域的新 period ，然后再启动先前停掉的网关。

要想把一个域（例如 ``us`` 域组内的 ``us-2`` 域）晋级为元数据主域，
在这个域上做如下操作：

.. prompt:: bash $

   radosgw-admin zone modify --rgw-zone=us-2 --master
   radosgw-admin zonegroup modify --rgw-zonegroup=us --master
   radosgw-admin period update --commit

这样就会生成一个新 period ，而且 ``us-2`` 域内的 radosgw 例程\
会把这个 period 发给其它域。


故障切换和灾难恢复
==================
.. Failover and Disaster Recovery

故障切换到副域的配置
--------------------
.. Setting Up Failover to the Secondary Zone

如果主域失败，则切换到副域以作灾难恢复，按照下列步骤：

#. 让副域成为默认的主域，例如：

   .. prompt:: bash #

      radosgw-admin zone modify --rgw-zone={zone-name} --master --default

   默认情况下， Ceph 对象网关运行在多活模式下。
   如果集群被配置成了主从模式，
   那么副域是个只读域，
   需要去除 ``--read-only`` 状态，
   以允许这个域处理写操作。例如：

   .. prompt:: bash #

      radosgw-admin zone modify --rgw-zone={zone-name} --master --default \
                                   --read-only=false

#. 更新 period 以使变更生效。

   .. prompt:: bash #

      radosgw-admin period update --commit

#. 最后，重启 Ceph 对象网关。

   .. prompt:: bash #

      systemctl restart ceph-radosgw@rgw.`hostname -s`

从故障切换回退
--------------
.. Reverting from Failover

如果前任主域恢复了，可以回退故障切换操作，按照下列步骤：

#. 在已恢复的域里，从当前的主域拉取最新的 realm 配置：

   .. prompt:: bash #

      radosgw-admin realm pull --url={url-to-master-zone-gateway} \
                                  --access-key={access-key} --secret={secret}

#. 让恢复的域成为默认的主域：

   .. prompt:: bash #

      radosgw-admin zone modify --rgw-zone={zone-name} --master --default

#. 更新 period 以使变更生效：

   .. prompt:: bash #

      radosgw-admin period update --commit

#. 然后，在恢复好的域里重启 Ceph 对象网关。

   .. prompt:: bash #

       systemctl restart ceph-radosgw@rgw.`hostname -s`

#. 如果副域还要恢复为只读配置，
   更新一下副域。

   .. prompt:: bash #

      radosgw-admin zone modify --rgw-zone={zone-name} --read-only

#. 更新 period 以使变更生效。

   .. prompt:: bash #

      radosgw-admin period update --commit

#. 最后，重启次域里的 Ceph 对象网关。

   .. prompt:: bash #

      systemctl restart ceph-radosgw@rgw.`hostname -s`


.. _rgw-multisite-migrate-from-single-site:

从单站迁移到多站配置
====================
.. Migrating a Single-Site Deployment to Multi-Site

要想从只有一个 ``default`` 域组和域的单站系统迁移到多站系统，
可以按如下步骤实施：

#. 创建一个 realm ，把下面命令中的 ``<name>`` 换成 realm 名字。

   .. prompt:: bash #

      radosgw-admin realm create --rgw-realm=<name> --default

#. 重命名默认域和域组，把 ``<name>`` 替换成域组和域名字。

   .. prompt:: bash #

      radosgw-admin zonegroup rename --rgw-zonegroup default --zonegroup-new-name=<name>
      radosgw-admin zone rename --rgw-zone default --zone-new-name us-east-1 --rgw-zonegroup=<name>

#. 重命名默认域组的 ``api_name`` ，把 ``<name>`` 换成域组名：

   .. prompt:: bash #

      radosgw-admin zonegroup modify --api-name=<name> --rgw-zonegroup=<name>

#. 配置主域组。把 ``<name>`` 替换成 realm 或域组的名字；
   ``<fqdn>`` 替换成域组内配置的全资域名。

   .. prompt:: bash #

      radosgw-admin zonegroup modify --rgw-realm=<name> --rgw-zonegroup=<name> --endpoints http://<fqdn>:80 --master --default

#. 配置主域。把 ``<name>`` 替换成
   realm 、域组或域的名字；
   ``<fqdn>`` 替换成域组内配置的全资域名。

   .. prompt:: bash #

      radosgw-admin zone modify --rgw-realm=<name> --rgw-zonegroup=<name> \
                                --rgw-zone=<name> --endpoints http://<fqdn>:80 \
                                --access-key=<access-key> --secret=<secret-key> \
                                --master --default

#. 创建一个系统用户。把 ``<user-id>`` 替换成用户名；
   ``<display-name>`` 替换成显示名称，
   它可以包含空格。

   .. prompt:: bash #

      radosgw-admin user create --uid=<user-id> \
      --display-name="<display-name>" \ 
      --access-key=<access-key> \ 
      --secret=<secret-key> --system

#. 提交更新过的配置：

   .. prompt:: bash #

      radosgw-admin period update --commit

#. 重启 Ceph 对象网关：

   .. prompt:: bash #

      systemctl restart ceph-radosgw@rgw.`hostname -s`

完成这一步以后，可以继续\
`配置一个副域 <#configure-secondary-zones>`_ ，
并在主域组中创建第二个域。


多站配置参考
============
.. Multi-Site Configuration Reference

以下是附上细节信息，以及与 realm 、 period 、 zonegroup （域组）、 zone （域）
相关的命令行用法。

要看每个可用配置选项的详细用法，请看 ``src/common/options/rgw.yaml.in`` 文件。

另外，可以看看 :ref:`mgr-dashboard` 配置页面（在 `Cluster` 内），
在这里你可以方便地查看和配置所有选项。
在这个页面上，把水平设置为 ``advanced`` 然后搜索 RGW ，
能看到所有基本和高级的配置选项。


.. _rgw-realms:

Realms
------

一个 realm 就是一个全局唯一的命名空间，它由一个或多个域组组成。
域组又包含一或多个域，域包含桶、桶内是对象。

realm 概念可以让 Ceph 对象网关在同一套硬件上配置多个命名空间。

realm 暗含了 period 概念，
每个 period 表示了域组和域在当时的配置状态。
每次更改域组或域后都需要更新 period 并提交它。

考虑到与 Infernalis 以及更早版本的向后兼容问题，
默认情况下， Ceph 对象网关不会创建 realm 。
然而，我们建议您最好在新集群上创建 realm 。


创建 realm
~~~~~~~~~~
.. Create a Realm

创建 realm 可用 ``realm create`` 命令，并加上 realm 名字。
如果要创建默认的 realm ，需指定 ``--default`` 参数。

.. prompt:: bash #

   radosgw-admin realm create --rgw-realm={realm-name} [--default]

例如：

.. prompt:: bash #

   radosgw-admin realm create --rgw-realm=movies --default

指定 ``--default`` 以后，每次调用 ``radosgw-admin`` 都会默认指向这个 realm ，
除非另外指定了 ``--rgw-realm`` 和 realm 名字。

让 realm 成为默认
~~~~~~~~~~~~~~~~~
.. Make a Realm the Default

一堆 realm 里应该有一个默认的，而且只能有一个默认的。如果只有\
一个 realm ，而且创建时没把它设置为默认，也可以稍后设置成默认\
的。或者，要把某个 realm 改成默认的，用命令：

.. prompt:: bash #

   radosgw-admin realm default --rgw-realm=movies

.. note:: 如果要操作的 realm 是默认的，命令会认为已经加了
   ``--rgw-realm=<realm-name>`` 参数。

删除 realm
~~~~~~~~~~
.. Delete a Realm

删除 realm 可用 ``realm rm`` 并加上其名字。

.. prompt:: bash #

   radosgw-admin realm rm --rgw-realm={realm-name}

例如：

.. prompt:: bash #

   radosgw-admin realm rm --rgw-realm=movies

查看 realm
~~~~~~~~~~
.. Get a Realm

查看 realm 可用 ``realm get`` 并加上其名字。

.. prompt:: bash #

   radosgw-admin realm get --rgw-realm=<name>

例如：

.. prompt:: bash #

   radosgw-admin realm get --rgw-realm=movies [> filename.json]

::

    {
        "id": "0a68d52e-a19c-4e8e-b012-a8f831cb3ebc",
        "name": "movies",
        "current_period": "b0c5bbef-4337-4edd-8184-5aeab2ec413b",
        "epoch": 1
    }

配置 realm
~~~~~~~~~~
.. Set a Realm

配置 realm 用 ``realm set`` ，还要指定其名字、和 ``--infile=`` 并输入文件名。

.. prompt:: bash #

   radosgw-admin realm set --rgw-realm=<name> --infile=<infilename>

例如：

.. prompt:: bash #

   radosgw-admin realm set --rgw-realm=movies --infile=filename.json

罗列 realm
~~~~~~~~~~
.. List Realms

罗列 realm 可用 ``realm list`` ：

.. prompt:: bash #

   radosgw-admin realm list

罗列 realm 的 period
~~~~~~~~~~~~~~~~~~~~
.. List Realm Periods

罗列 realm 的 period 可用 ``realm list-periods`` 。

.. prompt:: bash #

   radosgw-admin realm list-periods

拉取 realm 配置
~~~~~~~~~~~~~~~
.. Pull a Realm

要把 realm 配置从包含主域组和主域的节点拉取到包含副域组或副域的节点，
在接收 realm 配置的节点上执行 ``realm pull`` ：

.. prompt:: bash #

   radosgw-admin realm pull --url={url-to-master-zone-gateway} --access-key={access-key} --secret={secret}

重命名 realm
~~~~~~~~~~~~
.. Rename a Realm

realm 并非 period 的一部分，所以，对 realm 的重命名只在本地生效，
不会随 ``realm pull`` 拉过去。重命名一个包含多个域的 realm 时，
需要在各个域上分别执行 ``rename`` 命令。

要重命名一个 realm ，执行下列命令：

.. prompt:: bash #

   radosgw-admin realm rename --rgw-realm=<current-name> --realm-new-name=<new-realm-name>

.. note:: **不要**\ 用 ``realm set`` 更改 ``name`` 参数，这样只能更改内部名字，
   指定 ``--rgw-realm`` 时还是只能用老的 realm 名。


域组
----
.. Zonegroups

域组出现后， Ceph 对象网关就可以支持多站部署和全局命名空间。
域组在 Infernalis 版之前叫作 region 。

域组定义了各个域内一或多个 Ceph 对象网关例程的地理位置。

域组的配置与典型的配置过程有所不同，因为不是所有配置都在 Ceph 配置文件里。

你可以罗列域组、查看或更改域组配置。


创建域组
~~~~~~~~
.. Create a Zone Group

创建域组时需指定：域组名、默认 realm 里新创建的各个域
（或者用 ``--rgw-realm=<realm-name>`` 选项指定了别的 realm ）。

加 ``--default`` 参数则创建为默认域组；
加 ``--master`` 参数则创建为主域组。例如：

.. prompt:: bash #

   radosgw-admin zonegroup create --rgw-zonegroup=<name> [--rgw-realm=<name>][--master] [--default]

.. note:: 已存在域组的配置可用 \
   ``zonegroup modify --rgw-zonegroup=<zonegroup-name>`` 更改。


让域组成为默认
~~~~~~~~~~~~~~
.. Making a Zonegroup the Default

一堆域组里应该有一个默认的，且只能有一个默认域组。
如果只有一个域组，且创建时没指定为默认，可让它成为默认域组。用命令：

#. 把一个域组指定成默认域组：

   .. prompt:: bash #

      radosgw-admin zonegroup default --rgw-zonegroup=comedy

   .. note:: 有默认域组时，命令会认为那个域组的名字就是
      ``--rgw-zonegroup=<zonegroup-name>`` 选项的参数。（本例中，
      为了保持一致性和易读性，保留了 ``<zonegroup-name>`` 。）

#. 更新 period ：

   .. prompt:: bash #

      radosgw-admin period update --commit

把域加进域组
~~~~~~~~~~~~
.. Adding a Zone to a Zonegroup

这一步解释如何把域加入域组。

#. 执行下列命令，把域加进域组：

   .. prompt:: bash #

      radosgw-admin zonegroup add --rgw-zonegroup=<name> --rgw-zone=<name>

#. 更新 period ：

   .. prompt:: bash #

      radosgw-admin period update --commit

删除域组中的域
~~~~~~~~~~~~~~
.. Removing a Zone from a Zonegroup

#. 从域组删除域可以用下列命令：

   .. prompt:: bash #

      radosgw-admin zonegroup remove --rgw-zonegroup=<name> --rgw-zone=<name>

#. 更新 period ：

   .. prompt:: bash #

      radosgw-admin period update --commit

重命名域组
~~~~~~~~~~
.. Renaming a Zonegroup

#. 重命名一个域组可以用：

   .. prompt:: bash #

      radosgw-admin zonegroup rename --rgw-zonegroup=<name> --zonegroup-new-name=<name>

#. 更新 period ：

   .. prompt:: bash #

      radosgw-admin period update --commit

删除域组
~~~~~~~~
.. Deleting a Zonegroup

#. 要删除域组，执行下列命令：

   .. prompt:: bash #

      radosgw-admin zonegroup delete --rgw-zonegroup=<name>

#. 更新 period ：

   .. prompt:: bash #

      radosgw-admin period update --commit

罗列域组
~~~~~~~~
.. Listing Zonegroups

一个 Ceph 集群可以有很多域组，用以下命令可以罗列出来：

.. prompt:: bash #

   radosgw-admin zonegroup list

``radosgw-admin`` 会返回 JSON 格式的域组列表：

::

    {
        "default_info": "90b28698-e7c3-462c-a42d-4aa780d24eda",
        "zonegroups": [
            "us"
        ]
    }

查看域组映射图
~~~~~~~~~~~~~~
.. Getting a Zonegroup Map

要查看各域组的详情，执行下列命令：

.. prompt:: bash #

   radosgw-admin zonegroup-map get

.. note:: 如果你遇到了 ``failed to read zonegroup map`` 错误，\
   首先试一下以 root 身份运行 ``radosgw-admin zonegroup-map update`` 。

查看域组
~~~~~~~~
.. Getting a Zonegroup

查看域组配置可以用命令：

.. prompt:: bash #

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

配置域组
~~~~~~~~
.. Setting a Zonegroup

定义域组需创建一个 JSON 对象，并指定必需选项。下面是必需选项：

#. ``name``: 域组的名字，必需。

#. ``api_name``: 域组的 API 名字，可选。

#. ``is_master``: 决定此域组是否为主域组，必需。
   **注意：** 一套系统只能有一个主域组。

#. ``endpoints``: 此域组可服务的终结点列表，例如，你可以让多个\
   域名指向同一域组。记得转义正斜线（ ``\/`` ）。每个终结点都\
   可以分别指定端口（ ``fqdn:port`` ）。可选参数。

#. ``hostnames``: 域组内所有主机名的列表，例如，
   你可以让多个域名指向同一域组。可选参数。
   ``rgw dns name`` 选项会自动包含在这个列表内，
   更改此选项后需重启网关进程。

#. ``master_zone``: 域组的主域，不指定则为默认域，可选参数。\
   **注意：**\ 每个域组只能有一个主域。

#. ``zones``: 域组内所有域的列表，每个域需包含其名字（必需）、\
   终结点列表（可选）、以及网关是否需记录元数据和数据操作（默认为否）。

#. ``placement_targets``: 归置靶列表（可选），
   每个归置靶需包含其名字（必需）、和一个标签列表（可选），
   只有打了这些标签的用户才可以使用这个归置靶
   （即用户信息里的 ``placement_tags`` 字段）。

#. ``default_placement``: 对象索引和对象数据的默认归置靶，
   默认为 ``default-placement`` 。
   你也可以为每个用户分别设置它们自己的默认归置靶，设置在用户信息里。

配置域组 - 步骤
~~~~~~~~~~~~~~~
.. Setting a Zonegroup - Procedure

#. 要配置域组，需创建一个包含必需字段的 JSON 对象，并存入文件
   （例如 ``zonegroup.json`` ），然后执行下列命令：

   .. prompt:: bash #

      radosgw-admin zonegroup set --infile zonegroup.json

   其中 ``zonegroup.json`` 是刚刚创建的 JSON 文件。

   .. important:: 名为 ``default`` 的域组其 ``is_master`` 选项的值默认是 ``true`` 。
      如果你要新建域组并让它成为主域组，必须把域组 ``default`` 的
      ``is_master`` 选项设置为 ``false`` ，或者删除域组 ``default`` 。

#. 更新 period ：

   .. prompt:: bash #

      radosgw-admin period update --commit

配置域组映射图
~~~~~~~~~~~~~~
.. Setting a Zonegroup Map

配置域组映射图的过程包括： (1) 创建一个包含一或多个域组的 JSON 对象，
(2) 设置集群的 ``master_zonegroup`` 。域组映射图里的每个域组都包含一个键值对，
其中 ``key`` 选项相当于单个域组配置里的 ``name`` 选项，
``val`` 是包含着整个域组配置的 JSON 对象。

你只能有一个 ``is_master`` 为 ``true`` 的域组，而且它必须是域组映射图尾部
``master_zonegroup`` 选项的值。下面是默认域组映射图的一个实例：

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

#. 要配置域组映射图，执行下列命令：

   .. prompt:: bash #

      radosgw-admin zonegroup-map set --infile zonegroupmap.json

    其中 ``zonegroupmap.json`` 是你创建的 JSON 文件，
    需确保域组映射图里的域都已创建。

#. 更新 period ：

   .. prompt:: bash #

      radosgw-admin period update --commit


.. _radosgw-zones:

域
--
.. Zones

域定义的是一或多个 Ceph 对象网关例程的逻辑分组。在某一个域中，
所有 RGW 都可以提供 S3 对象服务，这些 S3 对象底层是 RADOS 对象，
这些 RADOS 对象存储在同一集群的同一组存储池中。Ceph 对象网关组成了域。

域的配置不同于典型配置过程，因为有些配置不在 Ceph 配置文件里。

你可以罗列域、查看或修改域配置。

创建域
~~~~~~
.. Create a Zone

创建域时，需指定其名字。如果创建的是主域，得加上 ``--master`` 选项，
一个域组只能有一个主域；若要把域加入域组，
需加上 ``--rgw-zonegroup`` 选项和域组名字。

.. prompt:: bash #

   radosgw-admin zone create --rgw-zone=<name> \
                    [--zonegroup=<zonegroup-name]\
                    [--endpoints=<endpoint>[,<endpoint>] \
                    [--master] [--default] \
                    --access-key $SYSTEM_ACCESS_KEY --secret $SYSTEM_SECRET_KEY

创建完域之后，更新 period ：

.. prompt:: bash #

   radosgw-admin period update --commit

删除域
~~~~~~
.. Deleting a Zone

删除域前，要先从域组删掉：

.. prompt:: bash #

   radosgw-admin zonegroup remove --zonegroup=<name>\
                                     --zone=<name>

然后，更新 period ：

.. prompt:: bash #

   radosgw-admin period update --commit

接下来，删除这个域：

.. prompt:: bash #

   radosgw-admin zone delete --rgw-zone<name>

最后，更新 period ：

.. prompt:: bash #

   radosgw-admin period update --commit

.. important:: 从域组删掉域之前先不要删除这个域，否则更新 period 时会失败。

域被删除后，如果其它地方也不需要与之相关的存储池，可以考虑删除掉，
把下面实例中的 ``<del-zone>`` 替换成已删除域的名字即可。

.. important:: 只能删除以域名打头的存储池。
   若删除根存储池（如 ``.rgw.root`` ），
   会删除整个系统的配置。

.. important:: 一旦删除存储池，其内的数据也会被删除，
   且不可恢复。所以，确定存储池内容不需要了再删除。

.. prompt:: bash #

   ceph osd pool rm <del-zone>.rgw.control <del-zone>.rgw.control --yes-i-really-really-mean-it
   ceph osd pool rm <del-zone>.rgw.meta <del-zone>.rgw.meta --yes-i-really-really-mean-it
   ceph osd pool rm <del-zone>.rgw.log <del-zone>.rgw.log --yes-i-really-really-mean-it
   ceph osd pool rm <del-zone>.rgw.otp <del-zone>.rgw.otp --yes-i-really-really-mean-it
   ceph osd pool rm <del-zone>.rgw.buckets.index <del-zone>.rgw.buckets.index --yes-i-really-really-mean-it
   ceph osd pool rm <del-zone>.rgw.buckets.non-ec <del-zone>.rgw.buckets.non-ec --yes-i-really-really-mean-it
   ceph osd pool rm <del-zone>.rgw.buckets.data <del-zone>.rgw.buckets.data --yes-i-really-really-mean-it


修改域配置
~~~~~~~~~~
.. Modifying a Zone

修改域配置需指定域名、以及你想更改的参数。

.. prompt:: bash #

   radosgw-admin zone modify [options]

其中 ``[options]`` 可以是： 

- ``--access-key=<key>``
- ``--secret 或 --secret-key=<key>``
- ``--master``
- ``--default``
- ``--endpoints=<list>``

然后，更新 period ：

.. prompt:: bash #

   radosgw-admin period update --commit

罗列域
~~~~~~
.. Listing Zones

以 ``root`` 身份罗列集群中的域：

.. prompt:: bash #

   radosgw-admin zone list

查看域
~~~~~~
.. Getting a Zone

以 ``root`` 身份查看某个域的配置：

.. prompt:: bash #

   radosgw-admin zone get [--rgw-zone=<zone>]

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


配置域
~~~~~~
.. Setting a Zone

配置域时需指定一系列 Ceph 对象网关例程的存储池，\
考虑到一致性，我们建议用域的名字作为存储池前缀。\
存储池如何配置见\
`存储池 <http://docs.ceph.com/en/latest/rados/operations/pools/#pools>`__ 。

要配置域，需创建一个包含存储池的 JSON 对象，并存入一个文件
（如 ``zone.json`` ），然后执行下列命令，把 ``{zone-name}`` 替换为域的名字：

.. prompt:: bash #

   radosgw-admin zone set --rgw-zone={zone-name} --infile zone.json

其中 ``zone.json`` 是你创建的 JSON 文件。

然后，以 ``root`` 用户身份更新 period ：

.. prompt:: bash #

   radosgw-admin period update --commit

重命名域
~~~~~~~~
.. Renaming a Zone

要重命名域，需指定域的名字和新的域名。

.. prompt:: bash #

   radosgw-admin zone rename --rgw-zone=<name> --zone-new-name=<name>

然后，更新 period ：

.. prompt:: bash #

   radosgw-admin period update --commit


域组和域选项
------------
.. Zonegroup and Zone Settings

配置默认的域组和域时，存储池名字里包含域的名字，
例如：

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



.. _`存储池`: ../pools
.. _`同步策略配置`: ../multisite-sync-policy
