:orphan:

=================================
 rbdmap -- 在引导时映射 RBD 设备
=================================

.. program:: rbdmap

提纲
====

| **rbdmap map**
| **rbdmap unmap**


描述
====

**rbdmap** 是个 shell 脚本，用于自动化一到多个 RBD (RADOS
Block Device) 映像的 ``rbd map`` 和 ``rbd unmap`` 操作，同\
时这个脚本也可以由系统管理员随时手动运行，但主要的用途还是\
把启动时 RBD 映像的映射、挂载（还有关机时的卸载、取消映射）\
操作自动化，由 init 系统（一个 systemd 的 unit 文件，为此
ceph-common 软件包打包了 ``rbdmap.service`` ）触发。

此脚本只需一个参数，可以是 map 或 unmap 。运行时，脚本会分\
析配置文件（默认为 ``/etc/ceph/rbdmap`` ，但可以用环境变量
``RBDMAPFILE`` 覆盖），配置文件的每行对应一个要映射、或取消\
映射的 RBD 映像。

配置文件格式为： ::

    IMAGESPEC RBDOPTS

其中， ``IMAGESPEC`` 的格式应该是 ``POOLNAME/IMAGENAME`` （存\
储池名、一个斜线、和映像名），或只有 ``IMAGENAME`` ，此时
``POOLNAME`` 默认为 rbd 。 ``RBDOPTS`` 是可选项列表，包含要传\
入底层 ``rbd map`` 命令的参数，这些参数及其取值应该是逗号分隔\
的字符串： ::

    PARAM1=VAL1,PARAM2=VAL2,...,PARAMN=VALN 

此配置会让脚本执行如下的 ``rbd map`` 命令： ::

    rbd map POOLNAME/IMAGENAME --PARAM1 VAL1 --PARAM2 VAL2 

（ ``rbd`` 命令的可用选项请参考其手册页。）

运行 ``rbdmap map`` 时，此脚本先分析配置文件，对于每个配置了\
的 RBD 映像，首先尝试映射它（用 ``rbd map`` 命令），然后，尝\
试挂载此映像。

运行 ``rbdmap unmap`` 时，罗列在配置文件内的映像会被依次卸\
载、取消映射。

``rbdmap unmap-all`` 尝试卸载、然后取消当前映射的所有 RBD 映\
像，不管它们是否在配置文件里。

如果操作成功， ``rbd map`` 操作会把映像映射到类似
``/dev/rbdX`` 的设备，此时，会触发 udev 规则来创建友好的设备\
名符号链接，如 ``/dev/rbd/POOLNAME/IMAGENAME`` ，链接到映射\
的真实设备。

挂载、卸载功能要正常运行的话，友好的设备名在 ``/etc/fstab``
里面还必须有相应配置。

在 ``/etc/fstab`` 里面写 RBD 映像的配置条目时，最好加上
noauto （或者 nofail ）挂载选项。这样可防止 init 系统过早地\
挂载设备——甚至早于相应设备存在时。（正因为 ``rbdmap.service``
是执行 shell 脚本的，所以在引导过程中都触发得很晚。）


实例
====

配置实例 ``/etc/ceph/rbdmap`` 内含两个 RBD 映像，分别名为
bar1 和 bar2 ，都在 foopool 存储池内： ::

    foopool/bar1    id=admin,keyring=/etc/ceph/ceph.client.admin.keyring
    foopool/bar2    id=admin,keyring=/etc/ceph/ceph.client.admin.keyring

此文件的每一行都有两种字符串：映像说明、和传递给 ``rbd map``
的选项。这两行将被转换为如下的命令： ::

    rbd map foopool/bar1 --id admin --keyring /etc/ceph/ceph.client.admin.keyring
    rbd map foopool/bar2 --id admin --keyring /etc/ceph/ceph.client.admin.keyring

假设这些映像上的文件系统为 XFS ，其对应的 ``/etc/fstab`` 配\
置可能如下： ::

    /dev/rbd/foopool/bar1 /mnt/bar1 xfs noauto 0 0
    /dev/rbd/foopool/bar2 /mnt/bar2 xfs noauto 0 0

创建好映像、并写好 ``/etc/ceph/rbdmap`` 配置文件后，只需启用\
这个 unit 就可让映像在启动时自动映射和挂载： ::

    systemctl enable rbdmap.service


选项
====

无


使用范围
========

**rbdmap** 是 Ceph 的一部分，这是个伸缩力强、开源、分布式的\
存储系统，更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`rbd <rbd>`\(8),
