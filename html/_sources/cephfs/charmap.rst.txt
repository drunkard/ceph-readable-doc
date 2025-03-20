.. _charmap:

CephFS 目录条目名字规范化和大小写折叠
=====================================
.. CephFS Directory Entry Name Normalization and Case Folding

CephFS 允许把目录树配置成可以对目录条目名字施行\ **规范化**\
并有可能\ **大小写折叠**\ 。对于像 Samba 这样的、通过网关导出的文件系统来说，
此功能一般是个有用的属性，因为这些网关会强制执行文件系统不区分大小写的视图，
所以，通常对接不区分大小写的文件系统会造成性能损失。

以下虚拟扩展属性可控制目录条目的\ **字符映射**\ （ character mapping ）规则： 

* ``ceph.dir.casesensitive``: 用于设置目录大小写敏感性的布尔值。
  如果为 true ，则对目录条目名称进行大小写折叠。
* ``ceph.dir.normalization``: 用于设置目录条目名称的 Unicode 规范化类型的字符串。
  目前，客户端支持 D (``nfd``)、C (``nfc``)、KD (``nfkd``)和 KC (``nfkc``)等规范化格式。
* ``ceph.dir.encoding``: 为目录条目名称使用和执行的编码设置字符串。
  默认和目前唯一支持的编码是 UTF-8 (``utf8``)。

还有一个方便的虚拟扩展属性，可用于获取大小写敏感性、
规范化和编码配置的 JSON 编码： 

* ``ceph.dir.charmap``: 目录的完整字符映射配置。

它还可用于\ **删除**\ 所有设置，并恢复目录条目名称的默认 CephFS 行为：
以 NUL 结尾、不含 ``/`` 的未解释字节。

操作这些扩展属性时，要注意以下限制： 

* 目录必须为空。
* 目录不得是快照的一部分。

在已配置了 ``charmap`` 的目录下创建的新子目录将继承（复制）父目录的配置。

.. note:: 您可以删除子目录继承的 ``charmap`` 配置，
   前提条件是：该子目录为空，且不属于现有快照的一部分。


规范化
------
.. Normalization

``ceph.dir.normalization`` 属性支持下列规范化格式：

* **nfd**: Form D (Canonical Decomposition ，标准分解)
* **nfc**: Form C (Canonical Decomposition, followed by Canonical Composition ，标准分解而后标准合成)
* **nfkd**: Form KD (Compatibility Decomposition， 兼容分解)
* **nfkc**: Form KC (Compatibility Decomposition, followed by Canonical Composition，兼容分解而后标准合成)

字符映射配置的默认规范化格式是 ``nfd`` 。

.. note:: 有关 Unicode 规范化格式的详情，参阅 `Unicode 规范化标准文档`_\ 。

每当在路径遍历或查找过程中生成目录条目名称时，
在向 MDS 提交任何操作之前，
客户端会对名字进行规范化处理。在 MDS 端，
存储的只是这些已经规范化过的目录条目名字。

例如，在目录上设置规范化： 

::

    $ setfattr -n ceph.dir.normalization -v "" foo/
    
    $ getfattr -n ceph.dir.charmap foo/
    # file: foo/
    ceph.dir.charmap="{\"casesensitive\":true,\"normalization\":\"nfd\",\"encoding\":\"utf8\"}"
    
    $ getfattr -n ceph.dir.normalization foo/
    # file: foo/
    ceph.dir.normalization="nfd"

.. note:: 设置空字符串时， MDS 将采用默认规范化。

所有字符映射配置都必须启用一种规范化。
删除规范化将恢复默认值： 

::

    $ setfattr -n ceph.dir.normalization -v nfc foo/
    $ getfattr -n ceph.dir.normalization foo/
    # file: foo/
    ceph.dir.normalization="nfc"

    $ setfattr -x ceph.dir.normalization foo/
    $ getfattr -n ceph.dir.normalization foo/
    # file: foo/
    ceph.dir.normalization="nfd"

要删除一个目录的规范化，
必须删除配置的 ``ceph.dir.charmap`` 。

.. note:: MDS 会给目录条目维护一个 ``alternate_name`` 元数据
   （也用于加密），可以给客户端保留应用程序要用的、
   原始的未规范化过的名字。 MDS 不会对此元数据做任何解释；
   它仅仅用于让客户端重建目录条目的原始名字。


大小写折叠
----------
.. Case Folding

``ceph.dir.casesensitive`` 属性应该设置布尔值。
默认情况下，名字区分大小写（是 POSIX 文件系统的正常情况）。
把此值设为 false 后，此目录（及其子目录）
将不区分大小写。

大小写折叠要求对名称也进行规范化处理。默认情况下，
在把目录设置为大小写不敏感后， ``charmap`` 配置是这样的：

::

    $ setfattr -n ceph.dir.casesensitive -v 0 foo/
    $ getfattr -n ceph.dir.casesensitive foo/
    # file: foo/
    ceph.dir.casesensitive="0"

    $ getfattr -n ceph.dir.charmap foo/
    # file: foo/
    ceph.dir.charmap="{\"casesensitive\":false,\"normalization\":\"nfd\",\"encoding\":\"utf8\"}"

注意，设置目录的大小写敏感性时，
会顺带选定默认的规范化配置。

.. note:: 规范化在大小写折叠之前应用。
   MDS 所用的目录条目名字是经过大小写折叠和规范化的名字。


删除字符映射
------------
.. Removing Character Mapping

如果目录是空的，也不属于快照，
那么它的 ``charmap`` 就可以删掉：

::

   $ setfattr -x ceph.dir.charmap foo/

可以用下面的方法确认，此目录已经恢复成了默认的 CephFS 行为：

::

   $ getfattr -n ceph.dir.charmap foo/
   foo/: ceph.dir.charmap: No such attribute

如果这个属性不存在，就说明这个目录没有配置字符映射。
注意，（以后）其子目录或父目录可能会有 charmap 配置，
但那个配置对本目录没有影响。
只有在创建目录时才会继承 charmap 配置。

.. note:: 默认的 charmap 包含规范化配置，此行为不能禁用。
   关闭此功能的唯一方法是删除这个
   ``charmap`` 虚拟扩展属性。


限制不兼容的客户端访问
----------------------
.. Restricting Incompatible Client Access

MDS 用一个新的客户端特性位来保护带有 ``charmap`` 的目录树的访问权限。
MDS 不允许不支持 ``charmap`` 功能的客户端修改那些配置了 ``charmap`` 的目录。
unlink 文件或删除子目录操作除外。

您也可以要求所有客户端都必须理解 ``charmap`` 功能，
才能使用文件系统： 

.. prompt:: bash #

    ceph fs required_client_features <fs_name> add charmap

.. note:: 内核驱动程序并不支持 ``charmap`` 功能，
   而且很可能也不会支持，因为现有的内核库对于\
   大小写折叠和规范化格式不赞同。因此，
   不建议把 ``charmap`` 加到必需的客户端功能中。


权限
----
.. Permissions

与其他 CephFS 虚拟扩展属性一样，客户端只能在具有 **p** 这个 MDS
auth cap 的目录上做 ``charmap`` 配置。查看配置不需要此 cap 。


.. _Unicode 规范化标准文档: https://unicode.org/reports/tr15/
