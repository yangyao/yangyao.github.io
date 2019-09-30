---
layout: post
title: github免密码push的方法
tags: ['git']
---



1.生成ssh key

ssh-keygen -t rsa -C "your_email@example.com"

2.添加到github

3.校验ssh key

ssh -T git@github.com

4.修改配置 
url 改成ssh的，而不是https的
[remote "origin"]
        url = git@github.com:yangyao/yangyao.git

5.以后避免

clone的时候就要选择ssh的url



