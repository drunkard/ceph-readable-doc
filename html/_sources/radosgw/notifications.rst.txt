====================
Bucket Notifications
====================

.. versionadded:: Nautilus

.. contents::

Bucket notifications provide a mechanism for sending information out of the radosgw when certain events are happening on the bucket.
Currently, notifications could be sent to: HTTP, AMQP0.9.1 and Kafka endpoints.

Note, that if the events should be stored in Ceph, in addition, or instead of being pushed to an endpoint,
the `PubSub Module`_ should be used instead of the bucket notification mechanism.

A user can create different topics. A topic entity is defined by its name and is per tenant. A
user can only associate its topics (via notification configuration) with buckets it owns.

In order to send notifications for events for a specific bucket, a notification entity needs to be created. A
notification can be created on a subset of event types, or for all event types (default).
The notification may also filter out events based on prefix/suffix and/or regular expression matching of the keys. As well as,
on the metadata attributes attached to the object, or the object tags.
There can be multiple notifications for any specific topic, and the same topic could be used for multiple notifications.

REST API has been defined to provide configuration and control interfaces for the bucket notification
mechanism. This API is similar to the one defined as the S3-compatible API of the pubsub sync module.

.. toctree::
   :maxdepth: 1

   S3 Bucket Notification Compatibility <s3-notification-compatibility>

.. note:: To enable bucket notifications API, the `rgw_enable_apis` configuration parameter should contain: "notifications".

Notification Reliability
------------------------

Notifications may be sent synchronously, as part of the operation that triggered them.
In this mode, the operation is acked only after the notification is sent to the topic's configured endpoint, which means that the
round trip time of the notification is added to the latency of the operation itself.

.. note:: The original triggering operation will still be considered as successful even if the notification fail with an error, cannot be deliverd or times out

Notifications may also be sent asynchronously. They will be committed into persistent storage and then asynchronously sent to the topic's configured endpoint.
In this case, the only latency added to the original operation is of committing the notification to persistent storage.

.. note:: If the notification fail with an error, cannot be deliverd or times out, it will be retried until successfully acked

.. tip:: To minimize the added latency in case of asynchronous notifications, it is recommended to place the "log" pool on fast media


Topic Management via CLI
------------------------

Configuration of all topics, associated with a tenant, could be fetched using the following command:

::

   # radosgw-admin topic list [--tenant={tenant}]


Configuration of a specific topic could be fetched using:

::

   # radosgw-admin topic get --topic={topic-name} [--tenant={tenant}]


And removed using:

::

   # radosgw-admin topic rm --topic={topic-name} [--tenant={tenant}]


Notification Performance Stats
------------------------------
The same counters are shared between the pubsub sync module and the bucket notification mechanism.

- ``pubsub_event_triggered``: running counter of events with at least one topic associated with them
- ``pubsub_event_lost``: running counter of events that had topics associated with them but that were not pushed to any of the endpoints
- ``pubsub_push_ok``: running counter, for all notifications, of events successfully pushed to their endpoint
- ``pubsub_push_fail``: running counter, for all notifications, of events failed to be pushed to their endpoint
- ``pubsub_push_pending``: gauge value of events pushed to an endpoint but not acked or nacked yet

.. note::

    ``pubsub_event_triggered`` and ``pubsub_event_lost`` are incremented per event, while:
    ``pubsub_push_ok``, ``pubsub_push_fail``, are incremented per push action on each notification

Bucket Notification REST API
----------------------------

Topics
~~~~~~

.. note::

    In all topic actions, the parameters are URL-encoded and sent in the
    message body using this content type:
    ``application/x-www-form-urlencoded``.
   

.. _Create a Topic:

Create a Topic
``````````````

This creates a new topic. Provide the topic with push endpoint parameters,
which will be used later when a notification is created. A response is
generated. A successful response includes the topic's `ARN
<https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html>`_
(the "Amazon Resource Name", a unique identifier used to reference the topic).
To update a topic, use the same command that you used to create it (but when
updating, use the name of an existing topic and different endpoint values).

.. tip:: Any notification already associated with the topic must be re-created
   in order for the topic to update.

::

   POST

   Action=CreateTopic
   &Name=<topic-name>
   [&Attributes.entry.1.key=amqp-exchange&Attributes.entry.1.value=<exchange>]
   [&Attributes.entry.2.key=amqp-ack-level&Attributes.entry.2.value=none|broker|routable]
   [&Attributes.entry.3.key=verify-ssl&Attributes.entry.3.value=true|false]
   [&Attributes.entry.4.key=kafka-ack-level&Attributes.entry.4.value=none|broker]
   [&Attributes.entry.5.key=use-ssl&Attributes.entry.5.value=true|false]
   [&Attributes.entry.6.key=ca-location&Attributes.entry.6.value=<file path>]
   [&Attributes.entry.7.key=OpaqueData&Attributes.entry.7.value=<opaque data>]
   [&Attributes.entry.8.key=push-endpoint&Attributes.entry.8.value=<endpoint>]
   [&Attributes.entry.9.key=persistent&Attributes.entry.9.value=true|false]
   [&Attributes.entry.10.key=cloudevents&Attributes.entry.10.value=true|false]
   [&Attributes.entry.11.key=mechanism&Attributes.entry.11.value=<mechanism>]
   [&Attributes.entry.12.key=time_to_live&Attributes.entry.12.value=<seconds to live>]
   [&Attributes.entry.13.key=max_retries&Attributes.entry.13.value=<retries number>]
   [&Attributes.entry.14.key=retry_sleep_duration&Attributes.entry.14.value=<sleep seconds>]
   [&Attributes.entry.15.key=Policy&Attributes.entry.15.value=<policy-JSON-string>]

Request parameters:

- push-endpoint: This is the URI of an endpoint to send push notifications to.
- OpaqueData: Opaque data is set in the topic configuration and added to all
  notifications that are triggered by the topic.
- persistent: This indicates whether notifications to this endpoint are
  persistent (=asynchronous) or not persistent. (This is "false" by default.)
- time_to_live: This will limit the time (in seconds) to retain the notifications.
  default value is taken from `rgw_topic_persistency_time_to_live`.
  providing a value overrides the global value.
  zero value means infinite time to live.
- max_retries: This will limit the max retries before expiring notifications.
  default value is taken from `rgw_topic_persistency_max_retries`.
  providing a value overrides the global value.
  zero value means infinite retries.
- retry_sleep_duration: This will control the frequency of retrying the notifications.
  default value is taken from `rgw_topic_persistency_sleep_duration`.
  providing a value overrides the global value.
  zero value mean there is no delay between retries.
- Policy: This will control who can access the topic in addition to the owner of the topic.
  The policy passed needs to be a JSON string similar to bucket policy.
  For example, one can send a policy string as follows::

    {
      "Version": "2012-10-17",
      "Statement": [{
        "Effect": "Allow",
        "Principal": {"AWS": ["arn:aws:iam::usfolks:user/fred:subuser"]},
        "Action": ["sns:GetTopicAttributes","sns:Publish"],
        "Resource": ["arn:aws:sns:default::mytopic"],
      }]
    }

  Currently, we support only the following actions:
  - sns:GetTopicAttributes  To list or get existing topics
  - sns:SetTopicAttributes  To set attributes for the existing topic
  - sns:DeleteTopic         To delete the existing topic
  - sns:Publish             To be able to create/subscribe notification on existing topic

- HTTP endpoint

 - URI: ``http[s]://<fqdn>[:<port]``
 - port: This defaults to 80 for HTTP and 443 for HTTPS.
 - verify-ssl: This indicates whether the server certificate is validated by
   the client. (This is "true" by default.)
 - cloudevents: This indicates whether the HTTP header should contain
   attributes according to the `S3 CloudEvents Spec`_. (This is "false" by
   default.)

- AMQP0.9.1 endpoint

 - URI: ``amqp[s]://[<user>:<password>@]<fqdn>[:<port>][/<vhost>]``
 - user/password: This defaults to "guest/guest".
 - user/password: This must be provided only over HTTPS. Topic creation
   requests will otherwise be rejected.
 - port: This defaults to 5672 for unencrypted connections and 5671 for
   SSL-encrypted connections.
 - vhost: This defaults to "/".
 - verify-ssl: This indicates whether the server certificate is validated by
   the client. (This is "true" by default.)
 - If ``ca-location`` is provided and a secure connection is used, the
   specified CA will be used to authenticate the broker. The default CA will
   not be used.  
 - amqp-exchange: The exchanges must exist and must be able to route messages
   based on topics. This parameter is mandatory.
 - amqp-ack-level: No end2end acking is required. Messages may persist in the
   broker before being delivered to their final destinations. Three ack methods
   exist:

  - "none": message is considered "delivered" if sent to broker
  - "broker": message is considered "delivered" if acked by broker (default)
  - "routable": message is considered "delivered" if broker can route to a consumer

.. tip:: The topic-name (see :ref:`Create a Topic`) is used for the
   AMQP topic ("routing key" for a topic exchange).

- Kafka endpoint

 - URI: ``kafka://[<user>:<password>@]<fqdn>[:<port]``
 - ``use-ssl``: If this is set to "true", a secure connection is used to
   connect to the broker. (This is "false" by default.)
 - ``ca-location``: If this is provided and a secure connection is used, the
   specified CA will be used instead of the default CA to authenticate the
   broker. 
 - user/password: This should be provided over HTTPS. If not, the config parameter `rgw_allow_notification_secrets_in_cleartext` must be `true` in order to create topics.
 - user/password: This should be provided together with ``use-ssl``. If not, the broker credentials will be sent over insecure transport.
 - mechanism: may be provided together with user/password (default: ``PLAIN``). The supported SASL mechanisms are:

  - PLAIN
  - SCRAM-SHA-256
  - SCRAM-SHA-512
  - GSSAPI
  - OAUTHBEARER

 - port: This defaults to 9092.
 - kafka-ack-level: No end2end acking is required. Messages may persist in the
   broker before being delivered to their final destinations. Two ack methods
   exist:

  - "none": Messages are considered "delivered" if sent to the broker.
  - "broker": Messages are considered "delivered" if acked by the broker. (This
    is the default.)

.. note::

    - The key-value pair of a specific parameter need not reside in the same
      line as the parameter, and need not appear in any specific order, but it
      must use the same index.
    - Attribute indexing need not be sequential and need not start from any
      specific value.
    - `AWS Create Topic`_ provides a detailed explanation of the endpoint
      attributes format. In our case, however, different keys and values are
      used.

The response will have the following format:

::

    <CreateTopicResponse xmlns="https://sns.amazonaws.com/doc/2010-03-31/">
        <CreateTopicResult>
            <TopicArn></TopicArn>
        </CreateTopicResult>
        <ResponseMetadata>
            <RequestId></RequestId>
        </ResponseMetadata>
    </CreateTopicResponse>

The topic `ARN
<https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html>`_
in the response has the following format:

::

   arn:aws:sns:<zone-group>:<tenant>:<topic>

Get Topic Attributes
````````````````````

This returns information about a specific topic. This includes push-endpoint
information, if provided.

::

   POST

   Action=GetTopicAttributes
   &TopicArn=<topic-arn>

Response will have the following format:

::

    <GetTopicAttributesResponse>
        <GetTopicAttributesResult>
            <Attributes>
                <entry>
                    <key>User</key>
                    <value></value>
                </entry> 
                <entry>
                    <key>Name</key>
                    <value></value>
                </entry> 
                <entry>
                    <key>EndPoint</key>
                    <value></value>
                </entry> 
                <entry>
                    <key>TopicArn</key>
                    <value></value>
                </entry> 
                <entry>
                    <key>OpaqueData</key>
                    <value></value>
                </entry> 
            </Attributes>
        </GetTopicAttributesResult>
        <ResponseMetadata>
            <RequestId></RequestId>
        </ResponseMetadata>
    </GetTopicAttributesResponse>

- User: name of the user that created the topic
- Name: name of the topic
- EndPoint: JSON formatted endpoint parameters, including:
   - EndpointAddress: the push-endpoint URL
   - EndpointArgs: the push-endpoint args
   - EndpointTopic: the topic name that should be sent to the endpoint (may be different than the above topic name)
   - HasStoredSecret: "true" if if endpoint URL contain user/password information. In this case request must be made over HTTPS. If not, topic get request will be rejected 
   - Persistent: "true" is topic is persistent
- TopicArn: topic ARN
- OpaqueData: the opaque data set on the topic

Get Topic Information
`````````````````````

