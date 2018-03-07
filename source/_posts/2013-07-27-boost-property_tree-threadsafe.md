---
layout: post
title: "boost::property_tree threadsafe"
description: "增加宏定义解决boost::property_tree的线程安全问题"
category: 
tags: [boost, property_tree, read_json, threadsafe]
---

##问题描述：
最近程序经常崩溃，GDB调试发现全部是read_json函数调用的问题。

##问题原因：
boost::property_tree是boost提供的文本解析库，可用于解析或生成json，xml，ini等文件。

property_tree依赖grammar库，grammar在多线程环境下使用（很）可能崩溃。

##问题解决：
引用与property_tree头文件之前定义宏：
	
	#define BOOST_SPIRIT_THREADSAFE
	
效果非常显著，建议把宏的定义直接写入Makefile。