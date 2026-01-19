.. _documenting_ceph:

================
 贡献 Ceph 文档
================

想帮助 Ceph 项目可以贡献文档。
即使是小小的贡献也对 Ceph 项目有益。

纠正文档最简单的方法就是往 `ceph-users@ceph.io` 发送电子邮件。
在标题行写上 "ATTN: DOCS" 或者 "Attention: Docs"
或者 "Attention: Documentation" 。
在 email 正文，写上要纠正的文本
（以便我在代码库里找到它）、还有你的更正。

纠正文档的另外一种方法是发出一个拉取请求。
向 Ceph 文档提出拉取请求的说明位于
:ref:`making_contributions` 。

如果这是你第一次改进文档、或者假设你发现了一个小小的错误
（比如拼写错误或者错别字），发一封邮件比提出拉取请求更简单。
这次改进将归功于你，除非你向 Ceph 文档上游明确提出不要归功于你。


文档在源码库里的位置
====================
.. Location of the Documentation in the Repository

Ceph 文档源码位于 Ceph 源码库的 ``ceph/doc`` 目录下，\
由 Python Sphinx 渲染成 HTML 和手册页。

怎么看以前的 Ceph 文档
======================
.. Viewing Old Ceph Documentation

https://docs.ceph.com 链接默认展示最新版
（例如，假设 Reef 是最新版，那么
https://docs.ceph.com 默认展示的就是 Reef 的文档），
也可以查看较早版本的文档（例如 ``quincy`` ），
把 URL 里的版本名（如
`https://docs.ceph.com/en/reef/ <https://docs.ceph.com/en/reef>`_ 里的 ``reef`` ）
替换成你想看的分支即可（如 ``quincy`` ， URL 就成了
`https://docs.ceph.com/en/quincy/ <https://docs.ceph.com/en/quincy/>`_ ）。


.. _making_contributions:

如何贡献
========
.. Making Contributions

贡献文档和贡献源码的过程基本相同，\
唯一不同的就是你编译的是文档源码而不是程序源码。\
大致顺序（编译文档源码的顺序）如下：

#. `获取源代码`_
#. `进入分支`_
#. `开始更改`_
#. `构建文档源码`_
#. `提交变更`_
#. `推送变更`_
#. `发出接收请求`_
#. `通知我们`_

获取源代码
----------
.. Get the Source

Ceph 文档的源码是一系列 ReStructured Text 文本文件，
位于和源码同一仓库内的 ``ceph/doc`` 目录下，\
关于 Github 和 Ceph 的关系细节情况见\ :ref:`Get Involved`\ 。

贡献文档的方法是\ `分支并拉取`_\ 。\
为此，必须先做到：

#. 在本地安装 git 。在 Debian/Ubuntu 下用此命令：

   .. prompt:: bash $

	sudo apt-get install git

   在 Fedora 下用此命令：

   .. prompt:: bash $

	sudo yum install git

   在 CentOS/RHEL 下用此命令：

   .. prompt:: bash $

	sudo yum install git

#. 在 ``.gitconfig`` 配置文件里写好自己的名字和邮件地址。

   .. code-block:: ini

	[user]
	   email = {your-email-address}
	   name = {your-name}

   例如：

   .. prompt:: bash $

	git config --global user.name "John Doe"
	git config --global user.email johndoe@example.com


#. 创建 `github`_ 帐户（假如没有的话）。

#. 创建 Ceph 项目的分支，见 https://github.com/ceph/ceph 。

#. 把你自己的 Ceph 分支项目克隆到本机。
   这样就创建了“本地工作副本”。


Ceph 文档是按它的组件来分类组织的：

- **Ceph 存储集群：**\ Ceph 存储集群文档位于
  ``doc/rados`` 目录下；

- **Ceph 块设备：**\ Ceph 块设备文档位于
  ``doc/rbd`` 目录下；

- **Ceph 对象存储：**\ Ceph 对象存储文档位于
  ``doc/radosgw`` 目录下；

- **Ceph 文件系统：**\ Ceph 文件系统文档位于
  ``doc/cephfs`` 目录下；

- **安装（快速）：**\ 快速入门文档位于
  ``doc/start`` 目录下；

- **安装（手动）：**\ 手动安装文档位于
  ``doc/install`` 目录下；

- **手册页：**\ 手册源码位于 ``doc/man`` 目录下；

- **开发者：**\ 开发者文档位于 ``doc/dev`` 目录下；

- **图片：**\ 如果你想上传文档，如 JPEG 或 PNG 文件，
  应该放到 ``doc/images`` 目录下。


进入分支
--------
.. Select a Branch

