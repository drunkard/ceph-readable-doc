================================================
 radosgw-admin -- rados REST 网关的用户管理工具
================================================

.. program:: radosgw-admin

提纲
====

| **radosgw-admin** *command* [ *options* *...* ]


描述
====

**radosgw-admin** 是 RADOS 网关用户管理工具，可用于创建和修改用户。


命令
====

*command* 是下列选项之一：

:command:`user create`
  创建一个新用户

:command:`user modify`
  修改一个用户

:command:`user info`
  显示用户信息，以及可能存在的子用户和密钥

:command:`user rm`
  删除一个用户

:command:`subuser create`
  新建一个子用户（适合使用 Swift API 的客户端）

:command:`subuser modify`
  修改子用户

:command:`subuser rm`
  删除子用户

:command:`bucket list`
  罗列所有桶

:command:`bucket unlink`
  删除一个桶

:command:`bucket rm`
  删除一个桶

:command:`object rm`
  删除一个对象

:command:`key create`
  创建一个访问密钥

:command:`key rm`
  删除一个访问密钥

:command:`pool add`
  增加一个已有存储池用于数据归置

:command:`pool rm`
  从数据归置集删除一个已有存储池

:command:`pools list`
  罗列归置活跃集

:command:`policy`
  显示桶或对象相关的策略

:command:`log show`
  查看一个桶的日志（可指定日期）

:command:`usage show`
  查看使用率信息（可选选项有用户和数据范围）

:command:`usage trim`
  修剪使用率信息（可选选项有用户和数据范围）


选项
====

.. option:: -c ceph.conf, --conf=ceph.conf

   用指定的 *ceph.conf* 配置文件而非默认的 ``/etc/ceph/ceph.conf`` 来确定启\
   动时所需的监视器地址。

.. option:: -m monaddress[:port]

   连接到指定监视器，而非通过 ceph.conf 查询。

.. option:: --uid=uid

   radosgw 用户的 ID 。

.. option:: --secret=secret

   指定密钥的密文

.. option:: --display-name=name

   配置用户的显示名称（昵称）

.. option:: --email=email

   用户的电子邮件地址

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

.. option:: --auth-uid=auid

   librados 认证所需的 auid

.. option:: --purge-data

   删除用户前先删除用户数据
   
.. option:: --purge-objects

   删除桶前先删除其内所有对象

.. option:: --lazy-remove

   推迟对象尾部的删除
   

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

**radosgw-admin** 是 Ceph 的一部分，这是个伸缩力强、开源、分布式的存储系统，\
更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph <ceph>`\(8)
