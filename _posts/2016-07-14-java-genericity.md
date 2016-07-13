---
layout: post
title: java的伪泛型
date: 2016-07-14
categories: blog
tags: [技术,java]
description: 关于java中的泛型的思考

---

jdk1.5以后加入类泛型，这样对于很操作都可以很方便了。比如一个List，jdk1.5之前都是里面Object，如果我们想用到特定的对象，需要get的时候强行转换，
而在jdk1.5之后，可以在new的时候直接声明List里面的类型。这样就可以不用强转了。使用泛型习惯后就会很自然的去使用它。之前看《Thinking in java》中说java实现的泛型是伪泛型，我还没有深刻的认识，直到前几天发现的一个问题。

..待续