如果只是细小的变更，像修正排版错误、或换一种措辞，
直接提交到 ``main`` 分支（默认的）即可；
为当前版本的功能提供文档时也应该提交到 ``main`` 分支。
``main`` 是最常用的分支。

.. prompt:: bash $

	git checkout main

给未来版本提供文档时应该提交到 ``next`` 分支，
``next`` 分支是第二常用的分支。

.. prompt:: bash $

	git checkout next

你在为尚未发布的功能写文档时，
如果这部分文档和已追踪的某个问题有关，
或者想在它被合并到 ``main`` 分支前看看它在 ceph.com 网站上的预览，
你应该另外创建个分支。为标识这是个只包含文档的更新，
按惯例用 ``wip-doc`` 作前缀，
按这个格式 ``wip-doc-{your-branch-name}`` 。
如果此分支和 http://tracker.ceph.com/issues 里的某个问题相关，
分支名最好包含问题编号，
例如，如果某文档分支是为 #4000 这个问题写的，
按惯例这个分支名就是 ``wip-doc-4000`` ，
对应的问题追踪 URL 就是 http://tracker.ceph.com/issues/4000 。

.. note:: 请不要把贡献的文档和源码混合到同一个提交里，
   您的文档提交和源码提交分开时，
   会简化审查进程。
   我们强烈建议所有新增功能或\
   配置选项的拉取请求也要包含一个文档提交，
   描述一下相关的变更、选项。

创建分支前，确保本地和远程都没有同名的。

.. prompt:: bash $

	git branch -a | grep wip-doc-{your-branch-name}

如果确实不存在，就可以创建了：

.. prompt:: bash $

	git checkout -b wip-doc-{your-branch-name}


开始更改
--------
.. Make a Change

修改文档很简单，打开 restructuredText 文件、修改、保存即可。
相关的语法请参考 `文档风格指南`_ 。

新增文档要在 ``doc`` 目录或其子目录下新建 restructuredText 文件，
并以 ``*.rst`` 作后缀。
还必须包含对它的引用：
如超链接或目录条目。某个顶极目录中的 ``index.rst`` 文件通常也包含一个 TOC ，
你可以在这里添加新文件名。
所有文档都必须有标题，详情见\ `标题`_\ 。

你新建的文档不会自动被 ``git`` 跟踪，
如果想把它加进仓库，必须用 ``git add {path-to-filename}`` 命令。
比如，在 Ceph 仓库的顶极目录下，
把 ``example.rst`` 文件加到 ``rados`` 子目录下，
可以这样：

.. prompt:: bash $

	git add doc/rados/example.rst

要删除一文档，应该用 ``git rm {path-to-filename}`` ，
比如：

.. prompt:: bash $

	git rm doc/rados/example.rst

还必须从其他文档删除与之相关的引用。


构建文档源码
------------
.. Build the Source

要想构建文档，先进入 ``ceph`` 库目录：

.. prompt:: bash $

	cd ceph

.. note::
   包含 ``build-doc`` 和 ``serve-doc`` 的目录必须加进
   ``PATH`` 环境变量里，这些命令才能好好运行。


在 Debian/Ubuntu 、 Fedora 或 CentOS/RHEL 上执行此命令构建文档：

.. prompt:: bash $

	admin/build-doc

要扫描外部链接是否都可达，执行：

.. prompt:: bash $

	admin/build-doc linkcheck

执行 ``admin/build-doc`` 之后，它会在 ``ceph`` 下创建一个
``build-doc`` 目录。你也许还得在 ``ceph/build-doc`` 下创建个目\
录用于 Javadoc 的输出。

.. prompt:: bash $

	mkdir -p output/html/api/libcephfs-java/javadoc

``build-doc`` 构建脚本可能会产生警告和报错，
有关语法的错误\ **必须**\ 修复才能提交，
警告\ **应该**\ 尽量消除。

.. important:: 你必须核实\ **所有超链接**\ ，
   损坏的超链接会中止构建过程。

文档构建完成后你就可以启动一个 HTTP 服务器、
通过 ``http://localhost:8080/`` 查看了：

.. prompt:: bash $

	admin/serve-doc

或者，你可以直接到 ``build-doc/output`` 下看看构建好的文档。\
那里应该有 ``html`` 目录和 ``man`` 目录分别存放着 HTML 和手册\
页格式的文档。

构建源码（首次）
~~~~~~~~~~~~~~~~
.. Build the Source (First Time)

Ceph 用 Python Sphinx 构建文档，此软件一般都没安装。
首次构建文档时，它会生成一个用于
doxygen 的 XML 树，这个过程比较耗时.

Python Sphinx 的依赖软件包根据发行版不同而有所区别。
首次构建文档时，如果你没安装必要工具，构建脚本会提示你。

要运行 Sphinx 并成功构建文档，至少要安装下面这些软件包：

