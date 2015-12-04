================
 贡献 Ceph 文档
================

帮助 Ceph 项目的\ **最简单的方法**\ 之一就是贡献文档，随着 Ceph 用户量的增长和\
开发的迅速推进，越来越多的人在更新文档、增加新条目。即使是修正拼写错误或增加说\
明这样的小贡献也会提升 Ceph 项目的品质。

Ceph 文档源码位于 Ceph 源码库的 ``ceph/doc`` 目录下，由 Python Sphinx 渲染成 \
HTML 和手册页。 http://ceph.com/docs 链接默认展示 ``master`` 分支，也可以查看\
较早分支（如 ``argonaut`` ）或者未来分支（如 ``next`` ）的文档，同样可以查看\
正在改进的分支，只需把 ``master`` 改成你想看的分支即可。


如何贡献
========

贡献文档和贡献源码的过程基本相同，唯一不同的就是你编译的是文档源码而不是程序源\
码。大致顺序如下：

#. `获取源代码`_
#. `进入分支`_
#. `开始更改`_
#. `构建文档源码`_
#. `提交变更`_
#. `推送变更`_
#. `发出接收请求`_
#. `通知相关人员`_


获取源代码
----------

Ceph 文档位于和源码同一仓库内的 ``ceph/doc`` 目录下，关于 github 和 Ceph 的详\
细情况见\ :ref:`Get Involved`\ 。

最常用的贡献方法是\ `分支并拉入`_\ 。为此，必须先做到：

#. 在本地安装 git 。在 Debian/Ubuntu 下用此命令： ::

	sudo apt-get install git

   在 Fedora 下用此命令： ::

	sudo yum install git

   在 CentOS/RHEL 下用此命令： ::

	sudo yum install git

#. 在 ``.gitconfig`` 配置文件里写好自己的名字和邮件地址。 ::

	[user]
	   email = {your-email-address}
	   name = {your-name}

   例如： ::

	git config --global user.name "John Doe"
	git config --global user.email johndoe@example.com


#. 创建 `github`_ 帐户（假如没有的话）。

#. 创建 Ceph 项目的分支，参见 https://github.com/ceph/ceph 。

#. 把已分支项目克隆到本机。


Ceph 文档按主要组件来分类组织。

- **Ceph 存储集群：**\ Ceph 存储集群文档位于 ``doc/rados`` 目录下；

- **Ceph 块设备：**\ Ceph 块设备文档位于 ``doc/rbd`` 目录下；

- **Ceph 对象存储：**\ Ceph 对象存储文档位于 ``doc/radosgw`` 目录下；

- **Ceph 文件系统：**\ Ceph 文件系统文档位于 ``doc/cephfs`` 目录下；

- **安装（快速）：**\ 快速入门文档位于 ``doc/start`` 目录下；

- **安装（手动）：**\ 手动安装文档位于 ``doc/install`` 目录下；

- **手册页：**\ 手册源码位于 ``doc/man`` 目录下；

- **开发者：**\ 开发者文档位于 ``doc/dev`` 目录下；

- **图片：**\ 如果你想上传文档，如 JPEG 或 PNG 文件，应该放到 ``doc/images`` \
  目录下。


进入分支
--------

如果只是细小的变更，像修正排版错误、或换一种措辞，直接提交到 ``master`` 分支即\
可；为当前版本的功能提供文档时也应该提交到 ``master`` 分支。 ``master`` 是最常\
用的分支。 ::

	git checkout master

给未来版本提供文档时应该提交到 ``next`` 分支， ``next`` 分支是第二常用的分支。 ::

	git checkout next

你在为尚未发布的功能写文档时，如果这部分文档和已追踪的某个问题有关，或者想在它\
被合并到 ``master`` 分支前看看它在 ceph.com 网站上的预览，你应该另外创建个分\
支。为标识这是个只包含文档的更新，按惯例用 ``wip-doc`` 作前缀，按这个格式 \
``wip-doc-{your-branch-name}`` 。如果此分支和 http://tracker.ceph.com/issues \
里的某个问题相关，分支名最好包含问题编号，例如，如果某文档分支是为 #4000 这个\
问题写的，按惯例这个分支名就是 ``wip-doc-4000`` ，对应的问题追踪 URL 就是 \
http://tracker.ceph.com/issues/4000 。

