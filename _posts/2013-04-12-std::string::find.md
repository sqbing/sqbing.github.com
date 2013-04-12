---
layout: post
title: "std::string::find和std::string::npos的使用备忘"
description: ""
category: 
tags: []
---
{% include JB/setup %}

项目中需要手动解析路径，我的开发机是32位系统，运行无问题，但是部署到64位的生产环境时，解析出现了莫名其妙的错误。
翻阅basic_string.h后发现了问题的原因，错误的将std::string:find的结果赋值给unsigned int。

	unsigned int begin;
	std::string test_str = "helloworld";
	if(std::string::npos != (begin = test_str.find("/"))) // 赋值时出现了问题
	{…}

改成如下的形式，问题解决。

	size_t begin;
	std::string test_str = "helloworld";
	if(std::string::npos != (begin = test_str.find("/"))) // 问题解决
	{…}

根本原因是basic_string.h将npos定义为size_type的-1。