.. raw:: html

	<style type="text/css">div.body h3{margin:5px 0px 0px 0px;}</style>
	<table cellpadding="10"><colgroup><col width="30%"><col width="30%"><col width="30%"></colgroup><tbody valign="top"><tr><td><h3>Debian/Ubuntu</h3>

- gcc
- python3-dev
- python3-pip
- python3-sphinx
- python3-venv
- libxml2-dev
- libxslt1-dev
- doxygen
- graphviz
- ant
- ditaa
- cython3

.. raw:: html

	</td><td><h3>Fedora</h3>

- gcc
- python-devel
- python-pip
- python-docutils
- python-jinja2
- python-pygments
- python-sphinx
- libxml2-devel
- libxslt1-devel
- doxygen
- graphviz
- ant
- ditaa

.. raw:: html

	</td><td><h3>CentOS/RHEL</h3>

- gcc
- python-devel
- python-pip
- python-docutils
- python-jinja2
- python-pygments
- python-sphinx
- libxml2-dev
- libxslt1-dev
- doxygen
- graphviz
- ant

.. raw:: html

	</td></tr></tbody></table>


缺少的依赖都要安装，基于 Debian/Ubuntu 发行版的系统\
执行此命令安装：

.. prompt:: bash $

	sudo apt-get install gcc python-dev python3-pip libxml2-dev libxslt-dev doxygen graphviz ant ditaa
	sudo apt-get install python3-sphinx python3-venv cython3

在 Fedora 发行版上可以执行：

.. prompt:: bash $

   sudo yum install gcc python-devel python-pip libxml2-devel libxslt-devel doxygen graphviz ant
   sudo pip install html2text
   sudo yum install python-jinja2 python-pygments python-docutils python-sphinx
   sudo yum install jericho-html ditaa

在 CentOS/RHEL 发行版上，最好安装 ``epel``
(Extra Packages for Enterprise Linux) 软件库，
因为它提供了很多默认软件库所没有的软件包。
可执行此命令安装 ``epel`` ：

.. prompt:: bash $

   sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

在 CentOS/RHEL 发行版上可以执行：

.. prompt:: bash $

   sudo yum install gcc python-devel python-pip libxml2-devel libxslt-devel doxygen graphviz ant
   sudo pip install html2text

对于 CentOS/RHEL 发行版，其余软件包不包含在默认及 ``epel`` 软件库内，
所以需要到 http://rpmfind.net/ 找，然后到合适的镜像下载并安装它们，
比如：

.. prompt:: bash $

	wget http://rpmfind.net/linux/centos/7/os/x86_64/Packages/python-jinja2-2.7.2-2.el7.noarch.rpm
	sudo yum install python-jinja2-2.7.2-2.el7.noarch.rpm
	wget http://rpmfind.net/linux/centos/7/os/x86_64/Packages/python-pygments-1.4-9.el7.noarch.rpm
	sudo yum install python-pygments-1.4-9.el7.noarch.rpm
	wget http://rpmfind.net/linux/centos/7/os/x86_64/Packages/python-docutils-0.11-0.2.20130715svn7687.el7.noarch.rpm
	sudo yum install python-docutils-0.11-0.2.20130715svn7687.el7.noarch.rpm
	wget http://rpmfind.net/linux/centos/7/os/x86_64/Packages/python-sphinx-1.1.3-11.el7.noarch.rpm
	sudo yum install python-sphinx-1.1.3-11.el7.noarch.rpm

Ceph 文档大量使用了 `ditaa`_ ，
它没有对应的 CentOS/RHEL7 二进制包。
如果你要修改 ``ditaa`` 图，
那你必须安装 ``ditaa`` 才能确认你新增或修改的 ``ditaa`` 图可以正确渲染。
你可以自己去找与 CentOS/RHEL7 发行版兼容的包，并手动安装。
在 CentOS/RHEL7 下 ``ditaa`` 依赖下列软件包：

- jericho-html
- jai-imageio-core
- batik

到 http://rpmfind.net/ 找兼容的 ``ditaa`` 及其依赖，
然后从某个镜像下载并安装它们。例如：

.. prompt:: bash $

	wget http://rpmfind.net/linux/fedora/linux/releases/22/Everything/x86_64/os/Packages/j/jericho-html-3.3-4.fc22.noarch.rpm
	sudo yum install jericho-html-3.3-4.fc22.noarch.rpm
	wget http://rpmfind.net/linux/centos/7/os/x86_64/Packages/jai-imageio-core-1.2-0.14.20100217cvs.el7.noarch.rpm
	sudo yum install jai-imageio-core-1.2-0.14.20100217cvs.el7.noarch.rpm
	wget http://rpmfind.net/linux/centos/7/os/x86_64/Packages/batik-1.8-0.12.svn1230816.el7.noarch.rpm
	sudo yum install batik-1.8-0.12.svn1230816.el7.noarch.rpm
	wget http://rpmfind.net/linux/fedora/linux/releases/22/Everything/x86_64/os/Packages/d/ditaa-0.9-13.r74.fc21.noarch.rpm
	sudo yum install ditaa-0.9-13.r74.fc21.noarch.rpm

