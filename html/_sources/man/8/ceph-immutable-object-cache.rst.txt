:orphan:

===================================================================
 ceph-immutable-object-cache -- 用于不可变对象缓存的 Ceph 守护进程
===================================================================

.. program:: ceph-immutable-object-cache

提纲
====

| **ceph-immutable-object-cache**


描述
====

:program:`ceph-immutable-object-cache` is a daemon for object cache of RADOS
objects among Ceph clusters. It will promote the objects to a local directory
upon promote requests and future reads will be serviced from these cached
objects.

It connects to local clusters via the RADOS protocol, relying on
default search paths to find ceph.conf files, monitor addresses and
authentication information for them, i.e. ``/etc/ceph/$cluster.conf``,
``/etc/ceph/$cluster.keyring``, and
``/etc/ceph/$cluster.$name.keyring``, where ``$cluster`` is the
human-friendly name of the cluster, and ``$name`` is the rados user to
connect as, e.g. ``client.ceph-immutable-object-cache``.


选项
====

.. option:: -c ceph.conf, --conf=ceph.conf

   Use ``ceph.conf`` configuration file instead of the default
   ``/etc/ceph/ceph.conf`` to determine monitor addresses during startup.

.. option:: -m monaddress[:port]

   Connect to specified monitor (instead of looking through ``ceph.conf``).

.. option:: -i ID, --id ID

   Set the ID portion of name for ceph-immutable-object-cache

.. option:: -n TYPE.ID, --name TYPE.ID

   Set the rados user name for the gateway (eg. client.ceph-immutable-object-cache)

.. option:: --cluster NAME

   Set the cluster name (default: ceph)

.. option:: -d

   Run in foreground, log to stderr

.. option:: -f

   Run in foreground, log to usual location


使用范围
========

:program:`ceph-immutable-object-cache` is part of Ceph, a massively scalable, open-source, distributed
storage system. Please refer to the Ceph documentation at http://ceph.com/docs for
more information.


参考
====

:doc:`rbd <rbd>`\(8)
