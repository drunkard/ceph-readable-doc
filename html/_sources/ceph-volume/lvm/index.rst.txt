.. _ceph-volume-lvm:

``lvm``
=======

实现的功能是用 ``lvm`` 子命令部署 OSD ： ``ceph-volume lvm``

**命令行子命令**

* :ref:`ceph-volume-lvm-prepare`

* :ref:`ceph-volume-lvm-activate`

* :ref:`ceph-volume-lvm-create`

* :ref:`ceph-volume-lvm-list`

* :ref:`ceph-volume-lvm-migrate`

* :ref:`ceph-volume-lvm-newdb`

* :ref:`ceph-volume-lvm-newwal`

.. not yet implemented
.. * :ref:`ceph-volume-lvm-scan`


**内部功能**

``lvm`` 子命令还有一些是内部功能，没有暴露给用户，
这几段解释了它们之间如何协作、并阐明该工具的工作流程。

:ref:`Systemd 单元 <ceph-volume-lvm-systemd>` |
:ref:`lvm <ceph-volume-lvm-api>`