安装好所有这些包之后，就可以按照\ `构建文档源码`_\
里的步骤构建文档了。


提交变更
--------
.. Commit the Change

Ceph文档的提交虽然简单，却遵循着严格的惯例：

- 一次提交\ **应该**\ 只涉及一个文件（方便回退），
  也\ **可以**\ 一次提交有关联的多个文件。
  不相干的变更\ **不应该**\ 放到同一提交内；
- 每个提交都\ **必须**\ 有注释；
- 提交的注释\ **必须**\ 以 ``doc:`` 打头（应严格遵守）；
- 注释摘要\ **必须**\ 只有一行（应严格遵守）；
- 额外的注释\ **可以**\ 写到摘要下面空一行的地方，
  但应该简单明了；
- 提交\ **可以**\ 包含 ``Fixes: https://tracker.ceph.com/issues/{bug number}`` 字样；
- 提交\ **必须**\ 包含 ``Signed-off-by: Firstname Lasname <email>`` （应严格遵守）。

.. tip:: 请遵守前述惯例，特别是标明了 ``（应严格遵守）`` 的那些，\
   否则你的提交会被打回，
   修正后才能重新提交。

下面是个通用提交的注释（首选）： ::

	doc: Fixes a spelling error and a broken hyperlink.

	Signed-off-by: John Doe <john.doe@gmail.com>


下面的注释里有到 BUG 的引用。 ::

	doc: Fixes a spelling error and a broken hyperlink.

	Fixes: https://tracker.ceph.com/issues/1234

	Signed-off-by: John Doe <john.doe@gmail.com>


下面的注释包含一句概要和详述，
在摘要和详述之间用空行隔开了： ::

	doc: Added mon setting to monitor config reference

	Describes 'mon setting', which is a new setting added
	to config_opts.h.

	Signed-off-by: John Doe <john.doe@gmail.com>


执行下列命令提交变更：

.. prompt:: bash $

	git commit -a


管理文档提交的一个比较简单的方法是用 ``git`` 的图形化前端，
如 ``gitk`` 提供了可查看仓库历史的图形界面；
``git-gui`` 提供的图形界面可查看未提交的变更、
把未提交变更暂存起来、提交变更、
并推送到自己的 Ceph 分支仓库。


在 Debian/Ubuntu 上执行以下命令安装：

.. prompt:: bash $

	sudo apt-get install gitk git-gui

在 Fedora/CentOS/RHEL 上执行以下命令安装：

.. prompt:: bash $

	sudo yum install gitk git-gui

然后执行：

.. prompt:: bash $

	cd {git-ceph-repo-path}
	gitk

最后，点击 **File->Start git gui** 打开图形界面。


推送变更
--------
.. Push the Change

你完成一或多个提交后，必须从本地推送到位于 ``github`` 的仓库。
某些图形化工具（如 ``git-gui`` ）有推送菜单。如果你之前创建了分支：

.. prompt:: bash $

	git push origin wip-doc-{your-branch-name}

否则：

.. prompt:: bash $

	git push


发出接收请求
------------
.. Make a Pull Request

前面已经说过了，你可以依照\ `分支并拉取`_\ 方法共享文档。


合并额外的提交
--------------
.. Squash Extraneous Commits

每个拉取请求（ pull request ）应该只关联一个提交（ commit ）。
如果您在新功能工作分支上做出了多个提交，则需要“合并（ squash ）”这些提交。
“合并”是一种特定类型“交互式重置（ interactive rebase ）”的俗称。
“合并”的方式有很多，但本例涉及的是有三个提交、
且三个提交中的改动都要保留的情形。这三个提交将被合并成一个提交。

