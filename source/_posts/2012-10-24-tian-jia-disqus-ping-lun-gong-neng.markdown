---
layout: post
title: "添加 DISQUS 评论功能"
date: 2012-10-24 22:02
comments: true
categories: About-blog 
---

博客建好，晾好几天了，抽了点空添加了评论系统。
DISQUS 无疑是首选，先到 [官网](http://http://disqus.com) 注册，添加 blog 域名，获得一个 shortname，这个可以先 copy 下来，下面要用。接下来到 ~/octopress 目录下编辑 _config.yml 文件，在 
    disqus_short_name: (键下刚才的 shortname) 
下面的 
    dispus_show_comment_count: true
终端里
<!--more-->
    rake generate 
    rake preview
没啥问题的话，
    rake deploy
Ok, Done. :-)
