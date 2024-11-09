================
 如何追踪各服务
================
.. Tracing Services

.. _cephadm-tracing:


Jaeger Tracing
==============

Ceph 用 Jaeger 作为追踪后端。要想用追踪功能，就得部署那些服务。

Further details on tracing in ceph:

`Ceph Tracing documentation <https://docs.ceph.com/en/latest/jaegertracing/#jaeger-distributed-tracing/>`_

Deployment
==========

Jaeger services consist of 3 services:

1. Jaeger Agent

2. Jaeger Collector

3. Jaeger Query

Jaeger requires a database for the traces. we use ElasticSearch (version 6) by default.


To deploy jaeger tracing service, when not using your own ElasticSearch:

#. Deploy jaeger services, with a new elasticsearch container:

    .. prompt:: bash #

        ceph orch apply jaeger


#. Deploy jaeger services, with existing elasticsearch cluster and existing jaeger query (deploy agents and collectors):

     .. prompt:: bash #

        ceph orch apply jaeger --without-query --es_nodes=ip:port,..