#. 先做出提交，稍后合并它们。

   A. 做出第一个提交。

      ::

         doc/glossary: improve "CephX" entry

         Improve the glossary entry for "CephX".

         Signed-off-by: Zac Dover <zac.dover@proton.me>

         # Please enter the commit message for your changes. Lines starting
         # with '#' will be ignored, and an empty message aborts the commit.
         #
         # On branch wip-doc-2023-03-28-glossary-cephx
         # Changes to be committed:
         #       modified:   glossary.rst
         #

   B. 做出第二个提交。

      ::

         doc/glossary: add link to architecture doc

         Add a link to a section in the architecture document, which link
         will be used in the process of improving the "CephX" glossary entry.

         Signed-off-by: Zac Dover <zac.dover@proton.me>

            # Please enter the commit message for your changes. Lines starting
            # with '#' will be ignored, and an empty message aborts the commit.
            #
            # On branch wip-doc-2023-03-28-glossary-cephx
            # Your branch is up to date with 'origin/wip-doc-2023-03-28-glossary-cephx'.
            #
            # Changes to be committed:
            #       modified:   architecture.rst

   C. 做出第三个提交。

      ::

         doc/glossary: link to Arch doc in "CephX" glossary

         Link to the Architecture document from the "CephX" entry in the
         Glossary.

         Signed-off-by: Zac Dover <zac.dover@proton.me>

         # Please enter the commit message for your changes. Lines starting
         # with '#' will be ignored, and an empty message aborts the commit.
         #
         # On branch wip-doc-2023-03-28-glossary-cephx
         # Your branch is up to date with 'origin/wip-doc-2023-03-28-glossary-cephx'.
         #
         # Changes to be committed:
         #       modified:   glossary.rst

#. 功能分支中现在有三个提交。
   现在我们开始将它们合并为一个提交。

   A. 执行命令 ``git rebase -i main`` ，它会把当前分支
      （功能分支）重置到 ``main`` 分支：

      .. prompt:: bash

         git rebase -i main

   B. 会出现一个提交列表，就是之前在功能分支上做出的那些提交，
      样子类似下面：

      ::

         pick d395e500883 doc/glossary: improve "CephX" entry
         pick b34986e2922 doc/glossary: add link to architecture doc
         pick 74d0719735c doc/glossary: link to Arch doc in "CephX" glossary

         # Rebase 0793495b9d1..74d0719735c onto 0793495b9d1 (3 commands)
         #
         # Commands:
         # p, pick <commit> = use commit
         # r, reword <commit> = use commit, but edit the commit message
         # e, edit <commit> = use commit, but stop for amending
         # s, squash <commit> = use commit, but meld into previous commit
         # f, fixup [-C | -c] <commit> = like "squash" but keep only the previous
         #                    commit's log message, unless -C is used, in which case
         #                    keep only this commit's message; -c is same as -C but
         #                    opens the editor
         # x, exec <command> = run command (the rest of the line) using shell
         # b, break = stop here (continue rebase later with 'git rebase --continue')
         # d, drop <commit> = remove commit
         # l, label <label> = label current HEAD with a name
         # t, reset <label> = reset HEAD to a label
         # m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
         #         create a merge commit using the original merge commit's
         #         message (or the oneline, if no original merge commit was
         #         specified); use -c <commit> to reword the commit message
         # u, update-ref <ref> = track a placeholder for the <ref> to be updated
         #                       to this position in the new commits. The <ref> is
         #                       updated at the end of the rebase
         #
         # These lines can be re-ordered; they are executed from top to bottom.
         #
         # If you remove a line here THAT COMMIT WILL BE LOST.

      找到屏幕上显示 pick 的部分，这是需要您修改的部分。
      现在有三个提交被标为 pick 。
      我们选择其中一个继续标记为 pick ，
      并将另外两个提交标记为 squash 。

#. 把三个提交中的两个标记为 ``squash``:

   ::

      pick d395e500883 doc/glossary: improve "CephX" entry
      squash b34986e2922 doc/glossary: add link to architecture doc
      squash 74d0719735c doc/glossary: link to Arch doc in "CephX" glossary

      # Rebase 0793495b9d1..74d0719735c onto 0793495b9d1 (3 commands)
      #
      # Commands:
      # p, pick <commit> = use commit
      # r, reword <commit> = use commit, but edit the commit message
      # e, edit <commit> = use commit, but stop for amending
      # s, squash <commit> = use commit, but meld into previous commit
      # f, fixup [-C | -c] <commit> = like "squash" but keep only the previous
      #                    commit's log message, unless -C is used, in which case
      #                    keep only this commit's message; -c is same as -C but
      #                    opens the editor
      # x, exec <command> = run command (the rest of the line) using shell
      # b, break = stop here (continue rebase later with 'git rebase --continue')
      # d, drop <commit> = remove commit
      # l, label <label> = label current HEAD with a name
      # t, reset <label> = reset HEAD to a label
      # m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
      #         create a merge commit using the original merge commit's
      #         message (or the oneline, if no original merge commit was
      #         specified); use -c <commit> to reword the commit message
      # u, update-ref <ref> = track a placeholder for the <ref> to be updated
      #                       to this position in the new commits. The <ref> is
      #                       updated at the end of the rebase
      #
      # These lines can be re-ordered; they are executed from top to bottom.
      #
      # If you remove a line here THAT COMMIT WILL BE LOST.

