---
layout: post
title: excel countif 使用说明
date: 2018-4-17
categories: blog
tags: [技术,excel]
description: excel countif
---

今天学习下excel中的countif函数
countif这个函数本质就是*数数*，只不过是带条件的*数数*

## count函数
首先说一下count函数，count函数可以统计一定范围内的*数值*数量。例如下图，想要统计c列的数值有多少，使用count(C:C)即可

![](/img/excel/excel-1.png)

注意，count函数不会统计字符串数据

## countif 函数
countif函数是count的改进，可以在统计范围后面加入一个条件。countif不仅可以查找数值，也可以查找字符串。

条件可以有两种：
+ 一个单纯的单元格
+ 一个数值表达式

### 单元格个数查找
例如下图，想在B列查找a的个数，使用 =COUNTIF($B$2:$B$15,E3)，这里的条件单纯是一个E3（a），即可查找到a的个数。

![](/img/excel/excel-2.png)

### 数值表达式查找
例如下图，想在c列查找大于5的个数，使用 =COUNTIF($C$2:$C$15,">=5") ，注意这里的条件要被双引号括起来。即可查找到c列大于5的个数。

![](/img/excel/excel-3.png)

## 字符串大于15位的解决办法
在查找字符串时候，如果字符串大于15位（一般身份证号等会有这个问题），直接使用countif函数只会比较前15位的字符串，会导致错误。如下图。

![](/img/excel/excel-4.png)

解决方法是使用在条件后面加&"*",如下图

![](/img/excel/excel-5.png)

## 范围绝对引用：按F4
一个很常见的使用countif错误是：选定范围时，没有把范围变成绝对引用，把相对引用变成绝对引用的快捷键是F4，建议每个countif的范围都用F4处理下。

## countif常见应用场景
countif 可以判断一个字符串是否存在于另外一列，例如，下图中的学生体检可以用countif处理，只要countif结果为0的，就是未体检。（同样也可以用vlookup函数处理，但相对麻烦）

![](/img/excel/excel-6.png)