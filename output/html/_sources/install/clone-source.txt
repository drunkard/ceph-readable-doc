==================
 克隆 Ceph 源码库
==================

你可以去\ `位于 github 的 Ceph 源码库`_\ 克隆某个 Ceph 分支，先选择一个分支（默认\
是 ``master`` ），然后点击 **Download ZIP** 按钮。

.. _位于 github 的 Ceph 源码库: https://github.com/ceph/ceph


要克隆整个 git 源码库，你得先安装、配置 ``git`` 。


安装 Git
========

在 Debian/Ubuntu 上执行下列命令安装 ``git`` ： ::

	sudo apt-get install git

在 CentOS/RHEL 上执行下列命令安装 ``git`` ： ::

	sudo yum install git

相应地，你必须有 ``github`` 帐户。如果你还没有，去 `github.com`_ 注册一个，然后按\
照\ `设置 Git`_ 指引配置 git 。

.. _github.com: http://github.com
.. _设置 Git: http://help.github.com/linux-set-up-git


添加 SSH 密钥（可选）
=====================

如果你计划向 Ceph 贡献代码、或者想通过 SSH 克隆（ \
``git@github.com:ceph/ceph.git`` ），你必须生成一个 SSH 密钥对。

.. tip:: 如果你只是想克隆，不需要 SSH 密钥也可用 \
   ``git clone --recursive https://github.com/ceph/ceph.git`` 克隆。

执行如下命令生成 SSH 密钥对用于 ``github`` ： ::

	ssh-keygen

把此密钥对的公钥加入 ``github`` 帐户（本例假设用了默认路径）： ::

	cat .ssh/id_rsa.pub

复制公钥。

进入 ``github`` 帐户，点击 “Account Settings” （即 ``tools`` 图标），然后点击导\
航条左边的 “SSH Keys” 。

点击 “SSH Keys” 列表里的 “Add SSH key” ，给密钥起个名字，把复制的公钥粘帖进去，最\
后点击 “Add key” 按钮。


克隆源码
========

执行下列命令克隆源码库： ::

	git clone --recursive https://github.com/ceph/ceph.git

``git clone`` 完成后，你应该已经得到了一份完整的 Ceph 源码库。

.. tip:: 确保你获取到的源码库之内的各子模块都是最新的，运行 ``git status`` 确认。 ::

	cd ceph
	git status

   如果你的子模块过时了，运行： ::

	git submodule update --force --init --recursive


选择分支
========

克隆完源码和子模块后，你的源码库将默认位于 ``master`` 分支上，这是个不稳定开发分\
支，你也可以切换到其他分支上。

- ``master``: 不稳定开发分支；
- ``stable``: 缺陷修正分支；
- ``next``: 发布候选分支。

::

	git checkout master
