====================
 安装 Ceph 对象网关
====================

从 `firefly` (v0.80) 起， Ceph 对象网关开始运行在 Civetweb
（嵌入进了 ``ceph-radosgw`` 守护进程）之上，不再依赖 Apache
和 FastCGI 。基于 Civetweb 的用法简化了 Ceph 对象网关的安装\
和配置。

.. note:: 要提供 Ceph 对象网关服务，你得有正常运行的 Ceph
   存储集群、且它的网关主机能访问存储集群的公共网。

.. note:: 0.80 版时， Ceph 对象网关还不支持 SSL ，你可以配\
   置个支持 SSL 的反向代理，用来把 HTTPS 请求脱壳为 HTTP 、\
   并调度到 Civetweb 。


安装前的准备工作
----------------

参考\ `飞前检查`_\ ，先在 Ceph 对象网关节点上完成安装前的准\
备工作。特别要注意， Ceph 部署用户的 ``requiretty`` 需禁用，
SELinux 需设置为 ``Permissive`` ， Ceph 部署用户执行
``sudo`` 要配置为无需密码。对于 Ceph 对象网关，你得开放
Civetweb 在生产环境下要用到的端口。

.. note:: Civetweb 默认使用 ``7480`` 端口。


安装 Ceph 对象网关
------------------

在管理服务器的工作目录下，把 Ceph 对象网关软件包安装到 Ceph
对象网关节点上。例如： ::

        ceph-deploy install --rgw <gateway-node1> [<gateway-node2> ...]

其中， ``ceph-common`` 软件包是依赖，所以 ``ceph-deploy``
也会安装它。 ``ceph`` 命令行工具包是用于管理的。要想让你的
Ceph 对象网关节点兼具管理节点的功能，可以在管理节点的工作\
目录下执行下列命令： ::

        ceph-deploy admin <node-name>


创建网关例程
------------

在管理服务器的工作目录下，为 Ceph 网关节点创建一个例程。\
例如： ::

        ceph-deploy rgw create <gateway-node1>

网关运行后，你就应该能通过 ``7480`` 端口访问了，比如用下面\
这个尚未认证的请求： ::

        http://client-node:7480

如果网关例程可以正常工作，你应该能看到类似如下的响应： ::

        <?xml version="1.0" encoding="UTF-8"?>
        <ListAllMyBucketsResult xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
            <Owner>
                <ID>anonymous</ID>
                <DisplayName></DisplayName>
            </Owner>
            <Buckets>
            </Buckets>
        </ListAllMyBucketsResult>

如果当中哪一步遇到问题了，想重来一次，可以用下列命令清除配置： ::

        ceph-deploy purge <gateway-node1> [<gateway-node2>]
        ceph-deploy purgedata <gateway-node1> [<gateway-node2>]

如果你执行的是 ``purge`` ，那你得重新安装 Ceph 软件包。


.. _Change the Default Port:

更改默认端口
------------

Civetweb 默认使用 ``7480`` 端口。要改变默认端口（如改成 ``80``
端口），可以在管理服务器的工作目录下修改配置文件，新增名为
``[client.rgw.<gateway-node>]`` 的段，其中，要把
``<gateway-node>`` 替换为你 Ceph 对象网关节点的短主机名（即
``hostname -s`` ）。

.. note:: 从 11.0.1 版起， Ceph 对象网关\ **才支持** SSL 。\
   如何配置请参考\ `在 Civetweb 上使用 SSL`_ 。

例如，假设你的节点名为 ``gateway-node1`` ，可以在 ``[global]``
段之后加这样一段： ::

        [client.rgw.gateway-node1]
        rgw_frontends = "civetweb port=80"

.. note:: 切记，在 ``rgw_frontends`` 键值对里的
   ``port=<port-number>`` 之中不要加空格。
   ``[client.rgw.gateway-node1]`` 标题表明，这部分 Ceph
   配置文件是用于配置 Ceph 存储集群客户端的、客户端类型\
   为 Ceph 对象网关（即 ``rgw`` ）、且例程名字是
   ``gateway-node1`` 。

