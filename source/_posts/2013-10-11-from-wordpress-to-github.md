---
layout: post
title: "尝试从Wordpress迁移到Github Pages"
description: "导出Wordpress数据，安装jekyll-import工具，完成迁移。"
category: 
tags: [wordpress]
---


我的另一个博客[sqbing.com](http://sqbing.com)使用Wordpress，目前挂在Appfog的免费主机上。当我们使用免费产品时，我们自己就变成了别人的产品。这个免费主机有很多问题，插件不时被删除，图片不时被删除，备份又麻烦。

参考[Jekyll](http://jekyllrb.com/docs/migrations/)提供的博客迁移方法，主要是jekyll-import工具的安装，我使用的是Ubuntu 12.04，默认ruby版本是1.8，需要安装最新版本的ruby。如果已经安装了合适版本的ruby，直接运行第3步。

1. 从[ruby官网](https://www.ruby-lang.org/zh_cn/)下载ruby 2.0.0源码包

		解压源码包
		进入源码目录，执行./configure
		make
		sudo make install

2. 使用update-alternatives修改默认ruby

		sudo update-alternatives --install /usr/bin/ruby ruby /usr/local/bin/ruby
		sudo update-alternatives --config ruby
		根据提示选择/usr/local/bin/ruby
		
3. 安装jekyll-import

		gem install jekyll-import --pre
		gem install hpricot

4. 解析Wordpress导出文件

		ruby -rubygems -e 'require "jekyll/jekyll-import/wordpressdotcom";
    		JekyllImport::WordpressDotCom.process({ :source => "wordpress.xml" })'
    	这里的wordpress.xml是文件名
    	
导出结果不是很让人满意，排版一塌糊涂，不过总比一篇篇的复制要强，找时间再重排版吧。

另外，如果嫌gem速度慢，试试换成taobao的源。

	gem sources --remove https://rubygems.org/
	gem sources -a http://ruby.taobao.org/
	gem sources -l
	*** CURRENT SOURCES ***
	 
	http://ruby.taobao.org