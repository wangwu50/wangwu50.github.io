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

Future从字面意思理解是未来，在java中其实是一个对未来结果的引用。我们使用Future很多情况都是在线程池中使用的，其实Future和线程池没有太直接的关系。可以理解为，线程池执行完任务时，把这个任务未来的引用返回给了使用者。所以在ThreadPoolExecutor中，使用的是RunnableFuture（将Runnable和Future结合的接口）。而对于真正纯粹的Future使用，其实并不限于一定要在ThreadPoolExecutor中。任何一个结果还没有确定的对象，我们对其未来结果的引用都可以使用Future来完成。也就是说，Future代表结果的将来时。对于一个未来结果的引用，我们关心什么呢？首先我们一定关心结果的完成情况、结果的获取。如果结果很难获取到，我们会取消结果。所以我们也会关心结果是否取消成功。这正是Future接口里面5个的方法的作用。


## Future的实现

### Future在jdk中的实现方法

谈了那么多为什么要使用Future，那么我们怎么样才能使用Future呢？首先先看看jdk中内置的Future使用方法。

#### FutureTask

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

这里，FutureTask其实就是一个的Future，我们可以执行有关Future的一系列操作。

但是，如果每次要使用Future，都需要new一个和业务无关的FutureTask，这样也会给使用带来不便。在ExecutorService接口中有一个submit的方法。这个方法可以直接返回一个Future，使用者不需要关心Future是怎么来的，直接使用即可。

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

#### CompletableFuture

前面说过，Future对象其实是一个对未来结果的引用，java8中的新的Future实现类CompletableFuture就很好的体现了这一点。我们可以单独使用它：

```java
@Test
public void testCompletableFuture() throws InterruptedException, ExecutionException
{
    CompletableFuture<String> cF=new CompletableFuture<>();
    cF.complete("123");
    String result = cF.get();
    System.out.println(result);
    cF.complete("222");
    String result2 = cF.get();
    System.out.println(result2);
}
```

输出：123 123

当CompletableFuture的get方法调用的时候，如果complete没有被调用，会一直阻塞，直到complete被调用。可以看出，complete的重复调用时不起作用的，只有第一次的调用生效。
这样的实现，其实非常清晰的将Future的功能和Executor分离出去了。我们可以在任意代码中使用CompletableFuture。当然，java8中也提供了封装好的方法来实现和之前线程池的submit一样的功能。

```java
@Test
public void testCompletableFutureAsync() throws InterruptedException, ExecutionException {
    Future<String> future = CompletableFuture.supplyAsync(() -> {
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "Done!";
    }, Executors.newCachedThreadPool());
    String result = future.get();
    System.out.println(result);
}
```

supplyAsync调用了asyncSupplyStage方法，asyncSupplyStage方法如下：

```java
static <U> CompletableFuture<U> asyncSupplyStage(Executor e,
                                                 Supplier<U> f) {
    if (f == null) throw new NullPointerException();
    CompletableFuture<U> d = new CompletableFuture<U>();
    e.execute(new AsyncSupply<U>(d, f));
    return d;
}
```

AsyncSupply实现了Runnable，其run方法将执行结果放到了CompletableFuture中。可以看出和FutureTask差不多，将任务组合成一个最后返回了一个CompletableFuture。

除此之外，CompletableFuture也提供了回调、组合Future的方法。但这些方法都和java8的新特性lambda表达式有关。
待续。

### guava中的Future

Guava是google提供一个开源工具包，里面对Future进行了很好的扩展，guava中的很多实现最后都被jdk8采用了。

#### ListenableFuture

ListenableFuture是guava提倡的使用Future的方式，它在Future的基础上扩展了一个*void addListener(Runnable listener, Executor executor)*方法。也就是说，在Future执行完之后，我们可以添加回调方法。当然，这个addListener使用起来也不是太方便。guava提供了Futures工具类中的addCallback(ListenableFuture<V> future,FutureCallback<? super V> callback)方便使用。为了能够返回ListenableFuture，guava提供了MoreExecutors工具类来修饰任意Executor。如下代码：

```java
@Test
@SneakyThrows
public void testGuavaFutureCallback() {
    ListeningExecutorService service = MoreExecutors.listeningDecorator(Executors.newSingleThreadExecutor());
    ListenableFuture<String> future = service.submit(() -> {
        TimeUnit.SECONDS.sleep(1);
        return "Done!";
    });
    Futures.addCallback(future, new FutureCallback<String>() {
        @Override
        public void onSuccess(String result) {
            System.out.println(result);
        }
        @Override
        public void onFailure(Throwable t) {
        }
    });
    TimeUnit.SECONDS.sleep(2);
}
```

Futures提供了很多有用的方法，如果有多个Future，可以使用allAsList方法将多个Future转为一个Future统一处理：

```java
@Test
@SneakyThrows
public void testAsList() {
    ListeningExecutorService service = MoreExecutors.listeningDecorator(Executors.newSingleThreadExecutor());
    ListenableFuture<String> future = service.submit(() -> {
        TimeUnit.SECONDS.sleep(1);
        return "Done!";
    });
    ListenableFuture<String> future2 = service.submit(() -> {
        TimeUnit.SECONDS.sleep(1);
        return "Done too!";
    });
    ListenableFuture<List<String>> allAsList = Futures.allAsList(future, future2);
    Futures.addCallback(allAsList, new FutureCallback<List<String>>() {
        @Override
        public void onSuccess(List<String> result) {
            result.forEach(a->System.out.println(a));
        }
        @Override
        public void onFailure(Throwable t) {
            t.printStackTrace();
        }
    });
    TimeUnit.SECONDS.sleep(2);
}
```

