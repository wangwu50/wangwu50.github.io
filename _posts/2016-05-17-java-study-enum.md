---
layout: post
title: java枚举类的学习
date: 2016-05-17
categories: blog
tags: [技术,java]
description: java枚举类的学习

---

java中的枚举（enum）是jdk1.5以后加入的，使用枚举类有些场景会带来很多方便，枚举的本质是继承java.lang.Enum类的，因此有了父类中的很多方法。java是单继承的，所以所有枚举类不能再继承父类了。

### 基本enum特性

以下是最普通的枚举代码

```java
public enum ColorEnum
{
red,green,yellow,blue;
}
``` 

在上面代码中我们定义了四种颜色的枚举，这些枚举代表了什么呢？其实每一种颜色可以理解为枚举类的实例，写法等同于：

```java
public  class ColorEnum
{
public static final ColorEnum red=new ColorEnum();
public static final ColorEnum green=new ColorEnum();
public static final ColorEnum yellow=new ColorEnum();
public static final ColorEnum blue=new ColorEnum();
}
```

可以看出，枚举在写法上更优雅，更简洁。

### 枚举中的构造器

就像普通类一样，枚举中的类是可以添加构造方法的，只需要在枚举后面定义一个构造器就可以了。构造器的各个参数，在枚举中加括号按顺序传入就可以了。如下所示：

```java
public enum ColorEnum
{
	red("red","1"),green("green","2"),yellow("yellow","3"),blue("blue","4");
	private ColorEnum(String firstName,String secondeName)
	{
		//do some with names;
	}
}
``` 

### 枚举中的自定义方法

我们也可以在枚举中定义其他方法，定义普通变量。就和普通类一样使用,也可以覆盖默认方法。

```java
public enum ColorEnum
{
red("red","1"),green("green","2"),yellow("yellow","3"),blue("blue","4");
private String name;
private ColorEnum(String firstName,String secondeName)
{
	this.name=firstName+"_"+secondeName;
}
public String getName()
{
	return this.name;
}
@Override
public String toString()
{
	return this.name;
}
}
```

### 枚举在switch中的使用

枚举的各个元素天生就是有次序的（可以通过ordinal()方法取得），所有很适合在switch中使用。如下测试代码。

```java
public static void main(String[] args)
{
ColorEnum colorEnum=ColorEnum.blue;
switch (colorEnum)
{
case red:
	System.out.println("color is red");
	break;
case green:
	System.out.println("color is green");
	break;
case yellow:
	System.out.println("color is yellow");
	break;
case blue:
	System.out.println("color is blue");
	break;
default:
	break;
}
}
```

可以看到在switch中使用枚举很清晰、自然。

### 枚举在抽象类的使用

枚举中可以有抽象方法，这样，每一个枚举实例都有一个该抽象方法的实现类，这样可以非常灵活。

```java
public enum ColorEnum
{
red("red","1"){
	public String getName()
	{
		return this.name;
	}
},green("green","2"){

	public String getName()
	{
		return this.name;
	}
	
},yellow("yellow","3"){
	public String getName()
	{
		return this.name;
	}
},blue("blue","4"){
public String getName()
{
	return this.name;
}
};
protected String name;
private ColorEnum(String firstName,String secondeName)
{
	this.name=firstName+"_"+secondeName;
}
public abstract String getName();
}
```

这样每一个枚举都是该枚举的子类（这里的name不能定义成private了，否则会报编译错误）。

### 枚举在接口中的使用

#### 枚举来简化接口实例

枚举虽然不能继承父类，但是可以实现接口的。java中有一个著名多线程接口java.util.concurrent.ThreadFactory，很多框架需要实现这个接口来产生线程。这个接口也有很多实现。但使用和感官最好的还是用枚举定义的接口实现。
以下代码引自disruptor中的守护线程工厂实现代码。

```java
import java.util.concurrent.ThreadFactory;
public enum DaemonThreadFactory implements ThreadFactory
{
INSTANCE;

@Override
public Thread newThread(final Runnable r)
{
    Thread t = new Thread(r);
    t.setDaemon(true);
    return t;
}
}
```

这样，我们需要一个[守护线程](http://baike.baidu.com/link?url=piC3OQoNvFv57IxX-Qu8p7F-hxREJkLfKOS1DI2yMbP283l1ZAiqIfk2duqumk04VS8oVVBUMgbkgmKxR_zUdi7P50Lzt5br1qHKT7otqcOK5mTFmUtdILE70RaH-RupEjMfZ2mRTSbTwStbhl0omHBTPfALuKI8a8O7TB7CjRhZTSKtDhCOC1rNIOVOvLRf)的工厂时候，就可以用DaemonThreadFactory.INSTANCE来调用。

#### 使用接口给枚举分组

枚举无法继承也无法被继承，但有时候我们需要给枚举分组，所以我们可以在一个接口内部创建实现该接口的枚举。假如你想给颜色分类，分为深色和浅色。

```java
public interface Color
{
	enum deep implements Color{
		red,blue
	}
	enum light implements Color{
		green,yellow
	}
}
```

这样每个枚举都是Color 的实例，但分为了深色和浅色两种色系。可以用Color.deep.blue来调用。

也可以使用枚举中的枚举。

```java
public enum ColorEnum
{
deep(Color.deep.class),light(Color.light.class);
private ColorEnum(Class<? extends Color> color)
{
	colors=color.getEnumConstants();
}
private Color[] colors;
public Color getOneColor()
{
	return colors[1];
}
enum deep {
	red,blue
}
enum light{
	green,yellow
}
public interface Color
{
	enum deep implements Color{
		red,blue
	}
	enum light implements Color{
		green,yellow
	}
}
}
```

### 枚举的高级特性

除了以上基础应用外，enum还有enumSet、enumMap、责任链、状态机、多路分发等各种高级特性。
待续。。。。。



