:orphan:

================================================
 radosgw-admin -- rados REST 网关的用户管理工具
================================================

.. program:: radosgw-admin

提纲
====

| **radosgw-admin** *command* [ *options* *...* ]


描述
====

:program:`radosgw-admin` 是 RADOS 网关用户管理工具，可用于创建\
和修改用户。


命令
====

:program:`radosgw-admin` 工具有很多命令，可完成各种管理任务：

:command:`user create`
  创建一个新用户。

:command:`user modify`
  修改一个用户。

:command:`user info`
  显示用户信息，以及可能存在的子用户和密钥。

:command:`user rm`
  删除一个用户。

:command:`user suspend`
  暂停某用户。

:command:`user enable`
  重新允许暂停的用户。

:command:`user check`
  检查用户信息。

:command:`user stats`
  显示配额子系统统计的用户状态。

:command:`user list`
  罗列出所有用户。

:command:`caps add`
  给用户分配能力。

:command:`caps rm`
  删除用户能力。

:command:`subuser create`
  新建一个子用户（适合使用 Swift API 的客户端）。

:command:`subuser modify`
  修改子用户。

:command:`subuser rm`
  删除子用户

:command:`key create`
  新建访问密钥。

:command:`key rm`
  删除访问密钥。

:command:`bucket list`
  罗列所有桶。

:command:`bucket limit check`
  显示桶的分片统计信息。

:command:`bucket link`
  把桶关联到指定用户。

:command:`bucket unlink`
  取消指定用户和桶的关联。

:command:`bucket stats`
  返回桶的统计信息。

:command:`bucket rm`
  删除一个桶。

:command:`bucket check`
  检查桶的索引信息。

:command:`bucket rewrite`
  重写指定桶内的所有对象。

:command:`bucket reshard`
  对桶进行重分片。

:command:`bucket sync disable`
  禁用桶同步。

:command:`bucket sync enable`
  允许桶同步。

:command:`bi get`
  检出桶索引对象条目。

:command:`bi put`
  存入桶索引对象条目。

:command:`bi list`
  罗列原始的桶索引条目。

:command:`bi purge`
  清除桶索引条目。

:command:`object rm`
  删除一个对象。

:command:`object stat`
  对某一对象执行 stat 操作，查看其元数据。

:command:`object unlink`
  从桶索引里去掉对象。

:command:`object rewrite`
  重写指定对象。

:command:`objects expire`
  启动过期对象的清理。

:command:`period rm`
  删除一个 period 。

:command:`period get`
  查看指定 period 的信息。

:command:`period get-current`
  查看当前 period 的信息。

:command:`period pull`
  拉取一个 period 。

:command:`period push`
  推送一个 period 。

:command:`period list`
  罗列所有 period 。

:command:`period update`
  更新暂存的 period 。

:command:`period commit`
  提交暂存的 period 。

:command:`quota set`
  设置配额参数。

:command:`quota enable`
  启用配额。

:command:`quota disable`
  禁用配额。

:command:`global quota get`
  查看全局配额参数。

:command:`global quota set`
  配置全局配额参数。

:command:`global quota enable`
  启用全局配额。

:command:`global quota disable`
  禁用全局配额。

:command:`realm create`
  新建一个 realm 。

:command:`realm rm`
  删除一个 realm 。

:command:`realm get`
  显示此 realm 的信息。

:command:`realm get-default`
  查看默认的 realm 名。

:command:`realm list`
  罗列所有 realm 。

:command:`realm list-periods`
  罗列所有 realm 的 period 。

:command:`realm rename`
  重命名一个 realm 。

:command:`realm set`
  设置 realm 信息（需要信息源 infile ）。

:command:`realm default`
  把此 realm 设置为默认的。

:command:`realm pull`
  拉取一个 realm 、及其当前的 period 。

:command:`zonegroup add`
  把一个域加入域组。

:command:`zonegroup create`
  新建一条域组信息。

:command:`zonegroup default`
  设置默认域组。

:command:`zonegroup rm`
  删除一条域组信息。

:command:`zonegroup get`
  显示域组信息。