把更新过的配置文件推送到 Ceph 对象网关节点（以及其它 Ceph
节点）： ::

        ceph-deploy --overwrite-conf config push <gateway-node> [<other-nodes>]

要使新的端口配置生效，重启 Ceph 对象网关即可： ::

        sudo systemctl restart ceph-radosgw.service

最后，检查下你选用的端口在防火墙上是否开放了（如 ``80``
端口），如果没开放，加上这个端口并重载防火墙配置。如果你\
用的是 ``firewalld`` 守护进程，可执行： ::

        sudo firewall-cmd --list-all
        sudo firewall-cmd --zone=public --add-port 80/tcp --permanent
        sudo firewall-cmd --reload

如果用的是 ``iptables`` ，可执行： ::

        sudo iptables --list
        sudo iptables -I INPUT 1 -i <iface> -p tcp -s <ip-address>/<netmask> --dport 80 -j ACCEPT

需把命令中的 ``<iface>`` 和 ``<ip-address>/<netmask>`` 替\
换成你的 Ceph 对象网关节点上的相关值。

配置完 ``iptables`` 之后，你得保存配置，如此一来你的 Ceph
对象网关节点重启后防火墙配置还依然有效。命令如下： ::

        sudo apt-get install iptables-persistent

会出现一个终端对话框。提示你是否把当前 iptables 的 ``IPv4``
规则保存为 ``/etc/iptables/rules.v4`` 、 ``IPv6`` 规则保存为
``/etc/iptables/rules.v6`` 时选 ``yes`` 。

前面我们配置的 ``IPv4`` 的 iptables 规则会从
``/etc/iptables/rules.v4`` 载入，且重启后仍然有效。

如果在安装 ``iptables-persistent`` 之后你还新增过 ``IPv4``
的 iptables 规则，你还得把它们手动存入规则文件，这时，你得\
以 ``root`` 用户执行下面的命令： ::

        iptables-save > /etc/iptables/rules.v4


.. _Using SSL with Civetweb:

在 Civetweb 上使用 SSL
----------------------

要在 civetweb 上启用 SSL ，首先你需要一个证书、它应该与提供
Ceph 对象网关服务的域名相对应。考虑到灵活性，你也许应该申请\
有 `subject alternate name` 字段的证书；如果你想使用 S3 风\
格的子域名（\ `增加 DNS 通配符`_\ ），你得申请通配型证书（
`wildcard certificate` ）。

.. note:: 译者注：你可以采用自认证证书。有些客户端 API 会检\
   查可信证书颁发机构，这样的话你只能向可信机构（如
   https://letsencrypt.org ）申请 SSL 证书，才能使用那些客\
   户端 API 。

Civetweb 要求服务器密钥、服务器证书、以及其它 CA 或者中间证\
书都在同一个文件里，而且它们都必须是 `pem` 格式。这个整合的\
文件包含了密钥，所以应该把它保护起来，防止未经授权的访问。

要启用 SSL ，在端口号后面加 ``s`` 。如： ::

        [client.rgw.gateway-node1]
        rgw_frontends = civetweb port=443s ssl_certificate=/etc/ceph/private/keyandcert.pem

.. versionadded:: Luminous

此外， civetweb 还能配置为绑定多个端口，在配置时用 ``+`` 分隔\
多个端口号即可。如此一来，单个 rgw 例程就能满足需要同时开放
SSL 和非 SSL 连接的场景。例如： ::

        [client.rgw.gateway-node1]
        rgw_frontends = civetweb port=80+443s ssl_certificate=/etc/ceph/private/keyandcert.pem


.. Additional Civetweb Configuration Options

额外的 Civetweb 配置选项
------------------------

