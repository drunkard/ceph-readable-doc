==================
 克隆 Ceph 源码库
==================
.. Cloning the Ceph Source Code Repository

你可以去\ `位于 github 的 Ceph 源码库`_\ 克隆某个 Ceph 分支，\
先选择一个分支（默认是 ``main`` ），然后点击 **Download ZIP** 按钮。

.. _位于 github 的 Ceph 源码库: https://github.com/ceph/ceph

要克隆整个 git 源码库，你得先\ :ref:`安装 <install-git>`\ 并配置 ``git`` 。


.. _install-git:

安装 Git
========

在 Debian/Ubuntu 上执行下列命令安装 ``git`` ：

.. prompt:: bash $

   sudo apt-get install git

在 CentOS/RHEL 上执行下列命令安装 ``git`` ：

.. prompt:: bash $

   sudo yum install git

相应地，你必须有 ``github`` 帐户。
如果你还没有，去 `github.com`_ 注册一个，
然后按照\ `设置 Git`_ 指引配置 git 。

.. _github.com: https://github.com
.. _设置 Git: https://help.github.com/linux-set-up-git



添加 SSH 密钥（可选）
=====================
.. Add SSH Keys (Optional)

如果你计划向 Ceph 贡献代码、或者想通过 SSH 克隆（ \
``git@github.com:ceph/ceph.git`` ），你必须生成一个 SSH 密钥对。

.. tip:: 如果你只是想克隆，不需要 SSH 密钥也可用
   ``git clone --recursive https://github.com/ceph/ceph.git`` 克隆。

执行如下命令生成 SSH 密钥对用于 ``github`` ：

.. prompt:: bash $

   ssh-keygen

打印出刚才生成的密钥，准备添加到你的 ``github`` 帐户，
用 ``cat`` 命令。（本例假设你用的是默认的文件路径）：

.. prompt:: bash $

   cat .ssh/id_rsa.pub

复制公钥。

进入 ``github`` 帐户，点击 “Account Settings”
（即 "tools" 图标），然后点击左侧导航条的 “SSH Keys” 。

点击 “SSH Keys” 列表里的 “Add SSH key” ，给密钥起个名字，
把复制的公钥粘帖进去，最后点击 “Add key” 按钮。


克隆源码
========
.. Clone the Source

执行下列命令克隆源码库：

.. prompt:: bash $

	git clone --recursive https://github.com/ceph/ceph.git

``git clone`` 完成后，你应该已经得到了一份完整的 Ceph 源码库。

.. tip:: 确保你获取到的源码库之内的各子模块都是最新的，
   运行 ``git status`` ，它会告诉你子模块是否过时了。
   详情见 :ref:`update-submodules` 。


.. prompt:: bash $

    cd ceph
    git status


.. _update-submodules:

更新子模块
----------
.. Updating Submodules

如果你的子模块（ submodule ）过时了，运行：

   .. prompt:: bash $

      git submodule update --force --init --recursive --progress
      git clean -fdx
      git submodule foreach git clean -fdx

如果你的子模块目录仍然有问题，用
``rm -rf [directory name]`` 删掉那个目录。然后再次运行
``git submodule update --init --recursive --progress`` 。


选择分支
========
.. Choose a Branch

克隆完源码和子模块后，你的源码库将默认位于 ``main`` 分支上，
这是个不稳定开发分支，你也可以切换到其他分支上。

- ``main``: 不稳定开发分支；
- ``stable-release-name``: 稳定的、 `活跃版本`_ 的名字，比如 ``Pacific`` ；
- ``next``: 发布候选分支。

::

	git checkout main

.. _活跃版本: https://docs.ceph.com/en/latest/releases/#active-releases