:command:`zonegroup modify`
  更改现有域组。

:command:`zonegroup set`
  设置域组信息（需要信息源 infile ）。

:command:`zonegroup remove`
  从一域组中删除一个域。

:command:`zonegroup rename`
  重命名一个域组。

:command:`zonegroup list`
  罗列此集群内配置的所有域组。

:command:`zonegroup placement list`
  罗列域组的归置靶。

:command:`zonegroup placement add`
  把一个归置靶 id 加进一个域组。

:command:`zonegroup placement modify`
  更改某一特定域组内的一个归置靶。

:command:`zonegroup placement rm`
  删除一个域组的一个归置靶。

:command:`zonegroup placement default`
  设置一域组的默认归置靶。

:command:`zone create`
  新建一个域。

:command:`zone rm`
  删除一个域。

:command:`zone get`
  显示区域集群参数。

:command:`zone set`
  设置区域集群参数（需要输入文件）。

:command:`zone modify`
  更改现有域。

:command:`zone list`
  列出本集群内配置的所有区域。

:command:`metadata sync status`
  查看元数据同步状态。

:command:`metadata sync init`
  初始化元数据同步。

:command:`metadata sync run`
  启动元数据同步。

:command:`data sync status`
  查看指定源 zone 的数据同步状态。

:command:`data sync init`
  初始化指定源 zone 的数据同步。

:command:`data sync run`
  启动指定源 zone 的数据同步。

:command:`sync error list`
  罗列同步错误。

:command:`sync error trim`
  清理同步错误。

:command:`zone rename`
  重命名一个 zone 。

:command:`zone placement list`
  罗列 zone 的归置靶。

:command:`zone placement add`
  新增一个 zone 归置靶。

:command:`zone placement modify`
  更改一个 zone 的归置靶。

:command:`zone placement rm`
  删除一个 zone 的归置靶。

:command:`pool add`
  增加一个已有存储池用于数据归置。

:command:`pool rm`
  从数据归置集删除一个已有存储池。

:command:`pools list`
  罗列归置活跃集。

:command:`policy`
  显示桶或对象相关的策略。

:command:`log list`
  罗列日志对象。

:command:`log show`
  显示指定对象内（或指定桶、日期、桶标识符）的日志。
  （注意：日期格式必须是 YYYY-MM-DD-hh ）

:command:`log rm`
  删除日志对象。

:command:`usage show`
  查看使用率信息（可选选项有用户和数据范围）。

:command:`usage trim`
  修剪使用率信息（可选选项有用户和数据范围）。

:command:`gc list`
  显示过期的垃圾回收对象（加 --include-all 选项罗列所有条目，\
  包括未过期的）。

:command:`gc process`
  手动处理垃圾。

:command:`lc list`
  罗列所有桶的生命周期进度。

:command:`lc process`
  手动处理生命周期。

:command:`metadata get`
  读取元数据信息。

:command:`metadata put`
  设置元数据信息。

:command:`metadata rm`
  删除元数据信息。

:command:`metadata list`
  罗列元数据信息。

:command:`mdlog list`
  罗列元数据日志。

:command:`mdlog trim`
  裁截元数据日志。

:command:`mdlog status`
  读取元数据日志状态。

:command:`bilog list`
  罗列桶索引日志。

:command:`bilog trim`
  裁截桶索引日志（需要起始标记、结束标记）。

:command:`datalog list`
  罗列数据日志。

:command:`datalog trim`
  裁截数据日志。

:command:`datalog status`
  读取数据日志状态。

:command:`opstate list`
  罗列含状态操作（需要 client_id 、 op_id 、对象）。

:command:`opstate set`
  设置条目状态（需指定 client_id 、 op_id 、对象、状态）。

:command:`opstate renew`
  更新某一条目的状态（需指定 client_id 、 op_id 、对象）。

:command:`opstate rm`
  删除条目（需指定 client_id 、 op_id 、对象）。

:command:`orphans find`
  初始化、并开始检索遗漏的 RADOS 对象。

:command:`orphans finish`
  清理遗漏 RADOS 对象的检索结果。

