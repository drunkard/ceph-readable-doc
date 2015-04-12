====================
 安装 Ceph 对象网关
====================

:term:`Ceph 对象网关`\ 守护进程运行在 Apache 和 FastCGI 之上。

要提供 :term:`Ceph 对象存储`\ 服务，你必须安装 Apache 和 FastCGI ，还必须安\
装 Ceph 对象网关守护进程。 Ceph 对象网关支持 100-continue 功能，但是要使用\
它，还必须安装 Ceph 社区构建的 Apache 和 FastCGI 软件包。要安装 Ceph 对象网\
关，先得安装并配置 Apache 和 FastCGI ，然后安装 Ceph 对象网关守护进程。如果\
你打算运行联盟架构（多个辖区和区域）的 Ceph 对象存储服务，还必须安装同步代理。

参考\ `获取软件包`_\ 把 Ceph 软件包安装到各节点，确保先在各节点上完成那些步骤。


Apache/FastCGI w/out 100-Continue
=================================

你可以用标准的 Apache 和 FastCGI 软件包做对象网关，只是它们不会提供 \
100-continue 功能。


Debian 软件包
-------------

要安装 Apache 和 FastCGI 的 Debian 软件包，可用下列命令： ::

	sudo apt-get install apache2 libapache2-mod-fastcgi


RPM 软件包
----------

要安装 Apache 和 FastCGI 的 RPM 包，可用下列命令： ::

	sudo rpm -ivh fcgi-2.4.0-10.el6.x86_64.rpm
	sudo rpm -ivh mod_fastcgi-2.4.6-2.el6.rf.x86_64.rpm

或者： ::

	sudo yum install httpd mod_fastcgi


Apache/FastCGI w/ 100-Continue
==============================

Ceph 社区提供了一个轻度优化过的 ``apache2`` 和 ``fastcgi`` 软件包，本质区别\
在于 Ceph 提供的包优化了 ``100-continue`` HTTP 响应支持，这样服务器会先评估\
请求头部、并以此决定是否接受请求。关于 ``100-continue`` 的详细信息见 \
`RFC 2616, Section 8`_ 。你可以在 `gitbuilder.ceph.com`_ 找到为 Ceph 优化过\
的最新 Apache 和 FastCGI 软件包。


Debian 软件包
-------------

#. 添加开发密钥： ::

	wget -q -O- https://raw.github.com/ceph/ceph/master/keys/autobuild.asc | sudo apt-key add -

#. 把 ``ceph-apache.list`` 文件加进你的 APT 源列表里。 ::

	echo deb http://gitbuilder.ceph.com/apache2-deb-$(lsb_release -sc)-x86_64-basic/ref/master $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph-apache.list

#. 把 ``ceph-fastcgi.list`` 文件加进你的 APT 源列表里。 ::

	echo deb http://gitbuilder.ceph.com/libapache-mod-fastcgi-deb-$(lsb_release -sc)-x86_64-basic/ref/master $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph-fastcgi.list

#. 更新软件库并安装 Apache 和 FastCGI::

	sudo apt-get update && sudo apt-get install apache2 libapache2-mod-fastcgi


RPM 软件包
----------

要安装支持 100-continue 的 Apache 服务器，可依照如下步骤：

#. 安装 ``yum-plugin-priorities`` 。 ::

	sudo yum install yum-plugin-priorities

#. 确认 ``/etc/yum/pluginconf.d/priorities.conf`` 文件存在。

#. 确认 ``priorities.conf`` 配置中已启用插件。 ::

	[main]
	enabled = 1

