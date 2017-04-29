.. _RGW Multisite:

==========
 RGW 多站
==========

.. versionadded:: Jewel

Ceph 从 Jewel 版起，你可以让各个 :term:`Ceph 对象网关`\ 在\
多活域内运行，多活域允许写入非主域。下面是会用到的基本术语：

- **域 (Zone)**: 域是一或多个 Ceph 对象网关例程的\ *逻辑*\
  分组。每个域组应该指定一个域为主域，由它负责所有桶和用户\
  的创建。

- **域组 (Zonegroup)**: 域组由多个域组成，此概念大致相当于
  Jewel 版以前联盟部署中的辖区（ region ）。应该有一个主\
  域组，负责处理系统配置变更。

- **域组映射图 (Zonegroup map)**: 域组映射图是个配置的数据\
  结构，它保存着整个系统的映射图，也就是哪个域组是主的、各\
  个域组间的关系、以及其它可配置信息，如存储策略。

- **组界 (Realm)**: 组界是域组的容器，有了它就能跨集群划分\
  域组。系统允许创建多个组界，这样就能轻易地在同一集群内跑\
  多个不同的配置。

- **界期 (Period)**: 界期保存着组界当前状态的配置数据结构。\
  每个界期都包含一个唯一标识符和一个时间结（ epoch )，每个\
  提交操作都会使界期的时间结递增。每个组界都有一个相关联的\
  当前界期，它保存着当前配置的域组和存储策略。任何非主域的\
  变更都会引起界期的时间结递增。把主域改为另一个域会牵涉以\
  下变更：

  - 根据新界期和为 1 的时间结生成新的界期；
  - 更新组界的当前界期，指向新生成的界期标识符；
  - 组界的时间结递增；


.. _About this guide:

关于此指南
==========

在本指南中，我们要用到两个 Ceph 集群，只创建一个域组，以及三\
个域，它们之间会活跃地同步数据。为达成目标，我们会在同一集群\
内创建 2 个域、在另一集群内创建 1 个域。不涉及在 Ceph 对象网\
关例程间镜像数据变更的同步代理，这样的话配置方案和多活配置简\
单得多。需要注意的是，像用户创建这类元数据操作仍然需要主域处\
理，然而，数据操作（像桶、对象的创建）可以由任意域处理。


.. _System Keys:

系统密钥
--------

配置各个域时， Ceph 对象网关例程希望创建的域内有系统用户，此\
用户有 S3 风格的访问密钥和秘密密钥。这样其它 Ceph 对象网关例\
程用访问密钥和秘密密钥就能远程拉取它的配置。为了能更简单地实\
现脚本化、或者配置管理工具的使用，应该事先生成访问密钥和秘密\
密钥，后面就用这些。在本文档中，我们把访问密钥和秘密密钥设置\
在环境变量中，例如： ::

  $ SYSTEM_ACCESS_KEY=1555b35654ad1656d805
  $ SYSTEM_SECRET_KEY=h7GhxuBLTrlhVUyxSPUKUV8r/2EI4ngqJxD7iBdBYLhwluN30JaT3Q12

通常，访问密钥是 20 个字母数字组成的字符串，秘密密钥是 40 个\
字母数字组成的字符串（它们也可以包含 +/= ），这些密钥可以在
shell 中生成，例如： ::

  $ SYSTEM_ACCESS_KEY=$(cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 20 | head -n 1)
  $ SYSTEM_SECRET_KEY=$(cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 40 | head -n 1)

然后我们用下列命令创建系统用户： ::

  $ radosgw-admin user create --uid=zone.user --display-name="Zone User" \
                 --access-key=$SYSTEM_ACCESS_KEY --secret=$SYSTEM_SECRET_KEY --system


.. _Naming conventions:

命名惯例
--------

本段描述了配置一个主域的过程。在本指南中，我们假设一个名为
``us`` 的域组，位于美国，它是我们的主域组，包含两个域，其名\
字格式为 ``{zonegroup}-{zone}`` ，这仅仅是个惯例，你可以用\
自己喜欢的方式命名。总的来说：

- 主域组：美国 ``us``
- 主域：美国，东区 1 ： ``us-east-1``
- 第二域：美国，东区 2 ： ``us-east-2``
- 第二域：美国，西区： ``us-west``

