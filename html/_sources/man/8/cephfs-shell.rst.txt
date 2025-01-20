:orphan:

.. _cephfs-shell:

==============================================
cephfs-shell -- 与 CephFS 对话的类 shell 工具
==============================================

.. program:: cephfs-shell

提纲
====

| **cephfs-shell** [options] [command]
| **cephfs-shell** [options] -- [command, command,...]


描述
====

CephFS Shell 提供了类似 shell 的命令，可以直接与 Ceph 文件系统交互。

此工具可以在交互模式下使用，也可以在非交互模式下使用。
在前一种模式下， cephfs-shell 会打开一个 shell 会话，
在指定命令完成后，它会打印出提示字符串，并无限期等待。
shell 会话完成后， cephfs-shell 会退出，
其状态码就是执行的最后一个完成命令的返回值。
在非交互式模式下， cephfs-shell 会发出一个命令，
并且在那个命令执行完毕后带着它的返回值退出。

CephFS Shell 的行为可以用 ``cephfs-shell.conf`` 调整。
详情见 `CephFS Shell 配置文件`_ 。


选项
====

.. option:: -b, --batch FILE

   批处理文件的路径。

.. option:: -c, --config FILE

   cephfs-shell.conf 的路径。

.. option:: -f, --fs FS

   要挂载的文件系统的名字。

.. option:: -t, --test FILE

   FILE 文件里是供测试的路径名录。

.. note:: cephfs-shell 依赖最新版的 cmd2 模块。
   如果 CephFS 是从源码安装的，在构建目录下执行 cephfs-shell 即可。
   也可以用 virtualenv 这样执行：

.. code:: bash

    [build]$ python3 -m venv venv && source venv/bin/activate && pip3 install cmd2
    [build]$ source vstart_environment.sh && source venv/bin/activate && python3 ../src/tools/cephfs/cephfs-shell


命令
====
.. Commands

.. note::

    除了 Ceph 文件系统， CephFS Shell 命令行\
    还能直接与本地文件系统交互。
    要这样做，必须在 CephFS Shell 命令前面加上 ``!`` （感叹号）。

    用法：

        !<cephfs_shell_command>

    例如，

    .. code:: bash

        CephFS:~/>>> !ls # Lists the local file system directory contents.
        CephFS:~/>>> ls  # Lists the Ceph File System directory contents.

mkdir
-----

如果指定目录不存在，就创建它们。

用法：

    mkdir [-option] <directory>...

* directory - 将要创建的目录名

选项：
  -m MODE    设置新目录的访问模式。
  -p, --parent         如有必要，创建其父目录。加上此选项后，目录已存在也不会报错。

put
---

把本地文件系统上的一个文件、目录复制到 Ceph 文件系统。

用法：

    put [options] <source_path> <target_path>

* source_path - 要复制到 cephfs 的本地文件、目录的路径
    * 如果是 `.` 就复制当前工作目录内的所有文件、目录。
    * 如果是 `-` 就从标准输入读取。

* target_path - 远程目录路径，文件、目录将复制到这里。
    * 如果是 `.` ，文件、目录将复制到远程的工作目录内。

选项：
   -f, --force        如果目的地已存在此文件就覆盖掉它。


get
---

把 Ceph 文件系统上的一个文件复制到本地文件系统。

用法：

    get [options] <source_path> <target_path>

* source_path - 将被复制到本地文件系统的远程文件、目录路径
    * 如果是 `.` ，就复制远程工作目录下的所有文件、目录。

* target_path - 本地目录路径，文件、目录将复制到这里。
    * 如果是 `.` ，文件、目录就会复制到本地工作目录。
    * 如果是 `-` ，把输出写到标准输出。

选项：
  -f, --force        如果目的地已存在此文件就覆盖掉它。

ls
--

罗列出当前工作目录内的所有文件和目录。

用法：

    ls [option] [directory]...

* directory - 目录名，会罗列出它里面的文件、目录
    * 默认会罗列出当前工作目录内的文件、目录。

选项：
  -l, --long	    以长格式罗列 - 显示权限
  -r, --reverse     反向排序
  -H                人类可读
  -a, -all          忽略以 . 打头的条目
  -S                按文件尺寸排序


cat
---

连结文件内容并打印在标准输出上。

用法：

    cat  <file>....

* file - 文件名

ln
--

给现有文件增加一个硬链接，或者给现有文件或目录创建一个符号链接。

用法：

    ln [options] <target> [link_name]

