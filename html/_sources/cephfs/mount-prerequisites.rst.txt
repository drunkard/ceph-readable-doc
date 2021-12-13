挂载 CephFS ：先决条件
======================
.. Mount CephFS: Prerequisites

要使用 CephFS ，你可以把它挂载到你的本地文件系统下、
或者通过 `cephfs-shell`_ 使用。挂载 CephFS 需要超级用户权限才能\
发起对它自己的重新挂载（ remount ），之后才能修剪条目。
可以 `用内核`_ 也可以 `用 FUSE`_ 挂载 CephFS ，二者各有优劣。
读一下下面的段落，了解一下两种挂载 CephFS 的方式。

要在 Windows 上挂载 CephFS ，请看 `ceph-dokan`_ 。

用哪个 CephFS 客户端？
----------------------
.. Which CephFS Client?

FUSE 客户端是最简便、也最容易升级到与存储集群一致的 Ceph 版本，\
但是内核客户端的性能通常更好。

遇到缺陷或性能问题时，最好试试另一个客户端，
以甄别此缺陷是否特定于客户端（然后报告给开发者）。

挂载 CephFS 的一般先决条件
--------------------------
.. General Pre-requisite for Mounting CephFS

挂载 CephFS 前，确保客户端主机（挂载和使用 CephFS 的地方）上\
有 Ceph 配置文件的副本（就是 ``ceph.conf`` ）、
和 CephX 用户的密钥环（有权限访问 MDS 的）。
这些文件必须已经存在于 Ceph MON 主机上。

#. 为客户端生成一份最小化的配置文件，并放到标准位置： ::

    # 在客户端主机上
    mkdir -p -m 755 /etc/ceph
    ssh {user}@{mon-host} "sudo ceph config generate-minimal-conf" | sudo tee /etc/ceph/ceph.conf

   另外，你也可以复制整个 conf 文件。只是，
   前述方法生成的配置文件只包含最少信息，通常足够了。
   要了解更多，见 `客户端认证`_ 和 :ref:`bootstrap-options` 。

#. 确保这个 conf 文件的权限没问题： ::

    chmod 644 /etc/ceph/ceph.conf

#. 创建一个 CephX 用户、并获取它的密钥： ::

    ssh {user}@{mon-host} "sudo ceph fs authorize cephfs client.foo / rw" | sudo tee /etc/ceph/ceph.client.foo.keyring

   在前面的命令里，把 ``cephfs`` 换成你的 CephFS 名字、
   ``foo`` 换成你的 CephX 用户名、
   ``/`` 换成客户端主机被允许访问的 CephFS 路径，
   ``rw`` 分别表示读取和写入权限。另外，
   你可以把 MON 主机上的 Ceph 密钥环复制到客户端主机上的 ``/etc/ceph`` 目录，
   但是新建一个客户端主机专用的密钥环更好。
   同时，创建一个 CephX 密钥环或客户端后，
   在很多机器上都使用相同的客户端也完全没问题。

   .. note:: 如果你运行前面两个命令时遇到两次输密码的提示符，
      先运行 ``sudo ls`` （或者用 sudo 运行个随便什么无关紧要的命令）
      再紧接着运行这些命令。

#. 确保这个密钥环的权限位合适： ::

    chmod 600 /etc/ceph/ceph.client.foo.keyring

.. note:: 用内核或 FUSE 挂载前还有些其它的先决条件，
   请分别查看相应的文档。


.. _客户端认证: ../client-auth
.. _cephfs-shell: ../cephfs-shell
.. _用内核: ../mount-using-kernel-driver
.. _用 FUSE: ../mount-using-fuse
.. _ceph-dokan: ../ceph-dokan
