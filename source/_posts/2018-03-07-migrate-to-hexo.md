---
title: migrate to hexo
date: 2018-03-07 13:18:09
tags:
---
受够了奇奇怪怪的Jekyll，迁移到hexo。

hexo把静态生成的文件放到了public文件夹里，导致无法用一个repo同时用于保存.md文件和发布blog。
目前的处理方法是额外创建一个branch用于存储.md文件和hexo本身，当前的master分支用于发布blog。
姑且试试吧。