* target - 准备创建一个链接的源文件/目录
* link_name - 用指定的名字 link_name 链接到目标

选项：
  -s, --symbolic  创建符号链接
  -v, --verbose   打印各个被链接文件的名字
  -f, --force     强行创建链接、符号链接

cd
--

改变当前工作目录。

用法：

    cd [directory]

* directory - 路径、目录名。如果没指定目录，它就改变到根目录。
    * 如果是 '..' ，就移动到当前目录的父目录。

cwd
---

获取当前工作目录。

用法：

    cwd


quit/Ctrl-D
-----------

关闭当前 shell 。

chmod
-----

更改文件、目录的权限。

用法：

    chmod <mode> <file/directory>

mv
--

把文件、目录从源头移动到目的地。

用法：

    mv <source_path> <destination_path>

rmdir
-----

删除一或多个目录。

用法：

    rmdir <directory_name>.....

rm
--

删除一或多个文件。

用法：

    rm <file_name/pattern>...


write
-----

创建并写入一个文件。

用法：

        write <file_name>
        <Enter Data>
        Ctrl+D Exit.

lls
---

罗列指定目录里的所有文件和目录。如果没指定 path ，就会罗列出当前本地目录内的文件和目录。

用法：

    lls <path>.....

lcd
---

进入指定的本地目录。

用法：

    lcd <path>

lpwd
----

打印出当前本地目录的绝对路径。

用法：

    lpwd


umask
-----

设置和获取文件模式的创建掩码。

用法：

    umask [mode]

alias
-----

定义或显示别名。

用法：

    alias [name] | [<name> <value>]

* name - 要查询、新增、或替换的别名的名字。
* value - 别名解析到的内容（新增或删除时），可以包含空格、并且不需要加引号。

run_pyscript
------------

在控制台里运行一个 python 脚本。

用法：

    run_pyscript <script_path> [script_arguments]

* 在这个脚本里，可以用 cmd （你的自定义命令）执行控制台命令。
  但是，在这个脚本里你不能运行嵌套的 py 或 pyscript 命令。
  包含空格的路径或参数必须用引号括起来。

.. note:: cmd2 版本为 0.9.13 或更低时，此命令名为 ``pyscript`` 。

py
--

调用 python 命令、 shell 或脚本。

用法：

        py <command>: 执行一个 Python 命令。
        py: 进入交互式 Python 模式。

shortcuts
---------

列出可用的快捷方式（别名）。

用法：

    shortcuts

history
-------

查看、运行、编辑、和保存之前输入的命令。

用法：

    history [-h] [-r | -e | -s | -o FILE | -t TRANSCRIPT] [arg]

选项：
   -h             显示此帮助信息而后退出
   -r             运行选定的（多条）历史条目
   -e             编辑而后运行选定的（多条）历史条目
   -s             脚本格式，没有分隔行
   -o FILE        把命令输出到一个脚本文件
   -t TRANSCRIPT  把命令及其结果输出到一个笔录文件

unalias
-------

取消别名。

用法：

    unalias [-a] name [name ...]

* name - 要取消的别名名字

选项：
   -a     删除所有别名定义

set
---

设置一个可设置参数、或显示参数的当前设置。

用法：

    set [-h] [-a] [-l] [settable [settable ...]]

* 调用时不加参数可罗列可设置参数及其取值。

选项：
  -h     显示此帮助信息而后退出
  -a     也显示只读设置
  -l     参数的描述函数

edit
----

在一个文本编辑器内编辑文件。

用法：

    edit [file_path]

* file_path - 要用编辑器打开的文件路径


run_script
----------

运行脚本文件里的命令，文本编码格式为 ASCII 或 UTF-8 。
脚本里的各个命令应该用换行符分隔。

用法：

    run_script <file_path>

* file_path - 脚本文件的路径

.. note:: cmd2 版本为 0.9.13 或更低时，此命令名为 ``load`` 。


shell
-----

像在操作系统提示符下一样，执行一个命令。

用法：

    shell <command> [arguments]

locate
------

在文件系统里查找一个条目。

用法：

     locate [options] <name>

选项：
  -c       统计找到的条数
  -i       忽略大小写

stat
------

显示文件状态。

用法：

     stat [-h] <file_name> [file_name ...]

选项：
  -h     显示帮助信息


snap
----

创建或删除快照。

用法：

     snap {create|delete} <snap_name> <dir_name>

* snap_name - 要创建或删除的快照名。
* dir_name - 目录，将在它下面创建或删除快照


setxattr
--------

