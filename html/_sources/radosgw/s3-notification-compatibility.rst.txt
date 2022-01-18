===================
 S3 桶通知的兼容性
===================
.. S3 Bucket Notifications Compatibility

Ceph 的 `桶通知`_ 和 `PubSub 模块`_ API 遵循 `AWS S3 桶通知 API`_ 。
然而还是有些不同，详述如下。

.. note::
   取决于采用的前述机制，兼容性的表现不一样。


支持的目的地
------------
.. Supported Destination

AWS supports: **SNS**, **SQS** and **Lambda** as possible destinations (AWS internal destinations). 
Currently, we support: **HTTP/S**, **Kafka** and **AMQP**. And also support pulling and acking of events stored in Ceph (as an internal destination).

We are using the **SNS** ARNs to represent the **HTTP/S**, **Kafka** and **AMQP** destinations.

通知的配置 XML
--------------
.. Notification Configuration XML

不支持下列标签（包括它们里面的标签）：

+-----------------------------------+----------------------------------------------+
| Tag                               | Remaks                                       |
+===================================+==============================================+
| ``<QueueConfiguration>``          | not needed, we treat all destinations as SNS |
+-----------------------------------+----------------------------------------------+
| ``<CloudFunctionConfiguration>``  | not needed, we treat all destinations as SNS |
+-----------------------------------+----------------------------------------------+

REST API 扩展
-------------
.. REST API Extension 

Ceph's bucket notification API has the following extensions:

- Deletion of a specific notification, or all notifications on a bucket, using the ``DELETE`` verb

 - In S3, all notifications are deleted when the bucket is deleted, or when an empty notification is set on the bucket

- Getting the information on a specific notification (when more than one exists on a bucket)

  - In S3, it is only possible to fetch all notifications on a bucket

- In addition to filtering based on prefix/suffix of object keys we support:

  - Filtering based on regular expression matching

  - Filtering based on metadata attributes attached to the object

  - Filtering based on object tags

- Each one of the additional filters extends the S3 API and using it will require extension of the client SDK (unless you are using plain HTTP). 

- Filtering overlapping is allowed, so that same event could be sent as different notification


事件记录里不支持的字段
----------------------
.. Unsupported Fields in the Event Record

为桶通知发出的记录遵循 `事件消息数据结构`_ 里描述的格式。
然而，在不同的部署选项下（ Notification/PubSub ），下列的字段可能会为空：

+----------------------------------------+--------------+---------------+------------------------------------------------------------+
| Field                                  | Notification | PubSub        | Description                                                |
+========================================+==============+===============+============================================================+
| ``userIdentity.principalId``           | Supported    | Not Supported | The identity of the user that triggered the event          |
+----------------------------------------+--------------+---------------+------------------------------------------------------------+
| ``requestParameters.sourceIPAddress``  |         Not Supported        | The IP address of the client that triggered the event      |
+----------------------------------------+--------------+---------------+------------------------------------------------------------+
| ``requestParameters.x-amz-request-id`` | Supported    | Not Supported | The request id that triggered the event                    |
+----------------------------------------+--------------+---------------+------------------------------------------------------------+
| ``requestParameters.x-amz-id-2``       | Supported    | Not Supported | The IP address of the RGW on which the event was triggered |
+----------------------------------------+--------------+---------------+------------------------------------------------------------+
| ``s3.object.size``                     | Supported    | Not Supported | The size of the object                                     |
+----------------------------------------+--------------+---------------+------------------------------------------------------------+

事件类型
--------
.. Event Types