:command:`orphans list-jobs`
  罗列当前正在进行的遗漏对象检索作业号。

:command:`role create`
  新建一个用于 STS 的 AWS 角色。

:command:`role rm`
  删除一个角色。

:command:`role get`
  获取一个角色。

:command:`role list`
  罗列带有指定路径前缀的角色。

:command:`role modify`
  修改现有角色的 assume role 策略。

:command:`role-policy put`
  新增、更新角色的权限策略。

:command:`role-policy list`
  罗列与一个角色相关的策略。

:command:`role-policy get`
  获取给定角色内嵌的指定内联策略文档。

:command:`role-policy rm`
  删除与一个角色相关的策略。

:command:`reshard add`
  安排一个桶进行重分片。

:command:`reshard list`
  罗列所有正在进行的桶重分片、或已安排准备重分片的作业。

:command:`reshard process`
  已安排重分片作业的进度。

:command:`reshard status`
  一个桶的重分片状态。

:command:`reshard cancel`
  取消一个桶的重分片。


选项
====

.. option:: -c ceph.conf, --conf=ceph.conf

   用指定的 ``ceph.conf`` 配置文件而非默认的
   ``/etc/ceph/ceph.conf`` 来确定启动时所需的监视器地址。

.. option:: -m monaddress[:port]

   连接到指定监视器，而非通过 ceph.conf 查询。

.. option:: --tenant=<tenant>

   租户的名字。

.. option:: --uid=uid

   radosgw 用户的 ID 。

.. option:: --subuser=<name>

   子用户名字。

.. option:: --access-key=<key>

   S3 访问密钥。

.. option:: --email=email

   用户的电子邮件地址。

.. option:: --secret/--secret-key=<key>

   私钥。

.. option:: --gen-access-key

   生成随机访问密钥（给 S3 ）。

.. option:: --gen-secret

   生成随机私钥。

.. option:: --key-type=<type>

   密钥类型，可用的有： swift 、 s3 。

.. option:: --temp-url-key[-2]=<key>

   临时 URL 密钥。

.. option:: --max-buckets

   一用户的最大桶数量（ 0 意为不限制，负值意为禁止创建桶）。
   默认为 1000 。

.. option:: --access=<access>

   为子用户设置访问权限。可用的访问权限有读、写、读写和完全。

.. option:: --display-name=<name>

   此用户的显示名字（昵称）。

.. option:: --admin

   给用户设置管理标志。

.. option:: --system

   给用户设置系统标识。

.. option:: --bucket=[tenant-id/]bucket

   指定桶名。如果未指定 tenant-id ，那就用用户（ --uid ）的
   tenant-id 。

.. option:: --pool=<pool>

   指定存储池名字。也可以用于 `orphans find` 指定数据存储池，\
   以扫描泄露的 rados 对象。

.. option:: --object=object

   指定对象名

.. option:: --date=yyyy-mm-dd

   格式为 yyyy-mm-dd 的日期。

.. option:: --start-date=yyyy-mm-dd

   格式为 yyyy-mm-dd 的起始日期。

.. option:: --end-date=yyyy-mm-dd

   格式为 yyyy-mm-dd 的终结日期。

.. option:: --bucket-id=<bucket-id>

   指定桶 id 。

.. option:: --bucket-new-name=[tenant-id/]<bucket>

   `bucket link` 命令的可选项，用于重命名一个桶。 tenant-id/
   可加可不加，常规操作一般没必要管。

.. option:: --shard-id=<shard-id>

   ``mdlog list`` 、 ``bi list`` 、 ``data sync status`` 命令\
   的可选项。对 ``mdlog trim`` 来说是必需的。

.. option:: --max-entries=<entries>

   罗列操作的可选参数，用于指定最大条数。

.. option:: --purge-data

   若加了此选项，删除用户时也一并删除用户的所有数据。

.. option:: --purge-keys

   若加了此选项，删除子用户时将一起删除其所有密钥。

.. option:: --purge-objects

   若加了此选项，删除此桶时也一并删除其内所有对象。