.. note:: 请不要把贡献的文档和源码混合到同一个 pull 请求里，因为文档由编辑审\
   阅、而源码由工程师审阅。您分别提交文档和源码时，合并进度会很快，我们也不用\
   让您重新提交。

创建分支前，确保本地和远程都没有同名的。 ::

	git branch -a | grep wip-doc-{your-branch-name}

如果确实不存在，就可以创建了： ::

	git checkout -b wip-doc-{your-branch-name}


开始更改
--------

修改文档很简单，打开 restructuredText 文件、修改、保存即可。相关的语法请参考 \
`文档风格手册`_ 。

新增文档要在 ``doc`` 目录或其子目录下新建 restructuredText 文件，并以 \
``*.rst`` 作后缀。还必须包含对它的引用：如超链接或目录条目。某个顶极目录中的 \
``index.rst`` 文件通常也包含一个 TOC ，你可以在这里添加新文件名。所有文档都必\
须有标题，详情见\ `标题`_\ 。

你新建的文档不会自动被 ``git`` 跟踪，如果想把它加进仓库，必须用 \
``git add {path-to-filename}`` 命令。比如，在 Ceph 仓库的顶极目录下，把 \
``example.rst`` 文件加到 ``rados`` 子目录下，可以这样： ::

	git add doc/rados/example.rst

要删除一文档，应该用 ``git rm {path-to-filename}`` ，比如： ::

	git rm doc/rados/example.rst

还必须从其他文档删除与之相关的引用。


构建文档源码
------------

要想构建文档，先进入 ``ceph`` 库目录： ::

	cd ceph

在 Debian/Ubuntu 上执行此命令构建文档： ::

	admin/build-doc

在 Fedora 上执行此命令构建文档： ::

	admin/build-doc

在 CentOS/RHEL 上执行此命令构建文档： ::

	admin/build-doc

执行 ``admin/build-doc`` 之后，它会在 ``ceph`` 下创建一个 ``build-doc`` 目录。\
你也许还得在 ``ceph/build-doc`` 下创建个目录用于 Javadoc 的输出。 ::

	mkdir -p output/html/api/libcephfs-java/javadoc

``build-doc`` 构建脚本可能会产生警告和报错，有关语法的错误\ **必须**\ 修复才能\
提交，警告\ **应该**\ 尽量消除。

.. important:: 你必须核实\ **所有超链接**\ ，损坏的超链接会中止构建过程。

文档构建完成后你就可以到源码目录下查看了： ::

	cd build-doc/output

那里应该有 ``html`` 目录和 ``man`` 目录分别存放着 HTML 和手册页格式的文档。


构建源码（首次）
~~~~~~~~~~~~~~~~

Ceph 用 Python Sphinx 构建文档，此软件一般都没安装。首次构建文档时，它会生成一\
个用于 doxygen 的 XML 树，这个过程比较耗时.

Python Sphinx 的依赖软件包根据发行版不同而有所区别。首次构建文档时，如果你没安\
装必要工具，构建脚本会提示你。要运行 Sphinx 并成功构建文档，至少要安装下面这些\
软件包：

.. raw:: html

	<style type="text/css">div.body h3{margin:5px 0px 0px 0px;}</style>
	<table cellpadding="10"><colgroup><col width="30%"><col width="30%"><col width="30%"></colgroup><tbody valign="top"><tr><td><h3>Debian/Ubuntu</h3>

- gcc
- python-dev
- python-pip
- python-virtualenv
- python-sphinx
- libxml2-dev
- libxslt1-dev
- doxygen
- graphviz
- ant
- ditaa

.. raw:: html

	</td><td><h3>Fedora</h3>

- gcc
- python-devel
- python-pip
- python-virtualenv
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
- python-virtualenv
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


缺少的依赖都要安装，基于 Debian/Ubuntu 发行版的系统可以用此命令安装： ::

	sudo apt-get install gcc python-dev python-pip python-virtualenv libxml2-dev libxslt-dev doxygen graphviz ant ditaa
	sudo apt-get install python-sphinx

在 Fedora 发行版上可以执行： ::

   sudo yum install gcc python-devel python-pip python-virtualenv libxml2-devel libxslt-devel doxygen graphviz ant
   sudo pip install html2text
   sudo yum install python-jinja2 python-pygments python-docutils python-sphinx
   sudo yum install jericho-html ditaa

