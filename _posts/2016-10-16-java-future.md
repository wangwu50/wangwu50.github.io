---
layout: post
title: java的伪泛型
date: 2016-10-16
categories: blog
tags: [技术,java]
description: java中的Future的思考

---

最近很长时间都在思考多线程和异步的课题。这是一个很大的课题，下面我对java中的异步计算返回的结果——Future做一个深入分析和总结。

## Future是什么，为什么要使用Future。

最开始使用Future是学习ThreadPoolExecutor的时候，当我们使用submit提交一个Callable任务的时候，可以返回一个Future<?>类型的对象。然后调用Future的get，可以获取异步任务的结果，而且有很多例子，是为了让异步任务执行完毕，也可以调用Future.get()来实现。当时觉得Future这个东西很神奇，后来逐步才对Future有了一些清晰的认识。

### Future的定义

Future类位于java.util.concurrent包下，它是一个接口：

```java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

在Future中声明了5个方法，下面依次解释每个方法的作用：

- *cancel(boolean mayInterruptIfRunning)*方法用来取消任务，如果取消任务成功则返回true，如果取消任务失败则返回false。参数mayInterruptIfRunning表示是否允许取消正在执行却没有执行完毕的任务，如果设置true，则表示可以取消正在执行过程中的任务。如果任务已经完成、已经被取消或者一些其他的原因无法取消，此方法会返回false；如果任务正在执行，若mayInterruptIfRunning设置为true，则返回true，若mayInterruptIfRunning设置为false，则返回false；如果任务还没有执行，则无论mayInterruptIfRunning为true还是false，肯定返回true。

- *isCancelled()*方法表示任务是否被取消成功，如果在任务正常完成前被取消成功，则返回 true。

- *isDone()*方法表示任务是否已经完成，若任务完成，则返回true；

- *get()*方法用来获取执行结果，这个方法会产生阻塞，会一直等到任务执行完毕才返回；

- *get(long timeout, TimeUnit unit)*用来获取执行结果，如果在指定时间内，还没获取到结果，就直接返回null。

## Future在jdk中的实现方法

## guava中的Future

## spring中的Future

## 如何使别的多线程框架也使用上Future

## 关于使用Future的一些性能优化的方向和想法