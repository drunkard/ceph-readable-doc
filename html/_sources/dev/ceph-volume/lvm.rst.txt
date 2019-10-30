
.. _ceph-volume-lvm-api:

LVM
===
The backend of ``ceph-volume lvm`` is LVM, it relies heavily on the usage of
tags, which is a way for LVM to allow extending its volume metadata. These
values can later be queried against devices and it is how they get discovered
later.

.. warning:: These APIs are not meant to be public, but are documented so that
             it is clear what the tool is doing behind the scenes. Do not alter
             any of these values.


.. _ceph-volume-lvm-tag-api:

Tag API
-------
The process of identifying logical volumes as part of Ceph relies on applying
tags on all volumes. It follows a naming convention for the namespace that
looks like::

    ceph.<tag name>=<tag value>

All tags are prefixed by the ``ceph`` keyword to claim ownership of that
namespace and make it easily identifiable. This is how the OSD ID would be used
in the context of lvm tags::

    ceph.osd_id=0


.. Metadata
.. _ceph-volume-lvm-tags:

元数据
------
下面描述的是存储在一个 LVM 卷中的所有 Ceph OSD 元数据：


``type``
--------
Describes if the device is an OSD or Journal, with the ability to expand to
other types when supported (for example a lockbox)

实例： ::

    ceph.type=osd


``cluster_fsid``
----------------
实例： ::

    ceph.cluster_fsid=7146B649-AE00-4157-9F5D-1DBFF1D52C26


``data_device``
---------------
实例： ::

    ceph.data_device=/dev/ceph/data-0


``data_uuid``
-------------
实例： ::

    ceph.data_uuid=B76418EB-0024-401C-8955-AE6919D45CC3


``journal_device``
------------------
实例： ::

    ceph.journal_device=/dev/ceph/journal-0


``journal_uuid``
----------------
实例： ::

    ceph.journal_uuid=2070E121-C544-4F40-9571-0B7F35C6CB2B


``encrypted``
-------------
用 ``luks`` 实现加密的实例： ::

    ceph.encrypted=1

不支持加密功能时、或完全禁用时： ::

    ceph.encrypted=0


``osd_fsid``
------------
实例： ::

    ceph.osd_fsid=88ab9018-f84b-4d62-90b4-ce7c076728ff


``osd_id``
----------
实例： ::

    ceph.osd_id=1


``block_device``
----------------
Just used on :term:`bluestore` backends. Captures the path to the logical
volume path.

实例： ::

    ceph.block_device=/dev/mapper/vg-block-0


``block_uuid``
--------------
Just used on :term:`bluestore` backends. Captures either the logical volume UUID or
the partition UUID.

实例： ::

    ceph.block_uuid=E5F041BB-AAD4-48A8-B3BF-31F7AFD7D73E


``db_device``
-------------
Just used on :term:`bluestore` backends. Captures the path to the logical
volume path.

实例： ::

    ceph.db_device=/dev/mapper/vg-db-0


``db_uuid``
-----------
Just used on :term:`bluestore` backends. Captures either the logical volume UUID or
the partition UUID.

实例： ::

    ceph.db_uuid=F9D02CF1-31AB-4910-90A3-6A6302375525


``wal_device``
--------------
Just used on :term:`bluestore` backends. Captures the path to the logical
volume path.

实例： ::

    ceph.wal_device=/dev/mapper/vg-wal-0


``wal_uuid``
------------
Just used on :term:`bluestore` backends. Captures either the logical volume UUID or
the partition UUID.

实例： ::

    ceph.wal_uuid=A58D1C68-0D6E-4CB3-8E99-B261AD47CC39


``vdo``
-------
A VDO-enabled device is detected when device is getting prepared, and then
stored for later checks when activating. This affects mount options by
appending the ``discard`` mount flag, regardless of mount flags being used.

Example for an enabled VDO device::

    ceph.vdo=1