还有些额外配置选项可用来调整嵌入式的 Civetweb 网页服务器，位于
``ceph.conf`` 文件的 **Ceph 对象网关**\ 一节。
支持的选项、包括例子，都在 `HTTP 前端`_\ 里。


从 Apache 迁移到 Civetweb
-------------------------

如果你是基于 Apache 和 FastCGI 运行 Ceph 对象网关的， 而且
Ceph 存储版本是 v0.80 或更高，那么你有条件切换到 Civetweb
——它随 ``ceph-radosgw`` 守护进程启动、默认运行在 7480 端口\
上，所以不会与 Apache 和 FastCGI 冲突，与其它常见的网页服\
务端口也没冲突。向 Civetweb 迁移主要包括删除 Apache 软件\
包、清除 Ceph 配置文件里的 Apache 和 FastCGI 配置、把
``rgw_frontends`` 重置为 Civetweb 。

根据前面的用 ``ceph-deploy`` 安装 Ceph 对象网关里面的描述，\
当时的配置文件只配置了 ``rgw_frontends`` （而且假设你更改了\
默认端口）， ``ceph-deploy`` 工具为你生成了数据目录和密钥环\
——且把密钥环放到了 ``/var/lib/ceph/radosgw/{rgw-intance}`` 。\
这个守护进程会到默认位置找密钥，除非你在 Ceph 配置文件里另有\
配置。你已经有密钥和数据目录了，如果不想用默认路径，你必须在
Ceph 配置文件里单独维护这些路径。

如果基于 Apache 部署，典型的 Ceph 对象网关配置文件类似如下：

在 Red Hat Enterprise Linux 上： ::

        [client.radosgw.gateway-node1]
        host = {hostname}
        keyring = /etc/ceph/ceph.client.radosgw.keyring
        rgw socket path = ""
        log file = /var/log/radosgw/client.radosgw.gateway-node1.log
        rgw frontends = fastcgi socket\_port=9000 socket\_host=0.0.0.0
        rgw print continue = false

在 Ubuntu 上： ::

        [client.radosgw.gateway-node]
        host = {hostname}
        keyring = /etc/ceph/ceph.client.radosgw.keyring
        rgw socket path = /var/run/ceph/ceph.radosgw.gateway.fastcgi.sock
        log file = /var/log/radosgw/client.radosgw.gateway-node1.log

要改成基于 Civetweb 的，删除 Apache 的专有配置即可，如
``rgw_socket_path`` 和 ``rgw_print_continue`` ；然后，把
``rgw_frontends`` 配置内容改成 Civetweb 的、而非 Apache
FastCGI 前端，并指定你想用的端口。例如： ::

        [client.radosgw.gateway-node1]
        host = {hostname}
        keyring = /etc/ceph/ceph.client.radosgw.keyring
        log file = /var/log/radosgw/client.radosgw.gateway-node1.log
        rgw_frontends = civetweb port=80

最后，重启 Ceph 对象网关进程。在 Red Hat Enterprise Linux
上执行： ::

        sudo systemctl restart ceph-radosgw.service

在 Ubuntu 上执行： ::

        sudo service radosgw restart id=rgw.<short-hostname>

如果你选用的端口没有开放，你还得在防火墙上开放它。


桶分片的配置
------------

Ceph 对象网关把桶的索引数据存储在 ``index_pool`` 里面，它\
默认是 ``.rgw.buckets.index`` 。有时候，用户们会用单个桶放\
置很多（数十到数百万个对象）对象，如果你没用网关管理接口配\
置配额、限制单个桶允许的最大对象数，这个桶索引的性能就会明\
显下降。

在 Ceph 0.94 版中，如果你允许桶内有很多对象，可以把桶索引\
分片，以此来防止性能瓶颈。
``rgw_override_bucket_index_max_shards`` 选项允许你设置每\
个桶所允许的最大对象数，其默认值是 ``0`` ，也就是说桶索引\
的分片功能默认是关闭的。

