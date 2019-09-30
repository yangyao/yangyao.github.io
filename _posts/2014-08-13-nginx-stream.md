---
layout: post
title: nginx流媒体支持配置
tags: ['nginx']
---

##配置##
1：视频播放中实现拖拽效果，可以通过nginx配置http协议的流媒体
nginx编译过程中 需要加入mp4,flv流媒体模块的支持

>
--with-http_flv_module<br>
--with-http_mp4_module

添加mp4,flv流媒体支持

>
location ~ .flv {<br>
    root /data/wwwroot/;<br>
    flv;<br>
}

这样我们就能够实现对mp4和flv的流媒体支持了，可以实现 视频的拖拽效果

2：视频rtmp视频点播,直播协议的支持
下载nginx-rtmp-module模块
编译过程中添加

>
--add-module=/usr/src/nginx-rtmp-module-master --with-http_ssl_module --with-debug<br>

点播配置：

>
rtmp {<br>
    server {<br>
    listen 1935;<br>
    chunk_size 4000;<br>
        application  vod {<br>
            play /data/wwwroot/;<br>
        }<br>
    }<br>
}

视频加载方式为 rtmp://xxx.xxx.xxx.xxx/play/视频路径
play代表rtmp提供的一个服务名

##盗链处理##
在处理视频资源文件安全型，保险一点的值能依靠视频的加密和播放器端解密的实现
服务器端 我们可以配合对访问来路判断，可以防止通过直接url下载视频资源

>
valid_referers blocked www.xxxx.com/paly.html;<br>
if ($invalid_referer) <br>
{<br>
  return 403;<br>
}<br>

需要说明一点的是 迅雷和360点下载工具 他们在下载你资源时 它是会自己带Referer头 所以你如果不讲Referer指定一个具体的来路是限制不了他们的下载的

在配合我们struts 拦截器的帮助下 能够很好的实现资源的被盗链