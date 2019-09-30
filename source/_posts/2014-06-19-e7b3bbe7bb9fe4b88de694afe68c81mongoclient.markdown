---
author: 269586320@qq.com
comments: true
date: 2014-06-19 08:59:32+00:00
layout: post
slug: '%e7%b3%bb%e7%bb%9f%e4%b8%8d%e6%94%af%e6%8c%81mongoclient'
title: 系统不支持:mongoClient
wordpress_id: 30
categories:
- 个人
---

安装了mongo扩展，在phpinfo里也能找到mongo但是就是报错。

原因是因为1.3版本的mongo驱动已经没有mongoClient类了，取而代之的是mongo

这个就坑爹了。因为TP的驱动里面用的是mongoClient。。。。尝试去修改驱动，发现改了就连不上mongo服务器了。

我上哪找1.3以前的驱动啊~知道的回个信。



The "problem" is the version on mongodriver in pagodabox.
I try with php 5.3.10, 5.3.23 and 5.4.14 but i have the same problem. With "mongo" php_extension in pagodabox i have Mongo driver 1.2.12 version.
Usually i use MongoClient() and not Mongo() because in other environment i have Mongo driver 1.3.X.
Mongo() is deprecated in 1.3.x and MongoClient is not present in the version of mongo driver of Pagodabox.
My question is: is it possible to have mongo driver 1.3.x on pagodabox ? If not i will refactor my app to use old Mongo interface.
