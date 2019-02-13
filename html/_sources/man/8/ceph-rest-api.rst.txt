:orphan:

==============================================
 ceph-rest-api -- ceph 的 REST 风格管理服务器
==============================================

.. program:: ceph-rest-api

提纲
====

| **ceph-rest-api** [ -c *conffile* ] [--cluster *clustername* ] [ -n *name* ] [-i *id* ]


描述
====

**ceph-rest-api** 是一个 WSGI （网页服务器网关接口）应用程序，可作为网页服\
务独立运行，也可在支持 WSGI 的网页服务器下运行。它通过 HTTP 访问接口提供了 \
**ceph** 命令行工具的大多数功能。


选项
====

.. option:: -c/--conf conffile

   指定配置所在的 ``ceph.conf`` 配置文件。若未指定 ``-c`` 选项，默认将取决于 \
   ``--cluster`` 选项（默认为 'ceph' ，见下文）的状态。配置文件将按如下顺序\
   确定：

   * $CEPH_CONF
   * /etc/ceph/${cluster}.conf
   * ~/.ceph/${cluster}.conf
   * ${cluster}.conf (in the current directory)
  
   所以说，你也可以用 CEPH_CONF 环境变量来传递此选项。

.. option:: --cluster clustername

   为定位 ceph.conf 文件，把 $cluster 元变量设置为 *clustername* 。默认为 \
   'ceph' 。

.. option:: -n/--name name

   指定客户端名字 'name' ，用于从配置文件里查找与此客户端相关的配置选项，也\
   用于连接集群（例如此名字出现在集群的 ``ceph auth list`` 输出中）时的认证\
   过程。默认为 'client.restapi' 。

.. option:: -i/--id id

   指定客户端 'id' ，在没有设置客户端名字时它将扩展为 'client.<id>' 。若设置\
   了 ``-n/--name`` 则优先采用此值。

   Ceph 全局选项在这里有效。
 

配置参数
========

支持的配置参数有：

* **keyring** 保存着 'clientname' 密钥的密钥环文件
* **public addr** 监听的 ip:port （默认为 0.0.0.0:5000 ）
* **log file** 通常用 Ceph 默认值
* **restapi base url** 向请求回应的前导 URL ，默认为 /api/v0.1 。
* **restapi log level** critical, error, warning, info, debug （默认 warning ）

配置参数按标准顺序搜索：先是名为 '<clientname>' 的段落、然后 'client' 、然\
后 'global' 。

<clientname> 的来源可以是 -n/--name ，或 "client.<id>" （ <id> 由 -i/--id 提\
供），都没配置的话就是 'client.restapi' 。

如果直接执行 ceph-rest-api 的话，一个单线程服务器将监听 **public addr** ；否\
则此配置将由 WSGI 网页服务器指定。


命令
====

命令通过 HTTP GET 请求（对于主要返回数据的命令）、或 PUT （对于影响集群状态\
的命令）提交的命令； HEAD 和 OPTIONS 也支持。返回标准的 HTTP 状态码。

对于返回大量数据的命令，发出的请求可用 Accept: application/json 或 \
Accept: application/xml 来指定想要的格式化输出，或者在请求的 PATH 里加 .json \
或 .xml 后缀。请求中的参数将作为查询参数；对于需要多个值的参数可重复提供 \
key=val 结构。例如，要删除 OSD 2 和 3 ，可发送 PUT 请求到 \
``osd/rm?ids=2&ids=3`` 。


发现
====

当请求的路径不完整或只有部分匹配时，服务器会返回人类可读的命令列表及其支持的\
参数、和简短描述。请求 / 将重定向到 **restapi base url** 的值，并返回所有已\
知命令的完整列表。例如，请求 ``api/vX.X/mon`` 将返回监视器相关的 API 调用列\
表； ``api/vX.X/osd`` 将返回 OSD 相关的 API 调用列表。

命令集与 **ceph** 工具所支持的子命令非常相似，但有一个明显的例外就是 \
``ceph pg <pgid> <command>`` 命令在这里将变为 ``tell/<pgid>/command?args`` 。


部署为 WSGI 应用程序
====================

部署为 WSGI 应用程序（比如用 Apache/mod_wsgi 、 nginx/uwsgi 或 gunicorn 等）\
时，需要用 ``ceph_rest_api.py`` 模块， ``ceph-rest-api`` 只是此模块的瘦前\
端。这里没用单独的网页服务器，所以地址和端口要在 WSGI 服务器上配置。用一个 \
python 的 .wsgi 模块或者类似模块调用 \
``app = generate_app(conf, cluster, clientname, clientid, args)`` ，其中：

* conf 相当于上述的 -c/--conf
* cluster 相当于上述的 --cluster
* clientname 即是 -n/--name
* clientid 相当 -i/--id
* args 是其它的 Ceph 常规参数

app 返回时，它将带有 'ceph_addr' 和 'ceph_port' 属性，分别被设置成了 Ceph 配\
置中的地址和端口，服务器可使用这些信息，或忽略掉。

任何读取配置或连接集群时的错误将引发一个异常，遇到问题时查看相关消息的方法可\
参考 WSGI 服务器文档。


使用范围
========

**ceph-rest-api** 是 Ceph 的一部分，这是个伸缩力强、开源、分布式的存储系统，\
更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph <ceph>`\(8)
