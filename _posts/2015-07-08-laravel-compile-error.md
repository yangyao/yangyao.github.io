---
layout: post
title:  laravel compile 的bug
tags: ['laravel']
---

### 错误日志

	'Declaration of Illuminate\View\Engines\CompilerEngine::handleViewException() should be compatible

### 错误原因

	原因是本地生成的compiler.php 被上传到其他环境的原因。

### 解决方案

	rm bootstrap/compile.php

	或者
 	
    php artisan clear-compiled

### 最佳方案，修改composer.json文件

    "pre-install-cmd" :[
        "php artisan clear-compiled"
    ],
    "pre-update-cmd": [
        "php artisan clear-compiled"
    ],


	

