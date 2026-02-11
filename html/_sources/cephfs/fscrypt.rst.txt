.. _fscrypt:

CephFS 上的 Fscrypt 加密
========================
.. Fscrypt Encryption on CephFS

Fscrypt 是一种文件系统级的加密实现。这种文件系统加密允许在单个目录的级别上\
进行加密。这样文件系统就可以同时包含加密部分和常规的非加密部分。
Fscrypt 加密会加密文件名和数据内容。


Fscrypt 如何加密
----------------
.. How Fscrypt Encryption Works

加密密钥
~~~~~~~~
.. Encryption Keys

每个 fscrypt 树都有一个主加密密钥。这个主密钥将提供加密目录所需的“密钥”。
这个密钥的长度最多可达 256 位。

策略
~~~~
.. Policies

fscrypt 根目录会被分配一个加密策略。
该策略包含诸如使用哪种加密密码和主密钥 ID 等项。
这告诉客户端如何加密/解密，以及如何验证指定主密钥 ID 与加密 inode 的对应关系。

加密完全在客户端进行。 MDS 和 OSD 并不知晓加密策略或主密钥，
这些服务器组件的改动极小，它们的大部分工作仍然是存储文件名和数据内容
（在这里，只是这些内容恰好是加密过的）。

存取语义
~~~~~~~~
.. Access Semantics

有些语义允许根据客户端是否拥有某个目录的主密钥来决定不同的访问权限。

有密钥的话

* 你可以像平时一样正常访问文件系统。
* 你可以看到文件名、数据内容和链接目标。

没有密钥的话

* 你看不到明文的文件名或者链接目标。
* 你打不开文件。
* 你不可能访问数据内容，设置是加密过的内容也看不到。
* 你不可能删减一个文件。
* 你能看到其他元数据，比如时间、权限模式、所有者、还有扩展属性。

.. note:: 没有密钥你不能做备份或恢复操作。

通过实例学习
~~~~~~~~~~~~
.. Learning by Example

比如有一个名为 ``cephfs`` 的文件系统，有个客户端有两个主密钥和两个目录
（ ``encdir1`` 和 ``encdira`` ），每个目录都有不同的加密主密钥；例如，
``encdir1`` 的主密钥是 ``key1`` ，且 ``encdira`` 的是 ``keyb`` 。
还有一个是普通目录 ``regdir`` 。请注意， ``regdir`` 是个未加密的目录，
会在多租户环境下显示。如下图 1 所示。

在目录上设置策略时，此目录必须为空。那么，在该子树中创建的任何目录、
文件或链接都将继承其父目录的策略信息。

.. figure:: cephfs_fscrypt_overview.svg

   图 1


密钥管理
--------
.. Key Management

每个客户端都有个独一无二的文件系统视图和 fscrypt 树视图。在此例中，
请参阅下图 2 。这里有三个客户端，前两个客户端使用的是支持 fscrypt 功能的
新版 CephFS 客户端，而第三个客户端则不支持。密钥管理是以单个客户端为准的。
一个客户端与 fscrypt 的关系和其他客户端并无瓜葛。
让我们仔细看看其中的细节差异。

第一个客户端（ Client 1 ）拥有主密钥，能够透明地查看加密树，这是未加锁模式。

第二个， Client 2 ，他没有主密钥，功能也不完整，即处于锁定模式。
在锁定模式下，用户无法查看明文文件名或数据内容。当用户罗列目录内容时，
只会看到加密文件名的哈希数值。然后，当做 ``open()`` 操作时，
系统会返回错误，并拒绝执行该操作。
文件大小、权限模式、时间戳和其他索引节点元数据\
会以明文存储，并且在此模式下可用。

最后， Client 3 使用的是较旧版本的 CephFS 客户端，且不具备 fscrypt 功能。
在此模式下，用户看到的视图与之前相同，
但可以对加密文件执行一些数据操作。
此模式并不推荐，也不受支持。

.. figure:: cephfs_fscrypt_multiclient.svg

   图 2


CephFS 支持情况
---------------
.. CephFS Support

CephFS 中存在两种 fscrypt 的实现方式。 CephFS 内核客户端支持 fscrypt 。这种实现方式扩展了内核库中的现有功能，并利用了内核加密密钥环。

其次，用户空间客户端支持“ceph-fuse”和“libcephfs”中的fscrypt。这两个版本旨在实现互操作性，但存在一些限制。


用户空间局限性
--------------
.. Userspace Limitations

要使用用户空间的 fscrypt ，需要一个自定义的 fscrypt 命令行界面（ CLI ）。
这是因为内核中（ ``ceph-fuse`` 所用的）永久配置定义有误。
相反，有一个作为 Ceph 项目一部分维护的 fscrypt 命令行工具。
这个版本包含了配置和使用 fscrypt 所需的必要变更。

这个版本位于： https://github.com/ceph/fscrypt/tree/wip-ceph-fuse

当前， fscrypt 用户空间支持部分加密算法，它们有：

- AES-256-XTS 用于内容加密
- AES-256-CBC-CTS 用于文件名加密

在文件夹上设置策略时，其他所有加密算法都会被拒绝。


如何使用
--------
.. How to Use

配置系统级的加密。这将需要配置 ``/etc/fscrypt.conf`` 。

.. prompt:: bash #
 
       fscrypt setup

设置挂载点级的加密。这必须应用于文件系统的挂载点。
这将设置内部 fscrypt CLI 配置文件，用于管理和跟踪加密密钥

.. prompt:: bash #

       fscrypt setup <mount pt>

要配置一个目录的加密（它必须是空的）

.. prompt:: bash #

       fscrypt encrypt <dir>

给加密目录上锁：

.. prompt:: bash #

       fscrypt lock <dir>

给加密目录解锁：

.. prompt:: bash #

       fscrypt unlock <dir>

查看一个目录的状态（可能是普通的或加密的目录）：

.. prompt:: bash #

       fscrypt status <dir>


主密钥在快照和克隆品中的行为
----------------------------
.. Behavior of Master Key in Snapshots and Clones

所有源自 fscrypt 目录的快照和克隆品的锁定状态将相互关联。
这意味着所有派生的数据集将同时被锁定或解锁。

例如，考虑以下情况：

#. 加密的 ``encdir1`` 已解锁。
#. ``encdir1`` 的快照名为 ``encdir1_snap`` 。
#. 快照 ``encdir1_snap`` 的克隆品是 ``encdir1_snap_clone1`` 。

在当前状态下， ``encdir1`` 、 ``encdir1_snap`` 和 ``encdir1_snap_clone1``
均已解锁，而且各个状态下的文件名和数据均可按预期访问。
如果您对这三个中的任何一个进行锁定，则所有三个都将被锁定。

.. note::

   快照名是未加密的。
