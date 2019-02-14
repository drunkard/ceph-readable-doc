.. _Object Store Architecture Overview:

==================
 对象存储架构概述
==================

.. graphviz::

  /*
   * Rough outline of object store module dependencies
   */

  digraph object_store {
    size="12,12";
    node [color=lightblue2, style=filled, fontname="Serif"];

    "testrados" -> "librados"
    "testradospp" -> "librados"

    "rbd" -> "librados"

    "radostool" -> "librados"

    "radosgw-admin" -> "radosgw"

    "radosgw" -> "librados"

    "radosacl" -> "librados"

    "librados" -> "objecter"

    "ObjectCacher" -> "Filer"

    "dumpjournal" -> "Journaler"

    "Journaler" -> "Filer"

    "SyntheticClient" -> "Filer"
    "SyntheticClient" -> "objecter"

    "Filer" -> "objecter"

    "objecter" -> "OSDMap"

    "ceph-osd" -> "PG"
    "ceph-osd" -> "ObjectStore"

    "crushtool" -> "CrushWrapper"

    "OSDMap" -> "CrushWrapper"

    "OSDMapTool" -> "OSDMap"

    "PG" -> "PrimaryLogPG"
    "PG" -> "ObjectStore"
    "PG" -> "OSDMap"

    "PrimaryLogPG" -> "ObjectStore"
    "PrimaryLogPG" -> "OSDMap"

    "ObjectStore" -> "FileStore"
    "ObjectStore" -> "BlueStore"

    "BlueStore" -> "rocksdb"

    "FileStore" -> "xfs"
    "FileStore" -> "btrfs"
    "FileStore" -> "ext4"
  }


.. todo:: write more here
