---
layout: post
title: 数据库字段太多引发的思考
date: 2020-03-18
categories: blog
tags: [技术,总结]
description: golang相关

---
今天发现cluster表竟然已经有了70多个列，想进行一些裁剪
主要思路是
1. 另外建一张表，一对一关联
2. 另外建一张表，通过继承使用

在django 中，有OneToOneField 和ForeignKey 两种设置，这两种有什么区别呢?
OneToOneField有点类似ForeignKey加上unique=True，它俩在使用方式有一些区别。OneToOneField反向关系中，是返回是一个直接的对象，而ForeignKey反向关系返回的是一个querySet（list）

比较好的方式是，新建一张表，使用OneToOneField和之前的cluster表做关联，例如叫做cluster_conf表,把一些新加的字段放到新表里面。使用方式：cluster.cluster_conf