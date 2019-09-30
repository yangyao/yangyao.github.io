---
layout: post
title: 为ThinkPhp设计一个Artisan 控制台
tags: ['php']
categories:
- php
---

# 为ThinkPhp设计一个Artisan 控制台

---

## artisan简单介绍

> Artisan 是 Laravel 中自带的命令行工具的名称。它提供了一些对应用开发有帮助的命令。它基于 Symfony Console 组件驱动的。

![](http://ww1.sinaimg.cn/large/7101c223ly1fey04u98yoj20oo0fkdgz.jpg)

---

## 首先需要有个命令的注册中心

- 建立Routes目录
- 为命令配置相应的别名

![](http://ww1.sinaimg.cn/large/7101c223ly1fey0b7jmh3j20zr0bq3zm.jpg)

---

## 核心的代码实现

![](http://ww1.sinaimg.cn/large/7101c223ly1fey0gfysr4j20ko0lfac0.jpg)

---

## 使用效果

- `php cli.php console/help`
- `php cli.php Console/crontab:list dresslink.com`

![](http://ww1.sinaimg.cn/large/7101c223ly1fey0po4238j20oo0fk3z3.jpg)


---

## 简化命令的使用

- `php artisan help`

![](http://ww1.sinaimg.cn/large/7101c223ly1fey1ntohshj20if0fo3zd.jpg)

---


# 使用山寨版的Artisan为Thinkphp增加计划任务管理功能


---

## 简单介绍

- 每个模块都可以配置计划任务
- 使用xml定义计划任务，在模块下面建立`crontab.xml`
- 将xml中的计划任务dump到数据库中
- 可以在后台界面配置计划任务的执行时间
- 可以在控制台中列出所有需要执行的计划任务
- 可以再控制台中手动运行某个计划任务


---


## 使用配置


![](http://ww1.sinaimg.cn/large/7101c223ly1fey1zz29vxj20l3098752.jpg)

---

## 实现逻辑

![](http://ww1.sinaimg.cn/large/7101c223ly1fey22dlcx5j20na0n6mz5.jpg)


---