#. 现在，我们创建一条提交信息，
   它适用于所有合并在一起的提交：

   A. 保存并关闭指定要合并的提交列表后，
      会出现包含所有三条提交信息的列表，
      如下所示：

      ::

         # This is a combination of 3 commits.
         # This is the 1st commit message:

         doc/glossary: improve "CephX" entry

         Improve the glossary entry for "CephX".

         Signed-off-by: Zac Dover <zac.dover@proton.me>

         # This is the commit message #2:

         doc/glossary: add link to architecture doc

         Add a link to a section in the architecture document, which link
         will be used in the process of improving the "CephX" glossary entry.

         Signed-off-by: Zac Dover <zac.dover@proton.me>

         # This is the commit message #3:

         doc/glossary: link to Arch doc in "CephX" glossary

         Link to the Architecture document from the "CephX" entry in the
         Glossary.

         Signed-off-by: Zac Dover <zac.dover@proton.me>

         # Please enter the commit message for your changes. Lines starting
         # with '#' will be ignored, and an empty message aborts the commit.
         #
         # Date:      Tue Mar 28 18:42:11 2023 +1000
         #
         # interactive rebase in progress; onto 0793495b9d1
         # Last commands done (3 commands done):
         #    squash b34986e2922 doc/glossary: add link to architecture doc
         #    squash 74d0719735c doc/glossary: link to Arch doc in "CephX" glossary
         # No commands remaining.
         # You are currently rebasing branch 'wip-doc-2023-03-28-glossary-cephx' on '0793495b9d1'.
         #
         # Changes to be committed:
         #       modified:   doc/architecture.rst
         #       modified:   doc/glossary.rst

   B. 提交信息已改编成了此处所示更简单的格式：

      ::

         doc/glossary: improve "CephX" entry

         Improve the glossary entry for "CephX".

         Signed-off-by: Zac Dover <zac.dover@proton.me>

         # Please enter the commit message for your changes. Lines starting
         # with '#' will be ignored, and an empty message aborts the commit.
         #
         # Date:      Tue Mar 28 18:42:11 2023 +1000
         #
         # interactive rebase in progress; onto 0793495b9d1
         # Last commands done (3 commands done):
         #    squash b34986e2922 doc/glossary: add link to architecture doc
         #    squash 74d0719735c doc/glossary: link to Arch doc in "CephX" glossary
         # No commands remaining.
         # You are currently rebasing branch 'wip-doc-2023-03-28-glossary-cephx' on '0793495b9d1'.
         #
         # Changes to be committed:
         #       modified:   doc/architecture.rst
         #       modified:   doc/glossary.rst

#. 将本地工作副本中合并后的提交强制推送（ push ）到远程的上游分支。
   之所以需要强制推送，是因为新合并的提交在远程分支中没有祖先（ ancestor ）。
   如果你对此感到困惑，只需运行此命令，
   先别多想：

   .. prompt:: bash $

      git push -f

   ::

      Enumerating objects: 9, done.
      Counting objects: 100% (9/9), done.
      Delta compression using up to 8 threads
      Compressing objects: 100% (5/5), done.
      Writing objects: 100% (5/5), 722 bytes | 722.00 KiB/s, done.
      Total 5 (delta 4), reused 0 (delta 0), pack-reused 0
      remote: Resolving deltas: 100% (4/4), completed with 4 local objects.
      To github.com:zdover23/ceph.git
       + b34986e2922...02e3a5cb763 wip-doc-2023-03-28-glossary-cephx -> wip-doc-2023-03-28-glossary-cephx (forced update)



通知我们
--------
.. Notify Us

如果发出的 PR 长时间没人审核，
请联系相应组件的负责人，询问一下这么久是什么原因。
组件负责人列表见 :ref:`ctl` 。


文档风格指南
============
.. Documentation Style Guide

Ceph 文档项目的目标之一就是可读性，
包括 restructuredText 和渲染后的 HTML 页面的可读性。
进入 Ceph 源码库，随便找个文档查看其源码，
你会发现它们在终端下就像已经渲染过的 HTML 页面一样清晰明了。
另外，也许你还看到 ``ditaa`` 格式的图表渲染的很漂亮。

.. prompt:: bash $

	less doc/architecture.rst

为了维持一致性，请遵守下面的风格手册。


标题
----
.. Headings

#. **文档标题：** 标题行的前、后各加一行 ``=`` ，
   且标题行首、行尾各有一个空格，
   详情见\ `文档标题`_\ 。

#. **段落标题：** 段标题行下是一行 ``=`` ，且标题行首、
   行尾都没有空格；段标题前应该有两个空行（除非前面是内嵌引用）。
   详情见\ `小节`_\ 。

