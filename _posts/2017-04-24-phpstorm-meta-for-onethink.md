---
layout: post
title: 为onethink增加phpstorm智能代码提示
tags: ['php']
categories:
- php
---

## 让phpstorm支持thinkphp中的全局函数跳转


---


### 如何编写phpstorm插件

- 可以使用 `PHPSTORM_META`来进行函数参数的智能映射
- 在项目根目录建立 `.phpstorm.meta.php`
- 映射非常的死板-所以需要有个脚本自动去生成


![](http://ww1.sinaimg.cn/large/7101c223ly1fey2sxdzoxj20s50863zf.jpg)


---

### 生成脚本


![](http://ww1.sinaimg.cn/large/7101c223ly1fey2ugo5sdj20ng0m30um.jpg)

[https://gist.github.com/yangyao/57a777a3c6f40b1324322efdf21f92dd](https://gist.github.com/yangyao/57a777a3c6f40b1324322efdf21f92dd "glist")

---