要打开桶索引分片功能，把
``rgw_override_bucket_index_max_shards`` 设置为大于 ``0``
的值即可。

想简化配置的话，把 ``rgw_override_bucket_index_max_shards``
选项写入 Ceph 配置文件的 ``[global]`` 段下即可，这样它是系\
统级的配置，你也可以分别给各个例程单独配置。

在 Ceph 配置文件里改完桶的分片配置后，需重启网关进程。在
Red Hat Enteprise Linux 上需执行： ::

        sudo systemctl restart ceph-radosgw.service

在 Ubuntu 上需执行： ::

        sudo service radosgw restart id=rgw.<short-hostname>

在联盟化配置中，为实现故障切换，各区域的 ``index_pool`` 可\
能配置得不一样，为使一个 zonegroup 内的各个域的配置保持一致，\
你可以在网关的 zonegroup 配置中设置
``rgw_override_bucket_index_max_shards`` 。例如： ::

        radosgw-admin zonegroup get > zonegroup.json

打开 ``zonegroup.json`` 文件、编辑相关各域的
``bucket_index_max_shards`` 选项，保存到 ``zonegroup.json``
文件、并重置此 zonegroup ，例如： ::

        radosgw-admin zonegroup set < zonegroup.json

更新完 zonegroup 后，再更新并提交 period ，例如： ::

   radosgw-admin period update --commit

.. note:: 把索引存储池（每个域的索引存储池，可能的话）映射到\
   使用 SSD 的 OSD 组成的 CRUSH 规则也有助于提升桶索引性能。


.. Add Wildcard to DNS

增加 DNS 通配符
---------------

要想在 Ceph 上使用 S3 风格的子域名（如
bucket-name.domain-name.com ），需要在 DNS 服务器上给这个
``ceph-radosgw`` 守护进程使用的域名加一条通配符记录。

DNS 的地址也必须写入 Ceph 配置文件，选项为
``rgw dns name = {hostname}`` 。

对 ``dnsmasq`` 来说，需增加下面的 address 选项，在主机名前\
加一点（ . ）： ::

        address=/.{hostname-or-fqdn}/{host-ip-address}

例如： ::

        address=/.gateway-node1/192.168.122.75

对 ``bind`` 来说，需加一条通配符记录，例如： ::

        $TTL    604800
        @       IN      SOA     gateway-node1. root.gateway-node1. (
                                      2         ; Serial
                                 604800         ; Refresh
                                  86400         ; Retry
                                2419200         ; Expire
                                 604800 )       ; Negative Cache TTL
        ;
        @       IN      NS      gateway-node1.
        @       IN      A       192.168.122.113
        *       IN      CNAME   @

重启 DNS 服务器、然后用一个子域名 ping 网关服务器，以确保 DNS \
解析配置无误： ::

        ping mybucket.{hostname}

例如： ::

        ping mybucket.gateway-node1


开启调试选项（如有必要）
------------------------

完成上述配置后，如果它们不如所愿，可以在 Ceph 配置文件的
``[global]`` 段下加入调试选项，并重启网关，试着排除配置问\
题。例如： ::

        [global]
        #append the following in the global section.
        debug ms = 1
        debug rgw = 20


试用网关
--------

要使用 REST 接口，首先得创建一个适用于 S3 接口的 Ceph 对象\
网关用户，然后创建用于 Swift 接口的子用户。之后验证创建的用\
户是否能访问网关。


创建用于 S3 访问的 RADOSGW 用户
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

需创建 ``radosgw`` 用户、并授予权限。其它的命令选项可以查看
``man radosgw-admin`` 。

创建用户可以在 ``gateway host`` 上执行下面的命令： ::

        sudo radosgw-admin user create --uid="testuser" --display-name="First User"