#. **小节标题：** 小节标题行下是一行 ``_`` ，
   且行首、行尾都没有空格；段标题前应该有两个空行
   （除非前面是内嵌引用）。


正文
----
.. Text Body

通常，我们把正文限制在 80 列之内，
这样它在任何标准终端内都可以正确显示，行首、行尾都不能有空格。
我们应该尽可能维持此惯例，包括文本、项目、文字文本
（允许例外）、表格、和 ``ditaa`` 图形。

#. **段落：** 段落前后各有一空行，且宽度不超过 80 字符，
   这样文档源码就可以在任何标准终端正确显示。

#. **引文文本：** 要创建引文文本（如展示命令行用法），
   前一段应以 ``::`` 结尾；
   或者先加一个空行、然后在新行上输入 ``::`` 、
   之后再加一个空行。
   之后以 TAB （首选）或 3 个空格缩进，
   开始输入引文了。

#. **缩进文本：** 像要点这样的缩进文本（如： ``- some text`` ）
   可能会延伸很多行，
   后续行应该延续和首行缩进
   （数字、圆点等）相同的起始列。

   缩进文本也可以包含引文。这时，
   缩进文本仍然用空格标记、引文仍用 TAB 标记。
   按照这个惯例，你就可以额外增加缩进段落，
   并在其中嵌入引文示例
   （引文段前加空行，行前用空格缩进）。

#. **编号项目：** 需编号的列表应该在行首\
   用 ``#`` 标识以实现自动编号，
   而不是手动标识，
   这样在条目顺序变更时就不用重新编号了。

#. **代码示例：** Ceph 文档中可以用 ``.. code-block::<language>`` 角色\
   按语种对源码进行高亮显示，
   对源代码应该这样标记。然而，
   使用这个标签时将导致编号项目从 1 开始重新编号，
   详情见\ `显示代码示例`_\ 。


段落分级标记
------------
.. Paragraph Level Markup

Ceph 文档项目用\ `段落分级标记`_\ 来高亮显示要点。

#. **Tip:** 提示：用 ``.. tip::`` 指令标识额外信息，
   以助读者或操作员脱困。

#. **Note:** 注：用 ``.. note::`` 指令来高亮显示一个要点。

#. **Important:** 重要：用 ``.. important::`` 指令来高亮显示重要依赖或警告
   （如可能导致数据丢失的事情）。
   尽量少用，因为它会渲染成红色背景。

#. **Version Added:** 版本新增：用 ``..versionadded::`` 指令\
   来标识新增功能或配置选项，这样用户才能知道此选项适用的最低版本。

#. **Version Changed:** 版本变更：用 ``.. versionchanged::`` 指令\
   标识用法或配置选项的变更。

#. **Deprecated:** 已过时：用 ``.. deprecated::`` 指令\
   标识不再推荐或将被移除的 CLI 用法、功能、或配置选项。

#. **Topic:** 论题：用 ``.. topic::`` 指令\
   来封装位于文档主体之外的文本。
   详情见 `topic 指令`_\ 。


TOC 和超链接
------------
.. TOC and Hyperlinks

所有文档都必须被链接到其他文档或列表内，
否则构建时会被警告。

Sphinx 管理的 Ceph 文档套件中的每个文档
（每个 ``.rst`` 文件）都必须链接到
(1) 文档套件中的另一个文档，或 (2) 目录 (TOC)。
如果文档套件中存在文档没有以这种方式链接，
则 ``build-doc`` 脚本在尝试构建文档时会产生警告。

Ceph 项目使用了 ``.. toctree::`` 指令（详情见 `TOC 树`_\ ）。
渲染内容表（ TOC ）时，最好指定 ``:maxdepth:`` 参数，
这样渲染出的 TOC 不至于太长。

链接目标是个惟一标识符（如 ``.. _unique-target-id:`` ）、
而且某一引用明确引用了它（如 ``:ref: `uniq-target-id``` ），
这时应该优先用 ``:ref:`` 语法。
遵循此惯例的话，即使
``.rst`` 源文件在 ``ceph/doc`` 内移动位置后链接仍然有效，
详情见\ `交叉引用任意位置`_\ 。


.. _start_external_hyperlink_example:

外部超链接实例
~~~~~~~~~~~~~~
.. External Hyperlink Example

还可以创建一个指向文档某段落的链接，并在链接正文中显示自定义文本。
当保留的链接句子文本比明确引用链接部分的标题更重要时，
这种方法就很有用。

例如，链接到 Sphinx Python 文档生成器主页的 RST ，
生成一句 "Click here to learn more about Python Sphinx." ，
像这样书写：

::

    ``Click `here <https://www.sphinx-doc.org>`_ to learn more about Python
    Sphinx.``

