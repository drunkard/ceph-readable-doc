.. _rados_operations_bluestore_migration:

==================
 迁移到 BlueStore
==================
.. BlueStore Migration

.. warning:: Filestore 在 Reef 版已经废弃了，且不再支持。请迁移到 BlueStore 。

每个 OSD 都必须格式化成 Filestore 或 BlueStore 。然而，
Ceph 集群可以同时靠 Filestore OSD 和 BlueStore OSD 的混合体运行。
由于 BlueStore 在性能和稳健性方面优于 Filestore ，并且从 Reef 版本开始，
Ceph 就不再支持 Filestore ，因此部署 Filestore OSD 的用户应迁移到 BlueStore 。
迁移到 BlueStore 有几种策略。

BlueStore 与 Filestore 截然不同，因此单个 OSD 无法直接转换。
相反，转换过程必须使用以下两种方法之一：
(1) 利用集群的常规复制和修复功能，或
(2) 用工具和策略将 OSD 内容从旧（ Filestore ）设备复制到新（ BlueStore ）设备。


用 BlueStore 部署新 OSD
=======================
.. Deploying new OSDs with BlueStore

在部署新的 OSD （例如，集群扩展时）时，要用 BlueStore 。
由于这是默认行为，所以无需任何特殊更改。

同样，在更换故障驱动器后，重新配置 OSD 时也要用 BlueStore 。


转换已有 OSD
============
.. Converting existing OSDs

“标记为 ``out`` ”的替代
-----------------------
.. "Mark-``out``" replacement

最简单的方法是先验证集群是否正常运行，
然后依次对每个 Filestore OSD 执行以下步骤：
把这个 OSD 标记为 ``out`` ，等待数据在集群中复制完成，
重新配置这个 OSD ，再把这个 OSD 标记回 ``in`` ，
并等待恢复完成，再继续处理下一个 OSD 。这种方法易于自动化，
但会带来不必要的数据迁移，从而产生时间和 SSD 磨损方面的成本。

#. 找一个要替换掉 Filestore OSD::

     ID=<osd-id-number>
     DEVICE=<disk-device>

   #. 判断指定的 OSD 是 Filestore 还是 BlueStore ：

      .. prompt:: bash $

         ceph osd metadata $ID | grep osd_objectstore

   #. 获取 Filestore 和 BlueStore OSD 当前的数量：

      .. prompt:: bash $

         ceph osd count-metadata osd_objectstore

#. 把一个 Filestore OSD 标记为 ``out`` ：

   .. prompt:: bash $

      ceph osd out $ID

#. 等待数据迁移出这个 OSD ：

   .. prompt:: bash $

      while ! ceph osd safe-to-destroy $ID ; do sleep 60 ; done

#. 停止 OSD ：

   .. prompt:: bash $

      systemctl kill ceph-osd@$ID

   .. _osd_id_retrieval: 

#. 记下这个 OSD 在使用哪个设备：

   .. prompt:: bash $

      mount | grep /var/lib/ceph/osd/ceph-$ID

#. 卸载这个 OSD ：

   .. prompt:: bash $

      umount /var/lib/ceph/osd/ceph-$ID

#. 销毁 OSD 的数据。请 *务必小心*\ ！
   这些命令将销毁设备的内容；在继续操作之前，
   必须确保设备上的数据不再需要（换句话说，集群状态良好）：

   .. prompt:: bash $

      ceph-volume lvm zap $DEVICE

#. 告诉集群这个 OSD 已经销毁了
   （并且可以用同一个 OSD ID 重新部署新 OSD 了）：

   .. prompt:: bash $

      ceph osd destroy $ID --yes-i-really-mean-it

#. 用同一个 OSD ID 在原地部署 BlueStore OSD 。
   这就要求你根据 :ref:`"注意 OSD 正在使用的设备" <osd_id_retrieval>`
   这一步骤中检索到的信息，确定要擦除的设备、
   并确保目标设备是正确且预期的设备。
   **务必小心！** 注意，在操作混合式 OSD 时，
   你可能需要修改这些命令：

   .. prompt:: bash $

      ceph-volume lvm create --bluestore --data $DEVICE --osd-id $ID

#. 如此往复。