还有transformAsync方法，可以把ListenableFuture和另外一个ListenableFuture结合起来，限于篇幅不在赘述。总之ListenableFuture是guava提供的一个很好用的api。

那么guava是怎么样实现任务完成回调的呢？

查看guava源代码可以发现，实现Future回调可以有两种方式：

- 继承FutureTask。 Futuretask中保留了 protected void done() { }的方法供继承使用，该方法会被在任务执行完毕后调用。重写done方法，在done方法中执行回调逻辑即可。

- 重写Future。 查看MoreExecutors.listeningDecorator可知，该方法返回的是guava自己实现的一套Future。该Executors重写了返回的Future实现类方法：

```java
@Override protected final <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) {
  return TrustedListenableFutureTask.create(runnable, value);
}
```

TrustedListenableFutureTask的Future主要实现在AbstractFuture中，run方法中调用了AbstractFuture的set方法。也就是说，其实guava在jdk8之前就实现了Future和Runnable的分离。在set()方法中，会调用complete()方法，该方法会执行listener的runnable方法。

#### SettableFuture

SettableFuture是guava对外提供的他们自己写的Future实现类。和jdk8的CompletableFuture使用方法几乎一样，先调用set放值，然后再调用get取值。

```java
@Test
@SneakyThrows
public void testSettableFuture() {
    SettableFuture<String> future = SettableFuture.create();
    future.set("123");
    String result = future.get();
    System.out.println(result);
    future.set("222");
    String result2 = future.get();
    System.out.println(result2);
}
```

输出：123 123

可以看出，SettableFuture和CompletableFuture几乎相同，其set方法也只是第一次调用生效。

google推荐使用他们重写的Future实现类，该实现类比FutureTask有更好的性能。可以看出，该Future和jdk8的CompletableFuture有异曲同工之妙。如果使用的java版本是jdk8，那推荐使用CompletableFuture；如果无法使用jdk8，那么强烈推荐使用SettableFuture。

### spring中的Future

#### spring对Future的处理

spring中对Future封装更多的是为了使用为主，spring定义了@Async注解，利用其aop的功能，可以方便的实现异步的代码。但如何在使用注解后还可以返回结果是个难题，注解不能改变本身方法的返回值。如果在aop中直接异步后调用get返回结果，导致异步方法阻塞，就失去了异步程序的意义。因此spring开发者使用了一个比较巧妙地方法，首先定义一个同步的Future实现类，在构造器中将result传入，get方法直接同步返回result。这样，方法可以返回一个“假的”Future<?>类型。然后通过spring的aop功能，将假的Future变成真的Future。其核心代码如下：

```java
Callable<Object> task = new Callable<Object>() {
@Override
public Object call() throws Exception {
    try {
        Object result = invocation.proceed();
        if (result instanceof Future) {
            return ((Future<?>) result).get();
        }
    }
    catch (ExecutionException ex) {
        handleError(ex.getCause(), userDeclaredMethod, invocation.getArguments());
    }
    catch (Throwable ex) {
        handleError(ex, userDeclaredMethod, invocation.getArguments());
    }
    return null;
}
};
```

```java
if (Future.class.isAssignableFrom(returnType)) {
        return executor.submit(task);
    }
    else {
        executor.submit(task);
        return null;
    }
```

这样，我们定义方法的时候就可以定义返回Future的API。

spring在高版本也定义了一套自己的ListenableFuture，其主要实现思路是继承FutureTask。我觉得不如guava提供的ListenableFuture好用。

### 如何使别的多线程框架也使用上Future

根据spring对Future的处理，我实现了基于disruptor的异步注解，主要思路是让disruptor抽象类继承AbstractExecutorService，实现execute方法。然后aop中使用和spring相似的处理方式。当然有所变化的是，我加入了guava的ListenableFuture。

```java
if (ListenableFuture.class.isAssignableFrom(returnType)) {
    return MoreExecutors.listeningDecorator(executor).submit(task);
} else if (Future.class.isAssignableFrom(returnType)) {
    return executor.submit(task);
} else {
    executor.submit(task);
    return null;
}
```

## 关于设置异步API

编写代码时，在大部分情况下，我们都会设计同步的API，好理解，容易编写。但是考虑到一些网络IO或者计算频繁的情景下，我们无法控制一些方法的调用时间，因此这时候同步代码就会有响应不及时的问题，有时候我们也会有需要根据返回的部分结果实时返回结果的情况，所以设计异步API是一个很好的选择。让一些耗时的操作慢慢执行，需要的时候再取用他们，这种形式正是Future的意义所在。之前mybatis和hibernate中的懒加载正是异步API的很好的体现。

## 关于使用Future的一些性能优化的方向和想法

无论什么Future，在使用中本质是需要new一个新的对象来控制结果的状态的。如果能够重用这些对象，比如加入一个销毁之前的结果的方法，使得set或者complete方法可以重新调用。这样就可以减少新创建对象的开销。或者在一个对象内部维护一个容器，把结果都放到容器里面。这些想法是我下一步可能会研究的方向。