.. option:: --metadata-key=<key>

   用 ``metadata get`` 检索元数据时用的密钥。

.. option:: --remote=<remote>

   远程网关的域或域组 id 。

.. option:: --period=<id>

   Period id.

.. option:: --url=<url>

   用于推送、拉取 period 或 realm 的 URL 。

.. option:: --epoch=<number>

   Period epoch.

.. option:: --commit

   在执行 ``period update`` 期间提交此 period 。

.. option:: --staging

   查看暂存的 period 信息。

.. option:: --master

   设置为 master 。

.. option:: --master-zone=<id>

   主域的 id 。

.. option:: --rgw-realm=<name>

   realm 的名字。

.. option:: --realm-id=<id>

   realm 的 id 。

.. option:: --realm-new-name=<name>

   realm 的新名字。

.. option:: --rgw-zonegroup=<name>

   域组的名字。

.. option:: --zonegroup-id=<id>

   域组的 id 。

.. option:: --zonegroup-new-name=<name>

   域组的新名字。

.. option:: --rgw-zone=<zone>

   radosgw 所在的区域。

.. option:: --zone-id=<id>

   域的 id 。

.. option:: --zone-new-name=<name>

   此域的新名字。

.. option:: --source-zone

   数据同步的源 zone 。

.. option:: --default

   Set the entity (realm, zonegroup, zone) as default.

.. option:: --read-only

   Set the zone as read-only when adding to the zonegroup.

.. option:: --placement-id

   Placement id for the zonegroup placement commands.

.. option:: --tags=<list>

   The list of tags for zonegroup placement add and modify commands.

.. option:: --tags-add=<list>

   The list of tags to add for zonegroup placement modify command.

.. option:: --tags-rm=<list>

   The list of tags to remove for zonegroup placement modify command.

.. option:: --endpoints=<list>

   The zone endpoints.

.. option:: --index-pool=<pool>

   The placement target index pool.

.. option:: --data-pool=<pool>

   The placement target data pool.

.. option:: --data-extra-pool=<pool>

   The placement target data extra (non-ec) pool.