您可以选择 (1) 在替换 BlueStore OSD 的同时，对下一个 Filestore OSD 进行数据迁移，
或者 (2) 并行地对多个 OSD 执行相同的操作。但无论哪种情况，在销毁任何 OSD 之前，
必须确保集群完全一致（即所有数据都复制够了所有副本）。
如果你选择并行地重新配置多个 OSD ，\ **务必小心**\ ，
只能销毁单个 CRUSH 故障域（例如 ``host`` 或 ``rack`` ）内的 OSD 。
如果不满足此要求，将降低数据的冗余性和可用性，
而且增加了数据丢失的风险（甚至导致数据丢失）。 

优点：

* 简单。
* 可以按设备逐个地完成。
* 不需要备用设备或主机。

缺点：

* 数据通过网络复制两次：一次是复制到集群中的另一个 OSD
  （以维持指定的副本数），另一次是复制回重新配置过的 BlueStore OSD 。


“整机”替换
----------
.. "Whole host" replacement

如果集群中有备用主机，或者有足够的空闲空间来腾出一整台主机作为备用，
那么就可以逐台主机进行转换，这样存储的每个数据副本只需迁移一次。 

要用这种方法，你需要一台没配置过 OSD 的空主机。有两种方法可以实现：
一是使用全新的、尚未加入集群的空主机，二是从已加入集群的现有主机上卸载数据。 


利用新的、空主机
^^^^^^^^^^^^^^^^
.. Using a new, empty host

理想情况下，一台主机的容量应该与将要转换的其他每台主机的容量差不多。
把这台主机加进 CRUSH 分级结构中，但不要把它绑到 root 节点：

.. prompt:: bash $

   NEWHOST=<empty-host-name>
   ceph osd crush add-bucket $NEWHOST host

确保在新主机上安装了 Ceph 软件包。


利用已有主机
^^^^^^^^^^^^
.. Using an existing host

如果你想用已经是集群一部分的现有主机，并且该主机上有足够的空闲空间，
足以将所有数据迁移到集群内的其他主机上，
那么您可以采用以下步骤（而不是用新的空主机）：

.. prompt:: bash $

   OLDHOST=<existing-cluster-host-to-offload>
   ceph osd crush unlink $OLDHOST default

其中， "default" 是 CRUSH 图中的直接父节点。
（对于配置未修改的小型集群，这通常为 "default" ，
但也可能是一个机架名。）现在，
您应该会在 OSD 树输出的顶部看到该主机，而且它没有父节点：

.. prompt:: bash $

   bin/ceph osd tree

::

  ID CLASS WEIGHT  TYPE NAME     STATUS REWEIGHT PRI-AFF
  -5             0 host oldhost
  10   ssd 1.00000     osd.10        up  1.00000 1.00000
  11   ssd 1.00000     osd.11        up  1.00000 1.00000
  12   ssd 1.00000     osd.12        up  1.00000 1.00000
  -1       3.00000 root default
  -2       3.00000     host foo
   0   ssd 1.00000         osd.0     up  1.00000 1.00000
   1   ssd 1.00000         osd.1     up  1.00000 1.00000
   2   ssd 1.00000         osd.2     up  1.00000 1.00000
  ...

如果一切安好，直接跳到下面 :ref:`"等待数据迁移完成"
<bluestore_data_migration_step>` 这个步骤，然后从那里开始清理旧 OSD 。

迁移过程
^^^^^^^^
.. Migration process

如果你用的是新主机，从 :ref:`第一步 <bluestore_migration_process_first_step>`
开始；如果你用的是现有主机，跳转到 
:ref:`这一步 <bluestore_data_migration_step>` 。

.. _bluestore_migration_process_first_step:

#. 在所有设备上配置新的 BlueStore OSD ：

   .. prompt:: bash $

      ceph-volume lvm create --bluestore --data /dev/$DEVICE

#. 核对一下这个 OSD 已加入集群：

   .. prompt:: bash $

      ceph osd tree

   你应该能看到这个新主机 ``$NEWHOST`` ，它下面还挂着所有的
   OSD ，但是这台主机\ *不应该*\ 嵌入分级结构的其它任何节点\
   （像 ``root default`` ）。例如，假设 ``newhost`` 就是这台\
   空主机，你可能看到类似的： ::

     $ bin/ceph osd tree
     ID CLASS WEIGHT  TYPE NAME     STATUS REWEIGHT PRI-AFF
     -5             0 host newhost
     10   ssd 1.00000     osd.10        up  1.00000 1.00000
     11   ssd 1.00000     osd.11        up  1.00000 1.00000
     12   ssd 1.00000     osd.12        up  1.00000 1.00000
     -1       3.00000 root default
     -2       3.00000     host oldhost1
      0   ssd 1.00000         osd.0     up  1.00000 1.00000
      1   ssd 1.00000         osd.1     up  1.00000 1.00000
      2   ssd 1.00000         osd.2     up  1.00000 1.00000
     ...

