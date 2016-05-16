---
layout: post
title: 关于linux的学习（二），文件与目录管理基础部分
date: 2016-05-16
categories: blog
tags: [技术,linux]
description: 文件目录管理

---

linux中最常用的命令就是显示属性（ls）、复制（cp）、删除（rm）、移动（mv）了。其实linux中一切都是文件，包括ls等。那么我们为什么可以直接使用呢？和windows一样，这些可执行文件被默认加入到了环境变量PATH下。
下面说说这些命令的常用方法

## 查看文件与目录：ls

#### 所有参数名称：

- 内容控制： ls [-aAdfFhilnrRSt] 目录名称 
- 颜色控制： ls [--color={never,auto,always}] 目录名称   
- 时间控制： ls [--full-time] 目录名称 

#### 比较常用的参数有：

- ls -l（查看详细属性）可以简写为ll
- ls -a（隐藏文件）
- ls -lh（查看文件详细信息，包括文件大小）
- ls -al --full-time （查看文件的完整修改时间）

## 查看文件与目录：cp

#### 所有参数名称：

-  cp [-adfilprsu] 源文件（source） 目标文件（destination）

#### 比较常用的参数有：

- cp -a 相当于-pdr的意思，可以复制文件可以连同复制文件的权限
- cp -d 若源文件是连接文件（我理解就是快捷方式）的属性，则复制连接文件属性而非文件本身
- cp -f 强制复制，若目标文件已存在且无法开启，会删除重试
- cp -i 要复制的文件已经存在时，在覆盖时候会询问（很多系统默认继承这个）
- cp -r 递归复制
- cp -s 复制快捷方式，软链接
- cp -p 会连同文件的属性一起复制过去，而非使用默认属性（备份常用）
- cp -u update操作，destination 比source旧会更新destination

#### 复制时要考虑的问题：

- 是否需要完整保留来源文件的信息？
- 源文件是否为[软连接文件](https://www.ibm.com/developerworks/cn/linux/l-cn-hardandsymb-links/)？
- 源文件是否为特殊文件，例如[FIFO](https://en.wikipedia.org/wiki/FIFO)，socket等？
- 源文件是否为目录？
 
## 移除文件或目录

#### 所有参数名称：

- rm [-fir]

#### 参数释义：

- rm -f ：强行删除
- rm -i ：互动删除，会询问
- rm -r ：递归删除，可以删除目录（慎用）

## 移动文件与目录，或更名

#### 所有参数名称：

- mv -[-fiu] source destination

#### 参数释义：

- mv -f ：强行覆盖
- mv -i ：互动覆盖，会询问
- mv -i ：update操作