然后，渲染成这样：

Click `here <https://www.sphinx-doc.org>`_ to learn more about Python Sphinx. 。

请特别注意反引号后面的下划线。如果你忘了加下划线，
而这又是你第一天使用 RST ，那么你有可能得花一整天去找到底哪里出了问题，
却不知道自己只是漏掉了下划线。此外，
请特别注意替换文本（本例中为 "here" ）与小于号之间的空格，
小于号把链接目标与显示的文本分隔开来。
如果没有这个空格，链接将无法正常渲染。

链接自定义
~~~~~~~~~~
.. Linking Customs

根据 Ceph 在 Inktank 时代形成的惯例，
Ceph 项目的文档贡献者倾向于把
``.. _Link Text: ../path`` 链接放在文档底部，
然后用 ``:ref:`path``` 形式的引用链接到它们。
这一约定更受欢迎，因为这样文档在命令行界面中的可读性更好。
但截至 2023 年，我们不再偏爱其中一种。
使用哪种惯例能让文本更容易阅读，就采用哪种惯例。

使用句子的一部分作为超链接， `像这样 <docs.ceph.com>`_\ ，不太好。
建议优先采用“参见 X ”这样的惯例。以下是一些推荐的表述方式：

#. 详情见 `docs.ceph.com <docs.ceph.com>`_ 。

#. 参见 `docs.ceph.com <docs.ceph.com>`_ 。


RST 格式的怪异之处
------------------
.. Quirks of ReStructured Text

外部链接
~~~~~~~~
.. External Links

.. _external_link_with_inline_text:

用下面的公式渲染链接，就是把读者导向 Ceph 文档之外的链接地址：

::

   `inline text <http:www.foo.com>`_

.. note:: 在内联文本和小于号之间不要丢了空格。

   最后一个反引号后面别丢了下划线。

   要链接到 Ceph 文档以外的地址，
   必须在内联文本和外部地址前的角括号之间加上空格。
   这个惯例与链接到 Ceph 文档内部位置的内联文本正好相反。
   参阅 :ref:`here <internal_link_with_inline_text>`
   以了解此惯例的实例。

   如果您觉得这玩意儿不一致且令人困惑，您的感觉是对的。
   它的确前后不一，令人困惑。

另请参阅 ":ref:`外部超链接示例 <start_external_hyperlink_example>`" 。


内部链接
~~~~~~~~
.. Internal Links

要链接到 Ceph 文档中的某个段落，您必须
（1）在该部分之前定义一个目标链接，然后（2）在文档中的另一个位置链接到该目标。
以下是目标和链接到这些目标的方式：

定义目标::

   .. _target:

   Title of Targeted Section
   =========================

   Lorem ipsum...

链接到目标::

   :ref:`target`

.. _internal_link_with_inline_text:

链接到目标时带上内联文本： ::

   :ref:`inline text<target>`

.. note::

   "inline text" 与紧随其后的角括号之间没有空格。这恰恰与
   :ref:`链接到 Ceph 文档之外位置的内联文本<external_link_with_inline_text>`
   语法相反。如果您觉得这似乎不一致且令人困惑，
   您的感觉没错。它就是前后不一，令人困惑。


在单词中标记粗体字符
~~~~~~~~~~~~~~~~~~~~
.. Escaping Bold Characters within Words

本节介绍如何将单词中的某些字母加粗，而让单词中的其他字母保持常规（非加粗）。

下面的单行段落就是一个例子：

**C**\eph **F**\ile **S**\ystem 。

在 RST 文本里，下面的写法行不通：

::

   **C**eph **F**ile **S**ystem

必须使用转义字符 (\\) 关闭加粗符号，如下所示：

::

   **C**\eph **F**\ile **S**\ystem


.. _Python Sphinx: http://sphinx-doc.org
.. _resturcturedText: http://docutils.sourceforge.net/rst.html
.. _分支并拉取: https://help.github.com/articles/using-pull-requests
.. _github: http://github.com
.. _ditaa: http://ditaa.sourceforge.net/
.. _文档标题: http://docutils.sourceforge.net/docs/user/rst/quickstart.html#document-title-subtitle
.. _小节: http://docutils.sourceforge.net/docs/user/rst/quickstart.html#sections
.. _交叉引用任意位置: http://www.sphinx-doc.org/en/master/usage/restructuredtext/roles.html#role-ref
.. _TOC 树: http://sphinx-doc.org/markup/toctree.html
.. _显示代码示例: http://sphinx-doc.org/markup/code.html
.. _段落级别标记: http://sphinx-doc.org/markup/para.html
.. _topic 指令: http://docutils.sourceforge.net/docs/ref/rst/directives.html#topic
