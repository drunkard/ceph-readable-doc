====================
 安装 Ceph 对象网关
====================

.. note:: 要提供 Ceph 网关服务，你得有正常运行的 Ceph 集群、网关主机能访问\
   存储集群的公共网、且在基于 RPM 的发行版上 SELinux 应该处于 permissive \
   模式。

:term:`Ceph 对象网关`\ 守护进程运行在 Apache 和 FastCGI 之上。

要提供 :term:`Ceph 对象存储`\ 服务，你必须在对应主机（即 ``gateway host`` ）\
上安装 Apache 和 Ceph 对象网关守护进程。如果你打算运行联盟架构（多个 region 和\
区域）的 Ceph 对象存储服务，还必须安装同步代理。

.. note:: Ceph 以前需要 ``mod_fastcgi`` ，而现在需要的是 ``mod_proxy_fcgi`` 。

默认提供 Apache 2.4 的发行版（像 RHEL 7 、 CentOS 7 或 Ubuntu 14.04 \
``Trusty`` ）上已经有 ``mod_proxy_fcgi`` 模块了。在用 ``yum`` 安装 \
``httpd`` 包、或者用 ``apt-get`` 装 ``apache2`` 的时候，就已经安装了 \
``mod_proxy_fcgi`` 模块。

默认提供 Apache 2.2 的发行版（像 RHEL 6 、 CentOS 6 或 Ubuntu 12.04 \
``Precise`` ）， ``mod_proxy_fcgi`` 在单独的软件包中。在 \
**RHEL 6/CentOS 6** 上，需启用 ``EPEL 6`` 软件库，然后用 \
``yum install mod_proxy_fcgi`` 命令安装；在 **Ubuntu 12.04** 上，移植 \
``mod_proxy_fcgi`` 的工作正在进行，详情见 \
`ceph radosgw 需要适用于 apache 2.2 的 mod-proxy-fcgi`_ 。


安装 Apache
===========

要在 ``gateway host`` 上安装 Apache ，按如下步骤：

在基于 Debian 的发行版上，执行： ::

	sudo apt-get install apache2

在基于 RPM 的发行版上，执行： ::

	sudo yum install httpd


配置 Apache
===========

``gateway host`` 上的 Apache 配置还得改一下：

基于 Debian 的发行版
--------------------

#. 在 ``/etc/apache2/apache2.conf`` 里新增一行 ``ServerName`` 配置，其值是\
   服务器的全资域名（如 ``hostname -f`` ）： ::

	ServerName {fqdn}

#. 加载 ``mod_proxy_fcgi`` 模块。

   执行： ::

		sudo a2enmod proxy_fcgi

#. 启动 Apache 服务： ::

	sudo service apache2 start

基于 RPM 的发行版
-----------------

#. 打开 ``httpd.conf`` 文件： ::

	sudo vim /etc/httpd/conf/httpd.conf

#. 删掉此文件内 ``#ServerName`` 前的 # ，后面再加上服务器名字，应该是服务\
   器的全资域名（如 ``hostname -f`` ）： ::

	ServerName {fqdn}

#. 确保 ``/etc/httpd/conf/httpd.conf`` 里加载了 ``mod_proxy_fcgi`` 模块，\
   加上下面这段： ::

	<IfModule !proxy_fcgi_module>
	LoadModule proxy_fcgi_module modules/mod_proxy_fcgi.so
	</IfModule>

#. 编辑 ``/etc/httpd/conf/httpd.conf`` 里的 ``Listen 80`` 这一行，前面加上\
   用作网关服务器的 IP 地址，改成 ``Listen {IP ADDRESS}:80`` 这样的。

#. 启动 httpd 服务。

   执行： ::

		sudo service httpd start

   或者： ::

		sudo systemctl start httpd


启用 SSL
========

有些 REST 客户端默认使用 HTTPS ，所以应该考虑在 Apache 上启用 SSL 。请按下述\
步骤启用 SSL 。

.. note:: 你可以采用自认证证书。有些客户端 API 会检查可信证书颁发机构，这样\
   的话你得向可信机构申请 SSL 证书，才能使用那些客户端 API 。


