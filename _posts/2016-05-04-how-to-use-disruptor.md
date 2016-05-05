---
layout: post
title: 关于disruptor的使用
date: 2016-05-04
categories: blog
tags: [技术,disruptor]
description: 关于disruptor的使用

---

## 摘要
公司有使用多线程的需求，之前leader自己写了两个多线程框架，一个是异步处理的抽象类，另外一个是多线程并发抽象类。这两个框架非常好用，大家也对其做了很多改进。
但用java中的线程池不得不使用锁，直到有一天，我们在开源社区发现了disruptor的存在。leader让我研究下这个多线程框架。我也顺便把对这个框架的心得记录下来。

## 目录
下面我从以下几点对disruptor进行总结：
 1. 简介。
 2. 原理。
 3. 抽象过程和相关代码。
 4. 优化和压测结果。
 
## 简介
disruptor是一个开源的并发框架。用它可以代替current包里面的Queue。
以下是Martin Fowler写的《[LMAX架构](http://ifeve.com/lmax/)》中的简介：
> LMAX是一种新型零售金融交易平台，它能够以很低的延迟(latency)产生大量交易(吞吐量). 这个系统是建立在JVM平台上，核心是一个业务逻辑处理器，它能够在一个线程里每秒处理6百万订单. 业务逻辑处理器完全是运行在内存中(in-memory)，使用事件源驱动方式(event sourcing). 业务逻辑处理器的核心是Disruptors，这是一个并发组件，能够在无锁的情况下实现网络的Queue并发操作。
总结来说，disruptor相对于Queue的优势在于：
- 高性能
- 使用简单
- 灵活可扩展

推荐两个我个人认为比较好的学习网站
首先是disruptor的github地址：
[disruptor源码中的perftest包](https://github.com/LMAX-Exchange/disruptor/tree/master/src/perftest/java/com/lmax/disruptor)
然后是[并发编程网](http://ifeve.com/disruptor/)
我的大部分知识都是看以上网站学习的。

## 原理
待续
## 抽象和代码
待续
## 抽象类
```java


```

## 优化建议和压测结果
待续





