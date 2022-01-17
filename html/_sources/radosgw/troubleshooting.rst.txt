==========
 故障排除
==========


网关启不起来
============
.. The Gateway Won't Start

如果你启动不了网关（就是没有 ``pid`` ），检查下有没有 ``.asok`` 文件、
它是不是属于另外一个用户。如果有 ``.asok`` 文件、但属于另外一个用户，
而且没有在运行的 ``pid`` ，删掉 ``.asok`` 文件并重启一下这个进程。
这种情况发生在你以 ``root`` 用户启动了进程、
而启动脚本尝试以 ``www-data`` 或 ``apache`` 用户启动、
而且已经存在的 ``.asok`` 妨碍了脚本启动这个进程。

radosgw 初始化脚本（ /etc/init.d/radosgw ）也有一个详情参数，
可以让你深入了解问题所在： ::

  /etc/init.d/radosgw start -v

或者： ::

  /etc/init.d radosgw start --verbose


HTTP 请求错误
=============
.. HTTP Request Errors

检查 web 服务器自身的访问日志和错误日志可能是找出问题的第一步。
如果有 500 错误，通常说明和 ``radosgw`` 守护进程的通讯有问题。
确保这个守护进程在运行，它的套接字路径配置了，
而且 web 服务器去正确的位置找它了。


``radosgw`` 进程崩溃
====================
.. Crashed ``radosgw`` process

如果 ``radosgw`` 进程挂了，一般会看到网页服务器（ apache 、 nginx \
等）返回 500 错误，这种情况下，只要重启 radosgw 就能恢复服务。

要调查崩溃原因，请检查 ``/var/log/ceph`` 里的日志、以及核心转储\
文件（如果生成了）。


阻塞的 ``radosgw`` 请求
=======================
.. Blocked ``radosgw`` Requests

如果某些（或者所有） radosgw 请求被阻塞了，你可以通过管理\
套接字深入了解 ``radosgw`` 守护进程的内部状态。默认情况下，\
套接字文件位于 ``/var/run/ceph`` ，可以这样查询： ::

    ceph daemon /var/run/ceph/client.rgw help

    help                list available commands
    objecter_requests   show in-progress osd requests
    perfcounters_dump   dump perfcounters value
    perfcounters_schema dump perfcounters schema
    version             get protocol version

某个特定请求： ::

    ceph daemon /var/run/ceph/client.rgw objecter_requests
    ...

它会显示当前正在进行的、发往 RADOS 集群的请求。用这个功能你就\
能确定是否有请求因 OSD 不响应而被阻塞，比如，你也许能看到： ::

  { "ops": [
        { "tid": 1858,
          "pg": "2.d2041a48",
          "osd": 1,
          "last_sent": "2012-03-08 14:56:37.949872",
          "attempts": 1,
          "object_id": "fatty_25647_object1857",
          "object_locator": "@2",
          "snapid": "head",
          "snap_context": "0=[]",
          "mtime": "2012-03-08 14:56:37.949813",
          "osd_ops": [
                "write 0~4096"]},
        { "tid": 1873,
          "pg": "2.695e9f8e",
          "osd": 1,
          "last_sent": "2012-03-08 14:56:37.970615",
          "attempts": 1,
          "object_id": "fatty_25647_object1872",
          "object_locator": "@2",
          "snapid": "head",
          "snap_context": "0=[]",
          "mtime": "2012-03-08 14:56:37.970555",
          "osd_ops": [
                "write 0~4096"]}],
  "linger_ops": [],
  "pool_ops": [],
  "pool_stat_ops": [],
  "statfs_ops": []}

在上面的显示中，有两个请求正在进行。 ``last_sent`` 字段是 RADOS \
请求发出的时间，如果以及有一段时间了，说明那个 OSD 没响应。例如，\
对于 1858 这个请求，你可以这样检查 OSD 状态： ::

    ceph pg map 2.d2041a48

    osdmap e9 pg 2.d2041a48 (2.0) -> up [1,0] acting [1,0]

