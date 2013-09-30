---
layout: post
title: "Fix: librtmp not found in ffmpeg"
description: ""
category: 
tags: [ffmpeg, librtmp]
---
{% include JB/setup %}


服务器的yum坏了，运维的同事说“无解”，好吧，好吧……

手动编译ffmpeg，配置参数为

	./configure --enable-libfaac --enable-libx264 --enable-nonfree --enable-gpl --enable-librtmp
   
然后就一直停在

    “librtmp not found”
   
下载编译了librtmp，搞定了头文件和库的链接还是报这个错，检查config.log发现check_pkg_config这个调用出错了，再看configure，这个调用貌似是检查librtmp这个包的安装情况，也许是手动安装没有更新包信息导致的。

找到configure中对应librtmp检查的那行，可能长这样

    enabled librtmp    && require_pkg_config librtmp librtmp/rtmp.h RTMP_Socket
   
注释该行，再配置一次，通过了。

经过这样的修改，编译时可能会报错，librtmp中的一些函数未定义，可以修改config.mak，找到EXTRALIBS这个宏，在后面追加“-lrtmp”即可。