#. 把 ``ceph-apache.repo`` 文件放入 ``/etc/yum.repos.d`` ，用你自己的发行版\
   名字（如 ``centos6`` 、 ``rhel6`` 等）替换掉 ``{distro}`` ： ::

	[apache2-ceph-noarch]
	name=Apache noarch packages for Ceph
	baseurl=http://gitbuilder.ceph.com/apache2-rpm-{distro}-x86_64-basic/ref/master
	enabled=1
	priority=2
	gpgcheck=1
	type=rpm-md
	gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/autobuild.asc

	[apache2-ceph-source]
	name=Apache source packages for Ceph
	baseurl=http://gitbuilder.ceph.com/apache2-rpm-{distro}-x86_64-basic/ref/master
	enabled=0
	priority=2
	gpgcheck=1
	type=rpm-md
	gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/autobuild.asc

#. 把 ``ceph-fastcgi.repo`` 文件放入 ``/etc/yum.repos.d`` ，，用你自己的发行\
   版名字（如 ``centos6`` 、 ``rhel6`` 等）替换掉 ``{distro}`` ： ::

	[fastcgi-ceph-basearch]
	name=FastCGI basearch packages for Ceph
	baseurl=http://gitbuilder.ceph.com/mod_fastcgi-rpm-{distro}-x86_64-basic/ref/master
	enabled=1
	priority=2
	gpgcheck=1
	type=rpm-md
	gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/autobuild.asc

	[fastcgi-ceph-noarch]
	name=FastCGI noarch packages for Ceph
	baseurl=http://gitbuilder.ceph.com/mod_fastcgi-rpm-{distro}-x86_64-basic/ref/master
	enabled=1
	priority=2
	gpgcheck=1
	type=rpm-md
	gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/autobuild.asc

	[fastcgi-ceph-source]
	name=FastCGI source packages for Ceph
	baseurl=http://gitbuilder.ceph.com/mod_fastcgi-rpm-{distro}-x86_64-basic/ref/master
	enabled=0
	priority=2
	gpgcheck=1
	type=rpm-md
	gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/autobuild.asc

   如果此软件库没有 ``noarch`` 段，你可以删除上述 ``noarch`` 段。

#. 更新软件库。在 RHEL 系统上需启用 ``rhel-6-server-optional-rpms`` 库。 ::

	sudo yum update --enablerepo=rhel-6-server-optional-rpms

#. 安装 Apache 和 FastCGI 。 ::

	sudo yum update && sudo yum install httpd mod_fastcgi


配置 Apache/FastCGI
===================

结束安装前，要确认启用了重写模块和 FastCGI ，具体步骤因安装方式略有差异。


基于 Debian 的软件包
--------------------

#. 打开 ``apache2.conf`` 文件。 ::

	sudo vim /etc/apache2/apache2.conf

#. 在 Apache 配置文件中增加 ``ServerName`` 选项，内容为服务器主机的全资域名\
   （如 ``hostname -f`` ）。 ::

	ServerName {fqdn}

#. 为 Apache 和 FastCGI 启用 URL 重写模块。 ::

	sudo a2enmod rewrite
	sudo a2enmod fastcgi

#. 重启 Apache ，以确保前面的更改生效。 ::

	sudo service apache2 restart


基于 RPM 的软件包
-----------------

#. 打开 ``httpd.conf`` 文件。 ::

	sudo vim /etc/httpd/conf/httpd.conf

#. 删除 ``#ServerName`` 的注释并加上你的服务器名，应该是服务器的全资域名（如 \
   ``hostname -f`` ）。 ::

	ServerName {fqdn}

#. 确认重写模块已启用。 ::

	#if not present, add:
	LoadModule rewrite_module modules/mod_rewrite.so

#. 保存到 ``httpd.conf`` 文件。

#. 确认 FastCGI 模块已启用。安装器应该会包含一个 \
   ``/etc/httpd/conf.d/fastcgi.conf`` 文件，用于载入 FastCGI 模块。 ::

	# 若没有，新增此行：
	LoadModule fastcgi_module modules/mod_fastcgi.so

#. 重启 Apache ，使前述更改生效。 ::

	sudo /etc/init.d/httpd restart


启用 SSL
========

有些 REST 客户端默认使用 HTTPS ，所以应该考虑在 Apache 上启用 SSL 。请按下述\
步骤启用 SSL 。

