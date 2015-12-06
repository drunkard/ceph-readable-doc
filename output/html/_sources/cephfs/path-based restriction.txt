========================
 限制某一目录的访问权限
========================

CephFS 假设自己通常位于可控的环境内，客户端们都没被限制哪些路径\
允许挂载。而且如果他们挂载了某个子目录后，如 /home/user ，当前\
的 MDS 也不会确保后续操作都“锁定”在那个目录里面。基于路径的限制\
可以让我们把一个客户端限定在文件系统的特定目录下。


语法
====

如果只想授予指定目录读写（ rw ）权限，我们在给这个客户端创建密钥\
时就要提及这个目录，语法如下： ::

	ceph auth get-or-create client.*client_name* mon 'allow r' mds 'allow r, allow rw path=/*specified_directory*' osd 'allow rwx'

比如，要想把 ``foo`` 客户端限定在 ``bar`` 目录下，命令如下： ::

	ceph auth get-or-create client.foo mon 'allow r' mds 'allow r, allow rw path=/bar' osd 'allow rwx'

要是只想让客户端在指定子目录下活动，在挂载时也要指定这个目录，\
语法如下： ::

	ceph-fuse -n client.*client_name* *mount_path* -r *directory_to_be_mounted*

比如，要想把 ``foo`` 客户端限定到 ``mnt/bar`` 目录下，可用此目录： ::

	ceph-fuse -n client.foo mnt -r /bar
