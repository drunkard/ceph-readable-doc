.. _ceph-conf-common-settings:

通用选项
========

The `Hardware Recommendations`_ section provides some hardware guidelines for
configuring a Ceph Storage Cluster. It is possible for a single :term:`Ceph
Node` to run multiple daemons. For example, a single node with multiple drives
may run one ``ceph-osd`` for each drive. Ideally, you will  have a node for a
particular type of process. For example, some nodes may run ``ceph-osd``
daemons, other nodes may run ``ceph-mds`` daemons, and still  other nodes may
run ``ceph-mon`` daemons.

Each node has a name identified by the ``host`` setting. Monitors also specify
a network address and port (i.e., domain name or IP address) identified by the
``addr`` setting.  A basic configuration file will typically specify only
minimal settings for each instance of monitor daemons. For example:

.. code-block:: ini

	[global]
	mon_initial_members = ceph1
	mon_host = 10.0.0.1


.. important:: The ``host`` setting is the short name of the node (i.e., not
   an fqdn). It is **NOT** an IP address either.  Enter ``hostname -s`` on
   the command line to retrieve the name of the node. Do not use ``host``
   settings for anything other than initial monitors unless you are deploying
   Ceph manually. You **MUST NOT** specify ``host`` under individual daemons
   when using deployment tools like ``chef`` or ``cephadm``, as those tools
   will enter the appropriate values for you in the cluster map.


.. _ceph-network-config:

网络
====
.. Networks

See the `Network Configuration Reference`_ for a detailed discussion about
configuring a network for use with Ceph.


Temporary Directory
===================

Some operations will cause a daemon to write to a temporary file. These files
are located according to the ``tmp_dir`` config.

.. confval:: tmp_dir

The ``$TMPDIR`` environment variable is used to initialize the config, if
present, but may be overriden on the command-line. A default may also
be set for the cluster using the usual ``ceph config`` API.

The template for the temporary files created by daemons is controlled
by the ``tmp_file_template`` config.

.. confval:: tmp_file_template

One example where temporary files are created by daemons is the use of the
``--daemon-output-file=:tmp:`` argument to the ``ceph tell`` command.


监视器
======
.. Monitors

Ceph production clusters typically deploy with a minimum 3 :term:`Ceph Monitor`
daemons to ensure high availability should a monitor instance crash. At least
three (3) monitors ensures that the Paxos algorithm can determine which version
of the :term:`Ceph Cluster Map` is the most recent from a majority of Ceph
Monitors in the quorum.

.. note:: You may deploy Ceph with a single monitor, but if the instance fails,
	       the lack of other monitors may interrupt data service availability.

Ceph Monitors normally listen on port ``3300`` for the new v2 protocol, and ``6789`` for the old v1 protocol.

By default, Ceph expects that you will store a monitor's data under the
following path::

	/var/lib/ceph/mon/$cluster-$id

You or a deployment tool (e.g., ``cephadm``) must create the corresponding
directory. With metavariables fully  expressed and a cluster named "ceph", the
foregoing directory would evaluate to::

	/var/lib/ceph/mon/ceph-a

详情见 :ref:`monitor-config-reference` 。


.. _ceph-osd-config:

认证
====
.. Authentication

.. versionadded:: Bobtail 0.56

For Bobtail (v 0.56) and beyond, you should expressly enable or disable
authentication in the ``[global]`` section of your Ceph configuration file.

.. code-block:: ini

	auth_cluster_required = cephx
	auth_service_required = cephx
	auth_client_required = cephx

另外，你应该打开消息签名功能，细节见 :ref:`rados-cephx-config-ref` 。


.. _ceph-monitor-config:


OSDs
====

By default, Ceph expects to store a Ceph OSD Daemon's data on the following
path::

    /var/lib/ceph/osd/$cluster-$id

You or a deployment tool (for example, ``cephadm``) must create the
corresponding directory. With metavariables fully expressed and a cluster named
"ceph", the path specified in the above example evaluates to::

    /var/lib/ceph/osd/ceph-0

You can override this path using the ``osd_data`` setting. We recommend that
you do not change the default location. To create the default directory on your
OSD host, run the following commands:

.. prompt:: bash $

    ssh {osd-host}
    sudo mkdir /var/lib/ceph/osd/ceph-{osd-number}

The ``osd_data`` path must lead to a device that is not shared with the
operating system. To use a device other than the device that contains the
operating system and the daemons, prepare it for use with Ceph and mount it on
the directory you just created by running commands of the following form:

.. prompt:: bash $

    ssh {new-osd-host}
    sudo mkfs -t {fstype} /dev/{disk}
    sudo mount -o user_xattr /dev/{disk} /var/lib/ceph/osd/ceph-{osd-number}

We recommend using the ``xfs`` file system when running :command:`mkfs`. (The
``btrfs`` and ``ext4`` file systems are not recommended and are no longer
tested.)

For additional configuration details, see `OSD Config Reference`_.


心跳
====
.. Heartbeats

During runtime operations, Ceph OSD Daemons check up on other Ceph OSD Daemons
and report their  findings to the Ceph Monitor. You do not have to provide any
settings. However, if you have network latency issues, you may wish to modify
the settings.

See `Configuring Monitor/OSD Interaction`_ for additional details.


.. _ceph-logging-and-debugging:

日志记录、调试
==============
.. Logs / Debugging

Sometimes you may encounter issues with Ceph that require
modifying logging output and using Ceph's debugging. See `Debugging and
Logging`_ for details on log rotation.

.. _Debugging and Logging: ../../troubleshooting/log-and-debug


ceph.conf 实例
==============
.. Example ceph.conf

Note that since the Mimic release the Monitors maintain a database of option
settings: the *central config*.  Node-local ``ceph.conf`` files are still
supported, but in most cases need only contain the first three lines shown
below.  Maintaining full ``ceph.conf`` files across cluster nodes can be
tedious and prone to omission and error, especially when daemons are containerized.
The below file is provided as a reference example. Clusters running recent
releases are best managed primary by central config, with a minimal ``ceph.conf``
file that defines only how to reach the Monitors and thus rarely requires
modification.

.. literalinclude:: demo-ceph.conf
   :language: ini

.. _ceph-runtime-config:


跑多个集群（已废弃）
====================
.. Naming Clusters (deprecated)

Each Ceph cluster has an internal name that is used as part of configuration
and log file names as well as directory and mountpoint names.  This name
defaults to "ceph".  Previous releases of Ceph allowed one to specify a custom
name instead, for example "ceph2".  This was intended to faciliate running
multiple logical clusters on the same physical hardware, but in practice this
was rarely exploited and should no longer be attempted.  Prior documentation
could also be misinterpreted as requiring unique cluster names in order to
use ``rbd-mirror``.

Custom cluster names are now considered deprecated and the ability to deploy
them has already been removed from some tools, though existing custom name
deployments continue to operate.  The ability to run and manage clusters with
custom names may be progressively removed by future Ceph releases, so it is
strongly recommended to deploy all new clusters with the default name "ceph".

Some Ceph CLI commands accept an optional ``--cluster`` (cluster name) option. This
option is present purely for backward compatibility and need not be accomodated
by new tools and deployments.

If you do need to allow multiple clusters to exist on the same host, please use
:ref:`cephadm`, which uses containers to fully isolate each cluster.





.. _Hardware Recommendations: ../../../start/hardware-recommendations
.. _Network Configuration Reference: ../network-config-ref
.. _OSD Config Reference: ../osd-config-ref
.. _Configuring Monitor/OSD Interaction: ../mon-osd-interaction
