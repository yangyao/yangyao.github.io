---
author: 269586320@qq.com
comments: true
date: 2014-07-02 06:04:01+00:00
layout: post
slug: mysql-you-cant-specify-target-table-table-name-for-update-in-from-clause
title: mysql You can't specify target table 'table name' for update in FROM clause
wordpress_id: 37
categories:
- 个人
---

create table tmp as select min(id) as col1 from blur_article group by title;
    delete from blur_article where id not in (select col1 from tmp); 
    drop table tmp;