基于 Debian 的发行版
--------------------

要在基于 Debian 的系统上启用 SSL ，执行下列步骤：

#. 确保装上了所需依赖。 ::

	sudo apt-get install openssl ssl-cert

#. 启用 SSL 模块。 ::

	sudo a2enmod ssl

#. 生成证书。 ::

	sudo mkdir /etc/apache2/ssl
	sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/ssl/apache.key -out /etc/apache2/ssl/apache.crt

#. 重启 Apache 。 ::

	sudo service apache2 restart

详情见 `Ubuntu 服务器指南`_\ 。


基于 RPM 的发行版
-----------------

在基于 RPM 的发行版上启用 SSL 可按如下步骤进行：

#. 确认所需软件包已安装。 ::

	sudo yum install mod_ssl openssl

#. 生成私钥。 ::

	openssl genrsa -out ca.key 2048

#. 生成 CSR 。 ::

	openssl req -new -key ca.key -out ca.csr

#. 生成证书。 ::

	openssl x509 -req -days 365 -in ca.csr -signkey ca.key -out ca.crt

#. 把前面的文件复制到合适的位置。 ::

	cp ca.crt /etc/pki/tls/certs
	cp ca.key /etc/pki/tls/private/ca.key
	cp ca.csr /etc/pki/tls/private/ca.csr

#. 更新 Apache 的 SSL 配置文件 ``/etc/httpd/conf.d/ssl.conf`` 。

   提供 ``SSLCertificateFile`` 的正确路径。 ::

		SSLCertificateFile /etc/pki/tls/certs/ca.crt

   提供 ``SSLCertificateKeyFile`` 的正确路径。 ::

		SSLCertificateKeyFile /etc/pki/tls/private/ca.key

   保存更改。

#. 重启 Apache 。

   执行： ::

		sudo service httpd restart

   或者： ::

		sudo systemctl restart httpd

详情见\ `用 CentOS 配置受 SSL 保护的网页服务器`_\ 。


安装 Ceph 对象网关守护进程
==========================

Ceph 对象存储服务通过 Ceph 对象网关守护进程（ ``radosgw`` ）提供网关。在联\
盟体系结构下，同步代理（ ``radosgw-agent`` ）可实现 region 和区域间的数据、元数\
据同步。


基于 Debian 的发行版
--------------------

执行下列命令在 ``gateway host`` 上安装 Ceph 对象网关守护进程： ::

	sudo apt-get install radosgw

执行下列命令安装 Ceph 对象网关同步代理： ::

	sudo apt-get install radosgw-agent


基于 RPM 的发行版
-----------------

执行下列命令在 ``gateway host`` 上安装 Ceph 对象网关守护进程： ::

	sudo yum install ceph-radosgw

执行下列命令安装 Ceph 对象网关同步代理： ::

	sudo yum install radosgw-agent


配置网关
========

安装好 Ceph 对象网关软件包之后，下一步就是配置 Ceph 对象网关，有两种方法：

- **简易配置：** `简易的`_ Ceph 对象网关配置意味着你在单个数据中心运行了一套 \
  Ceph 对象存储服务，所以你配置 Ceph 对象网关时无需考虑 region 和区域。

- **联盟配置：** `联盟的`_ Ceph 对象网关配置意味着你在地理上分散的方式运行着\
  一套 Ceph 对象存储服务，以实现更好的容灾和故障转移。这需要给 Ceph 对象网关\
  例程配置 region 和区域。

请选择最适合自己的方式。


.. _ceph radosgw 需要适用于 apache 2.2 的 mod-proxy-fcgi: https://bugs.launchpad.net/precise-backports/+bug/1422417
.. _Ubuntu 服务器指南: https://help.ubuntu.com/12.04/serverguide/httpd.html
.. _用 CentOS 配置受 SSL 保护的网页服务器: http://wiki.centos.org/HowTos/Https
.. _简易的: ../../radosgw/config
.. _联盟的: ../../radosgw/federated-config
