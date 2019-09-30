---
author: 269586320@qq.com
comments: true
date: 2015-06-06 06:25:02+00:00
layout: post
slug: mongodb-%e5%88%a0%e9%99%a4collections%e4%b8%ad%e7%9a%84%e6%9f%90%e4%b8%aakey
title: mongodb 删除collections中的某个key
wordpress_id: 70
categories:
- CodeIgniter
tags:
- mongodb
---





delete db.ocm.videoId


  


  

db.domain.update({},{$unset: {affLink:1}},{multi: true});  

db.user_account_gym.update({},{$unset:{mutexCodes:1}},{multi:true});  

