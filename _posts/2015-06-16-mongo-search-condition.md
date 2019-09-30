---
layout: post
title: mongo的条件查询
tags: ['mongo']
---


##比较查询

$gt:大于

$lt:小于

$gte:大于或等于

$lte:小于或等于

$ne:不等于


###in查询

`db.test.find({"name":{"$in":["stephen","stephen1"]}})`

php的用法： 

`$querys = ["number"=>['$in' => [1,2,9]]]`

###not in 查询

`db.test.find({"name": {"$not": {"$in":["stephen2","stephen1"]}}})`


##关于字段值为null 或者为 ""或者不存在该字段的一些查询。

1.查询 不存在该字段或者值为null的记录

`db.test.find({"x":null})`

2.查询 存在该字段并且值为null的记录

`db.test.find({sex:{$in:[null],$exists:true}})`

3.查询 存在该字段的记录

`db.test.find({sex:{$exists:true}})`

4.查询 空字符串

`db.test.find({"x":''})`




