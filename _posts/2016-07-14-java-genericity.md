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

实现场景是这样的：有一个框架的方法，入参需要一个泛型数组T[]。因为那是一个抽象类，而数组这种形式平日里用的不是很多.所有我们把它封装成一个入参为集合的方法:

```java
	public boolean add(Collection<T2> values)
	{
		@SuppressWarnings("unchecked")
		T2[] val = (T2[]) values.toArray();
		return add(val);
	}
```

因为Collection的toArray()返回的数组的一个Object数组，所以经过了一次强转。这个方法一直用的很好，没有出过差错。


今天我在工作中偶然遇到了一个需要数组强转的地方，我毫不犹豫得进行了强转。如下示例代码所示：

```java
	@Test(expected=ClassCastException.class)
	public void testGenericity()
	{
		Object[] a=new Object[4];
		String[] b=(String[]) a;
	}
```

然后我发现竟然报错了：java.lang.ClassCastException: [Ljava.lang.Object; cannot be cast to [Ljava.lang.String;

这让我很不解，然后我查资料发现，数据在java中和int等一样，是不属于对象的。一个object数组和一个String数组是完全不同的两个容器，彼此之间没有任何联系。因此会强转失败。

然后我就想起之前用的泛型也进行强转了。我开始以为这个方法出错了。后来进行测试竟然没有发现任何毛病。

那么到底是怎么回事呢？

#### java中的语法糖

泛型这个概念是jdk1.5之后加入的，java是一种编译型语言，也就是说很多东西在编译的时候就已经定了。而泛型是一种动态过程，直到运行使用时才会确定其类型。
从这个角度来说，java中是不应该有泛型的。其实确实是这样的，泛型只是java为程序员提供的一个语法糖。我们写的List<T>到了java虚拟机中其实都是List<Object>。所以：**对泛型来说，强制转换其实并没有用**。我所定义的T2[]在jvm中本来就是Object[]。所以不会出现强转异常。

>语法糖：指计算机语言中添加的某种语法，这种语法对语言的功能并没有影响，但是更方便程序员使用

java中的语法糖除了泛型之外，还有：

 - 自动拆箱/装箱
 - 循环历遍（foreach）
 - 枚举
 - 变长参数
 



