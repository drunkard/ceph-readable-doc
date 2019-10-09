=======================
 Ceph 对象网关快速入门
=======================

从 `firefly` (v0.80) 起， Ceph 存储系统极大地简化了 Ceph 对象\
网关的安装和配置；网关守护进程内嵌了 Civetweb ，这样你就无需\
再安装网页服务器或配置 FastCGI 了。另外， ``ceph-deploy`` 也\
能帮你安装网关软件包、生成密钥、配置数据目录、创建网关例程。

.. tip:: Civetweb 默认使用 ``7480`` 端口，所以你得开放 \
   ``7480`` 端口、或者在 Ceph 配置文件里设置想要的端口号（如
   ``80`` 端口）。

下面我们开始 Ceph 对象网关之旅：

安装 Ceph 对象网关
==================

#. 在 ``client-node`` 节点上执行安装前的准备工作。如果就用
   Civetweb 的默认端口 ``7480`` ，你得用 ``firefly-cmd`` 或
   ``iptables`` 先开放这个端口，详情见\ `飞前检查`_\ 。

#. 在你管理服务器上的工作目录下，把 Ceph 对象网关软件包装进
   ``client-node`` 节点。例如：::

    ceph-deploy install --rgw <client-node> [<client-node> ...]

创建这个 Ceph 对象网关例程
==========================

在你管理服务器上的工作目录下，创建 Ceph 对象网关例程到
``client-node`` 节点上。例如：::

    ceph-deploy rgw create <client-node>

这个网关开始运行后，你应该就能通过 ``7480`` 端口访问了。（如
``http://client-node:7480`` ）。

配置这个 Ceph 对象网关例程
==========================

#. 要更改默认端口（如改为 ``80`` ），可修改 Ceph 配置文件。增\
   加一段名为 ``[client.rgw.<client-node>]`` 的配置，把其中的
   ``<client-node>`` 改为你的 Ceph 客户端节点的短主机名（即 \
   ``hostname -s`` ）。例如，假设你的节点名为 \
   ``client-node`` ，就在 ``[global]`` 段之后新增：::

    [client.rgw.client-node]          
    rgw_frontends = "civetweb port=80"

   .. note:: 在 ``rgw_frontends`` 键值对里面，切记不要在 \
      ``port=<port-number>`` 之中加空格。

   .. important:: 如果你想用 80 端口，需确保没运行 Apache 服务\
      器，否则它会与 Civetweb 冲突。这种情况下，我们建议您卸载
      Apache 。

#. 要使配置生效，需重启 Ceph 对象网关。在 Red Hat Enterprise
   Linux 7 和 Fedora 上可用下列命令：::

    sudo systemctl restart ceph-radosgw.service

   在 Red Hat Enterprise Linux 6 和 Ubuntu 上可用下列命令：::

    sudo service radosgw restart id=rgw.<short-hostname>

#. 最后，确保你选用的端口在此节点的防火墙上（如 ``80`` 端口）\
   已开放。如果没开放，就加上这个端口、然后重载下防火墙配置。\
   例如：::

    sudo firewall-cmd --list-all
    sudo firewall-cmd --zone=public --add-port 80/tcp --permanent
    sudo firewall-cmd --reload

   关于用 ``firewall-cmd`` 或 ``iptables`` 配置防火墙的详情见\
   \ `飞前检查`_\ 。

   现在，你应该能发起未经认证的请求、并收到响应了。例如，发出\
   一个类似下面的无参数请求：::

    http://<client-node>:80

   应该会收到这样的响应：::

    <?xml version="1.0" encoding="UTF-8"?>
    <ListAllMyBucketsResult xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
      <Owner>
        <ID>anonymous</ID>
        <DisplayName></DisplayName>
      </Owner>
    	<Buckets>
      </Buckets>
    </ListAllMyBucketsResult>

其它管理和 API 方面的详情可参考 `Ceph 对象网关的配置`_\ 手册。


.. _Ceph 对象网关的配置: ../../radosgw/config-ref
.. _飞前检查: ../quick-start-preflight
