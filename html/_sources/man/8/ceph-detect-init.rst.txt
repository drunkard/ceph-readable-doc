:orphan:

====================================================
 ceph-detect-init -- 显示 Ceph 应当使用的 init 系统
====================================================

.. program:: ceph-detect-init

提纲
====

| **ceph-detect-init** [--verbose] [--use-rhceph] [--default *init*]


描述
====

:program:`ceph-detect-init` 工具用于探测 Ceph 会使用的 init 系\
统，输出有可能是 ``sysvinit`` 、 ``upstart`` 、 ``systemd`` 其\
中之一。 Ceph 使用的 init 系统未必是宿主操作系统的默认 init 系\
统，比如在 Debian Jessie 上 Ceph 就会用 ``sysvinit`` ，而默认\
的却是 ``systemd`` 。

如果宿主操作系统的 init 系统是未知的，它会返回错误，除非指定了
:option:`--default` 选项。


选项
====

.. option:: --use-rhceph

   当操作系统自认为是 Red Hat 时，它会被当作 CentOS ；可以用、
   :option:`--use-rhceph` 选项把它当作 RHEL 。

.. option:: --default INIT

   如果宿主操作系统的 init 系统是未知的，就返回 *INIT* 值，而\
   非报错。

.. option:: --verbose

   显示用于调试的额外信息。


已知缺陷
========

:program:`ceph-detect-init` 是为 :program:`ceph-disk` 探测 init
系统类型的，以便管理 OSD 的挂载目录。但是我们只全面测试过下面\
列出的环境：

- `upstart` on `Ubuntu 14.04`
- `systemd` on `Ubuntu 15.04` and up
- `systemd` on `Debian 8` and up
- `systemd` on `RHEL/CentOS 7` and up
- `systemd` on `Fedora 22` and up


使用范围
========

:program:`ceph-detect-init` 是 Ceph 的一部分，这是个伸缩力强、\
开源、分布式的存储系统，更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph-disk <ceph-disk>`\(8),
:doc:`ceph-deploy <ceph-deploy>`\(8)