.. note:: 你可以采用自认证证书。有些客户端 API 会检查可信证书颁发机构，这样\
   的话你得向可信机构申请 SSL 证书，才能使用那些客户端 API 。


Debian 软件包
-------------

要在 Debian/Ubuntu 系统上启用 SSL ，执行下列步骤：

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


RPM 软件包
----------

在基于 RPM 的系统上启用 SSL 可按如下步骤进行：

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

#. 重启 Apache 。 ::

	sudo /etc/init.d/httpd restart

详情见\ `用 CentOS 配置受 SSL 保护的网页服务器`_\ 。


使 DNS 支持通配符
=================

要通过 S3 风格（如 ``bucket-name.domain-name.com`` ）的子域名使用 Ceph ，你\
得在 DNS 服务器上加一条通配符记录，给对应的 ``radosgw`` 守护进程使用。

.. tip:: DNS 的地址也必须在 Ceph 配置文件里设置，用 \
   ``rgw dns name = {hostname}`` 选项。

对于 ``dnsmasq`` ，要配置 ``address`` 选项，主机名前要加一个点（ . ）： ::

	address=/.{hostname-or-fqdn}/{host-ip-address}
	address=/.ceph-node/192.168.0.1

对于 ``bind`` ，要加一条通配符记录： ::

	$TTL	604800
	@	IN	SOA	ceph-node. root.ceph-node. (
				      2		; Serial
				 604800		; Refresh
				  86400		; Retry
				2419200		; Expire
				 604800 )	; Negative Cache TTL
	;
	@	IN	NS	ceph-node.
	@	IN	A	192.168.122.113
	*	IN	CNAME	@

重启 DNS 服务器，并用一个子域 ping 一下服务器，以确认 Ceph 对象存储 \
``radosgw`` 守护进程能处理此子域的请求。 ::

	ping mybucket.{fqdn}
	ping mybucket.ceph-node


安装 Ceph 对象网关
==================

Ceph 对象存储服务通过 Ceph 对象网关守护进程（ ``radosgw`` ）提供网关。在联\
盟体系结构下，同步代理（ ``radosgw-agent`` ）可实现辖区和区域间的数据、元数\
据同步。


Debian 软件包
-------------

执行下列命令安装 Ceph 对象网关守护进程： ::

	sudo apt-get install radosgw

执行下列命令安装 Ceph 对象网关同步代理： ::

	sudo apt-get install radosgw-agent


RPM 软件包
----------

执行下列命令安装 Ceph 对象网关守护进程： ::

	sudo yum install ceph-radosgw ceph

执行下列命令安装 Ceph 对象网关同步代理： ::

	sudo yum install radosgw-agent


配置网关
========

安装好 Ceph 对象网关软件包之后，下一步就是配置 Ceph 对象网关，有两种方法：

- **简易配置：** `简易的`_ Ceph 对象网关配置意味着你在单个数据中心运行了一套 \
  Ceph 对象存储服务，所以你配置 Ceph 对象网关时无需考虑辖区和区域。

- **联盟配置：** `联盟的`_ Ceph 对象网关配置意味着你在地理上分散的方式运行着\
  一套 Ceph 对象存储服务，以实现更好的容灾和故障转移。这需要给 Ceph 对象网关\
  例程配置辖区和区域。

请选择最适合自己的方式。


.. _获取软件包: ../get-packages
.. _Ubuntu 服务器指南: https://help.ubuntu.com/12.04/serverguide/httpd.html
.. _用 CentOS 配置受 SSL 保护的网页服务器: http://wiki.centos.org/HowTos/Https
.. _RFC 2616, Section 8: http://www.w3.org/Protocols/rfc2616/rfc2616-sec8.html
.. _gitbuilder.ceph.com: http://gitbuilder.ceph.com
.. _Installing YUM Priorities: ../yum-priorities
.. _简易的: ../../radosgw/config
.. _联盟的: ../../radosgw/federated-config
