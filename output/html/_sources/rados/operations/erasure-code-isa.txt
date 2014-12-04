================
 ISA 纠删码插件
================

The *isa* plugin is encapsulates the `ISA
<https://01.org/intel%C2%AE-storage-acceleration-library-open-source-version/>`_
library. It only runs on Intel processors.


创建 isa 配置
=============

要新建 *isa* 纠删码配置：
::

        ceph osd erasure-code-profile set {name} \
             plugin=isa \
             technique={reed_sol_van|cauchy} \
             [k={data-chunks}] \
             [m={coding-chunks}] \
             [ruleset-root={root}] \
             [ruleset-failure-domain={bucket-type}] \
             [directory={directory}] \
             [--force]

其中：


``k={data chunks}``

:Description: 各对象都被分割为\ **数据块**\ ，分别存储于不同 OSD 。
:Type: Integer
:Required: No.
:Default: 7


``m={coding-chunks}``

:Description: 计算各对象的\ **编码块**\ 、并存储于不同 OSD 。编码块的数量等同于在\
              不丢数据的前提下允许同时失效的 OSD 数量。

:Type: Integer
:Required: No.
:Default: 3


``technique={reed_sol_van|cauchy}``

:Description: The ISA plugin comes in two `Reed Solomon
              <https://en.wikipedia.org/wiki/Reed%E2%80%93Solomon_error_correction>`_
              forms. If *reed_sol_van* is set, it is `Vandermonde
              <https://en.wikipedia.org/wiki/Vandermonde_matrix>`_, if
              *cauchy* is set, it is `Cauchy
              <https://en.wikipedia.org/wiki/Cauchy_matrix>`_.

:Type: String
:Required: No.
:Default: reed_sol_van


``ruleset-root={root}``

:Description: The name of the crush bucket used for the first step of
              the ruleset. For intance **step take default**.

:Type: String
:Required: No.
:Default: default


``ruleset-failure-domain={bucket-type}``

:Description: Ensure that no two chunks are in a bucket with the same
              failure domain. For instance, if the failure domain is
              **host** no two chunks will be stored on the same
              host. It is used to create a ruleset step such as **step
              chooseleaf host**.

:Type: String
:Required: No.
:Default: host


``directory={directory}``

:Description: 设置纠删码插件的路径，需是\ **目录**\ 。
:Type: String
:Required: No.
:Default: /usr/lib/ceph/erasure-code


``--force``

:Description: 覆盖同名配置。

:Type: String
:Required: No.
