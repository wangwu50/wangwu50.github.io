---
layout: post
title: golang 并行去执行任务，提高效率
date: 2019-06-14
categories: blog
tags: [技术,总结]
description: golang相关

---

这两天完成了两次将串行的任务转为并行，以此来提高效率，技术栈是golang，由此来记录一下。
### 串行任务转并行几种模型
#### 消费者消费的时候也需要生成者的数据
把串行改成并行，直觉反映就是把任务用多个goruntine去并发请求。
第一个难点是，for循环中，生成者和消费者都需要使用之前list的一些数据，所以最开始想的，用一个channel来放消费者数据就行不通了。
第二个难点是，如何去await。保证所有任务能够执行完。
最后采用了map[string]chan obj这种格式。新建一个map，用于盛放所有的chan，然后并行去请求，把结果放到chan中。这样也解决了await的情况。这个存在的问题是，在处理结果的时候，由于是串行的，可能中间的一个goruntine卡住，会影响整个结果遍历,不过大部分时间不需要考虑这种可能。
```golang
chanMap := make(map[string]chan result)
for _,item := range list{
    key := getKey(item)
    c := make(chan result)
    chanMap[key] = c
    go func(c chan result) {
			c <- productResult(item)
		}(c)
}
for _,item := range list{
    key := getKey(item)
    res := <-chanMap[key]
    doWithResult(res)
}
```
其实上述代码也可以用另外一种数据结构来代替，就是给入参的item里面加一个属性：replyc chan。这样和一个map的效果是差不多的，然后每个item去自己的replyc里面获取结果。



#### 简单的生产者消费者模型
除了上述的情况外，也遇到了一种简单的生产者消费者模型，消费者不需要获取生产者的信息，直接消费就行。这时候想要用一个chan作为数据传输的载体。
难点是，由于用了for range chan，必须关闭循环才能从for跳出不至于卡死。但生产者数量不固定，生产时间也不固定，如何去确定什么时候关闭chan是个问题。
最后用单开一个goruntine+waitGroup专门来关闭chan解决了这个问题,如下图
![73ef87838ecda4cd22f68d1fed715fdc.png](evernotecid://40D35847-66BB-4EC9-830D-E85DC87326F7/appyinxiangcom/24260166/ENResource/p122)

示例代码如下：
```golang
c := make(chan int)
wg := &sync.WaitGroup{}
for _, item := root{
    wg.Add(1)
    go func (item Item){
        c <- doWithItem(item)
        wg.Done()
    }(item)
}
go func() {
	wg.Wait()
	close(c)
}()
for r := range c {
  doWithResult(r)
}
```
### 控制goruntine数量
在优化过程中，发现下游的服务性能有限，不能承受太多的并发，所有想控制goruntine的数量，以免下游服务崩溃。开始调研了一些golang的协程池框架，发现可以设置协程数，例如ant或者async等。但由于go不支持泛型，这些框架写起来比较繁琐，而且性能几乎没有提升，只是增加了理解成本，所有没有采用这个方案。

在网上查到了使用带缓冲的chan来控制并发度。
```golang
var taskLimit = make(chan bool, MaxParallel)
for _, domainID := range domainIDList {
    taskLimit <- true
    go func(id uint64){
        doTask()
        <-taskLimit
    }
}
```
taskLimit <- true 这一行也可以放在goruntine里面。区别就是进程卡在主协程还是子协程里面。由于这是一个定时任务，主协程必须等所有的子协程完成才能下一步，所以选择卡在了主协程里面。
### feature模式
之前使用java的时候，会用到feature模式。其实就是多线程获取结果。java的实现是new了一个featureTask的对象，里面包装了run方法，保留出一个call方法来。然后把结果存在该对象的一个字段中。之前我写过一篇博客分析过
golang里面由于原生支持channel（其实java里面也有queue）,可以用channel来模拟实现feature模式。
具体方法就是上面说道的，生产者的struct加入一个channel作为保存结果的容器。如果要支持取消和超时的话，可以使用select关键字，加一个quit的chan，或者使用time包的after（本质也是一个chan）
```golang
for {
		select {
		case req := <-service:
			go run(op, req)
		case <-quit:
			return
		}

}
```

其实如果是简单的生产者消费者的话，不用给每个结果都配置一个channel，用一个channel就好
