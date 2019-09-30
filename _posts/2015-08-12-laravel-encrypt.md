---
layout: post
title:  laravel encrypt
tags: ['laravel']
---

## 加密方式

默认的加密模式是 128位AES 使用 CBC模式加密，PKCS7进行pad.

block size 是16位。

使用base64_encode来进行输出。

## 使用方式 
	
	默认的key 在 config/app.php里面设置

    Crypt::ecrypt()

## 相关介绍文章 

 http://laravel-recipes.com/categories/15

	

