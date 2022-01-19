Hello World 模块
================

这是一份简单的模块框架，只是为了演示文档结构。

启用
----

*hello* 模块可以这样启用::

  ceph mgr module enable hello

要验证它确实启用了，运行::

  ceph mgr module ls

编辑完模块文件（ ``src/pybind/mgr/hello/module.py`` ）之后，你可以这样看变更后的::

  ceph mgr module disable hello
  ceph mgr module enable hello

或者::

  init-ceph restart mgr

执行模块可以用::

  ceph hello

日志的位置::

  build/out/mgr.x.log

模块文档
--------
.. Documenting

加入新的 mgr 模块之后，还要把对应的文档放到 ``doc/mgr/module_name.rst`` 。
还有，在 ``doc/mgr/index.rst`` 里加上新模块的链接。
