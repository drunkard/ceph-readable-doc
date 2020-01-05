.. Open Policy Agent Integration

===========================
 与 Open Policy Agent 对接
===========================

Open Policy Agent (OPA) is a lightweight general-purpose policy engine
that can be co-located with a service. OPA can be integrated as a
sidecar, host-level daemon, or library.

Services can offload policy decisions to OPA by executing queries. Hence,
policy enforcement can be decoupled from policy decisions.

.. Configure OPA

配置 OPA
========

To configure OPA, load custom policies into OPA that control the resources users
are allowed to access. Relevant data or context can also be loaded into OPA to make decisions.

Policies and data can be loaded into OPA in the following ways::
  * OPA's RESTful APIs
  * OPA's *bundle* feature that downloads policies and data from remote HTTP servers
  * Filesystem

.. Configure the Ceph Object Gateway

配置 Ceph 对象网关
==================

The following configuration options are available for OPA integration::

     rgw use opa authz = {use opa server to authorize client requests}
     rgw opa url = {opa server url:opa server port}
     rgw opa token = {opa bearer token}
     rgw opa verify ssl = {verify opa server ssl certificate}

.. How does the RGW-OPA integration work

RGW-OPA 是如何对接的
====================

After a user is authenticated, OPA can be used to check if the user is authorized
to perform the given action on the resource. OPA responds with an allow or deny
decision which is sent back to the RGW which enforces the decision.

请求实例： ::

   POST /v1/data/ceph/authz HTTP/1.1
   Host: opa.example.com:8181
   Content-Type: application/json
   
   {
       "input": {
           "method": "GET",
           "user_info": {
               "user_id": "john",
               "display_name": "John"  
           },
           "bucket_info": {
               "bucket": {
                   "name": "Testbucket",
                   "bucket_id": "testbucket" 
               },
               "owner": "john" 
           }             
       }
   }

响应： ::

   {"result": true}

The above is a sample request sent to OPA which contains information about the
user, resource and the action to be performed on the resource. Based on the polices
and data loaded into OPA, it will verify whether the request should be allowed or denied.
In the sample request, RGW makes a POST request to the endpoint */v1/data/ceph/authz*,
where *ceph* is the package name and *authz* is the rule name.
