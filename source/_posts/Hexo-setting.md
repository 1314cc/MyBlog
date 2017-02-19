---
title: Hexo 常用命令 
date: 2017*02*19 05:13:47
tags: [Hexo]
category: [Blog]
comments:
---


>*总有一片风景属于你*

![](http://olmfaph6j.bkt.clouddn.com/70219211125.png)


Hexo  配置与常用命令
=========

# hexo常用命令

* 初始化
	` hexo init dirname `
* 新建文章
	` hexo n "myblog"  == hexo new "myblog"`

* 新建草稿
	`hexo new draft " new_draft"`
	草稿在使用`hexo g `的时候并不发布
* 发布草稿
	`hexo publish "new_draft"`
* 生成网页
	` hexo g   == hexo generate`
* 启动服务器预览
	` hexo  s == hexo server`
* 部署到git
	`hexo d == hexo deploy`
* 清除缓存
	`hexo clean`


# hexo 文章属性

文章生成默认使用blog/scaffolds/post.md 中的配置
* title :
* date :
* tags : [ C++ , LINUX] ##标签
* category : [ 编程 ] ##分类
* comments :
注! **冒号后面有空格** 

![文章属性配置](http://olmfaph6j.bkt.clouddn.com/A395.tmp.png)

# 摘要

写文章时,`<!**more**>` 之上为摘要