Returns information about specific topic. This includes push-endpoint information, if provided.
Note that this API is now deprecated in favor of the AWS compliant `GetTopicAttributes` API.

::

   POST

   Action=GetTopic
   &TopicArn=<topic-arn>

Response will have the following format:

::

    <GetTopicResponse>
        <GetTopicResult>
            <Topic>
                <User></User>
                <Name></Name>
                <EndPoint>
                    <EndpointAddress></EndpointAddress>
                    <EndpointArgs></EndpointArgs>
                    <EndpointTopic></EndpointTopic>
                    <HasStoredSecret></HasStoredSecret>
                    <Persistent></Persistent>
                </EndPoint>
                <TopicArn></TopicArn>
                <OpaqueData></OpaqueData>
            </Topic>
        </GetTopicResult>
        <ResponseMetadata>
            <RequestId></RequestId>
        </ResponseMetadata>
    </GetTopicResponse>

- User: name of the user that created the topic
- Name: name of the topic
- EndpointAddress: the push-endpoint URL
- EndpointArgs: the push-endpoint args
- EndpointTopic: the topic name that should be sent to the endpoint (may be different than the above topic name)
- HasStoredSecret: "true" if endpoint URL contain user/password information. In this case request must be made over HTTPS. If not, topic get request will be rejected 
- Persistent: "true" is topic is persistent
- TopicArn: topic ARN
- OpaqueData: the opaque data set on the topic