.. option:: --placement-index-type=<type>

   The placement target index type (normal, indexless, or #id).

.. option:: --tier-type=<type>

   The zone tier type.

.. option:: --tier-config=<k>=<v>[,...]

   Set zone tier config keys, values.

.. option:: --tier-config-rm=<k>[,...]

   Unset zone tier config keys.

.. option:: --sync-from-all[=false]

   Set/reset whether zone syncs from all zonegroup peers.

.. option:: --sync-from=[zone-name][,...]

   Set the list of zones to sync from.

.. option:: --sync-from-rm=[zone-name][,...]

   Remove the zones from list of zones to sync from.

.. option:: --fix

   除了检查桶索引，还修复它。

.. option:: --check-objects

   检查桶：根据对象的实际状态重建桶索引。

.. option:: --format=<format>

   为某些操作指定输出格式： xml 、 json 。

.. option:: --sync-stats

   ``user stats`` 的选项。若加了此选项，它就会用当前来自用户\
   桶索引的统计信息更新用户的统计信息。

.. option:: --show-log-entries=<flag>

   执行 ``log show`` 时，显示或不显示日志条目。

.. option:: --show-log-sum=<flag>

   执行 ``log show`` 时，显示或不显示日志汇总。

.. option:: --skip-zero-entries

   让 ``log show`` 只显示数字字段非零的日志。

.. option:: --infile

   设置时指定要读取的文件。

.. option:: --categories=<list>

   逗号分隔的一系列类目，显示使用情况时需要。

.. option:: --caps=<caps>

   能力列表，如 "usage=read, write; user=read" 。

.. option:: --compression=<compression-algorithm>

   归置靶的压缩算法（ lz4|snappy|zlib|zstd ）

.. option:: --yes-i-really-mean-it

   某些特定操作需要。

.. option:: --min-rewrite-size

   指定桶重写时的最小对象尺寸（默认 4M ）。

.. option:: --max-rewrite-size

   指定桶重写时的最大对象尺寸（默认 ULLONG_MAX ）。

.. option:: --min-rewrite-stripe-size

   指定对象重写时的最小条带尺寸（默认 0 ）。如果此值设置为 0 ，\
   那么被指定对象被重写后还需重新条带化。

.. option:: --warnings-only

   进行桶超限检查时若加了此选项，仅罗列出那些当前分片内\
   最大对象数接近或超过的桶。

.. option:: --bypass-gc

   删除桶时若加了此选项，则跳过 GC 直接触发对象删除。

.. option:: --inconsistent-index

   删除桶时若加了此选项、且加了 ``--bypass-gc`` 选项，则无视\
   桶索引的一致性。

.. option:: --max-concurrent-ios

   进行桶操作时的最大并行 IO 数。影响的操作诸如扫描桶索引，\
   如罗列、删除；还有所有的扫描、搜索操作，比如捡漏或\
   检查桶索引。默认值为 32 。


配额选项
========

.. option:: --max-objects

   指定最大对象数（负数为禁用）。

.. option:: --max-size

   指定最大尺寸（单位为 B/K/M/G/T ，负数为禁用）。

.. option:: --quota-scope

   配额有效范围（桶、用户）。


捡漏（ Orphans ）选项
=====================

.. option:: --num-shards

   用多少个分片临时保存扫描信息。

.. option:: --orphan-stale-secs

   对象被遗漏多久才被当作孤儿，单位是秒。
   默认是 86400 （ 24 小时）。

.. option:: --job-id

   设置作业标识符（适用于 ``orphans find`` ）。


.. Orphans list-jobs options

``orphans list-jobs`` 选项
==========================

.. option:: --extra-info

   在作业列表中展示额外信息。


.. Role Options

角色选项
========

.. option:: --role-name

   要创建角色的名字。

.. option:: --path

   角色的路径。

.. option:: --assume-role-policy-doc

   信任关系策略文档，用于授予一个实体权限，以担任此角色。

.. option:: --policy-name

   策略文档的名字。

.. option:: --policy-doc

   权限策略文档。

.. option:: --path-prefix

   用于过滤角色的路径前缀。


实例
====

生成一新用户： ::

        $ radosgw-admin user create --display-name="johnny rotten" --uid=johnny
        { "user_id": "johnny",
          "rados_uid": 0,
          "display_name": "johnny rotten",
          "email": "",
          "suspended": 0,
          "subusers": [],
          "keys": [
                { "user": "johnny",
                  "access_key": "TCICW53D9BQ2VGC46I44",
                  "secret_key": "tfm9aHMI8X76L3UdgE+ZQaJag1vJQmE6HDb5Lbrz"}],
          "swift_keys": []}

删除一用户： ::

        $ radosgw-admin user rm --uid=johnny
        
删除一个用户和与他相关的桶及内容： ::

        $ radosgw-admin user rm --uid=johnny --purge-data

删除一个桶： ::

	$ radosgw-admin bucket rm --bucket=foo

把桶链接到指定用户： ::

	$ radosgw-admin bucket link --bucket=foo --bucket_id=<bucket id> --uid=johnny

切断桶与指定用户的链接： ::

        $ radosgw-admin bucket unlink --bucket=foo --uid=johnny

显示一个桶从 2012 年 4 月 1 日起的日志： ::

        $ radosgw-admin log show --bucket=foo --date=2012-04-01-01 --bucket-id=default.14193.1

显示某用户 2012 年 3 月 1 日（不含）到 4 月 1 日期间的使用情况： ::

        $ radosgw-admin usage show --uid=johnny \
                        --start-date=2012-03-01 --end-date=2012-04-01

只显示所有用户的使用情况汇总： ::

        $ radosgw-admin usage show --show-log-entries=false

裁剪掉某用户 2012 年 4 月 1 日之前的使用信息： ::

        $ radosgw-admin usage trim --uid=johnny --end-date=2012-04-01


使用范围
========

:program:`radosgw-admin` 是 Ceph 的一部分，这是个伸缩力强、开源、分布式的\
存储系统，更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph <ceph>`\(8)
:doc:`radosgw <radosgw>`\(8)
