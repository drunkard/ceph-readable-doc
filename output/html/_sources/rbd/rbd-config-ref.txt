=============
 librbd 选项
=============

详情见\ `块设备`_\ 。

缓存选项
========

.. sidebar:: 内核缓存

	Ceph 块设备的内核驱动可利用 Linux 页缓存来提升性能。

Ceph 块设备的用户空间实现（即 ``librbd`` ）不能利用 Linux 页缓存，所以它自己实现了\
内存缓存，名为“ RBD 缓存”。 RBD 缓存行为就像硬盘缓存一样端正，当 OS 发送了 barrier \
或 flush 请求时，所有脏数据都会写入 OSD ，这意味着只要 VM 会正确地发送回写命令（即\
内核版本大于 2.6.32 ），使用回写缓存和常见物理硬盘一样安全。此缓存用最近最少使用\
（ Least Recently Used, LRU ）算法，而且在回写模式下它能合并相邻请求以提高吞吐量。

.. versionadded:: 0.46

Ceph 可为 RBD 做回写缓存，要启用此功能，在 ``ceph.conf`` 配置文件的 ``[client]`` \
段下添加 ``rbd cache = true`` 。 ``librbd`` 默认不会进行任何缓存，写和读都直接到\
达存储集群，而且所有数据都完成复制后写动作才会返回；启用缓存后，写动作会立即返回，除\
非未回写的字节数大于 ``rbd cache max dirty`` ，这种情况下，写动作会触发回写机制并\
一直阻塞着，直到回写完了足够多的字节数。

.. versionadded:: 0.47

Ceph 支持为 RBD 写透做缓存。你可以设置缓存尺寸、还能设置从写回缓存切换到写透缓存的\
目标和临界点。要启用写透模式，把 ``rbd cache max dirty`` 设为 ``0`` ，这意味着数据\
的所有复制都完成时写才会返回，但是读可以来自缓存。在客户端，缓存位于内存中，且个 \
RBD 映像有自己的缓存。对客户端来说正因为缓存位于本地，所以对映像的访问没有相干性。\
打开缓存时，在 RBD 之上不能运行 GFS 或 OCFS 。

RBD 选项应该位于 ``ceph.conf`` 配置文件的 ``[client]`` 段下，可用选项有：


``rbd cache``

:Description: 允许为 RADOS 块设备提供缓存。
:Type: Boolean
:Required: No
:Default: ``true``


``rbd cache size``

:Description: RBD 缓存尺寸，字节。
:Type: 64-bit Integer
:Required: No
:Default: ``32 MiB``


``rbd cache max dirty``

:Description: 使缓存触发写回的 ``dirty`` 临界点，若为 ``0`` ，直接使用写透缓存。
:Type: 64-bit Integer
:Required: No
:Constraint: 必须小于 ``rbd cache size`` 。
:Default: ``24 MiB``


``rbd cache target dirty``

:Description: 缓存开始写回数据的目的地 ``dirty target`` ，不会阻塞到缓存的写动作。
:Type: 64-bit Integer
:Required: No
:Constraint: 必须小于 ``rbd cache max dirty``.
:Default: ``16 MiB``


``rbd cache max dirty age``

:Description: 写回开始前，脏数据在缓存中的暂存时间。
:Type: Float
:Required: No
:Default: ``1.0``

.. versionadded:: 0.60

``rbd cache writethrough until flush``

:Description: 开始进入写透模式，并且在首个 flush 请求收到后切回写回模式。启用它保\
              守但安全，以防 rbd 之上的虚拟机内核太老、不能发送 flush ，像 2.6.32 \
              之前的 virtio 驱动。

:Type: Boolean
:Required: No
:Default: ``true``

.. _块设备: ../../rbd/rbd/


预读选项
========

.. versionadded:: 0.86

RBD supports read-ahead/prefetching to optimize small, sequential reads.
This should normally be handled by the guest OS in the case of a VM,
but boot loaders may not issue efficient reads.
Read-ahead is automatically disabled if caching is disabled.


``rbd readahead trigger requests``

:Description: Number of sequential read requests necessary to trigger read-ahead.
:Type: Integer
:Required: No
:Default: ``10``


``rbd readahead max bytes``

:Description: Maximum size of a read-ahead request.  If zero, read-ahead is disabled.
:Type: 64-bit Integer
:Required: No
:Default: ``512 KiB``


``rbd readahead disable after bytes``

:Description: After this many bytes have been read from an RBD image, read-ahead is disabled for that image until it is closed.  This allows the guest OS to take over read-ahead once it is booted.  If zero, read-ahead stays enabled.
:Type: 64-bit Integer
:Required: No
:Default: ``50 MiB``