#. 找出第一台要转换的主机：

   .. prompt:: bash $

      OLDHOST=<existing-cluster-host-to-convert>

#. 交换集群里新旧主机的位置：

   .. prompt:: bash $

      ceph osd crush swap-bucket $NEWHOST $OLDHOST

   此时， ``$OLDHOST`` 上的所有数据将开始迁移到 ``$NEWHOST`` 上的 OSD 。
   如果旧主机的总容量与新主机的总容量有差异，
   您可能还会看到一些数据迁移到集群中的其他节点，
   或从其他节点迁移出来。然而，
   如果主机的大小相似，那么迁移的数据量将相对较少。

   .. _bluestore_data_migration_step:

#. 等待数据迁移完成：

   .. prompt:: bash $

      while ! ceph osd safe-to-destroy $(ceph osd ls-tree $OLDHOST); do sleep 60 ; done

#. 现在 ``$OLDHOST`` 主机空了，停止其上的所有旧 OSD ：

   .. prompt:: bash $

      ssh $OLDHOST
      systemctl kill ceph-osd.target
      umount /var/lib/ceph/osd/ceph-*

#. 销毁并擦除旧的 OSD ：

   .. prompt:: bash $

      for osd in `ceph osd ls-tree $OLDHOST`; do
         ceph osd purge $osd --yes-i-really-mean-it
      done

#. 擦除旧的 OSD 。你得手动找出要擦除的设备。
   **谨慎！** 在每个设备上：

   .. prompt:: bash $

      ceph-volume lvm zap $DEVICE

#. 把当前为空的主机作为新主机，重复：

   .. prompt:: bash $

      NEWHOST=$OLDHOST

优点：

* 数据通过网络只复制一次。
* 整台主机上的 OSD 一次就能转换完毕。
* 可以并行化，有可能实现多台主机同时转换。
* 此过程中涉及的所有主机都不需要备用设备。

缺点：

* 需要一台备用主机。
* 整台主机上健在的 OSD 将会同时迁移数据。
  这很可能影响整个集群的性能。
* 所有迁移的数据仍然需要通过网络完整地进行一次传输。


逐个 OSD 设备进行复制
---------------------
.. Per-OSD device copy

单个逻辑 OSD 可以用 ``ceph-objectstore-tool`` 中的 ``copy`` 功能来转换。
这要求主机有一或多个空闲设备来配置一个新的、空的 BlueStore OSD 。
例如，如果集群中的每个主机有 12 个 OSD ，那么你需要第 13 个未使用的 OSD ，
这样就能在回收前一个 OSD 之后逐个转换下一个 OSD 。

注意事项：

* 此方法要求我们准备一个空的 BlueStore OSD ，但不给它分配新 OSD ID 。
  ``ceph-volume`` 工具不支持这样的操作。 **重要提示**\ ：
  由于 *dmcrypt* 的配置与 OSD 的身份识别紧密相关，
  因此这种方法不适用于加密 OSD 。

* 这个设备必须手动分区。

* 有个无支持的、用户贡献的脚本，演示了这个过程，脚本在这里：
  https://github.com/ceph/ceph/blob/master/src/script/contrib/ceph-migrate-bluestore.bash

优点：

* 转换期间，只要 OSD 或集群上设置了
  `noout` 或者 `norecover`/`norebalance`  标记，
  就只会有少量或没有数据通过网络迁移。

缺点：

* 工具链尚未完全实现、支持、或有文档；

* 每台主机必须有一个合适的备用或空闲设备，用于腾挪。

* 在转换期间， OSD 处于离线状态，这意味着\
  在目标 OSD 启动并恢复之前，对 acting set 中\
  包含这个 OSD 的 PG 的新写入可能不会实现预期的冗余度。
  这会增加由于重叠故障而导致数据丢失的风险。
  然而，如果在转换和启动完成之前另一个 OSD 发生故障，
  还可以启动原先的 Filestore OSD 以访问其原始数据。
