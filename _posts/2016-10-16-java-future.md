---
layout: post
title: java中的future总结
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

- *cancel(boolean mayInterruptIfRunning)*方法用来取消任务，如果取消任务成功则返回true，如果取消任务失败则返回false。参数mayInterruptIfRunning表示是否允许取消正在执行却没有执行完毕的任务，如果设置true，则表示可以取消正在执行过程中的任务。如果任务已经完成、已经被取消或者一些其他的原因无法取消，此方法会返回false；如果任务正在执行，若mayInterruptIfRunning设置为true，则返回true，若mayInterruptIfRunning设置为false，则返回false；如果任务还没有执行，被取消后就不会被执行了。

- *isCancelled()*方法表示任务是否被取消成功，如果在任务正常完成前被取消成功，则返回 true。

- *isDone()*方法表示任务是否已经完成，若任务完成，则返回true；

- *get()*方法用来获取执行结果，这个方法会产生阻塞，会一直等到任务执行完毕才返回；

- *get(long timeout, TimeUnit unit)*用来获取执行结果，如果在指定时间内，还没获取到结果，会抛出TimeoutException的异常。

### 为什么要使用Future

Future从字面意思理解是未来，在java中其实是一个对未来结果的引用。我们使用Future很多情况都是在线程池中使用的，其实Future和线程池没有太直接的关系。或者可以理解为，线程池执行完任务时，把这个任务未来的引用返回给了使用者。所以在ThreadPoolExecutor中，使用的是RunnableFuture（将Runnable和Future结合的接口）。而对于真正纯粹的Future使用，其实并不限于一定要在ThreadPoolExecutor中。任何一个结果还没有确定的对象，我们对其未来结果的引用都可以使用Future来完成。也就是说，Future代表结果的将来时。对于一个未来结果的引用，我们关心什么呢？首先我们一定关心结果的完成情况、结果的获取。如果结果很难获取到，我们会取消结果。所以我们也会关心结果是否取消成功。这正是Future接口里面5个的方法的作用。

## Future在jdk中的实现方法

谈了那么多为什么要使用Future，那么我们怎么样才能使用Future呢？首先先看看jdk中内置的Future使用方法。

### FutureTask

在java8之前，FutureTask是默认且唯一的Future实现类。它实现了RunnableFuture接口，RunnableFuture继承了Runnable和Future。本质上，FutureTask是一个实现了Runnable的类。因此可以放到线程池中去执行。FutureTask构造的时候需要Callable（或者一个Runnable和result，但其也会转化为Callable）。我们观察一下它的run方法，其核心是
```java
     Callable<V> c = callable;
            if (c != null && state == NEW) {
                V result;
                boolean ran;
                try {
                    result = c.call();
                    ran = true;
                } catch (Throwable ex) {
                    result = null;
                    ran = false;
                    setException(ex);
                }
                if (ran)
                    set(result);
            }
```
可以发现，FutureTask的run方法中执行了Callable的call方法，并把结果放到了一个地方供外部取用。而在FutureTask内部有一系列状态标志，这样在任务执行过程中，状态会跟着改变。这样外部就可以根据状态来判断完成情况了。

因此，使用Future的一种情况就是直接使用FutureTask，如下代码所示：
```java
	@Test
	public void testFutureTask() throws InterruptedException, ExecutionException {
		FutureTask<String> ft = new FutureTask<String>(new Callable<String>() {
			@Override
			public String call() throws Exception {
				TimeUnit.SECONDS.sleep(1);
				return "Done!";
			}
		});
		ExecutorService service = Executors.newCachedThreadPool();
		service.execute(ft);
		String result = ft.get();
		System.out.println(result);
	}
```
这里，FutureTask其实就是一个的Future，我们可以执行有关Future一系列操作。

但是，如果每次要使用Future，都需要new一个和业务无关的FutureTask，这样也会给使用带来不便。在ExecutorService接口中有一个submit的方法。这个方法可以直接返回一个Future，使用者不需要关心Future是怎么来的，直接使用即可。这样给使用带来了很大的方便。
```java
	@Test
	public void testFuture() throws InterruptedException, ExecutionException {
		ExecutorService service = Executors.newCachedThreadPool();
		Future<String> future = service.submit(new Callable<String>() {
			@Override
			public String call() throws Exception {
				TimeUnit.SECONDS.sleep(1);
				return "Done!";
			}
		});
		String result = future.get();
		System.out.println(result);
	}
```
看一下submit的代码:
```java
public Future<?> submit(Runnable task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<Void> ftask = newTaskFor(task, null);
        execute(ftask);
        return ftask;
    }
    
 protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) {
        return new FutureTask<T>(runnable, value);
    }
```
我们可以发现，submit其实也是new了一个FutureTask，然后将其返回的。因此，在ThreadPoolExecutor中，其实是使用FutureTask来实现Future的。

### CompletableFuture

## guava中的Future

### ListenableFuture

### SettableFuture

## spring中的Future

### spring对Future的处理

## 如何使别的多线程框架也使用上Future

## 关于使用Future的一些性能优化的方向和想法