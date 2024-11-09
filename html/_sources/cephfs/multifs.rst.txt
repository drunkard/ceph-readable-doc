.. _cephfs-multifs:

多套 Ceph 文件系统
==================
.. Multiple Ceph File Systems

从 Pacific 版起，对多套文件系统的支持终于稳定了，可以使用了。
这个功能允许你配置独立的文件系统，
它的数据完全分离，位于独立的存储池中。

已有集群必须设置一个标记来启用多套文件系统功能： ::

    ceph fs flag set enable_multiple true

新的 Ceph 集群会自动设置此标记。


新建一个 Ceph 文件系统
----------------------
.. Creating a new Ceph File System

新的 ``volumes`` 插件接口（见 :doc:`/cephfs/fs-volumes` ）把很多\
配置新文件系统的工作自动化了。"volume" 概念就是一个新文件系统。可以这样创建::

    ceph fs volume create <fs_name>

Ceph 会创建新存储池、并自动部署新的 MDS 以支撑这个新文件系统。
所使用的部署技术，就是 cephadm ，还会配置与这个新文件系统配套的\
新 MDS 守护进程的 MDS 亲和性（见 :ref:`mds-join-fs` ）。


对访问权限做安全加固
--------------------
.. Securing access

``fs authorize`` 命令可以用于配置客户端到某个特定文件系统的访问权限。
也看看 :ref:`fs-authorize-multifs` 。客户端只能看到已经授权的文件系统，
监视器和 MDS 会拒绝没有授权的客户端访问。


其他注意事项
------------
.. Other Notes

多套文件系统不会共享存储池。这对于快照来说非常重要，
还因为没有任何防止重复 inode 的措施。
Ceph 命令会防止这种危险的配置。

各个文件系统都有它自己的一套 MDS rank ，相应地，
每个新文件系统也需要更多的 MDS 守护进程来运营、
运营成本也就相应增加了。
这方便了按应用或用户来提升元数据吞吐量，
但同时也增加了新建文件系统的代价。
一般来说，划分了子树的单个文件系统相比按应用隔离负载要好一些。