这些只是更大组界（假设为 ``gold`` ）的一部分， ``us-east-1``
和 ``us-east-2`` 是同一 Ceph 集群的一部分，其中，把
``us-east-1`` 配置为主的， ``us-west`` 将位于另一个不同的
Ceph 集群。


创建存储池
----------

如果权限配置得当， Ceph 对象网关例程可以自己创建各存储池，需\
要用到 ``ceph.conf`` 里的 ``pg_num`` 和 ``pgp_num`` 的配置值。\
具体细节请参考 `Ceph 对象网关——创建存储池`_\ 。某一域内的存\
储池按惯例以 ``{zone-name}.pool-name`` 命名，以 ``us-east-1``
域为例，按默认配置将创建以下存储池：

- ``.rgw.root``
- ``us-east-1.rgw.control``
- ``us-east-1.rgw.data.root``
- ``us-east-1.rgw.gc``
- ``us-east-1.rgw.log``
- ``us-east-1.rgw.intent-log``
- ``us-east-1.rgw.usage``
- ``us-east-1.rgw.users.keys``
- ``us-east-1.rgw.users.email``
- ``us-east-1.rgw.users.swift``
- ``us-east-1.rgw.users.uid``
- ``us-east-1.rgw.buckets.index``
- ``us-east-1.rgw.buckets.data``
- ``us-east-1.rgw.meta``

其它域内也需要创建这些存储池，但需把存储池名中的 ``us-east-1``
替换为规划的名字。


.. _Configuring the master zone in the primary cluster:

配置主集群的主域
================

首先，我们要配置主域组 ``us`` 内的主域 ``us-east-1`` ，它将\
作为主域负责所有的原数据操作，因此所有的用户创建之类的操作将\
由这个域处理。


.. _Creating a realm:

创建组界
--------

配置一个名为 ``gold`` 的组界，并使其成为默认的： ::

  # radosgw-admin realm create --rgw-realm=gold --default
  {
    "id": "4a367026-bd8f-40ee-b486-8212482ddcd7",
    "name": "gold",
    "current_period": "09559832-67a4-4101-8b3f-10dfcd6b2707",
    "epoch": 1
  }

注意，每个组界都有一个 id ，这样就能为某些操作提供便利，像\
后面的组界重命名就会用到； ``current_period`` 会随主域的任\
何变动而变化； ``epoch`` 在主域配置变更时会递增，因为它导致\
当前界期变化了。


.. _Deleting the default zonegroup:

删除默认域组
------------

安装 Ceph 对象网关时，它会假设默认的域组名为 default ，我们\
不需要这个，所以先删掉默认域组： ::

  # radosgw-admin zonegroup delete --rgw-zonegroup=default


.. _Creating a master zonegroup:

创建主域组
----------

我们将创建名为 ``us`` 的域组，作为主域组。主域组将会控制域组\
映射图、并负责把变更传达给系统的其余部分。我们还得把它设置为\
默认域组，这样后续命令就可以直接使用此域组了，无须再加
``--rgw-zonegroup`` 选项了。

::

   # radosgw-admin zonegroup create --rgw-zonegroup=us --endpoints=http://rgw1:80 --master --default
   {
    "id": "d4018b8d-8c0d-4072-8919-608726fa369e",
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
    "realm_id": "4a367026-bd8f-40ee-b486-8212482ddcd7"
    }

或者用下面的命令把此域组设置为默认域组： ::

  # radosgw-admin zonegroup default --rgw-zonegroup=us


.. _Creating a master zone:

创建主域
--------

下面创建一个域，并使其成为默认域。注意，进行元数据操作（如\
创建用户）时要用到此域。我们还要把它加进前面创建的域组里。

