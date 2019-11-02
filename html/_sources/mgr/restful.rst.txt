rest 风格的模块
===============

REST 风格的模块提供了通过 SSL 加密连接访问集群状态的 REST 风格\
接口。


.. Enabling

启用此功能
----------
*restful* 模块用此命令启用： ::

  ceph mgr module enable restful

在 API　终结点上线前你还得按下面的文档配置一个 SSL 证书。默认\
情况下，此模块会在 ``8003`` 端口上接受此主机上所有 IPv4 和 IPv6
地址的 HTTPS 请求。


.. Securing

安全加固
--------

到 *restful* 的所有连接都是用 SSL 加密过的。用下列命令可以生成\
一个自签名的证书： ::

  ceph restful create-self-signed-cert

请注意，用自签名证书时，大多数客户端都需要多加个选项以允许这种\
连接，并平息警告消息。例如，如果 ``ceph-mgr`` 守护进程在同一主\
机上， ::

  curl -k https://localhost:8003/

要给一套设施做好安全加固，应该使用由证书颁发机构签署过的证书。\
例如，可以用类似如下的命令生成密钥对： ::

  openssl req -new -nodes -x509 \
    -subj "/O=IT/CN=ceph-mgr-restful" \
    -days 3650 -keyout restful.key -out restful.crt -extensions v3_ca

生成的 ``restful.crt`` 应该再由您所在组织的 CA （证书颁发机构）\
签署。签完后，你可以这样使用它： ::

  ceph config-key set mgr/restful/$name/crt -i restful.crt
  ceph config-key set mgr/restful/$name/key -i restful.key

其中 ``name`` 是 ``ceph-mgr`` 例程的名字（通常是主机名）。如果\
所有管理器例程共用同一证书，可以省略 ``$name`` 部分： ::

  ceph config-key set mgr/restful/crt -i restful.crt
  ceph config-key set mgr/restful/key -i restful.key


.. Configuring IP and port

IP 和端口的配置
---------------

和其它 REST 风格的 API 终结点一样， *restful* 也是绑定到一个 IP
和端口的。默认情况下，当前活跃的 ``ceph-mgr`` 守护进程会绑定到
8003 端口和本主机上所有的可用 IPv4 和 IPv6 地址。

由于各个 ``ceph-mgr`` 管理着它自己的 *restful* 例程，所以可能\
得分别配置它们。 IP 和端口可以通过配置键更改： ::

  ceph config set mgr mgr/restful/$name/server_addr $IP
  ceph config set mgr mgr/restful/$name/server_port $PORT

其中，i ``$name`` 是 ceph-mgr 守护进程的 ID （通常是主机名）。

这些配置也可以是集群范围的，而不只是特定于管理器的。例如， ::

  ceph config set mgr mgr/restful/server_addr $IP
  ceph config set mgr mgr/restful/server_port $PORT

如果没配置端口， *restful* 将绑定到 ``8003`` 端口；如果没配置\
地址， *restful* 将绑定到 ``::`` ，意思是所有可用的 IPv4 和
IPv6 地址。


.. Creating an API User
.. _creating-an-api-user:

创建一个 API 用户
-----------------

To create an API user, please run the following command::

  ceph restful create-key <username>

Replace ``<username>`` with the desired name of the user. For example, to create a user named
``api``::

  $ ceph restful create-key api
  52dffd92-a103-4a10-bfce-5b60f48f764e

The UUID generated from ``ceph restful create-key api`` acts as the key for the user.

To list all of your API keys, please run the following command::

  ceph restful list-keys

The ``ceph restful list-keys`` command will output in JSON::

  {
  	"api": "52dffd92-a103-4a10-bfce-5b60f48f764e"
  }

You can use ``curl`` in order to test your user with the API. Here is an example::

  curl -k https://api:52dffd92-a103-4a10-bfce-5b60f48f764e@<ceph-mgr>:<port>/server

In the case above, we are using ``GET`` to fetch information from the ``server`` endpoint.


.. Load balancer

负载均衡器
----------

请注意，\ *只有*\ 当下管理器活跃着时， *restful* 才能启动。\
查询 Ceph 集群状态以确定哪个管理器活跃着（例如
``ceph mgr dump`` ）。为了让 API 有一致的 URL ，而无须操心当\
前哪个管理器守护进程活跃着，你可以配置一个负载均衡器前端，用\
以把流量引导到可用的管理器终结点上。


.. Available methods

可用方法
--------

你可以浏览 ``/doc`` 终结点来获取完整的可用终结点列表，以及各\
终结点已实现的 HTTP 方法。

例如，如果你想用 ``/osd/<id>`` 终结点的 PATCH 方法，把 OSD
``1`` 的状态设置为 ``up`` ，可以用下面的 curl 命令： ::

  echo -En '{"up": true}' | curl --request PATCH --data @- --silent --insecure --user <user> 'https://<ceph-mgr>:<port>/osd/1'

或者用 python 这样做： ::

  $ python
  >> import requests
  >> result = requests.patch(
         'https://<ceph-mgr>:<port>/osd/1',
         json={"up": True},
         auth=("<user>", "<password>")
     )
  >> print result.json()

*result* 模块里实现的其它终结点包括：

* ``/config/cluster``: **GET**
* ``/config/osd``: **GET**, **PATCH**
* ``/crush/rule``: **GET**
* ``/mon``: **GET**
* ``/osd``: **GET**
* ``/pool``: **GET**, **POST**
* ``/pool/<arg>``: **DELETE**, **GET**, **PATCH**
* ``/request``: **DELETE**, **GET**, **POST**
* ``/request/<arg>``: **DELETE**, **GET**
* ``/server``: **GET**


.. The ``/request`` endpoint

``/request`` 终结点
-------------------

你用 **DELETE** 、 **POST** 或 **PATCH** 这些方法做过操作后，\
可以用 ``/request`` 终结点来轮询这个请求的状态。这些方法默认情\
况下是异步执行的，因为它们可能得花费较长的时间才能完成；你可以\
在请求的 URL 后面加上 ``?wait=1`` 来改变此行为，这样返回的请求\
就肯定是已完成的。

``/request`` 的 **POST** 方法提供了直接执行定义在
``src/mon/MonCommands.h`` 内的各个 ceph mon 命令的方法： ::

  COMMAND("osd ls " \
          "name=epoch,type=CephInt,range=0,req=false", \
          "show all OSD ids", "osd", "r", "cli,rest")

命令\ **前缀**\ 是 **osd ls** 。可选参数的名字是 **epoch** 、\
其类型是 ``CephInt`` ，即 ``integer`` 。如此一来，你可以用下面\
的 **POST** 请求来调用命令： ::

  $ python
  >> import requests
  >> result = requests.post(
         'https://<ceph-mgr>:<port>/request',
         json={'prefix': 'osd ls', 'epoch': 0},
         auth=("<user>", "<password>")
     )
  >> print result.json()
