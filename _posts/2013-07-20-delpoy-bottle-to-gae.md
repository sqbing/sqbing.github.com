---
layout: post
title: "Delpoy Bottle To GAE"
description: "少走一些弯路"
category: 
tags: [bottle, GAE]
---
{% include JB/setup %}

1. 创建framework目录，并创建framework/__init__.py文件，拷贝bottle.py到framework/中
2. 修改app.yaml如下：

    	application: YOUR_APP_NAME
		version: 1
		runtime: python27
		api_version: 1
		threadsafe: yes
	
		handlers:
			- url: /favicon\.ico
  			static_files: favicon.ico
  			upload: favicon\.ico

			- url: /.*
  			script: main.app


3. 修改main.py如下：

		#!/usr/bin/env python
		from framework import bottle
		from framework.bottle import *
		
		app = Bottle()
		@app.get('/')
		def DisplayForm():
		    return "Hello World"
		 
		bottle.run(app=app, server='gae')
		

另外，也可以参考[这里](http://petergao.com/blog/using-bottle-with-python2-7-on-google-app-engine/)的介绍。