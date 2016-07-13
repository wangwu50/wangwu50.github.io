---
layout: post
title: 一个spring线程安全问题
date: 2016-07-13
categories: blog
tags: [技术,spring]
description: spring

---

今天在做压力测试的时候，忽然发现一个线程安全的问题，困扰一天中终于解决了。特此记录下心得。
项目技术背景是使用spring框架的一个服务，我在一个类里面定义了一些全局的变量。在调用服务的时候，执行init()方法为全局变量赋值，然后在后面的服务中使用这些变量。
这时候如果多线程请求时，全局变量就会有线程安全问题。
以下是测试代码：

```java
@Service
public class Car
{
	private String name;
	public String getName()
	{
		return name;
	}
	public void setName(String name)
	{
		this.name = name;
	}
	public String run(String name)
	{
		setName(name);
		System.out.println("receive:"+name);
		return getName();
	}
}
``` 

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath*:applicationContext-base.xml")
public class SpringTest
{
	@Autowired
	private Car car;
	@Test
	public void test() throws InterruptedException
	{
		ExecutorService service = Executors.newFixedThreadPool(2);
		while (true)
		{
			service.execute(new Slow());
			service.execute(new Fast());
			TimeUnit.SECONDS.sleep(1);
			System.out.println("stop!");
		}
	}
	class Fast implements Runnable
	{
		@Override
		public void run()
		{
			String run = car.run("50");
			System.out.println("return :"+run);
			
		}
	}
	class Slow implements Runnable
	{
		@Override
		public void run()
		{
			String run = car.run("20");
			System.out.println("return :"+run);
			
		}
	}
}
```
执行结果为：
		receive:20
receive:50
return :50
return :50
stop!
receive:20
return :50
receive:50
return :50
stop!

由此可见，对于String维护的单实例car里面的全局变量name并不是线程安全的。之前我这样使用的原因是因为网上有一些文字说spring把bean放入到Threadlocal里面来实现线程安全。
我仔细研究了下这块内容，发现一下几个问题：

- spring并没有把所有的bean都放入threadlocal中，只有一些本来不是线程安全的dao，它用threadlocal封装成了线程安全的。因此以下代码是没有问题的：

```java
@Service
public class DaoTest
{
   @Autowired
   private Dao dao;
   public String find(String name)
   {
     return dao.getByName(name);
   }
}
```

虽然数据库连接不是线程安全的，但我们可以单实例来使用它。

- 对于我们自己定义的全局域，spring并没有进行特殊处理，因此这个就不是线程安全的，在并发访问的时候会导致错误。而使用这种形式的类较多的是实体bean。
这也给我们一个启示：**实体bean最好使用时自己new，不要交给spring进行管理**。