在 CentOS/RHEL 发行版上，最好安装 ``epel`` (Extra Packages for Enterprise \
Linux) 软件库，因为它提供了很多默认软件库所没有的软件包。可执行此命令安装 \
``epel`` ： ::


	wget http://ftp.riken.jp/Linux/fedora/epel/7/x86_64/e/epel-release-7-2.noarch.rpm
	sudo yum install epel-release-7-2.noarch.rpm

在 CentOS/RHEL 发行版上可以执行： ::

	sudo yum install gcc python-devel python-pip python-virtualenv libxml2-devel libxslt-devel doxygen graphviz ant
	sudo pip install html2text

对于 CentOS/RHEL 发行版，其余软件包不包含在默认及 ``epel`` 软件库内，所以得到 \
http://rpmfind.net/ 找，然后到合适的镜像下载并安装它们，比如： ::

	wget ftp://rpmfind.net/linux/centos/7.0.1406/os/x86_64/Packages/python-jinja2-2.7.2-2.el7.noarch.rpm
	sudo yum install python-jinja2-2.7.2-2.el7.noarch.rpm
	wget ftp://rpmfind.net/linux/centos/7.0.1406/os/x86_64/Packages/python-pygments-1.4-9.el7.noarch.rpm
	sudo yum install python-pygments-1.4-9.el7.noarch.rpm
	wget ftp://rpmfind.net/linux/centos/7.0.1406/os/x86_64/Packages/python-docutils-0.11-0.2.20130715svn7687.el7.noarch.rpm
	sudo yum install python-docutils-0.11-0.2.20130715svn7687.el7.noarch.rpm
	wget ftp://rpmfind.net/linux/centos/7.0.1406/os/x86_64/Packages/python-sphinx-1.1.3-8.el7.noarch.rpm
	sudo yum install python-sphinx-1.1.3-8.el7.noarch.rpm

Ceph 文档大量使用了 `ditaa`_ ，它没有对应的 CentOS/RHEL7 二进制包。如果你要修\
改 `ditaa`_ 图，那你必须安装 `ditaa`_ 才能确认你新增或修改的 `ditaa`_ 图可以正\
确渲染。你可以自己去找与 CentOS/RHEL7 发行版兼容的包，并手动安装。在 \
CentOS/RHEL7 下 `ditaa`_ 依赖下列软件包：

- jericho-html
- jai-imageio-core
- batik

到 http://rpmfind.net/ 找兼容的 ``ditaa`` 及其依赖，然后从某个镜像\
下载并安装它们。例如： ::

	wget ftp://rpmfind.net/linux/fedora/linux/releases/20/Everything/x86_64/os/Packages/j/jericho-html-3.2-6.fc20.noarch.rpm
	sudo yum install jericho-html-3.2-6.fc20.noarch.rpm
	wget ftp://rpmfind.net/linux/centos/7.0.1406/os/x86_64/Packages/jai-imageio-core-1.2-0.14.20100217cvs.el7.noarch.rpm
	sudo yum install jai-imageio-core-1.2-0.14.20100217cvs.el7.noarch.rpm
	wget ftp://rpmfind.net/linux/centos/7.0.1406/os/x86_64/Packages/batik-1.8-0.12.svn1230816.el7.noarch.rpm
	sudo yum install batik-1.8-0.12.svn1230816.el7.noarch.rpm
	wget ftp://rpmfind.net/linux/fedora/linux/releases/20/Everything/x86_64/os/Packages/d/ditaa-0.9-10.r74.fc20.noarch.rpm
	sudo yum install ditaa-0.9-10.r74.fc20.noarch.rpm

.. important:: 不要安装含 ``fc21`` 的 ``ditaa`` rpm包，因为它使用\
   的 ``JRE`` 比 CentOS/RHEL7 自带的新，这样会导致冲突并抛出异常 \
   ``Exception`` ，程序也因此不能运行。

安装好所有这些包之后，就可以按照\ ``构建文档源码``\ 里的步骤构建\
文档了。


提交变更
--------

Ceph文档的提交虽然简单，却遵循着严格的惯例：

