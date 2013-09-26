---
layout: post
title: "Something About Nginx and HLS"
description: ""
category: 
tags: [nginx, HLS]
---
{% include JB/setup %}

不涉及具体实现，只是总结一下近半个月来研究nginx和HLS以及综合两者时遇到的一些困难。

## ngx_open_cached_file()

完整原型如下：

	ngx_int_t
	ngx_open_cached_file(ngx_open_file_cache_t *cache, 
		ngx_str_t *name,
		ngx_open_file_info_t *of, 
		ngx_pool_t *pool)

其中的name参数比较有意思，在nginx中，ngx_str_t比u_char\*多了一个长度字段。小心！这个函数内部并没有考虑name的长度，而是粗暴的调用了：

	fd = ngx_open_file(name->data, mode, create, access);

ngx_open_file()是对open()的简单封装，如下：

	#ifdef __CYGWIN__
		#define NGX_HAVE_CASELESS_FILESYSTEM  1
		#define ngx_open_file(name, mode, create, access)\
			open((const char *) name, mode|create|O_BINARY, access)
	#else
		#define ngx_open_file(name, mode, create, access)\
			open((const char *) name, mode|create, access)
	#endif

所以，在调用这个函数打开文件时，除了要用ngx_str_t封装文件路径，还要注意这个路径一定要用\0结尾。
