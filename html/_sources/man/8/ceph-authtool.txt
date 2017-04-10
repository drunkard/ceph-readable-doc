:orphan:

======================================
 ceph-authtool -- ceph 密钥环操作工具
======================================

.. program:: ceph-authtool

提纲
====

| **ceph-authtool** *keyringfile*
  [ -l | --list ]
  [ -p | --print-key ]
  [ -C | --create-keyring ]
  [ -g | --gen-key ]
  [ --gen-print-key ]
  [ --import-keyring *otherkeyringfile* ]
  [ -n | --name *entityname* ]
  [ -u | --set-uid *auid* ]
  [ -a | --add-key *base64_key* ]
  [ --cap *subsystem* *capability* ]
  [ --caps *capfile* ]


描述
====

**ceph-authtool** 工具用于创建、查看和修改 Ceph 密钥环文件。\
密钥环文件内存储着一或多个 Ceph 认证密钥、可能还有被授予的能\
力。每个密钥都与其类型名关联，格式为
``{client,mon,mds,osd}.name`` 。

**警告：** 在私钥保护得当的前提下， Ceph 能提供认证和防中间\
人攻击的保护。然而，传输中的数据是未加密的，这些数据中可能就\
有配置密钥的消息。此系统意在运行于可信环境中。


选项
====

.. option:: -l, --list

   列出密钥环内的所有密钥及其能力

.. option:: -p, --print-key

   打印指定条目的已编码密钥，它适合作为 ``mount -o secret=``
   的参数

.. option:: -C, --create-keyring

   创建新密钥环，覆盖已有密钥环文件

.. option:: -g, --gen-key

   为指定实体名生成新私钥

.. option:: --gen-print-key

   为指定条目（ entityname ）生成新密钥，不会修改密钥环文件（
   keyringfile ），只打印到标准输出。

.. option:: --import-keyring *secondkeyringfile*

   把此选项所指定密钥环的内容导入密钥环（ keyringfile ）

.. option:: -n, --name *name*

   指定要操作的条目名（ entityname ）

.. option:: -u, --set-uid *auid*

   设置指定条目（ entityname ）的 auid （已认证用户的标识符）

.. option:: -a, --add-key *base64_key*

   把编码好的密钥加进密钥环

.. option:: --cap *subsystem* *capability*

   设置指定子系统的能力

.. option:: --caps *capsfile*

   在所有子系统内设置与给定密钥相关的所有能力


能力
====

subsystem 代表 Ceph 子系统的名字： ``mon`` 、 ``mds`` 、或
``osd`` 。

能力是一个字符串，描述了允许此用户干什么。格式为逗号分隔的允\
许声明列表，此声明包含一或多个 rwx （分别表示读、写、执行权\
限）。 ``allow *`` 将在指定子系统下授予完整的超级用户权限。

例如： ::

        # 可读、写、执行对象
        osd = "allow rwx"

        # 可访问 MDS 服务器
        mds = "allow"

        # 可更改集群状态（即它是服务器守护进程）
        mon = "allow rwx"

被限定到单个存储池的 librados 用户的能力大致如此： ::

        mon = "allow r"

        osd = "allow rw pool foo"

一个 RBD 客户端有一个存储池的读权限和另一个存储池的读写权限： ::

        mon = "allow r"

        osd = "allow class-read object_prefix rbd_children, allow pool templates r class-read, allow pool vms rwx"

权限最小化的文件系统客户端，其能力大致如此： ::

        mds = "allow"

        osd = "allow rw pool data"

        mon = "allow r"


OSD 能力
========

一般来说， OSD 能力遵循以下语法： ::

        osdcap  := grant[,grant...]
        grant   := allow (match capspec | capspec match)
        match   := [pool[=]<poolname> | object_prefix <prefix>]
        capspec := * | [r][w][x] [class-read] [class-write]

capspec 决定了此实体可执行哪些操作： ::

    r           = 可读取对象
    w           = 可写入对象
    x           = 可调用任何类方法（等同于 class-read 、 class-write ）
    class-read  = 可调用读数据的类方法
    class-write = 可调用写数据的类方法
    *           = 等价于 rwx ，另外还可运行 OSD 管理命令，即 ceph osd tell ...

匹配规则限制了授权是基于被访问存储池的，客户端满足匹配条件时授权会叠加。例\
如，假设客户端的 OSD 能力为： \
"allow r object_prefix prefix, allow w pool foo, allow x pool bar" ，那么它\
有 foo 存储池的读写权限（ rw ）、有 bar 存储池的读和执行权限（ rx ）、还有\
任意存储池中以 prefix 打头的对象的读（ r ）权限。


能力文件的格式
==============

能力配置文件是格式化的零或多个键值对，每条一行。键和值以 ``=`` 分隔，且值内\
包含空格时必须用 ``'`` 或 ``"`` 包起来。键是某个 Ceph 子系统（ ``osd`` 、 \
``mds`` 、 ``mon`` ），值是能力字符串（见上文）。


实例
====

给 client.foo 生成密钥并新建密钥环： ::

        ceph-authtool -C -n client.foo --gen-key keyring

给此密钥关联一些能力（也就是挂载 Ceph 文件系统的能力）： ::

        ceph-authtool -n client.foo --cap mds 'allow' --cap osd 'allow rw pool=data' --cap mon 'allow r' keyring

查看密钥环内容： ::

        ceph-authtool -l keyring

挂载 Ceph 文件系统时，你可以用此命令获取编码好的私钥： ::

        mount -t ceph serverhost:/ mountpoint -o name=foo,secret=`ceph-authtool -p -n client.foo keyring`


使用范围
========

:program:`ceph-authtool` 是 Ceph 的一部分，这是个伸缩力强、开源、分布式的\
存储系统，更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph <ceph>`\(8)
