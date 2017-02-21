==========
 故障排除
==========


网关启不起来
============

If you cannot start the gateway (i.e., there is no existing ``pid``), 
check to see if there is an existing ``.asok`` file from another 
user. If an ``.asok`` file from another user exists and there is no
running ``pid``, remove the ``.asok`` file and try to start the
process again.

This may occur when you start the process as a ``root`` user and 
the startup script is trying to start the process as a 
``www-data`` or ``apache`` user and an existing ``.asok`` is 
preventing the script from starting the daemon.

The radosgw init script (/etc/init.d/radosgw) also has a verbose argument that
can provide some insight as to what could be the issue:

  /etc/init.d/radosgw start -v

or

  /etc/init.d radosgw start --verbose


HTTP 请求错误
=============

Examining the access and error logs for the web server itself is
probably the first step in identifying what is going on.  If there is
a 500 error, that usually indicates a problem communicating with the
``radosgw`` daemon.  Ensure the daemon is running, its socket path is
configured, and that the web server is looking for it in the proper
location.


``radosgw`` 崩溃
================

如果 ``radosgw`` 进程挂了，一般会看到网页服务器（ apache 、 nginx \
等）返回 500 错误，这种情况下，只要重启 radosgw 就能恢复服务。

要调查崩溃原因，请检查 ``/var/log/ceph`` 里的日志、以及核心转储\
文件（如果生成了）。


阻塞的 ``radosgw`` 请求
=======================

如果某些（或者所有） radosgw 请求被阻塞了，你可以通过管理套接字深\
入了解 ``radosgw`` 守护进程的内部状态。默认情况下，套接字文件位于\
\ ``/var/run/ceph`` ，可以这样查询： ::

	ceph daemon /var/run/ceph/client.rgw help

	help                list available commands
	objecter_requests   show in-progress osd requests
	perfcounters_dump   dump perfcounters value
	perfcounters_schema dump perfcounters schema
	version             get protocol version

某个特定请求： ::

	ceph daemon /var/run/ceph/client.rgw objecter_requests
	...

它会显示当前正在进行的、发往 RADOS 集群的请求。用这个功能你就能\
确定是否有请求因 OSD 不响应而被阻塞，比如，你也许能看到： ::

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
	"statfs_ops": []
	}

在上面的显示中，有两个请求正在进行。 ``last_sent`` 字段是 RADOS \
请求发出的时间，如果以及有一段时间了，说明那个 OSD 没响应。例如，\
对于 1858 这个请求，你可以这样检查 OSD 状态： ::

	ceph pg map 2.d2041a48

	osdmap e9 pg 2.d2041a48 (2.0) -> up [1,0] acting [1,0]

这说明，我们得查看 ``osd.1`` ，它是这个 PG 的主副本： ::

	ceph daemon osd.1 ops
	{
	    "num_ops": 651,
	    "ops": [
	        {
		 "description": "osd_op(client.4124.0:1858 fatty_25647_object1857 [write 0~4096] 2.d2041a48)",
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


Peer Not Authenticated
----------------------

You may receive an error that looks like this:: 

     [java] INFO: Unable to execute HTTP request: peer not authenticated

The Java SDK for S3 requires a valid certificate from a recognized certificate
authority, because it uses HTTPS by default. If you are just testing the Ceph
Object Storage services, you can resolve this problem in a few ways:  

#. Prepend the IP address or hostname with ``http://``. For example, change this::

	conn.setEndpoint("myserver");

   To:: 

	conn.setEndpoint("http://myserver")

#. After setting your credentials, add a client configuration and set the 
   protocol to ``Protocol.HTTP``. :: 

			AWSCredentials credentials = new BasicAWSCredentials(accessKey, secretKey);

			ClientConfiguration clientConfig = new ClientConfiguration();
			clientConfig.setProtocol(Protocol.HTTP);

			AmazonS3 conn = new AmazonS3Client(credentials, clientConfig);



405 MethodNotAllowed
--------------------

If you receive an 405 error, check to see if you have the S3 subdomain set up correctly. 
You will need to have a wild card setting in your DNS record for subdomain functionality
to work properly.

Also, check to ensure that the default site is disabled. ::

     [java] Exception in thread "main" Status Code: 405, AWS Service: Amazon S3, AWS Request ID: null, AWS Error Code: MethodNotAllowed, AWS Error Message: null, S3 Extended Request ID: null
  
  
  