这说明，我们得查看 ``osd.1`` ，它是这个 PG 的主副本： ::

 ceph daemon osd.1 ops
 { "num_ops": 651,
  "ops": [
        { "description": "osd_op(client.4124.0:1858 fatty_25647_object1857 [write 0~4096] 2.d2041a48)",
          "received_at": "1331247573.344650",
          "age": "25.606449",
          "flag_point": "waiting for sub ops",
          "client_info": { "client": "client.4124",
              "tid": 1858}},
 ...

``flag_point`` 字段表明这个 OSD 正在等待副本（本例中是 ``osd.0`` ）\
的响应。


Java S3 API 故障排除
====================
.. Java S3 API Troubleshooting

互联点没认证
------------
.. Peer Not Authenticated

你可能会遇到类似这样的错误： ::

    [java] INFO: Unable to execute HTTP request: peer not authenticated

S3 的 Java SDK 需要一个认可的证书机构颁发的合法证书，因为它默认使用 HTTPS 。
如果你只是想测试一下 Ceph 对象存储服务，可以用这几种方法解决这个问题：

#. 在 IP 地址或主机名前加 ``http://`` 。例如，改这行： ::

    conn.setEndpoint("myserver");

   改成:: 

    conn.setEndpoint("http://myserver")

#. 配置好凭证后，加上客户端配置、并把协议设置成 
   ``Protocol.HTTP`` 。 ::

        AWSCredentials credentials = new BasicAWSCredentials(accessKey, secretKey);

        ClientConfiguration clientConfig = new ClientConfiguration();
        clientConfig.setProtocol(Protocol.HTTP);

        AmazonS3 conn = new AmazonS3Client(credentials, clientConfig);


405 MethodNotAllowed
--------------------

如果你遇到了 405 错误，检查一下你是否配置好了 S3 子域。
你得在子域的 DNS 记录里配置通配符，这样它才能正常工作。

还有，要确保禁用了默认站点。 ::

     [java] Exception in thread "main" Status Code: 405, AWS Service: Amazon S3, AWS Request ID: null, AWS Error Code: MethodNotAllowed, AWS Error Message: null, S3 Extended Request ID: null


default.rgw.meta 存储池里的几个对象
===================================
.. Numerous objects in default.rgw.meta pool

在 *jewel* 之前创建的集群上有一个元数据存档功能是默认启用的，
用的是 ``default.rgw.meta`` 存储池。这个存档保留着用户和桶元数据的所有老版本，
导致 ``default.rgw.meta`` 存储池里产生了大量的对象。

禁用 Metadata Heap 字段
-----------------------
.. Disabling the Metadata Heap

想要禁用此功能的用户可以把 ``metadata_heap`` 字段设置为空字符串 ``""``::

  $ radosgw-admin zone get --rgw-zone=default > zone.json
  [edit zone.json, setting "metadata_heap": ""]
  $ radosgw-admin zone set --rgw-zone=default --infile=zone.json
  $ radosgw-admin period update --commit

这样新的元数据就不会再写入 ``default.rgw.meta`` 存储池了，
但是所有现有的对象或存储池都不会删除。

清理 Metadata Heap 存储池
-------------------------
.. Cleaning the Metadata Heap Pool

在 *jewel* 之前创建的集群的 ``default.rgw.meta`` 存储池通常只用作元数据归档。

而从 *luminous* 起， radosgw 利用 ``default.rgw.meta`` 存储池的 :ref:`存储池命名空间 <radosgw-pool-namespaces>` 的目的完全变了，就是说，用来存储 ``user_keys`` 和其它关键元数据了。

用户们在清理之前应该核对一下域配置信息： ::

  $ radosgw-admin zone get --rgw-zone=default | grep default.rgw.meta
  [should not match any strings]

确认过这个存储池确实没在用，用户就可以安全删除
``default.rgw.meta`` 存储池里的所有对象了，或者干脆删除整个存储池。