设置一个文件的扩展属性。

用法：

     setxattr [-h] <path> <name> <value>

*  path - 文件的路径
*  name - 查看或设置的扩展属性名字。
*  value - 要设置的扩展属性值。

选项：
  -h, --help   显示帮助信息


getxattr
--------

获取指定路径和名字的扩展属性的值。

用法：

     getxattr [-h] <path> <name>

*  path - 文件的路径
*  name - 要获取或设置的扩展属性名

选项：
  -h, --help   显示帮助信息


listxattr
---------

罗列指定路径的扩展属性名。

用法：

     listxattr [-h] <path>

*  path - 文件的路径

选项：
  -h, --help   显示帮助信息

df
--

显示可用磁盘空间的数量。

用法：

    df [-h] [file [file ...]]

* file - 文件名

选项：
  -h, --help   显示帮助信息


du
--

显示一个目录占用的磁盘空间。

用法：

    du [-h] [-r] [paths [paths ...]]

* paths - 目录名

选项：
  -h, --help   显示帮助信息
  -r     所有目录的递归磁盘占用量。


quota
-----

一个目录的配额管理。

用法：

    quota [-h] [--max_bytes [MAX_BYTES]] [--max_files [MAX_FILES]] {get,set} path

* {get,set} - 配额操作类型。
* path - 目录名.

选项：
  -h, --help   显示帮助信息
  --max_bytes MAX_BYTES    设置此目录下数据的最大累计尺寸
  --max_files MAX_FILES    设置此目录树下的文件总数


CephFS Shell 配置文件
=====================
.. CephFS Shell Configuration File

默认情况下， CephFS Shell 会在 ``CEPHFS_SHELL_CONF`` 环境变量里的路径内寻找
``cephfs-shell.conf`` ，而后才是用户的家目录（ ``~/.cephfs-shell.conf`` ）。

现在， CephFS Shell 从它依赖的 ``cmd2`` 那里继承了所有选项，
因此，这些选项可能会因你安装的 ``cmd2`` 版本而有很大差异。
关于这些选项的描述可以参考 ``cmd2`` 文档。

下面是个 ``cephfs-shell.conf`` 样板：

.. code-block:: ini

    [cephfs-shell]
    prompt = CephFS:~/>>>
    continuation_prompt = >

    quiet = False
    timing = False
    colors = True
    debug = False

    abbrev = False
    autorun_on_edit = False
    echo = False
    editor = vim
    feedback_to_output = False
    locals_in_py = True

退出代码
========
.. Exit Code

cephfs shell 能够返回下列退出代码：

+-----------------------------------------------+-----------+
| 错误类型                                      | 退出代码  |
+===============================================+===========+
| Miscellaneous                                 |     1     |
+-----------------------------------------------+-----------+
| Keyboard Interrupt                            |     2     |
+-----------------------------------------------+-----------+
| Operation not permitted                       |     3     |
+-----------------------------------------------+-----------+
| Permission denied                             |     4     |
+-----------------------------------------------+-----------+
| No such file or directory                     |     5     |
+-----------------------------------------------+-----------+
| I/O error                                     |     6     |
+-----------------------------------------------+-----------+
| No space left on device                       |     7     |
+-----------------------------------------------+-----------+
| File exists                                   |     8     |
+-----------------------------------------------+-----------+
| No data available                             |     9     |
+-----------------------------------------------+-----------+
| Invalid argument                              |     10    |
+-----------------------------------------------+-----------+
| Operation not supported on transport endpoint |     11    |
+-----------------------------------------------+-----------+
| Range error                                   |     12    |
+-----------------------------------------------+-----------+
| Operation would block                         |     13    |
+-----------------------------------------------+-----------+
| Directory not empty                           |     14    |
+-----------------------------------------------+-----------+
| Not a directory                               |     15    |
+-----------------------------------------------+-----------+
| Disk quota exceeded                           |     16    |
+-----------------------------------------------+-----------+
| Broken pipe                                   |     17    |
+-----------------------------------------------+-----------+
| Cannot send after transport endpoint shutdown |     18    |
+-----------------------------------------------+-----------+
| Connection aborted                            |     19    |
+-----------------------------------------------+-----------+
| Connection refused                            |     20    |
+-----------------------------------------------+-----------+
| Connection reset                              |     21    |
+-----------------------------------------------+-----------+
| Interrupted function call                     |     22    |
+-----------------------------------------------+-----------+

相关文件
========

``~/.cephfs-shell.conf``