此命令的输出类似如下： ::

        {
            "user_id": "testuser",
            "display_name": "First User",
            "email": "",
            "suspended": 0,
            "max_buckets": 1000,
            "auid": 0,
            "subusers": [],
            "keys": [{
                "user": "testuser",
                "access_key": "I0PJDPCIYZ665MW88W9R",
                "secret_key": "dxaXZ8U90SXydYzyS5ivamEP20hkLSUViiaR+ZDA"
            }],
            "swift_keys": [],
            "caps": [],
            "op_mask": "read, write, delete",
            "default_placement": "",
            "placement_tags": [],
            "bucket_quota": {
                "enabled": false,
                "max_size_kb": -1,
                "max_objects": -1
            },
            "user_quota": {
                "enabled": false,
                "max_size_kb": -1,
                "max_objects": -1
            },
            "temp_url_keys": []
        }

.. note:: 访问验证需要 ``keys->access_key`` 和
   ``keys->secret_key`` 的值。

.. important:: 仔细检查输出的密钥。有时候 ``radosgw-admin``
   生成的 JSON 里的 ``access_key`` 或 ``secret_key`` 里面有\
   转义字符 ``\`` ，而有些客户端不能处理 JSON 转义字符。补\
   救方法可以是删除 JSON 转义字符 ``\`` 、并用引号把它们包\
   起来；重新生成不含 JSON 转义字符的 key ；或者手动指定 key
   和密钥。或者，如果 ``radosgw-admin`` 生成的 JSON 同时包\
   含转义字符 ``\`` 和正斜线 ``/`` ，如 ``\/`` ，只需删除
   JSON 转义字符即可，不要删除正斜线 ``/`` ，因为它是 key
   的合法字符。


创建 Swift 用户
^^^^^^^^^^^^^^^

如果要通过 Swift 接口访问，需创建 Swift 子用户。 Swift 用\
户的创建包含两步，第一步是创建用户，第二步是创建密钥。

在 ``gateway host`` 上执行以下各步骤：

创建 Swift 用户： ::

        sudo radosgw-admin subuser create --uid=testuser --subuser=testuser:swift --access=full

其输出类似如下： ::

        {
            "user_id": "testuser",
            "display_name": "First User",
            "email": "",
            "suspended": 0,
            "max_buckets": 1000,
            "auid": 0,
            "subusers": [{
                "id": "testuser:swift",
                "permissions": "full-control"
            }],
            "keys": [{
                "user": "testuser:swift",
                "access_key": "3Y1LNW4Q6X0Y53A52DET",
                "secret_key": ""
            }, {
                "user": "testuser",
                "access_key": "I0PJDPCIYZ665MW88W9R",
                "secret_key": "dxaXZ8U90SXydYzyS5ivamEP20hkLSUViiaR+ZDA"
            }],
            "swift_keys": [],
            "caps": [],
            "op_mask": "read, write, delete",
            "default_placement": "",
            "placement_tags": [],
            "bucket_quota": {
                "enabled": false,
                "max_size_kb": -1,
                "max_objects": -1
            },
            "user_quota": {
                "enabled": false,
                "max_size_kb": -1,
                "max_objects": -1
            },
            "temp_url_keys": []
        }

创建密钥： ::

        sudo radosgw-admin key create --subuser=testuser:swift --key-type=swift --gen-secret

其输出类似如下： ::

        {
            "user_id": "testuser",
            "display_name": "First User",
            "email": "",
            "suspended": 0,
            "max_buckets": 1000,
            "auid": 0,
            "subusers": [{
                "id": "testuser:swift",
                "permissions": "full-control"
            }],
            "keys": [{
                "user": "testuser:swift",
                "access_key": "3Y1LNW4Q6X0Y53A52DET",
                "secret_key": ""
            }, {
                "user": "testuser",
                "access_key": "I0PJDPCIYZ665MW88W9R",
                "secret_key": "dxaXZ8U90SXydYzyS5ivamEP20hkLSUViiaR+ZDA"
            }],
            "swift_keys": [{
                "user": "testuser:swift",
                "secret_key": "244+fz2gSqoHwR3lYtSbIyomyPHf3i7rgSJrF\/IA"
            }],
            "caps": [],
            "op_mask": "read, write, delete",
            "default_placement": "",
            "placement_tags": [],
            "bucket_quota": {
                "enabled": false,
                "max_size_kb": -1,
                "max_objects": -1
            },
            "user_quota": {
                "enabled": false,
                "max_size_kb": -1,
                "max_objects": -1
            },
            "temp_url_keys": []
        }


