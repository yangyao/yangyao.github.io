---
layout: post
title: 下载laracasts视频
tags: ['laracasts']
---

* 使用 `iamfreee/laracasts-downloader`

<pre>

	git clone https://github.com/iamfreee/laracasts-downloader.git

</pre>



* 安装依赖 

<pre>
	
	cd laracasts-downloader
	composer install

</pre>


* 修改配置

<pre>

	cp config.example.ini config.ini

	vim config.ini


</pre>

* 运行下载

<pre>

	php start.php

</pre>

* 使用中的问题

如果突然下载变得很慢，可以关闭脚本，重启下载。这样快很多。

* 如何优化、

做个多线程下载的，但是我想我也不会有时间去`pr`

