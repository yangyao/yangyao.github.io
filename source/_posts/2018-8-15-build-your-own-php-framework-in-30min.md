---
layout: post
title: 30分钟，从0构建自己的PHP框架
tags: ['php']
date: 2018-08-05 06:40:12
categories:
- php
---



## composer 一切的基础

- `composer` 是什么? 在 `python` 语言中它叫 `pip` 在 `javascript` 中它叫 `npm`, 用于安装和管理第三方库
- 安装 `composer` [http://getcomposer.org](http://getcomposer.org "composer")
- 初始项目 `composer init` 
- 创建基础类  `Application.php`
- 使用 `composer psr-4` 加载基础类




## 异常的处理 - 踏出坚实的第一步

- 安装 `symfony/debug` 引入基础的异常类
- 注册异常处理函数 (`set_error_handler`,`set_exception_handler`,`register_shutdown_function`)
- 异常的处理 (渲染输出,记录日志，邮件上报)
- 什么时候注册异常处理函数呢？ Laravel 是在 Handle http 请求的时候，注册 `HandleExceptions`

- 直接抄袭 `Laravel` 即可[ 代码位于 `Illuminate/Foundation/Bootstrap/HandleExceptions.php`](https://github.com/laravel/framework/blob/7212b1e9620c36bf806e444f6931cf5f379c68ff/src/Illuminate/Foundation/Bootstrap/HandleExceptions.php#L28)
- laravel 的异常处理，还依赖`illuminate/http` 同样也安装一下
- 在 `Application` 中执行异常处理



## IOC - 框架的灵魂

- IOC 容器是什么?一个配置中心,一个数组
- 什么叫依赖注入? 从配置中心找到你需要的那一个实例
- Laravel 容器的介绍 [https://gist.github.com/davejamesmiller/bd857d9b0ac895df7604dd2e63b23afe](https://gist.github.com/davejamesmiller/bd857d9b0ac895df7604dd2e63b23afe "IOC")
- 看一个简单的例子
- 安装 容器 `illuminate/container`
- 后面会介绍容器使用的典型的例子




## `MVC`架构的实现 - 从路由开始

- fastroute 介绍 [http://nikic.github.io/2014/02/18/Fast-request-routing-using-regular-expressions.html](http://nikic.github.io/2014/02/18/Fast-request-routing-using-regular-expressions.html "fastroute")
- 安装 fastroute `composer require nikic/fast-route`、
- 在入库初始化 fastroute
- 如何使用控制器而不是闭包?



## 模板引擎 - 从外到里的美

- 控制器有了，我们来输出页面
- twig [https://twig.symfony.com/](https://twig.symfony.com/ "twig")
- 安装 `twig/twig`
- 使用 IOC 容器，加载视图




## 配置文件的加载 - 扩展只为未来

- 在进行真正的业务代码编写之前，我们需要一个配置文件
- 安装 dotEnv [https://symfony.com/doc/current/components/dotenv.html](https://symfony.com/doc/current/components/dotenv.html "dotEnv")

- 配置基础信息 `环境和数据库`



## orm 最受争议的存在

- 一个特别简单的`ORM`
-  redbean [https://github.com/gabordemooij/redbean](https://github.com/gabordemooij/redbean "redbean")
- 安装 `gabordemooij/redbean`
- 配置 ，此时就用到了我们的配置文件了
- 运行我们的 Hello world 


## 命令行 - geek的工具箱

- [https://symfony.com/doc/current/components/console.html](https://symfony.com/doc/current/components/console.html "console")
- 安装 `symfony/console`
- 演示 hello world


## 完工了?? 也许仅仅只是开始！

- 日志
- 验证器 validate
- http 中间件
- session 支持
- 小项目最终都变成了大项目。



## 感谢大家！