访问验证
^^^^^^^^

测试 S3 访问
""""""""""""

你得写个 Python 测试脚本并运行它来验证 S3 访问。这个 S3 访问测\
试脚本会连接到 ``radosgw`` 、创建新桶、并罗列所有桶。其中，
``aws_access_key_id`` 和 ``aws_secret_access_key`` 的值取自
``radosgw-admin`` 命令返回的 ``access_key`` 和 ``secret_key`` \
的值。

执行下列步骤：

#. 先要安装 ``python-boto`` 软件包： ::

    sudo yum install python-boto

#. 创建 Python 脚本： ::

    vi s3test.py

#. 把下面的内容写入这个脚本文件： ::

    import boto.s3.connection

    access_key = 'I0PJDPCIYZ665MW88W9R'
    secret_key = 'dxaXZ8U90SXydYzyS5ivamEP20hkLSUViiaR+ZDA'
    conn = boto.connect_s3(
            aws_access_key_id=access_key,
            aws_secret_access_key=secret_key,
            host='{hostname}', port={port},
            is_secure=False, calling_format=boto.s3.connection.OrdinaryCallingFormat(),
           )

    bucket = conn.create_bucket('my-new-bucket')
    for bucket in conn.get_all_buckets():
        print "{name} {created}".format(
            name=bucket.name,
            created=bucket.creation_date,
        )

   把上面的 ``{hostname}`` 替换成你配置了网关服务（即
   ``gateway host`` ）的主机名，把 ``{port}`` 替换为你配置\
   的 Civetweb 端口号。

#. 运行此脚本： ::

    python s3test.py

   其输出应该类似如下： ::

    my-new-bucket 2015-02-16T17:09:10.000Z


测试 Swift 访问
"""""""""""""""

Swift 访问可用 ``swift`` 命令行客户端验证，其命令行选项可用
``man swift`` 获取。

``swift`` 客户端的安装可用下列命令。在 Red Hat Enteprise
Linux 上： ::

        sudo yum install python-setuptools
        sudo easy_install pip
        sudo pip install --upgrade setuptools
        sudo pip install --upgrade python-swiftclient

在基于 Debian 的发行版上： ::

        sudo apt-get install python-setuptools
        sudo easy_install pip
        sudo pip install --upgrade setuptools
        sudo pip install --upgrade python-swiftclient

swift 访问可用下列命令验证： ::

        swift -V 1 -A http://{IP ADDRESS}:{port}/auth -U testuser:swift -K '{swift_secret_key}' list

把 ``{IP ADDRESS}`` 替换成网关服务器的公网 IP 地址；
``{swift_secret_key}`` 替换为创建 ``swift`` 用户（从
``radosgw-admin key create`` 命令的输出里提取）时取得的密\
钥； ``{port}`` 替换为你配置的 Civetweb 端口号（如默认的
``7480`` ），如果你不替换，它将默认为 ``80`` 端口。

例如： ::

        swift -V 1 -A http://10.19.143.116:7480/auth -U testuser:swift -K '244+fz2gSqoHwR3lYtSbIyomyPHf3i7rgSJrF/IA' list

其输出应该是： ::

        my-new-bucket


.. _飞前检查:  ../../start/quick-start-preflight
.. _HTTP 前端: ../../radosgw/frontends
