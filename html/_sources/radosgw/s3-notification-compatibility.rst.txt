===================
 S3 桶通知的兼容性
===================
.. S3 Bucket Notifications Compatibility

Ceph 的 `桶通知`_ API 遵循 `AWS S3 桶通知 API`_ 。
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
然而，发出的 ``requestParameters.sourceIPAddress`` 字段会为空。

事件类型
--------
.. Event Types

+--------------------------------------------------------+-------------------------------------------+
| Event                                                  | Note                                      |
+========================================================+===========================================+
| ``s3:ObjectCreated:*``                                 | Supported                                 |
+--------------------------------------------------------+-------------------------------------------+
| ``s3:ObjectCreated:Put``                               | Supported                                 |
+--------------------------------------------------------+-------------------------------------------+
| ``s3:ObjectCreated:Post``                              | Supported                                 |
+--------------------------------------------------------+-------------------------------------------+
| ``s3:ObjectCreated:Copy``                              | Supported                                 |
+--------------------------------------------------------+-------------------------------------------+
| ``s3:ObjectCreated:CompleteMultipartUpload``           | Supported                                 |
+--------------------------------------------------------+-------------------------------------------+
| ``s3:ObjectRemoved:*``                                 | Supported                                 |
+--------------------------------------------------------+-------------------------------------------+
| ``s3:ObjectRemoved:Delete``                            | Supported                                 |
+--------------------------------------------------------+-------------------------------------------+
| ``s3:ObjectRemoved:DeleteMarkerCreated``               | Supported                                 |
+--------------------------------------------------------+-------------------------------------------+
| ``s3:ObjectLifecycle:Expiration:Current``              | Ceph extension                            |
+--------------------------------------------------------+-------------------------------------------+
| ``s3:ObjectLifecycle:Expiration:NonCurrent``           | Ceph extension                            |
+--------------------------------------------------------+-------------------------------------------+
| ``s3:ObjectLifecycle:Expiration:DeleteMarker``         | Ceph extension                            |
+--------------------------------------------------------+-------------------------------------------+
| ``s3:ObjectLifecycle:Expiration:AbortMultipartUpload`` | Ceph extension                            |
+--------------------------------------------------------+-------------------------------------------+
| ``s3:ObjectLifecycle:Transition:Current``              | Ceph extension                            |
+--------------------------------------------------------+-------------------------------------------+
| ``s3:ObjectLifecycle:Transition:NonCurrent``           | Ceph extension                            |
+--------------------------------------------------------+-------------------------------------------+
| ``s3:LifecycleExpiration:*``                           | Supported. Equivalent to                  |
|                                                        | s3:LifecycleExpiration:Delete,            |
|                                                        | s3:LifecycleExpiration:DeleteMarkerCreated|
+--------------------------------------------------------+-------------------------------------------+
| ``s3:LifecycleExpiration:Delete``                      | Supported. Equivalent to                  |
|                                                        | s3:ObjectLifecycle:Expiration:Current     |
+--------------------------------------------------------+-------------------------------------------+
| ``s3:LifecycleExpiration:DeleteMarkerCreated``         | Supported. Equivalent to                  |
|                                                        | s3:ObjectLifecycle:Expiration:DeleteMarker|
+--------------------------------------------------------+-------------------------------------------+
| ``s3:LifecycleTransition``                             | Supported. Equivalent to                  |
|                                                        | s3:ObjectLifecycle:Transition:Current     |
+--------------------------------------------------------+-------------------------------------------+
| ``s3:ObjectSynced:*``                                  | Ceph extension                            |
+--------------------------------------------------------+-------------------------------------------+
| ``s3:ObjectSynced:Create``                             | Ceph Extension                            |
+--------------------------------------------------------+-------------------------------------------+
| ``s3:ObjectSynced:Delete``                             | Ceph extension                            |
+--------------------------------------------------------+-------------------------------------------+
| ``s3:ObjectSynced:DeletionMarkerCreated``              | Defined, Ceph extension (not generated)   |
+--------------------------------------------------------+-------------------------------------------+
| ``s3:Replication:*``                                   | Supported. Equivalent to                  |
|                                                        | s3:ObjectSynced:Create,                   |
|                                                        | s3:ObjectSynced:Delete                    |
+--------------------------------------------------------+-------------------------------------------+
| ``s3:Replication:Create``                              | Supported. Equivalent to                  |
|                                                        | s3:ObjectSynced:Create                    |
+--------------------------------------------------------+-------------------------------------------+
| ``s3:Replication:Delete``                              | Supported. Equivalent to                  |
|                                                        | s3:ObjectSynced:Delete                    |
+--------------------------------------------------------+-------------------------------------------+
| ``s3:Replication:DeletionMarkerCreated``               | Defined, Supported (not generated)        |
+--------------------------------------------------------+-------------------------------------------+
| ``s3:ObjectRestore:Post``                              | Not applicable                            |
+--------------------------------------------------------+-------------------------------------------+
| ``s3:ObjectRestore:Complete``                          | Not applicable                            |
+--------------------------------------------------------+-------------------------------------------+
| ``s3:ReducedRedundancyLostObject``                     | Not applicable                            |
+--------------------------------------------------------+-------------------------------------------+

.. note:: 

   The ``s3:ObjectRemoved:DeleteMarkerCreated`` event presents information on the latest version of the object

.. note::

   In case of multipart upload, an ``ObjectCreated:CompleteMultipartUpload`` notification will be sent at the end of the process.

.. note::

   The ``s3:ObjectSynced:Create`` event is sent when an object successfully syncs to a zone. It must be explicitly set for each zone. 

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
.. _`桶通知`: ../notifications
.. _`boto3 SDK filter extensions`: https://github.com/ceph/ceph/tree/main/examples/boto3
