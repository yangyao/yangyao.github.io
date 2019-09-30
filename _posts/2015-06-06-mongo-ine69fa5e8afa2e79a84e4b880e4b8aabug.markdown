---
author: 269586320@qq.com
comments: true
date: 2015-06-06 06:25:04+00:00
layout: post
slug: mongo-in%e6%9f%a5%e8%af%a2%e7%9a%84%e4%b8%80%e4%b8%aabug
title: mongo $in查询的一个bug
wordpress_id: 76
categories:
- CodeIgniter
tags:
- mongodb,php
---






问题重现

$condition [**'userId'**]=[ **'$in'**=>$userIds ];

  


报错，其中$userIds 为数组类型。

  



Can't canonicalize query: BadValue $in needs an array


  


强制类型转换

$condition [ **'userId'**]=[ **'$in'** =>(array)$userIds ];

  


依然报错。

  


Can't canonicalize query: BadValue $in needs an array

  


  


最终解决办法（让人很不理解，可能是php mongo驱动的问题）

  


$condition [**'userId'**]=[ **'$in'**=>_array_values_ ($userIds )];

  