::

   # radosgw-admin zone create --rgw-zonegroup=us --rgw-zone=us-east-1 --endpoints=http://rgw1:80 --access-key=$SYSTEM_ACCESS_KEY --secret=$SYSTEM_SECRET_KEY --default --master
   {
    "id": "83859a9a-9901-4f00-aa6d-285c777e10f0",
    "name": "us-east-1",
    "domain_root": "us-east-1.rgw.data.root",
    "control_pool": "us-east-1.rgw.control",
    "gc_pool": "us-east-1.rgw.gc",
    "log_pool": "us-east-1.rgw.log",
    "intent_log_pool": "us-east-1.rgw.intent-log",
    "usage_log_pool": "us-east-1.rgw.usage",
    "user_keys_pool": "us-east-1.rgw.users.keys",
    "user_email_pool": "us-east-1.rgw.users.email",
    "user_swift_pool": "us-east-1.rgw.users.swift",
    "user_uid_pool": "us-east-1.rgw.users.uid",
    "system_key": {
        "access_key": "1555b35654ad1656d804",
        "secret_key": "h7GhxuBLTrlhVUyxSPUKUV8r\/2EI4ngqJxD7iBdBYLhwluN30JaT3Q=="
    },
    "placement_pools": [
        {
            "key": "default-placement",
            "val": {
                "index_pool": "us-east-1.rgw.buckets.index",
                "data_pool": "us-east-1.rgw.buckets.data",
                "data_extra_pool": "us-east-1.rgw.buckets.non-ec",
                "index_type": 0
            }
        }
    ],
    "metadata_heap": "us-east-1.rgw.meta",
    "realm_id": "4a367026-bd8f-40ee-b486-8212482ddcd7"
    }

上面的命令中， ``--rgw-zonegroup`` 和 ``--default`` 选项把域\
加进了域组、也使其成为默认域了。下列命令也异曲同工： ::

    # radosgw-admin zone default --rgw-zone=us-east-1
    # radosgw-admin zonegroup add --rgw-zonegroup=us --rgw-zone=us-east-1


.. _Creating system users:

创建系统用户
------------

接下来我们创建用于访问域存储池的系统用户，注意，这些密钥在配\
置第二域时还要用到： ::

  # radosgw-admin user create --uid=zone.user --display-name="Zone
  User" --access-key=$SYSTEM_ACCESS_KEY --secret=$SYSTEM_SECRET_KEY --system
  {
    "user_id": "zone.user",
    "display_name": "Zone User",
    "email": "",
    "suspended": 0,
    "max_buckets": 1000,
    "auid": 0,
    "subusers": [],
    "keys": [
        {
            "user": "zone.user",
            "access_key": "1555b35654ad1656d804",
            "secret_key": "h7GhxuBLTrlhVUyxSPUKUV8r\/2EI4ngqJxD7iBdBYLhwluN30JaT3Q=="
        }
    ],
    "swift_keys": [],
    "caps": [],
    "op_mask": "read, write, delete",
    "system": "true",
    "default_placement": "",
    "placement_tags": [],
    "bucket_quota": {
        "enabled": false,
        "max_size_kb": -1,
        "max_objects": -1
    },
    "user_quota": {
        "enabled": false,
        "max_size_kb": -1,
        "max_objects": -1
    },
    "temp_url_keys": []
  }


.. note:: 要注意的是，系统用户在整个域里都有超级用户权限，其\
   操作的行为也和普通用户不一样，像创建桶、对象等等操作，因\
   为它的输出包含额外的 json 字段，这些字段维护着元数据。


.. _Update the period:

更新界期
--------

我们更改了主域配置，因此需要提交这些变更，以反映组界的配置结\
构。这是初始的界期： ::

  # radosgw-admin period get
   {
    "id": "09559832-67a4-4101-8b3f-10dfcd6b2707",
    "epoch": 1,
    "predecessor_uuid": "",
    "sync_status": [],
    "period_map": {
        "id": "09559832-67a4-4101-8b3f-10dfcd6b2707",
        "zonegroups": [],
        "short_zone_ids": []
    },
    "master_zonegroup": "",
    "master_zone": "",
    "period_config": {
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
    },
    "realm_id": "4a367026-bd8f-40ee-b486-8212482ddcd7",
    "realm_name": "gold",
    "realm_epoch": 1
    }

