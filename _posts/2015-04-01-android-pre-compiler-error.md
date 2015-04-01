---
layout: post
title: "Errors running builder 'Android Pre Compiler' on project"
description: "求人还是stackoverflow"
category: 
tags: [android, eclipse]
---
{% include JB/setup %}

很久没用eclipse写android，今天想写个demo，先是启动是告诉我没有java 1.6，这个先不说，编译项目的时候居然报了如题错误。
试了这个[方法](
http://stackoverflow.com/questions/14455018/eclipse-android-errors-running-builder-android-pre-compiler-on-project)
顺利解决。

似乎是预编译器和.svn文件夹闹别扭，罢工了，过滤掉.svn文件夹就行了。

这里顺便吐嘈下某些中文博客，原因讲不清楚就算了，给的解决方案也是够坑的。