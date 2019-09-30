---
layout: post
title: 安装node-rtmp-server
tags: ['node']
---

1.安装server
 
	git clone https://github.com/iizukanao/node-rtsp-rtmp-server.git
	cd node-rtsp-rtmp-server
	npm install -d

2.配置运行

	vim config.coffee
	sudo coffee server.coffee


3.音频视频发布(rtmp的方式)

	下载安装 Flash Media Live Encoder 3.2

	FMS URL :rtmp://192.168.199.165/live
	Stream :STREAM_NAME
	
	安装AAC插件

	Audio Format : AAC

	connect ! start

4.视频发布（rtsp的方式）

	rtsp://192.168.199.165:8080/live/STREAM_NAME

5.客户端接受