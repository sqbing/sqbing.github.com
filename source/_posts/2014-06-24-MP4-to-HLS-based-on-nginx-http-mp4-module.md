---
layout: post
title: MP4 to HLS based on nginx-http-mp4-module
tagline: null
category: null
tags: []
published: true
---

这里列出了我在基于nginx-http-mp4-module开发MP4直出HLS流时遇到的坑，哎，说多了都是泪啊……
希望对你有帮助，如果你发现了一些没有列出来的问题，欢迎反馈，你知道怎么联系我的。


* SPS和PPS何时发送？

     IDR(nalu_type=5)之前发送

* PMT和PAT如何构造？

     PMT和PAT为固定字段，每ts发送一次即可。

* 加入条带分割NAL头部对现有的H.264数据有何影响？

     目前看无影响，每一sample前发送条带分割NAL(nalu_type=9)

* 码流和网络流形式的NAL在转换时需要注意哪些？

     一般情况下发送0x00 00 00 01，仅在sample的非首slice时发送0x00 00 01

* AAC头部是否需要修改？

     AAC需要使用ADST封装

* ADST包含哪些参数？从MP4文件的何box中读取参数？

     profile: (object_type - 1)

     sampling frequency: sample_rate_index

     channel configuration: channels

     通过mp4a->esds读取上述参数

* 音频数据如何汇总成一个PES包再发送，而不是每一个sample封装一个PES？

     音频打包到一个PES再发送，大小为2930

* 生成m3u8时如何分割？

     以GOP为分割

* aac的duration为什么固定是1024？遇到非固定duration的aac应如何处理？

     aac规定每个frame包含1024个samples；忽略stts中的delta，固定输出1024

* MP4中的elst atom应如何处理？

     pts -= elst_start_time

     dts -= elst_start_time

     http://wiki.multimedia.cx/index.php?title=QuickTime_container#elst

* WARNING: Unable to read video timestamps in track 1; this may be due to not having a key frame in this segment.

     dts时间戳不为0即可，apple提供的切片工具(mediafilesegmenter)，使用的是10秒前缀;

     某些安卓平台的播放器，不支持非0开始的流，比如UC，遨游和小米自带的浏览器等等，表现为播放视频时自动跳到10秒之后播放。

* PAT中最重要的字段

     PMT的PID，其他字段一般不变

* PMT中最重要的字段

     PID

     PCR PID

     音频类型

     音频PID

     视频类型

     视频PID

* PAT和PMT中的continuity counter是否需要递增？

     建议连续，以兼容某些平台，比如安卓

* 音、视频的continuity counter是否需要递增？

     iOS平台和Mac平台的safari浏览器能够处理continuity counter跳跃，但是安卓的自带浏览器不行，如果安卓平台也在考虑范围内，最好安安分分的处理continuity counter