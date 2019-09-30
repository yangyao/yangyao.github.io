---
author: 269586320@qq.com
comments: true
date: 2015-06-06 06:25:05+00:00
layout: post
slug: mongodb%e7%9a%84max%e5%92%8cmin%e6%93%8d%e4%bd%9c
title: mongodb的max和min操作
wordpress_id: 78
categories:
- CodeIgniter
tags:
- mongodb
---




`db.person.``find``({}).``sort``({``"age"` `: -1}).limit(1)`

``

`-1 得到的是最大的值。max`

` 1 得到最小值 min`
