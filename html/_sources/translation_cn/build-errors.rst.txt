文档编译错误
============

万精油
------

有错误就试试升级 python 库，或者哪个库出问题了，就试试它的其它\
版本（开发者可能只是测试过了他自己的版本，没考虑其它版本能不能\
跑通）。


ImportError: librados.so: cannot open shared object file: No such file or directory
-----------------------------------------------------------------------------------

错误消息::

   (virtualenv) 18:14:06 /git/ceph/build-doc/virtualenv $ python
   >>> import rados
   Traceback (most recent call last):
     File "<stdin>", line 1, in <module>
   ImportError: librados.so: cannot open shared object file: No such file or directory
   >>> 

原因：没找到对应的库。

解决方法：手动链接到系统库位置，需 root 权限。

.. code:: bash

   18:15:27 ~ # cd /usr/lib64/
   18:16:54 /usr/lib64 # ln -s /git/ceph/build-doc/virtualenv/lib64/lib* .


Inline emphasis start-string without end-string
-----------------------------------------------

错误消息::

   [...]
   Running Sphinx v1.8.3
   loading pickled environment... done
   building [mo]: all of 0 po files
   building [dirhtml]: all source files
   updating environment: [extensions changed] 286 added, 3 changed, 128 removed
   reading sources... [100%] translation-convention                                                                                                                                              
   Warning, treated as error:
   docstring of rados.Rados.require_state:1:Inline emphasis start-string without end-string.

原因：可能是文档没更新到此处。主流文档可能已经删除了 rados.Rados.require_state ，而翻译的文档还存在此条。

解决方法：更新文档，删除相应条目。


rados.Rados size changed
------------------------

错误消息::

	[...]
	Successfully built rgw
	Installing collected packages: rgw
	Successfully installed rgw-2.0.0
	Running Sphinx v1.8.3
	building [mo]: all of 0 po files
	building [dirhtml]: all source files
	updating environment: 285 added, 0 changed, 0 removed
	reading sources... [100%] translation-convention

	Warning, treated as error:
	autodoc: failed to import module u'rgw'; the following exception was raised:
	Traceback (most recent call last):
	  File "/git/ceph/build-doc/virtualenv/lib/python2.7/site-packages/sphinx/ext/autodoc/importer.py", line 154, in import_module
	    __import__(modname)
	  File "rados.pxd", line 14, in init rgw
	ValueError: rados.Rados size changed, may indicate binary incompatibility. Expected 80 from C header, got 32 from PyObject

解决方法：

- 升级 Python，如 2.7 -> 3.6
- 升级 Cython
- 升级前述二者


bad magic number in 'rados': b'\x03\xf3\r\n'
--------------------------------------------

错误消息::

	22:22:13 /git/ceph $ ./admin/build-doc
	Top Level States:  ['RecoveryMachine']
	Branch 'py3' set up to track remote branch 'py3' from 'origin'.
	Switched to a new branch 'py3'
	Looking in indexes: https://pypi.tuna.tsinghua.edu.cn/simple
	Processing /git/ceph/src/pybind/rados
	Installing collected packages: rados
	  Running setup.py install for rados ... done
	Successfully installed rados-2.0.0
	Looking in indexes: https://pypi.tuna.tsinghua.edu.cn/simple
	Processing /git/ceph/src/pybind/rbd
	Installing collected packages: rbd
	  Running setup.py install for rbd ... done
	Successfully installed rbd-2.0.0
	Looking in indexes: https://pypi.tuna.tsinghua.edu.cn/simple
	Processing /git/ceph/src/pybind/cephfs
	Installing collected packages: cephfs
	  Running setup.py install for cephfs ... done
	Successfully installed cephfs-2.0.0
	Looking in indexes: https://pypi.tuna.tsinghua.edu.cn/simple
	Processing /git/ceph/src/pybind/rgw
	Installing collected packages: rgw
	  Running setup.py install for rgw ... done
	Successfully installed rgw-2.0.0
	Running Sphinx v1.8.3
	building [mo]: all of 0 po files
	building [dirhtml]: all source files
	updating environment: 285 added, 0 changed, 0 removed
	reading sources... [100%] translation-convention

	Warning, treated as error:
	autodoc: failed to import method 'Rados.conf_get' from module 'rados'; the following exception was raised:
	bad magic number in 'rados': b'\x03\xf3\r\n'

解决方法：

#. Python 版本可能不对，我这里是用了 Python 3.6，但当前只支持
   3.4 、 3.5 ::

      cd /git/ceph/build-doc
      python3.5 -m venv virtualenv/

#. 用更高版本的 Python 。 Python 3 向前兼容，但未必向后兼容；\
   有些开发者可能用了最新版 Python 的功能，而这个功能可能在
   ``setup.py`` 里声明的 Python 版本中不支持，所以应该尝试下更\
   高版本的 Python ::

      cd /git/ceph/build-doc
      python3.6 -m venv virtualenv/

#. Python 的缓存文件问题，删除即可::

      cd /git/ceph
      find build-doc/ src/ -name \*.pyc -exec rm -vf {} +

