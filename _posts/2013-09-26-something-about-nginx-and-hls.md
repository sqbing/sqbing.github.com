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

## tips
1. SPS和PPS何时发送？

	IDR(nalu_type=5)之前发送

2. PMT和PAT如何构造？

	PMT和PAT为固定字段，每GOP发送一次即可。

3. 加入条带分割NAL头部对现有的H.264数据有何影响？

	目前看无影响，每一sample前发送条带分割NAL(nalu_type=9)

4. 码流和网络流形式的NAL在转换时需要注意哪些？

	一般情况下发送0x00 00 00 01，仅在sample的非首slice时发送0x00 00 01

5. AAC头部是否需要修改？

	AAC需要使用ADST封装

6. ADST包含哪些参数？从MP4文件的何box中读取参数？

	profile: (object_type - 1)

	sampling frequency: sample_rate_index
	
	channel configuration: channels
	
	通过mp4a->esds读取上述参数

7. 音频数据如何汇总成一个PES包再发送，而不是每一个sample封装一个PES？

	音频打包到一个PES再发送，大小为2930

8. 生成m3u8时如何分割？

	以GOP为分割

9. aac的duration为什么固定是1024？遇到非固定duration的aac应如何处理？

	aac规定每个frame包含1024个samples；忽略stts中的delta，固定输出1024

10. MP4中的elst atom应如何处理？

	pts -= elst_start_time
