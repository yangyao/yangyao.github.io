---
layout: post
title: 如何设计一个好的异常
tags: ['php']
categories:
- php
---

# 如何设计一个好用的异常

---

## 为什么要使用异常

### 明明可以使用返回值来判定一个程序的运行状况？ 

> 异常机制初衷就是自行触发来转移程序流程， 解决逻辑嵌套太深时错误处理的困难

- 通过返回值判断，使得语意模糊，代码混乱
- 通过返回值判断，不利于错误的集中处理
- 可以让用户有机会干预PHP的默认行为（`display_errors`和`error_reporting`）
- 在发生异常的时候能做一些log或者cleanup
- 合理的使用异常可以帮我们设计结构优良的程序
- 增加了可维护性，便于定位线上环境的bug


---

## 主流的框架都是怎么处理异常的

### 几乎所有框架在初始化的时候都会注册异常处理的函数，但都无非下面3个函数


  - register_shutdown_function
  - set_error_handler
  - set_exception_handler

---

## thinkphp

### 框架初始化

[ 代码位于 `thinkphp/ThinkPHP/Library/Think/Think.class.php`](https://github.com/top-think/thinkphp/blob/955404d6782672b656a8d6799e87d8f47ea042ae/ThinkPHP/Library/Think/Think.class.php#L31)

![thinkphp](http://ww1.sinaimg.cn/large/7101c223gy1fdrwaxlsptj20f908jmxl)


---


### 异常的处理

[ 代码位于 `thinkphp/ThinkPHP/Library/Think/Think.class.php`](https://github.com/top-think/thinkphp/blob/955404d6782672b656a8d6799e87d8f47ea042ae/ThinkPHP/Library/Think/Think.class.php#L241)


![](http://ww1.sinaimg.cn/large/7101c223gy1fdrxfvi5gzj20f60eljsf)


---

### 异常的渲染与输出

[ 代码位于 `thinkphp/ThinkPHP/Library/Think/Think.class.php`](https://github.com/top-think/thinkphp/blob/955404d6782672b656a8d6799e87d8f47ea042ae/ThinkPHP/Library/Think/Think.class.php#L316)

![](http://ww1.sinaimg.cn/large/7101c223gy1fdrxgxbn1mj20r70mvjt9)


---


## laravel

### 框架初始化

[ 代码位于 `Illuminate/Foundation/Bootstrap/HandleExceptions.php`](https://github.com/laravel/framework/blob/7212b1e9620c36bf806e444f6931cf5f379c68ff/src/Illuminate/Foundation/Bootstrap/HandleExceptions.php#L28)

![](http://ww1.sinaimg.cn/large/7101c223gy1fdrwngimc7j20e60dpmxx)


---


### 异常的处理

[ 代码位于 `Illuminate/Foundation/Bootstrap/HandleExceptions.php`](https://github.com/laravel/framework/blob/7212b1e9620c36bf806e444f6931cf5f379c68ff/src/Illuminate/Foundation/Bootstrap/HandleExceptions.php#L74)


![](http://ww1.sinaimg.cn/large/7101c223gy1fdryme3mlvj20gw0e0q3w)

---

### 异常的渲染

[ 代码位于 `Illuminate/Foundation/Bootstrap/HandleExceptions.php`](https://github.com/laravel/framework/blob/7212b1e9620c36bf806e444f6931cf5f379c68ff/src/Illuminate/Foundation/Bootstrap/HandleExceptions.php#L95)


![](http://ww1.sinaimg.cn/large/7101c223gy1fdryv04wm4j20kb0cdq3r)


---


## 两者的比较

- 都能捕获到用户应用层面没能catch到的异常，并作相应的处理
- laravel兼容php7，可以捕获 `fatal error` 并作处理
- laravel中用户通过定义`Handler`渲染输出异常（TP可配置自定义错误页面）
- ThinkPHP在所有异常抛出的时候都给客户端发送了一个404的响应头 令人费解


---

## 应用层的异常处理


> 框架只是解决了基础的问题，要把异常系统做好，还需要在应用层面下功夫


Laravel推介的方式：每一个模块或者库异常都建立自己的异常处理类


---

## 分享一种异常处理方案

- 按照模块来定义异常类
- 精细化异常CODE || IDE便可以定位异常，并找到注解
- 全局CODE定义 ，抛出异常时引用全局CODE 便于定位异常
- 捕获的时候使用case 进行CODE比对 


---

## 基础父类

> talk is cheep , show me the code !

![](http://ww1.sinaimg.cn/large/7101c223gy1fds9wqhqjvj20rt0lm0uc)


---


## 自定义异常子类

> 按照不同模块定义不同的子类

### 系统异常

![](http://ww1.sinaimg.cn/large/7101c223gy1fdsa8n2l67j20qj0ehabs)

---

## 抛出并捕获异常

抛出

![](http://ww1.sinaimg.cn/large/7101c223gy1fdsabfjtcjj20sy04eq3f)

捕获

![](http://ww1.sinaimg.cn/large/7101c223gy1fdsadms955j20m10dq75z)

---

## 在什么时候什么地方抛出异常

 - 当然是Validate 、Model、Service、Api 这些地方
 - 在控制器中捕获异常、并根据错误码结合请求客户端给予合适的响应
 - 没有被抓住的异常，按照框架的要求，需要定义handler去处理，确保万无一失


---

## 注意事项

- 除了调试需要尽量不要在流程里面使用 die或者exit 这种会直接导致异常失效


---

# 感谢大家的参与！

 

