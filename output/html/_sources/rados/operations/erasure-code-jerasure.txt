=====================
 Jerasure 纠删码插件
=====================

*jerasure* 插件是最通用、最灵活的插件，也是 Ceph 纠删码存储池的默认插件。

The *jerasure* plugin encapsulates the `Jerasure
<https://bitbucket.org/jimplank/jerasure/>`_ library. It is
recommended to read the *jerasure* documentation to get a better
understanding of the parameters.

创建 jerasure 配置
==================

要新建 *jerasure* 纠删码配置：
::

        ceph osd erasure-code-profile set {name} \
             plugin=jerasure \
             k={data-chunks} \
             m={coding-chunks} \
             technique={reed_sol_van|reed_sol_r6_op|cauchy_orig|cauchy_good|liberation|blaum_roth|liber8tion} \
             [ruleset-root={root}] \
             [ruleset-failure-domain={bucket-type}] \
             [directory={directory}] \
             [--force]

其中：


``k={data chunks}``

:Description: 各对象都被分割为\ **数据块**\ ，分别存储于不同 OSD 。
:Type: Integer
:Required: Yes.
:Example: 4


``m={coding-chunks}``

:Description: 计算各对象的\ **编码块**\ 、并存储于不同 OSD 。编码块的数量等同于在\
              不丢数据的前提下允许同时失效的 OSD 数量。

:Type: Integer
:Required: Yes.
:Example: 2


``technique={reed_sol_van|reed_sol_r6_op|cauchy_orig|cauchy_good|liberation|blaum_roth|liber8tion}``

:Description: The more flexible technique is *reed_sol_van* : it is
              enough to set *k* and *m*. The *cauchy_good* technique
              can be faster but you need to chose the *packetsize*
              carefully. All of *reed_sol_r6_op*, *liberation*,
              *blaum_roth*, *liber8tion* are *RAID6* equivalents in
              the sense that they can only be configured with *m=2*. 

:Type: String
:Required: No.
:Default: reed_sol_van


``packetsize={bytes}``

:Description: The encoding will be done on packets of *bytes* size at
              a time. Chosing the right packet size is difficult. The
              *jerasure* documentation contains extensive information
              on this topic.

:Type: Integer
:Required: No.
:Default: 2048


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
