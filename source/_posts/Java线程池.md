---
title: JAVA线程池创建及原理
description: 合理利用线程池,能带来大量好处,一减少资源消耗,频繁创建销毁线程会销毁大量资源,二是提高响应速度,减少创建的时间,还可以进行线程的管理
date: 2018-07-20:40:00
comments: true
tags: 
    - Java
categories:
    - Java
---

# 线程池构造
线程池类图
![threadPool][threadPool]

线程池构造方法

```java
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.acc = System.getSecurityManager() == null ?
                null :
                AccessController.getContext();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```
corePoolSize 核心线程数,
maximumPoolSize 最大线程数
keepAliveTime 空闲线程存活时间
unit 单位
workQueue 任务队列
threadFactory 创建新线程工厂
handler 当线程数量及队列数量达到最大值时线程池处理策略

# 线程池执行任务流程
```java
public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        int c = ctl.get();
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        else if (!addWorker(command, false))
            reject(command);
    }
```

线程池执行流程为,提交任务,如果线程数小于corePoolSize 创建新线程,否则判断任务队列是否饱和,不饱和将任务添加到队列中,如果饱和判断,核心线程数是否小于maximumPoolSize,是的创建新线程,否则执行线程策略

流程图
![process][process]

# Executors 工具类
1. newFixedThreadPool(),创建固定数量线程线程池,共享同一个无边队列,如果所有线程都处于工作状态是添加任务,会将任务添加到队列中等待,直到线程可用来为止,如果有线程执行中出现异常,会创建一新线程代替
```java
    ExecutorService executors = Executors.newFixedThreadPool(5);

    //Executors#newFixedThreadPool
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```

2. newSingleThreadExecutor ,创建只包含一个线程的线程池,感觉与newFixedThreadPool(1)类似,两者区别查半天没有结果
```java
    ExecutorService executors = Executors.newSingleThreadExecutor();

    //Executors#()
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```
3. newCachedThreadPool,根据任务创建线程,最大值为Integer.MAX_VALUE,适用于大量短暂任务,如果任务量过多,可能产生大量线程,任务队列为SynchronousQueue
```java
    ExecutorService executors = Executors.newCachedThreadPool();

    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```
4. newScheduledThreadPool,该线程池可定时执行任务,任务队列为DelayedWorkQueue
```java
    ExecutorService executors = Executors.newScheduledThreadPool(4);

    public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }

    public ScheduledThreadPoolExecutor(int corePoolSize) {
        super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
              new DelayedWorkQueue());
    }
```
ScheduledThreadPoolExecutor 类图
![scheduled][scheduled]

# 线程池处理策略

当任务队列饱和,并且线程数已达到最大线程数,会执行拒绝策略,拒绝策略分为四种
1. AbortPolicy 该策略抛异常 RejectedExecutionException.该策略为ThreadPoolExecutor默认策略
```java
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            throw new RejectedExecutionException("Task " + r.toString() +
                                                 " rejected from " +
                                                 e.toString());
    }
```

2. DiscardPolicy策略 该策略什么也不做,直接抛弃任务
```java
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
}
```
3. DiscardOldestPolicy策略,将队列中最老的的任务丢弃,并执行新的任务
```java
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            if (!e.isShutdown()) {
                e.getQueue().poll();
                e.execute(r);
            }
        }
```
4. CallerRunsPolicy 使用当前调用线程执行

```java
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            if (!e.isShutdown()) {
                r.run();
            }
        }
```
[threadPool]:/images/jdk/threadPool.png
[process]:/images/jdk/process.png
[scheduled]:/images/jdk/scheduled.png