+------------------------------------------------+-----------------+-------------------------------------------+
| Event                                          | Notification    | PubSub                                    |
+================================================+=================+===========================================+
| ``s3:ObjectCreated:*``                         | Supported                                                   |
+------------------------------------------------+-----------------+-------------------------------------------+
| ``s3:ObjectCreated:Put``                       | Supported       | Supported at ``s3:ObjectCreated:*`` level |
+------------------------------------------------+-----------------+-------------------------------------------+
| ``s3:ObjectCreated:Post``                      | Supported       | Not Supported                             |
+------------------------------------------------+-----------------+-------------------------------------------+
| ``s3:ObjectCreated:Copy``                      | Supported       | Supported at ``s3:ObjectCreated:*`` level |
+------------------------------------------------+-----------------+-------------------------------------------+
| ``s3:ObjectCreated:CompleteMultipartUpload``   | Supported       | Supported at ``s3:ObjectCreated:*`` level |
+------------------------------------------------+-----------------+-------------------------------------------+
| ``s3:ObjectRemoved:*``                         | Supported       | Supported only the specific events below  |
+------------------------------------------------+-----------------+-------------------------------------------+
| ``s3:ObjectRemoved:Delete``                    | Supported                                                   |
+------------------------------------------------+-----------------+-------------------------------------------+
| ``s3:ObjectRemoved:DeleteMarkerCreated``       | Supported                                                   |
+------------------------------------------------+-----------------+-------------------------------------------+
| ``s3:ObjectLifecycle:Expiration:Current``      | Supported, Ceph extension                                   |
+------------------------------------------------+-----------------+-------------------------------------------+
| ``s3:ObjectLifecycle:Expiration:NonCurrent``   | Supported, Ceph extension                                   |
+------------------------------------------------+-----------------+-------------------------------------------+
| ``s3:ObjectLifecycle:Expiration:DeleteMarker`` | Supported, Ceph extension                                   |
+------------------------------------------------+-----------------+-------------------------------------------+
| ``s3:ObjectLifecycle:Expiration:AbortMultipartUpload`` | Defined, Ceph extension (not generated)             |
+------------------------------------------------+-----------------+-------------------------------------------+
| ``s3:ObjectLifecycle:Transition:Current``      | Supported, Ceph extension                                   |
+------------------------------------------------+-----------------+-------------------------------------------+
| ``s3:ObjectLifecycle:Transition:NonCurrent``   | Supported, Ceph extension                                   |
+------------------------------------------------+-----------------+-------------------------------------------+
| ``s3:ObjectRestore:Post``                      | Not applicable to Ceph                                      |
+------------------------------------------------+-----------------+-------------------------------------------+
| ``s3:ObjectRestore:Complete``                  | Not applicable to Ceph                                      |
+------------------------------------------------+-----------------+-------------------------------------------+
| ``s3:ReducedRedundancyLostObject``             | Not applicable to Ceph                                      |
+----------------------------------------------+-----------------+---------------------------------------------+

.. note:: 

   The ``s3:ObjectRemoved:DeleteMarkerCreated`` event presents information on the latest version of the object

.. note::

   In case of multipart upload, an ``ObjectCreated:CompleteMultipartUpload`` notification will be sent at the end of the process.

Topic Configuration
-------------------
In the case of bucket notifications, the topics management API will be derived from `AWS 简单通知服务 API`_. 
Note that most of the API is not applicable to Ceph, and only the following actions are implemented:

 - ``CreateTopic``
 - ``DeleteTopic``
 - ``ListTopics``

We also have the following extensions to topic configuration: 

 - In ``GetTopic`` we allow fetching a specific topic, instead of all user topics
 - In ``CreateTopic``

  - we allow setting endpoint attributes
  - we allow setting opaque data that will be sent to the endpoint in the notification


.. _AWS 简单通知服务 API: https://docs.aws.amazon.com/sns/latest/api/API_Operations.html
.. _AWS S3 桶通知 API: https://docs.aws.amazon.com/AmazonS3/latest/dev/NotificationHowTo.html
.. _事件消息数据结构: https://docs.aws.amazon.com/AmazonS3/latest/dev/notification-content-structure.html
.. _`PubSub 模块`: ../pubsub-module
.. _`桶通知`: ../notifications
.. _`boto3 SDK filter extensions`: https://github.com/ceph/ceph/tree/master/examples/boto3
