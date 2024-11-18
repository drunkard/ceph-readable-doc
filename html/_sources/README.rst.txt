:orphan:

=======================
 Ceph 文档中文翻译说明
=======================

This is Chinese translation of Ceph distributed storage system documents
(mainly ``doc`` and ``src/common/options/*yaml.in``, from https://github.com/ceph/ceph),
the translation tooling suits for any language, maybe :-D, you are allowed to
reuse them for your translation, just let me known,
email me <gongfan193@gmail.com>

----

这里是 Ceph 文档中文版的源码，编译好的位于：
https://github.com/drunkard/ceph-readable-doc ，可以克隆到本地，启动HTTP服务，
直接飞速浏览。


中文版文档风格
==============

译文的语言风格：

- 通俗易懂，尽量不用生僻的词语，前人没有翻译过的术语、又不太好直译的，想办法用\
  转义、通感之类的修辞方法翻译；
- 尽量用本土化的语言风格；中文就按汉语习惯翻译，不做直译，有时候反而不好理解；
- 废话少说，捞干的。

词语的翻译应该尽量一致，请参考\ `翻译惯例 </translation_cn/translation-convention>`_\
（即 translation-convention.rst ）。

尽量遵守原文档的书写风格，如缩进、不超过 80 列宽、等。

有些风格是中文版特有的，如：

- 中文和非中文用空格分隔；
- 插入命令，前一段行尾加空格和双引号（" ::"），否则编译器不能正确识别；
- 尽量保留原文的标题、并改为注释或链接，以防别的文档里有（自动生成的）引用，\
  且便于后续更新；如果链接与别的冲突了，可改为注释（行首加两点和空格）；
- 用反斜线（"\\"）换行，这样不会额外增加空格，即不留白；
- 原文中用双引号引用的词语在中文文档里可去掉引号，因为原文引用它已经很显眼了；
- 断行考虑因素：

   * 为防止中文版和英文版文档的行数差得太多、在后续更新时找不到位置，\
     中文版断行要提前，比如在 64 列左右断行；
   * **断行列数要与 vi_two_files.sh 脚本内的一致；**
   * 断行尽量避开词语、术语，后续要批量更改的时候方便用 ``grep`` 过滤；
   * 尽量按语义断行，即语义所在行和原文基本一致，语义一致时不必太在意\
     列是否整齐，反正编译后会合并成一行；好处也是方便用 ``grep`` 过滤。


翻译流程
========

下面大致是一个完整的工作流程。

先是配置环境
------------

#. 配置 Python venv 环境。

   用于辅助翻译质量的工具 ``qa`` 是用 Python 写的，主要用了 ``pandas`` ，
   我这里配置了 venv 环境配置::

        python3.9 -m venv /cc
        . /cc/bin/activate
        pip install pandas

#. 把代码库复制到本地，然后才能开始翻译、编译文档。 ::

        mkdir /git && cd /git
        git clone https://github.com/ceph/ceph.git
        git clone https://github.com/{你的github用户名}/ceph-Chinese-doc.git

#. 配置环境，做软链接，否则编译可能会出错。 ::

    cd /git/ceph
    ln -s /git/ceph-Chinese-doc doc-zh

    cd /git/ceph/admin
    ln -sf ../doc-zh/translation_cn/zh_build-doc .
    ln -sf ../doc-zh/translation_cn/zh_serve-doc serve-doc_zh


然后开始翻译
------------

本项目的更新有两种：

#. 对照着 Ceph 项目源码内与 doc/ 目录相关的 git commit ，把文档里变更过的部分\
   同步到本项目的对应文件；
#. 根据本项目脚本 ./qa 的输出结果，选择自己喜欢的子系统、文件进行翻译。

下面会分别介绍各自的流程。

按 git commit 更新
``````````````````

以下只是大致步骤，路径不同时还需修改脚本。

#. 执行 ``update-doc.sh`` 脚本，可以看到接下来需要同步的 git commit 历史。\
   这个脚本用到了 ``git`` 和 ``tig`` 命令，最好先检查下安装了没。

   对照着 tig 里的 commit 历史开始更新中文文档！ ::

        cd /git/ceph-Chinese-doc/ && ./translation_cn/update-doc.sh

#. 必须按日期更新！！！一次最少要更新一天的。如果遇上 commit 历史交叉得厉害，
   可能一次必须更新很多天的。

   更新完这一批后，需要更改脚本 ``env.sh`` 里的 ``SYNC_TO`` 变量。
   日期需要自己计算，下一个日期是当前的日期加下面 ``STEP`` 的天数。比如：现在是
   ``SYNC_TO="2022-01-05"`` ，而且 ``STEP=7`` ，那么下一个 ``SYNC_TO`` 就应该是
   ``2022-01-12`` 。 ::

        vi env.sh
        # 更改 SYNC_TO 变量，日期格式不要变

#. 翻译完、核对完之后，用 ``commit-updated.sh`` 脚本提交（如果你用
   ``zh_build-doc`` 编译了中文文档，
   这个脚本也会自动同步编译后的库，如果没有就只是提交当前库）::

        ./translation_cn/commit-updated.sh

按 qa 脚本的指引翻译
````````````````````
#. 先看看这个项目的翻译情况： ::

        cd /git/ceph-Chinese-doc/

        # 看整个项目的翻译情况
        ./qa

        # 看单个子系统的翻译情况（不含 ``dev/`` ，开发文档）
        ./qa rbd

        # 看单个文档的翻译情况
        ./qa rbd/rbd-mirroring.rst

   ``qa`` 工具的输出仅作参考，应该能够涵盖大部分应该翻译的内容。


翻译的注意事项
--------------

ditaa 图还不能翻译为中文，因为渲染时的字体问题还未解决。

配置选项的翻译特殊，不同于常规文档翻译，单独介绍：


配置选项的翻译
``````````````
原文档的配置选项用 ``.. confval::`` 来解释含义，源文件位于 \
``src/common/options/*.yaml.in`` 里。为翻译这部分内容，我把它们复制到了 \
``zh_options/`` 里面，只需翻译描述部分，字段名不能改，否则渲染系统就识别不到了，\
也就是说， ``type | default | min | max`` 之类的字段名只能维持原样，不能\
像原来一样翻译成 ``类型 | 默认值 | 最小值 | 最大值`` 。

此外，还需更改 ``*.yaml.in`` 文件的路径，让渲染系统采用译文：

.. code:: sh

    # conf.py -> ceph_confval_imports ，需要改成 doc-zh/zh_options

     ceph_confval_imports = glob.glob(os.path.join(top_level,
    -                                              'doc-zh/zh_options',
    +                                              'src/common/options',
                                                   '*.yaml.in'))

desc
''''
yaml.in 文件里同时有 fmt_desc|long_desc|desc 时，优先级为
fmt_desc > long_desc > desc ，见 _ext/ceph_confval.py 里：

.. code:: python

    desc = opt.get('fmt_desc') or opt.get('long_desc') or opt.get('desc')

字段名
''''''
TODO: 翻译字段名，位于 _ext/ceph_confval.py -> TEMPLATE



翻译后，测试下编译是否有报错
----------------------------

编译是为了看看还有没有什么问题。如果有报错，最好编译一下原版的文档，
对比一下出错的地方，有些错误原文里就有，修正需要的工作量较大。

#. 编译环境配置。

   * 安装 ``ditaa`` ，编译文档里的 ditaa 图需要这个软件。它依赖 Java 虚拟机，
     应该用 JDK，比如 oracle-jdk-bin-1.8 或 oracle-jdk-bin-1.7 ，jre 缺少\
     必要的库文件。
   * 安装 Sphinx ，这个文档编译系统是基于 Python 的，我现在用的是 Python 3.11 ；

#. 执行 ceph 库内 admin 目录下的 build-doc 开始构建文档； ::

    cd /git/ceph
    ./admin/zh_build-doc
    ./admin/zh_build-doc linkcheck    # 检查链接是否有效，耗时很长

    # 编译原版文档
    ./admin/build-doc

#. 启动文档服务器，这样就可以通过 http://localhost:9080/ 阅读文档了。 ::

    cd /git/ceph
    ./admin/serve-doc_zh    # 在 http://localhost:9080/ 看中文版文档

    ./admin/serve-doc       # 在 http://localhost:8080  看原版文档

如果编译失败，请参考\ `此文档 </translation_cn/build-errors>`_\
（即 build-errors.rst ）解决。


编译没问题的话，可以在 github 上向我反馈您的更新，比如 `pull request` :-)

.. vim: set colorcolumn=80 smarttab:
