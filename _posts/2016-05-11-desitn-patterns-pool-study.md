---
layout: post
title: 设计模式之池的学习
date: 2016-05-11
categories: blog
tags: [技术,设计模式]
description: 设计模式学习

---

## 池的概念

池在java中的有很广泛的使用，例如数据库连接池、线程池等。

池的基本思想是将一个对象保存起来达到复用的目的

思路是：池中维护一定量的对象，当需要使用对象的时候，只需要从池中拿出一个，使用完毕后再放回去。这样解决了资源频繁创建、释放的问题。

## 池的使用场景

1. 创建一个对象的开销非常大
2. 对象使用比较频繁
3. 对象可以复用

## 池和缓存的区别和联系

池和缓存都是为了数据复用可以采取的策略，都是基于“空间换时间”的思想。

#### 区别

 - 池中的所有对象都是无状态，使用者不关心每个对象的区别。缓存是有状态的，使用者一定要找到某一个对应的对象
 - 池一般会主动创建对象，以便将来使用，而缓存是在使用后存储，不会主动创建对象。也可以说池控制着对象的整个生命周期，而缓存只保管对象在缓存中的生命周期。
 
下面是摘自stackoverflow的回答：

>Cache - store frequently used values, typically because the lookup and/or creation is non-trivial. e.g. if a lookup table from a database is frequently used, or values are read from a file on disk, it's more efficient to keep it in memory and refresh it periodically.
A cache only manages object lifetime in the cache, but does not impose semantics on what is held in the cache. A cache also doesn't create the items, but just stores objects.
>Pool - term to describe a group of resources that are managed by the pool itself. e.g. (Database) Connection Pool - When a connection is needed it is obtained from the pool, and when finished with is returned to the pool.
The pool itself handles creation and destruction of the pooled objects, and manages how many objects can be created at any one time.
>Pools are typically used to reduce overhead and throttle access to resources. You wouldn't want every servlet request opening a new connection to the database. Because then you have a 1:1 relationship between active requests and open connections. The overhead of creating an destroying these connections is wasteful, plus you could easily overwhelm your database. by using a pool, these open connections can be shared. For example 500 active requests might be sharing as little as 5 database connections, depending on how long a typical request needs the connection.
>Cache Pool - mostly seems to describe the number of (independent?) cache's that exist. E.g. an asp.net application has 1 cache per Application Domain (cache isn't shared between asp.net applications). Literally a pool of caches, although this term seems to be used rarely.

## pool简单的实现思路及代码

首先我们设想这种场景：一个池维护这一类对象，假设这类对象是T，每次使用的时候我们可以从池中获取一个T，使用完的时候收回。我们可以把获取T的方法叫做checkOut，收回T的方法叫做checkIn。我们需要一个容器来保存未被使用的T，可以用hashset实现。比如这个set叫做available。我们还需要另外一个容器来保存在使用的T，可以叫做inUse。
当池中没有对象的时候，需要有一个create方法来创建T。如下代码。

```java
public abstract class ObjectPool<T> {

  private HashSet<T> available = new HashSet<>();
  private HashSet<T> inUse = new HashSet<>();

  protected abstract T create();
  
  public synchronized T checkOut() {
    if (available.size() <= 0) {
      available.add(create());
    }
    T instance = available.iterator().next();
    available.remove(instance);
    inUse.add(instance);
    return instance;
  }

  public synchronized void checkIn(T instance) {
    inUse.remove(instance);
    available.add(instance);
  }

  @Override
  public String toString() {
    return String.format("Pool available=%d inUse=%d", available.size(), inUse.size());
  }
}

```

当我们要使用具体的一类对象容器是，可以继承这个抽象类，实现其create方法。例如：

```java
public class OliphauntPool extends ObjectPool<Oliphaunt> {

  @Override
  protected Oliphaunt create() {
    return new Oliphaunt();
  }
}
public class Oliphaunt {

  private static int counter = 1;

  private final int id;

  public Oliphaunt() {
    id = counter++;
    try {
      Thread.sleep(1000);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }

  public int getId() {
    return id;
  }

  @Override
  public String toString() {
    return String.format("Oliphaunt id=%d", id);
  }
}
```

以下是测试代码：

```java
 public static void main(String[] args) {
    OliphauntPool pool = new OliphauntPool();
    System.out.println(pool);
    Oliphaunt oliphaunt1 = pool.checkOut();
    System.out.println("Checked out " + oliphaunt1);
    System.out.println(pool);
    Oliphaunt oliphaunt2 = pool.checkOut();
    System.out.println("Checked out " + oliphaunt2);
    Oliphaunt oliphaunt3 = pool.checkOut();
    System.out.println("Checked out " + oliphaunt3);
    System.out.println(pool);
    System.out.println("Checking in " + oliphaunt1);
    pool.checkIn(oliphaunt1);
    System.out.println("Checking in " + oliphaunt2);
    pool.checkIn(oliphaunt2);
    System.out.println(pool);
    Oliphaunt oliphaunt4 = pool.checkOut();
    System.out.println("Checked out " + oliphaunt4);
    Oliphaunt oliphaunt5 = pool.checkOut();
    System.out.println("Checked out " + oliphaunt5);
    System.out.println(pool);
  }
```
这样我们就实现了一个简单的池。



