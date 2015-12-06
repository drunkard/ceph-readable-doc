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

:program:`radosgw-admin` 是 RADOS 网关用户管理工具，可用于创建和修改用户。


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

:command:`object rm`
  删除一个对象。

:command:`object unlink`
  从桶索引里去掉对象。

:command:`quota set`
  设置配额参数。

:command:`quota enable`
  启用配额。

:command:`quota disable`
  禁用配额。

:command:`region get`
  显示 region 信息。

:command:`regions list`
  列出本集群配置的所有 region 。

:command:`region set`
  设置 region 信息（需要输入文件）。

:command:`region default`
  设置默认 region 。

:command:`region-map get`
  显示 region-map 。

:command:`region-map set`
  设置 region-map （需要输入文件）。

:command:`zone get`
  显示区域集群参数。

:command:`zone set`
  设置区域集群参数（需要输入文件）。

:command:`zone list`
  列出本集群内配置的所有区域。

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

:command:`log rm`
  删除日志对象。

:command:`usage show`
  查看使用率信息（可选选项有用户和数据范围）。

:command:`usage trim`
  修剪使用率信息（可选选项有用户和数据范围）。

:command:`temp remove`
  删除指定日期（时间可选）之前创建的临时对象。

:command:`gc list`
  显示过期的垃圾回收对象（加 --include-all 选项罗列所有条目，包括未过期的）。

:command:`gc process`
  手动处理垃圾。

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

:command:`bilog list`
  罗列桶索引日志。

:command:`bilog trim`
  裁截桶索引日志（需要起始标记、结束标记）。

:command:`datalog list`
  罗列数据日志。

:command:`datalog trim`
  裁截数据日志。

:command:`opstate list`
  罗列含状态操作（需要 client_id 、 op_id 、对象）。

:command:`opstate set`
  设置条目状态（需指定 client_id 、 op_id 、对象、状态）。

:command:`opstate renew`
  更新某一条目的状态（需指定 client_id 、 op_id 、对象）。

:command:`opstate rm`
  删除条目（需指定 client_id 、 op_id 、对象）。

:command:`replicalog get`
  读取复制元数据日志条目。

:command:`replicalog delete`
  删除复制元数据日志条目。


选项
====

.. option:: -c ceph.conf, --conf=ceph.conf

   用指定的 ``ceph.conf`` 配置文件而非默认的 ``/etc/ceph/ceph.conf`` 来确定启\
   动时所需的监视器地址。

.. option:: -m monaddress[:port]

   连接到指定监视器，而非通过 ceph.conf 查询。

.. option:: --uid=uid

   radosgw 用户的 ID 。

.. option:: --subuser=<name>

   子用户名字。

.. option:: --email=email

   用户的电子邮件地址。

.. option:: --display-name=name

   配置用户的显示名称（昵称）

.. option:: --access-key=<key>

   S3 访问密钥。

.. option:: --gen-access-key

   生成随机访问密钥（给 S3 ）。

.. option:: --secret=secret

   指定密钥的密文。

.. option:: --gen-secret

   生成随机密钥。

.. option:: --key-type=<type>

   密钥类型，可用的有： swift 、 S3 。

.. option:: --temp-url-key[-2]=<key>

   临时 URL 密钥。

.. option:: --system

   给用户设置系统标识。

.. option:: --bucket=bucket

   指定桶名

.. option:: --object=object

   指定对象名

.. option:: --date=yyyy-mm-dd

   某些命令所需的日期

.. option:: --start-date=yyyy-mm-dd

   某些命令所需的起始日期

.. option:: --end-date=yyyy-mm-dd

   某些命令所需的终结日期

.. option:: --shard-id=<shard-id>

   执行 ``mdlog list`` 时为可选项。对 ``mdlog trim`` 、 \
   ``replica mdlog get/delete`` 、 ``replica datalog get/delete`` 来说是必\
   须的。

.. option:: --auth-uid=auid

   librados 认证所需的 auid 。

.. option:: --purge-data

   删除用户前先删除用户数据。

.. option:: --purge-keys

   若加了此选项，删除子用户时将一起删除其所有密钥。

.. option:: --purge-objects

   删除桶前先删除其内所有对象。

.. option:: --metadata-key=<key>

   用 ``metadata get`` 检索元数据时用的密钥。

.. option:: --rgw-region=<region>

   radosgw 所在的 region 。

.. option:: --rgw-zone=<zone>

   radosgw 所在的区域。

.. option:: --fix

   除了检查桶索引，还修复它。

.. option:: --check-objects

   检查桶：根据对象的实际状态重建桶索引。

.. option:: --format=<format>

   为某些操作指定输出格式： xml 、 json 。

.. option:: --sync-stats

   ``user stats`` 的选项，收集用户的桶索引状态、并同步到用户状态。

.. option:: --show-log-entries=<flag>

   执行 ``log show`` 时，显示或不显示日志条目。

.. option:: --show-log-sum=<flag>

   执行 ``log show`` 时，显示或不显示日志汇总。

.. option:: --skip-zero-entries

   让 ``log show`` 只显示数字字段非零的日志。

.. option:: --infile

   设置时指定要读取的文件。

.. option:: --state=<state string>

   给 ``opstate set`` 命令指定状态。

.. option:: --replica-log-type

   复制日志类型（ metadata 、 data 、 bucket ），操作复制日志时需要。

.. option:: --categories=<list>

   逗号分隔的一系列类目，显示使用情况时需要。

.. option:: --caps=<caps>

   能力列表，如 "usage=read, write; user=read" 。

.. option:: --yes-i-really-mean-it

   某些特定操作需要。


配额选项
========

.. option:: --bucket

   为配额命令指定桶。

.. option:: --max-objects

   指定最大对象数（负数为禁用）。

.. option:: --max-size

   指定最大尺寸（单位为字节，负数为禁用）。

.. option:: --quota-scope

   配额有效范围（桶、用户）。


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

        $ radosgw-admin bucket unlink --bucket=foo

显示一个桶从 2012 年 4 月 1 日起的日志： ::

        $ radosgw-admin log show --bucket=foo --date=2012-04-01

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