- 一次提交\ **应该**\ 只涉及一个文件（方便回退），也\ **可以**\ \
  一次提交有关联的多个文件。不相干的变更\ **不应该**\ 放到同一提\
  交内；
- 每个提交都\ **必须**\ 有注释；
- 提交的注释\ **必须**\ 以 ``doc:`` 打头（应严格遵守）；
- 注释摘要\ **必须**\ 只有一行（应严格遵守）；
- 额外的注释\ **可以**\ 写到摘要下面空一行的地方，但应该简单明了；
- 提交\ **可以**\ 包含 ``Fixes: #{bug number}`` 字样；
- 提交\ **必须**\ 包含 \
  ``Signed-off-by: Firstname Lasname <email>`` （应严格遵守）。

.. tip:: 请遵守前述惯例，特别是标明了 ``（应严格遵守）`` 的那些，\
   否则你的提交会被打回，修正后才能重新提交。

下面是个通用提交的注释（首选）： ::

	doc: Fixes a spelling error and a broken hyperlink.

	Signed-off-by: John Doe <john.doe@gmail.com>


下面的注释里有到 BUG 的引用。 ::

	doc: Fixes a spelling error and a broken hyperlink.

	Fixes: #1234

	Signed-off-by: John Doe <john.doe@gmail.com>


下面的注释包含一句概要和详述，在摘要和详述之间用空行隔开了： ::

	doc: Added mon setting to monitor config reference

	Describes 'mon setting', which is a new setting added
	to config_opts.h.

	Signed-off-by: John Doe <john.doe@gmail.com>


执行下列命令提交变更： ::

	git commit -a


管理文档提交的一个比较简单的方法是用 ``git`` 的图形化前端，如 \
``gitk`` 提供了可查看仓库历史的图形界面； ``git-gui`` 提供的图\
形界面可查看未提交的变更、把未提交变更暂存起来、提交变更、并推\
送到自己的 Ceph 分支仓库。


在 Debian/Ubuntu 上执行以下命令安装： ::

	sudo apt-get install gitk git-gui

在 Fedora/CentOS/RHEL 上执行以下命令安装： ::

	sudo yum install gitk git-gui

然后执行 ::

	cd {git-ceph-repo-path}
	gitk

最后，点击 **File->Start git gui** 打开图形界面。


推送变更
--------

你完成一或多个提交后，必须从本地推送到位于 ``github`` 的仓库。某些图形化工具\
（如 ``git-gui`` ）有推送菜单。如果你之前创建了分支： ::

	git push origin wip-doc-{your-branch-name}

否则： ::

	git push


发出接收请求
------------

前面已经说过了，你可以依照\ `分支并拉入`_\ 方法共享文档。


通知相关人员
------------

发出接收请求后，还需通知相关人员。通常，文档的接收请求应该发给 `John Wilkins`_ 。


文档风格手册
============

Ceph 文档项目的目标之一就是可读性，包括 restructuredText 和渲染后的 HTML 页面\
的可读性。进入 Ceph 源码库，随便找个文档查看其源码，你会发现它们在终端下就像已\
经渲染过的 HTML 页面一样清晰明了。另外，也许你还看到 ``ditaa`` 格式的图表渲染\
的很漂亮。 ::

	cat doc/architecture.rst | less

为了维持一致性，请遵守下面的风格手册。


标题
----

#. **文档标题：** 标题行的前/后各加一行 ``=`` ，且标题行首、行尾各有一个空格，\
   详情见\ `文档标题`_\ 。

#. **段落标题：** 段标题行下是一行 ``=`` ，且标题行首、行尾都没有空格；段标题\
   前应该有两个空行（除非前面是内嵌引用）。详情见\ `小节`_\ 。

#. **小节标题：** 小节标题行下是一行 ``-`` ，且行首、行尾都没有空格；段标题前\
   应该有两个空行（除非前面是内嵌引用）。


正文
----

通常，我们把正文限制在 80 列之内，这样它在任何标准终端内都可以正确显示，行首、\
行尾都不能有空格。我们应该尽可能维持此惯例，包括文本、项目、文字文本（允许例\
外）、表格、和 ``ditaa`` 图形。

#. **段落：** 段落前后各有一空行，且宽度不超过 80 字符，这样文档源码就可以在任\
   何标准终端正确显示。