现在，更新界期并提交变更： ::

  # radosgw-admin period update --commit
  {
    "id": "b5e4d3ec-2a62-4746-b479-4b2bc14b27d1",
    "epoch": 1,
    "predecessor_uuid": "09559832-67a4-4101-8b3f-10dfcd6b2707",
    "sync_status": [ ""... # truncating the output here
    ],
    "period_map": {
        "id": "b5e4d3ec-2a62-4746-b479-4b2bc14b27d1",
        "zonegroups": [
            {
                "id": "d4018b8d-8c0d-4072-8919-608726fa369e",
                "name": "us",
                "api_name": "us",
                "is_master": "true",
                "endpoints": [
                    "http:\/\/rgw1:80"
                ],
                "hostnames": [],
                "hostnames_s3website": [],
                "master_zone": "83859a9a-9901-4f00-aa6d-285c777e10f0",
                "zones": [
                    {
                        "id": "83859a9a-9901-4f00-aa6d-285c777e10f0",
                        "name": "us-east-1",
                        "endpoints": [
                            "http:\/\/rgw1:80"
                        ],
                        "log_meta": "true",
                        "log_data": "false",
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
                "realm_id": "4a367026-bd8f-40ee-b486-8212482ddcd7"
            }
        ],
        "short_zone_ids": [
            {
                "key": "83859a9a-9901-4f00-aa6d-285c777e10f0",
                "val": 630926044
            }
        ]
    },
    "master_zonegroup": "d4018b8d-8c0d-4072-8919-608726fa369e",
    "master_zone": "83859a9a-9901-4f00-aa6d-285c777e10f0",
    "period_config": {
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
    },
    "realm_id": "4a367026-bd8f-40ee-b486-8212482ddcd7",
    "realm_name": "gold",
    "realm_epoch": 2
    }


.. _Starting the Ceph Object Gateway instance:

启动 Ceph 对象网关例程
----------------------

启动 Ceph 对象网关例程前，需在配置文件里配置好 ``rgw_zone``
和 ``rgw_frontends`` 选项，请参考\ `安装 Ceph 网关`_\ 。
Ceph 对象网关例程的配置段应该类似如下： ::

  [client.rgw.us-east-1]
  rgw_frontends="civetweb port=80"
  rgw_zone=us-east-1

然后启动 Ceph 对象网关例程（有别于你的操作系统）： ::

  $ sudo systemctl start ceph-radosgw@rgw.us-east-1


.. _Configuring the second zone in same cluster:

在同一集群内配置第二域
======================

接下来配置第二域 ``us-east-2`` ，还在同一集群内。在同一集群\
内，下面的所有命令都能在主域所在的节点上执行。


.. _Creating the second zone:

创建第二域
----------

与创建主域的步骤相似，只是这里要去掉 ``--master`` 选项： ::

  # radosgw-admin zone create --rgw-zonegroup=us --rgw-zone=us-east-2 --access-key=$SYSTEM_ACCESS_KEY --secret=$SYSTEM_SECRET_KEY --endpoints=http://rgw2:80
  {
    "id": "950c1a43-6836-41a2-a161-64777e07e8b8",
    "name": "us-east-2",
    "domain_root": "us-east-2.rgw.data.root",
    "control_pool": "us-east-2.rgw.control",
    "gc_pool": "us-east-2.rgw.gc",
    "log_pool": "us-east-2.rgw.log",
    "intent_log_pool": "us-east-2.rgw.intent-log",
    "usage_log_pool": "us-east-2.rgw.usage",
    "user_keys_pool": "us-east-2.rgw.users.keys",
    "user_email_pool": "us-east-2.rgw.users.email",
    "user_swift_pool": "us-east-2.rgw.users.swift",
    "user_uid_pool": "us-east-2.rgw.users.uid",
    "system_key": {
        "access_key": "1555b35654ad1656d804",
        "secret_key": "h7GhxuBLTrlhVUyxSPUKUV8r\/2EI4ngqJxD7iBdBYLhwluN30JaT3Q=="
    },
    "placement_pools": [
        {
            "key": "default-placement",
            "val": {
                "index_pool": "us-east-2.rgw.buckets.index",
                "data_pool": "us-east-2.rgw.buckets.data",
                "data_extra_pool": "us-east-2.rgw.buckets.non-ec",
                "index_type": 0
            }
        }
    ],
    "metadata_heap": "us-east-2.rgw.meta",
    "realm_id": "815d74c2-80d6-4e63-8cfc-232037f7ff5c"
    }


.. _Updating the period:

更新界期
--------

接下来，我们把系统映射的变更通知到所有 Ceph 对象网关例程，需\
更新界期、并提交变更： ::

  # radosgw-admin period update --commit
  {
    "id": "b5e4d3ec-2a62-4746-b479-4b2bc14b27d1",
    "epoch": 2,
    "predecessor_uuid": "09559832-67a4-4101-8b3f-10dfcd6b2707",
    "sync_status": [ ""... # truncating the output here
    ],
    "period_map": {
        "id": "b5e4d3ec-2a62-4746-b479-4b2bc14b27d1",
        "zonegroups": [
            {
                "id": "d4018b8d-8c0d-4072-8919-608726fa369e",
                "name": "us",
                "api_name": "us",
                "is_master": "true",
                "endpoints": [
                    "http:\/\/rgw1:80"
                ],
                "hostnames": [],
                "hostnames_s3website": [],
                "master_zone": "83859a9a-9901-4f00-aa6d-285c777e10f0",
                "zones": [
                    {
                        "id": "83859a9a-9901-4f00-aa6d-285c777e10f0",
                        "name": "us-east-1",
                        "endpoints": [
                            "http:\/\/rgw1:80"
                        ],
                        "log_meta": "true",
                        "log_data": "false",
                        "bucket_index_max_shards": 0,
                        "read_only": "false"
                    },
                    {
                        "id": "950c1a43-6836-41a2-a161-64777e07e8b8",
                        "name": "us-east-2",
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
                "realm_id": "4a367026-bd8f-40ee-b486-8212482ddcd7"
            }
        ],
        "short_zone_ids": [
            {
                "key": "83859a9a-9901-4f00-aa6d-285c777e10f0",
                "val": 630926044
            },
            {
                "key": "950c1a43-6836-41a2-a161-64777e07e8b8",
                "val": 4276257543
            }

        ]
    },
    "master_zonegroup": "d4018b8d-8c0d-4072-8919-608726fa369e",
    "master_zone": "83859a9a-9901-4f00-aa6d-285c777e10f0",
    "period_config": {
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
    },
    "realm_id": "4a367026-bd8f-40ee-b486-8212482ddcd7",
    "realm_name": "gold",
    "realm_epoch": 2
    }


启动 Ceph 对象网关
------------------

现在，在对应节点上启动属于第二域的 Ceph 对象网关例程，与
``client.rgw.us-east-1`` 这个 Ceph 对象网关例程的类似，这里\
只需把配置文件里的 ``rgw zone=us-east-2`` 改掉即可，例如： ::

  [client.rgw.us-east-2]
  rgw_frontends="civetweb port=80"
  rgw_zone=us-east-2

然后启动这个 Ceph 对象网关例程（有别于你的操作系统）： ::

  $ sudo systemctl start ceph-radosgw@rgw.us-east-2


.. _Configuring the Ceph Object Gateway in the second Ceph cluster:

在第二集群配置 Ceph 对象网关
============================

下面继续配置第二个集群里的 Ceph 对象网关，它可以是地理上分开\
的，但划分进了相同的域组。

因为我们已经在第一个集群里配置了组界，这里把配置拉取过来、并\
使其成为默认组界即可。还需要从主域拉取配置，拉取界期即可： ::

  # radosgw-admin realm pull --url=http://rgw1:80
  --access-key=$SYSTEM_ACCESS_KEY --secret=$SYSTEM_SECRET_KEY
  {
    "id": "4a367026-bd8f-40ee-b486-8212482ddcd7",
    "name": "gold",
    "current_period": "b5e4d3ec-2a62-4746-b479-4b2bc14b27d1",
    "epoch": 2
  }

  # radosgw-admin period pull --url=http://rgw1:80 --access-key=$SYSTEM_ACCESS_KEY --secret=$SYSTEM_SECRET_KEY
  # radosgw-admin realm default --rgw-realm=gold

也把它的默认域组设置为前面创建的 ``us`` 域组： ::

  # radosgw-admin zonegroup default --rgw-zonegroup=us


配置第二域
----------

用相同的系统密钥创建个新的域 ``us-west`` ： ::

  # radosgw-admin zone create --rgw-zonegroup=us --rgw-zone=us-west
  --access-key=$SYSTEM_ACCESS_KEY --secret=$SYSTEM_SECRET_KEY --endpoints=http://rgw3:80 --default
  {
    "id": "d9522067-cb7b-4129-8751-591e45815b16",
    "name": "us-west",
    "domain_root": "us-west.rgw.data.root",
    "control_pool": "us-west.rgw.control",
    "gc_pool": "us-west.rgw.gc",
    "log_pool": "us-west.rgw.log",
    "intent_log_pool": "us-west.rgw.intent-log",
    "usage_log_pool": "us-west.rgw.usage",
    "user_keys_pool": "us-west.rgw.users.keys",
    "user_email_pool": "us-west.rgw.users.email",
    "user_swift_pool": "us-west.rgw.users.swift",
    "user_uid_pool": "us-west.rgw.users.uid",
    "system_key": {
        "access_key": "1555b35654ad1656d804",
        "secret_key": "h7GhxuBLTrlhVUyxSPUKUV8r\/2EI4ngqJxD7iBdBYLhwluN30JaT3Q=="
    },
    "placement_pools": [
        {
            "key": "default-placement",
            "val": {
                "index_pool": "us-west.rgw.buckets.index",
                "data_pool": "us-west.rgw.buckets.data",
                "data_extra_pool": "us-west.rgw.buckets.non-ec",
                "index_type": 0
            }
        }
    ],
    "metadata_heap": "us-west.rgw.meta",
    "realm_id": "4a367026-bd8f-40ee-b486-8212482ddcd7"
    }


更新界期
--------

要让域组映射图的变更传播出去，我们得更新并提交界期：::

  # radosgw-admin period update --commit --rgw-zone=us-west
  {
    "id": "b5e4d3ec-2a62-4746-b479-4b2bc14b27d1",
    "epoch": 3,
    "predecessor_uuid": "09559832-67a4-4101-8b3f-10dfcd6b2707",
    "sync_status": [
        "", # truncated
    ],
    "period_map": {
        "id": "b5e4d3ec-2a62-4746-b479-4b2bc14b27d1",
        "zonegroups": [
            {
                "id": "d4018b8d-8c0d-4072-8919-608726fa369e",
                "name": "us",
                "api_name": "us",
                "is_master": "true",
                "endpoints": [
                    "http:\/\/rgw1:80"
                ],
                "hostnames": [],
                "hostnames_s3website": [],
                "master_zone": "83859a9a-9901-4f00-aa6d-285c777e10f0",
                "zones": [
                    {
                        "id": "83859a9a-9901-4f00-aa6d-285c777e10f0",
                        "name": "us-east-1",
                        "endpoints": [
                            "http:\/\/rgw1:80"
                        ],
                        "log_meta": "true",
                        "log_data": "true",
                        "bucket_index_max_shards": 0,
                        "read_only": "false"
                    },
                                    {
                        "id": "950c1a43-6836-41a2-a161-64777e07e8b8",
                        "name": "us-east-2",
                        "endpoints": [
                            "http:\/\/rgw2:80"
                        ],
                        "log_meta": "false",
                        "log_data": "true",
                        "bucket_index_max_shards": 0,
                        "read_only": "false"
                    },
                    {
                        "id": "d9522067-cb7b-4129-8751-591e45815b16",
                        "name": "us-west",
                        "endpoints": [
                            "http:\/\/rgw3:80"
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
                "realm_id": "4a367026-bd8f-40ee-b486-8212482ddcd7"
            }
        ],
        "short_zone_ids": [
            {
                "key": "83859a9a-9901-4f00-aa6d-285c777e10f0",
                "val": 630926044
            },
            {
                "key": "950c1a43-6836-41a2-a161-64777e07e8b8",
                "val": 4276257543
            },
            {
                "key": "d9522067-cb7b-4129-8751-591e45815b16",
                "val": 329470157
            }
        ]
    },
    "master_zonegroup": "d4018b8d-8c0d-4072-8919-608726fa369e",
    "master_zone": "83859a9a-9901-4f00-aa6d-285c777e10f0",
    "period_config": {
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
    },
    "realm_id": "4a367026-bd8f-40ee-b486-8212482ddcd7",
    "realm_name": "gold",
    "realm_epoch": 2
    }

可以看到，界期的时间结（ epoch ）数字递增了，说明配置有变化。


启动 Ceph 对象网关例程
----------------------

Ceph 对象网关例程的启动与在第一个域时相似，唯一不同的就是
``rgw zone`` 选项，这里应该设置为 ``us-west`` ： ::

  [client.rgw.us-west]
  rgw_frontends="civetweb port=80"
  rgw_zone=us-west

然后启动 Ceph 对象网关例程（有别于你的操作系统）： ::

  $ sudo systemctl start ceph-radosgw@rgw.us-west


.. _`Ceph 对象网关——创建存储池`: ../config#create-pools
.. _`安装 Ceph 网关`: ../../install/install-ceph-gateway
