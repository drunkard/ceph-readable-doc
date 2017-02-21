======================
 Rados 网关的数据布局
======================

虽说源代码才是终极手册，但本文档能让新开发者更快地了解实现细节。


简介
----

Swift 提供了名为容器（ container ）的东西，在这里它等价于桶\
（ bucket ）概念。也可以说 RGW 用桶实现了 Swift 的容器功能。

本文档没有考虑 RGW 如何操作这些数据结构，比如说，为了序列化怎样使用 \
encode() 和 decode() 方法、等等。


概念预览
--------

虽然 RADOS 只知道 pool 、对象（及其 xattr ）和 omap[1] ，但是从概念\
上 RGW 把它的数据组织成了三种不同的类型：元数据、桶索引和数据。

* 元数据

我们有三“部分”元数据： user 、 bucket 、和 bucket.instance 。你可以\
用下面的命令调查这几种元数据： ::

        radosgw-admin metadata list

        radosgw-admin metadata list bucket
        radosgw-admin metadata list bucket.instance
        radosgw-admin metadata list user

        radosgw-admin metadata get bucket:<bucket>
        radosgw-admin metadata get bucket.instance:<bucket>:<bucket_id>
        radosgw-admin metadata get user:<user>   # or set

user: 保存着用户信息
bucket: 保存着桶名与其例程 ID 的映射关系
bucket.instance: 保存着桶例程信息 [2]

每个元数据条目都存储于独立的 rados 对象。
实现细节见下文。

要注意，元数据没被索引过。所以，罗列元数据部分的时候，其实是在包含它\
们的存储池里做 rados pgls 操作。

* 桶索引

它是另一种元数据，也是独立保存的。桶索引是在 rados 对象里保存的键值\
映射，默认情况下，每个桶占用一个 rados 对象，但是从 Hammer 起，这些\
映射可以分片存入多个 rados 对象。这个映射本身是保存在 omap 里的，并\
与各 rados 对象关联。各 omap 的键名是对象名、值是此对象的基本元数据\
——罗列此桶时显示的就是元数据。另外，各 omap 都有一个头，保存着桶的\
一些统计元数据（如对象数量、总尺寸等等）。

另外，桶索引里还有其它信息，是保存在其它键名空间里的。比如，我们可\
以在这里存桶索引日志、而且对于版本化的对象，有更多信息要保存。

* 数据

各个 rgw 对象的数据保存在一或多个 rados 对象里。


对象定位路径
------------

访问对象时， ReST 风格的 API 要提供给 RGW 三个参数：帐户信息（ S3 \
的访问密钥或者 Swift 的帐户名）、桶或容器名、和对象名（或键名）。当\
前， RGW 只用帐户信息查找用户 ID 并用于访问控制，只用桶名和对象键名\
就能定位存储池内的对象。

在 RGW 里，用户 ID 是个字符串，通常是用户凭据里的真实用户名，而不是\
其哈希值或者映射过的标识符。

访问用户的数据时，用户记录要从 .users.uid 存储池里的 <user_id> 对象\
载入。

桶名即 .rgw 存储池内的对象名。要载入桶记录，以便获取所谓的记号（ \
marker ），它作为桶的唯一标识符。

对象数据位于 .rgw.buckets 存储池内。对象名是 "<marker>_<key>" ，如 \
default.7593.4_image.png ，其中 marker 是 default.7593.4 、键名是 \
image.png 。正因为这些组合起来的名字没被解析过，只是传递给了 \
RADOS ，所以分隔符的选择就没那么重要、不会引起歧义；正因为如此，斜\
杠也允许作为对象名（键名）。

也可以创建并使用多个数据存储池，这样不同的用户、桶就可以默认放到不\
同的 rados 存储池里了，以此提供了必要的伸缩性。这样的布局和这些存储\
池的命名是由 policy 选项 [3] 配置的。

一个 RGW 对象可由多个 RADOS 对象组成，其中的第一个对象是头儿，它包\
含着元数据，像载荷清单、 ACL 、内容类型、 ETag 、和用户自定义的元数\
据，这些元数据是存储在 xattr 里的。为保证高效性、原子性，这个头还可\
以存储最多 512 KB 的对象数据。载荷清单描述了各对象是如何存储在 \
RADOS 对象里的。


桶和对象列表
------------

某一用户的桶存在 .users.uid 存储池里、名为 <user_id>.buckets 的对象\
（如 foo.buckets ）的 omap 里。在罗列桶、更新桶内容、以及更新和检索\
桶的统计信息（例如配额信息）时要访问这些对象。

这些 omap 条目的值位于用户可见、编码过的 cls_user_bucket_entry 类和\
它的嵌套类 cls_user_bucket 里面。

这些列表以桶名持久化、存储在 .rgw 存储池里面。

某一桶内所包含的对象罗列在桶索引里，这已经在前面的“桶索引”小段里说\
过了。在 .rgw.buckets.index 存储池里、索引对象的默认命名规则是 \
.dir.<marker> 。


附注
----

[1] Omap 是个与对象相关联的键值存储，类似扩展属性与 POSIX 文件的关\
联一样。对象的 omap 与对象数据在物理上是分离的，但是对 RADOS 网关来\
说是不可见、无形的。在 Hammer 版里，每个 OSD 都有一个用于存储 omap \
的 LevelDB 数据库。

[2] 在 Dumpling 版之前还没有 bucket.instance 元数据，这些信息存储在\
\ bucket 元数据里。所以，在较老的集群里有可能碰到这样的桶。

[3] 在 Infernalis 版有个尚未被主线合并的提交，它实现了这样的功能，\
不必再以句点作为 rgw 系统存储池的前缀，并且重命名了所有默认存储池。\
详情见 Github 里的拉取请求 #4944 "rgw noperiod" 。


附录：简介
----------

已知存储池：

.rgw.root
  不确定的 region 、 zone 以及全局信息，每条使用一个对象。

.rgw.control
  notify.<N>

.rgw
  <bucket>
  .bucket.meta.<bucket>:<marker>

.rgw.gc
  gc.<N>

.users.uid
  包含两种信息，存储于 <user> 对象里的各个用户信息（RGWUserInfo）、\
  及其桶列表，以 omap 格式储存在名为 "<user>.buckets" 的对象里。

.users.email
  不重要

.users
  47UA98JSTJZ9YAN3OS3O
  不明白为什么这里没用用户 ID 作为对象名。

.users.swift
  test:tester

.rgw.buckets.index
  对象命名为 .dir.<marker> ，它们分别包含一个桶索引。

.rgw.buckets
  default.7593.4__shadow_.488urDFerTYXavx4yAd-Op8mxehnvTI_1
  <marker>_<key>

marker 长得像 "default.16004.1" 或者 "default.7593.4" 。