#. **引文文本：** 要创建引文文本（如展示命令行用法），前一段应以 ``::`` 结尾；\
   或者先加一个空行、然后在新行上输入 ``::`` 、之后再加一个空行。之后以 TAB \
   （首选）或 3 个空格缩进，开始输入引文了。

#. **缩进文本：** 像要点这样的缩进文本（如： ``- some text`` ）可能会延伸很多\
   行，后续行应该延续和首行缩进（数字、圆点等）相同的起始列。

   缩进文本也可以包含引文。这时，缩进文本仍然用空格标记、引文仍用 TAB 标记。按\
   照这个惯例，你就可以额外增加缩进段落，并在其中嵌入引文示例（引文段前加空\
   行，行前用空格缩进）。

#. **编号项目：** 需编号的列表应该在行首用 ``#`` 标识以实现自动编号，而不是手\
   动标识，这样在条目顺序变更时就不用重新编号了。

#. **代码示例：** Ceph 文档中可以用 ``.. code-block::<language>`` 按语种对源码\
   进行高亮显示，对源代码应该这样标记。然而，使用这个标签时将导致编号项目从 1 \
   开始重新编号，详情见\ `显示代码示例`_\ 。


段落分级标记
------------

Ceph 文档项目用\ `段落分级标记`_\ 来高亮显示要点。

#. **Tip:** 提示：用 ``.. tip::`` 指令标识额外信息，以助读者或操作员脱困。

#. **Note:** 注：用 ``.. note::`` 指令来高亮显示一个要点。

#. **Important:** 重要：用 ``.. important::`` 指令来高亮显示重要依赖或警告（如\
   可能导致数据丢失的事情）。尽量少用，因为它会渲染成红色背景。

#. **Version Added:** 版本新增：用 ``..versionadded::`` 指令来标识新增功能或配\
   置选项，这样用户才能知道此选项适用的最低版本。

#. **Version Changed:** 版本变更：用 ``.. versionchanged::`` 指令标识用法或配\
   置选项的变更。

#. **Deprecated:** 已过时：用 ``.. deprecated::`` 指令标识不再推荐或将被移除的 \
   CLI 用法、功能、或配置选项。

#. **Topic:** 论题：用 ``.. topic::`` 指令来封装位于文档主体之外的文本。详情\
   见 `topic 指令`_\ 。


TOC 和超链接
------------

所有文档都必须被链接到其他文档或列表内，否则构建时会被警告。

Ceph 项目采用 ``.. toctree::`` 指令（详情见 `TOC 树`_\ ）。渲染时，最好用 \
``:maxdepth:`` 参数把 TOC 修饰得简洁些。

链接目标是个惟一标识符（如 ``.. _unique-target-id:`` ）、而且某一引用明确引用\
了它（如 ``:ref: `uniq-target-id``` ），这时应该优先用 ``:ref:`` 语法。这样，\
如果源文件位置或文档结构变更之后链接仍然有效，详情见\ `交叉引用任意位置`_\ 。

Ceph 文档内的链接可以这样写：反引号（重音符号）、之后跟着链接文本、另一个反引\
号、最后是下划线； Sphinx 允许你内联链接目标。然而，我们喜欢这样用：在文档底部\
加 ``.. _Link Text: ../path`` ，因为这样的写法在命令行下可读性好。


.. _Python Sphinx: http://sphinx-doc.org
.. _resturcturedText: http://docutils.sourceforge.net/rst.html
.. _分支并拉入: https://help.github.com/articles/using-pull-requests
.. _github: http://github.com
.. _ditaa: http://ditaa.sourceforge.net/
.. _文档标题: http://docutils.sourceforge.net/docs/user/rst/quickstart.html#document-title-subtitle
.. _小节: http://docutils.sourceforge.net/docs/user/rst/quickstart.html#sections
.. _交叉引用任意位置: http://sphinx-doc.org/markup/inline.html#ref-role
.. _TOC 树: http://sphinx-doc.org/markup/toctree.html
.. _显示代码示例: http://sphinx-doc.org/markup/code.html
.. _段落级别标记: http://sphinx-doc.org/markup/para.html
.. _topic 指令: http://docutils.sourceforge.net/docs/ref/rst/directives.html#topic
.. _John Wilkins: mailto:jowilkin@redhat.com