Delete Topic
````````````

::

   POST

   Action=DeleteTopic
   &TopicArn=<topic-arn>

Delete the specified topic.

.. note::

  - Deleting an unknown notification (e.g. double delete) is not considered an error
  - Deleting a topic does not automatically delete all notifications associated with it

The response will have the following format:

::

    <DeleteTopicResponse xmlns="https://sns.amazonaws.com/doc/2010-03-31/">
        <ResponseMetadata>
            <RequestId></RequestId>
        </ResponseMetadata>
    </DeleteTopicResponse>

List Topics
```````````

List all topics associated with a tenant.

::

   POST

   Action=ListTopics

Response will have the following format:

::

    <ListTopicsResponse xmlns="https://sns.amazonaws.com/doc/2010-03-31/">
        <ListTopicsResult>
            <Topics>
                <member>
                    <User></User>
                    <Name></Name>
                    <EndPoint>
                        <EndpointAddress></EndpointAddress>
                        <EndpointArgs></EndpointArgs>
                        <EndpointTopic></EndpointTopic>
                    </EndPoint>
                    <TopicArn></TopicArn>
                    <OpaqueData></OpaqueData>
                </member>
            </Topics>
        </ListTopicsResult>
        <ResponseMetadata>
            <RequestId></RequestId>
        </ResponseMetadata>
    </ListTopicsResponse>

- if endpoint URL contain user/password information, in any of the topic, request must be made over HTTPS. If not, topic list request will be rejected.

Notifications
~~~~~~~~~~~~~

Detailed under: `Bucket Operations`_.

.. note::

    - "Abort Multipart Upload" request does not emit a notification
    - Both "Initiate Multipart Upload" and "POST Object" requests will emit an ``s3:ObjectCreated:Post`` notification

Events
~~~~~~

The events are in JSON format (regardless of the actual endpoint), and share the same structure as the S3-compatible events
pushed or pulled using the pubsub sync module. For example:

::

   {"Records":[
       {
           "eventVersion":"2.1",
           "eventSource":"ceph:s3",
           "awsRegion":"us-east-1",
           "eventTime":"2019-11-22T13:47:35.124724Z",
           "eventName":"ObjectCreated:Put",
           "userIdentity":{
               "principalId":"tester"
           },
           "requestParameters":{
               "sourceIPAddress":""
           },
           "responseElements":{
               "x-amz-request-id":"503a4c37-85eb-47cd-8681-2817e80b4281.5330.903595",
               "x-amz-id-2":"14d2-zone1-zonegroup1"
           },
           "s3":{
               "s3SchemaVersion":"1.0",
               "configurationId":"mynotif1",
               "bucket":{
                   "name":"mybucket1",
                   "ownerIdentity":{
                       "principalId":"tester"
                   },
                   "arn":"arn:aws:s3:us-east-1::mybucket1",
                   "id":"503a4c37-85eb-47cd-8681-2817e80b4281.5332.38"
               },
               "object":{
                   "key":"myimage1.jpg",
                   "size":"1024",
                   "eTag":"37b51d194a7513e45b56f6524f2d51f2",
                   "versionId":"",
                   "sequencer": "F7E6D75DC742D108",
                   "metadata":[],
                   "tags":[]
               }
           },
           "eventId":"",
           "opaqueData":"me@example.com"
       }
   ]}

- awsRegion: zonegroup
- eventTime: timestamp indicating when the event was triggered
- eventName: for list of supported events see: `S3 Notification Compatibility`_. Note that the eventName values do not start with the `s3:` prefix.
- userIdentity.principalId: user that triggered the change
- requestParameters.sourceIPAddress: not supported
- responseElements.x-amz-request-id: request ID of the original change
- responseElements.x_amz_id_2: RGW on which the change was made
- s3.configurationId: notification ID that created the event
- s3.bucket.name: name of the bucket
- s3.bucket.ownerIdentity.principalId: owner of the bucket
- s3.bucket.arn: ARN of the bucket
- s3.bucket.id: Id of the bucket (an extension to the S3 notification API)
- s3.object.key: object key
- s3.object.size: object size
- s3.object.eTag: object etag
- s3.object.versionId: object version in case of versioned bucket. 
  When doing a copy, it would include the version of the target object. 
  When creating a delete marker, it would include the version of the delete marker.
- s3.object.sequencer: monotonically increasing identifier of the change per object (hexadecimal format)
- s3.object.metadata: any metadata set on the object sent as: ``x-amz-meta-`` (an extension to the S3 notification API)
- s3.object.tags: any tags set on the object (an extension to the S3 notification API)
- s3.eventId: unique ID of the event, that could be used for acking (an extension to the S3 notification API)
- s3.opaqueData: opaque data is set in the topic configuration and added to all notifications triggered by the topic (an extension to the S3 notification API)

.. _PubSub Module : ../pubsub-module
.. _S3 Notification Compatibility: ../s3-notification-compatibility
.. _AWS Create Topic: https://docs.aws.amazon.com/sns/latest/api/API_CreateTopic.html
.. _Bucket Operations: ../s3/bucketops
.. _S3 CloudEvents Spec: https://github.com/cloudevents/spec/blob/main/cloudevents/adapters/aws-s3.md
