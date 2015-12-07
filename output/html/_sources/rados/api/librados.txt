==============
 Librados (C)
==============

.. highlight:: c

`librados` 提供了 RADOS 服务的底层访问功能， RADOS 概览参见\ \
:doc:`../../architecture`\ 。


实例：连接并写入一个对象
========================

要使用 `Librados` ，先实例化一个 :c:type:`rados_t` 变量（集群\
句柄）、再用指向它的指针调用 c:func:`rados_create()` ： ::

	int err;
	rados_t cluster;

	err = rados_create(&cluster, NULL);
	if (err < 0) {
		fprintf(stderr, "%s: cannot create a cluster handle: %s\n", argv[0], strerror(-err));
		exit(1);
	}

然后配置 :c:type:`rados_t` 以连接集群，可以挨个配置（ \
:c:func:`rados_conf_set()` ）、或用配置文件（ \
:c:func:`rados_conf_read_file()` ）、或命令行选项（ \
:c:func:`rados_conf_parse_argv()` ）、或环境变量（ \
:c:func:`rados_conf_parse_env()` ）： ::

	err = rados_conf_read_file(cluster, "/path/to/myceph.conf");
	if (err < 0) {
		fprintf(stderr, "%s: cannot read config file: %s\n", argv[0], strerror(-err));
		exit(1);
	}

集群句柄配置好后，就可以用 :c:func:`rados_connect()` 连接了： ::

	err = rados_connect(cluster);
	if (err < 0) {
		fprintf(stderr, "%s: cannot connect to cluster: %s\n", argv[0], strerror(-err));
		exit(1);
	}

然后打开一个“ IO 上下文”，用 :c:func:`rados_ioctx_create()` 打开\
:c:type:`rados_ioctx_t` ： ::

	rados_ioctx_t io;
	char *poolname = "mypool";

	err = rados_ioctx_create(cluster, poolname, &io);
	if (err < 0) {
		fprintf(stderr, "%s: cannot open rados pool %s: %s\n", argv[0], poolname, strerror(-err));
		rados_shutdown(cluster);
		exit(1);
	}

注意，你访问的存储池必须存在。

这时你能用 RADOS 数据修改函数了，例如用 \
:c:func:`rados_write_full()` 写入名为 ``greeting`` 的对象： ::

	err = rados_write_full(io, "greeting", "hello", 5);
	if (err < 0) {
		fprintf(stderr, "%s: cannot write pool %s: %s\n", argv[0], poolname, strerror(-err));
		rados_ioctx_destroy(io);
		rados_shutdown(cluster);
		exit(1);
	}

最后，用 :c:func:`rados_ioctx_destroy()` 和 \
:c:func:`rados_shutdown()` 分别关闭 IO 上下文、到 RADOS 的连接： ::

	rados_ioctx_destroy(io);
	rados_shutdown(cluster);


异步 IO
=======

处理大量 IO 时，通常不必等一个完成再开始下一个。 `Librados` 提供\
了几种操作的异步版本：

* :c:func:`rados_aio_write`
* :c:func:`rados_aio_append`
* :c:func:`rados_aio_write_full`
* :c:func:`rados_aio_read`

对每种操作，都必须先创建一个 :c:type:`rados_completion_t` 数据结\
构来表达做什么、何时安全或显式地调用 \
:c:func:`rados_aio_create_completion()` 来结束，如果没什么特殊需\
求，可以仅传递 NULL ： ::

	rados_completion_t comp;
	err = rados_aio_create_completion(NULL, NULL, NULL, &comp);
	if (err < 0) {
		fprintf(stderr, "%s: could not create aio completion: %s\n", argv[0], strerror(-err));
		rados_ioctx_destroy(io);
		rados_shutdown(cluster);
		exit(1);
	}

现在你可以调用任意一种异步 IO 操作了，然后等它出现在内存、所有复\
制所在的硬盘里： ::

	err = rados_aio_write(io, "foo", comp, "bar", 3, 0);
	if (err < 0) {
		fprintf(stderr, "%s: could not schedule aio write: %s\n", argv[0], strerror(-err));
		rados_aio_release(comp);
		rados_ioctx_destroy(io);
		rados_shutdown(cluster);
		exit(1);
	}
	rados_wait_for_complete(comp); // in memory
	rados_wait_for_safe(comp); // on disk

最后，用 :c:func:`rados_aio_release()` 释放内存： ::

	rados_aio_release(comp);

你可以用各种回叫函数告知应用程序何时可以持续写入、或何时读缓冲是\
满的。例如，如果你追加几个对象时想衡量每个操作的延时，可以调度几\
个写操作、并把确认和提交时间保存到相应回叫函数，然后用 \
:c:func:`rados_aio_flush()` 等它们完成，然后就可以分析延时了： ::

	typedef struct {
		struct timeval start;
		struct timeval ack_end;
		struct timeval commit_end;
	} req_duration;

	void ack_callback(rados_completion_t comp, void *arg) {
		req_duration *dur = (req_duration *) arg;
		gettimeofday(&dur->ack_end, NULL);
	}

	void commit_callback(rados_completion_t comp, void *arg) {
		req_duration *dur = (req_duration *) arg;
		gettimeofday(&dur->commit_end, NULL);
	}

	int output_append_latency(rados_ioctx_t io, const char *data, size_t len, size_t num_writes) {
		req_duration times[num_writes];
		rados_completion_t comps[num_writes];
		for (size_t i = 0; i < num_writes; ++i) {
			gettimeofday(&times[i].start, NULL);
			int err = rados_aio_create_completion((void*) &times[i], ack_callback, commit_callback, &comps[i]);
			if (err < 0) {
				fprintf(stderr, "Error creating rados completion: %s\n", strerror(-err));
				return err;
			}
			char obj_name[100];
			snprintf(obj_name, sizeof(obj_name), "foo%ld", (unsigned long)i);
			err = rados_aio_append(io, obj_name, comps[i], data, len);
			if (err < 0) {
				fprintf(stderr, "Error from rados_aio_append: %s", strerror(-err));
				return err;
			}
		}
		// wait until all requests finish *and* the callbacks complete
		rados_aio_flush(io);
		// the latencies can now be analyzed
		printf("Request # | Ack latency (s) | Commit latency (s)\n");
		for (size_t i = 0; i < num_writes; ++i) {
			// don't forget to free the completions
			rados_aio_release(comps[i]);
			struct timeval ack_lat, commit_lat;
			timersub(&times[i].ack_end, &times[i].start, &ack_lat);
			timersub(&times[i].commit_end, &times[i].start, &commit_lat);
			printf("%9ld | %8ld.%06ld | %10ld.%06ld\n", (unsigned long) i, ack_lat.tv_sec, ack_lat.tv_usec, commit_lat.tv_sec, commit_lat.tv_usec);
		}
		return 0;
	}

注意，所有 :c:type:`rados_completion_t` 都必须用 \
:c:func:`rados_aio_release()` 释放内存，以免造成内存泄漏。


API calls
=========

 .. autodoxygenfile:: rados_types.h
 .. autodoxygenfile:: librados.h
