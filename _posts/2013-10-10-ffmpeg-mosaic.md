---
layout: post
title: "使用ffmpeg为视频打码"
description: "ffmpeg提供的滤镜为视频打码"
category: 
tags: [ffmpeg]
---
{% include JB/setup %}

涉及的滤镜包括:crop，boxblur和overlay。

示例：

	ffmpeg -i input.mp4 -vf "[in]split[blurin][originalin] [blurin]crop=60:30:in_w-70:10,boxblur=5:5[blurout];[originalin][blurout]overlay=x=main_w-70:y=10[out]" -y output.mp4

其中，

	“60:30”为马赛克面积
	
	“in_w-70:10”为马赛克位置
	
	boxblur的参数决定了马赛克的模糊程度

crop滤镜和overlay滤镜的位置相同效果最好。