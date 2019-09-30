---
layout: post
title: 从wordpress迁移到jekyll
tags: ['blog']
---


##从wordpress迁移到jekyll

1.将wordpress内容导出成xml文件

2.将xml变成markdown:https://github.com/thomasf/exitwp

###exitwp的使用方法

1. git clone https://github.com/thomasf/exitwp

2. sudo apt-get install python-yaml python-bs4 python-html2text

3. Lint and place all wordpress xml files in the wordpress-xml directory 

4. run python exitwp.py