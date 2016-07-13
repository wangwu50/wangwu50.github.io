---
layout: post
title: spring的一个线程安全问题
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


