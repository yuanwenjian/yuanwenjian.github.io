---
title: ThreadLocal详解
description: ThreadLocal类主要解决的就是让每个线程绑定自己的值，可以将ThreadLocal类形象的比喻成存放数据的盒子，盒子中可以存储每个线程的私有数据
date: 2019-06-24 20:40:00
comments: true
tags: 
    - Java
categories:
    - Java
---
通常情况下，我们创建的变量是可以被任何一个线程访问并修改的。如果想实现每一个线程都有自己的专属本地变量该如何解决呢？ JDK中提供的ThreadLocal类正是为了解决这样的问题。 ThreadLocal类主要解决的就是让每个线程绑定自己的值，可以将ThreadLocal类形象的比喻成存放数据的盒子，盒子中可以存储每个线程的私有数据


如果你创建了一个ThreadLocal变量，那么访问这个变量的每个线程都会有这个变量的本地副本，这也是ThreadLocal变量名的由来。他们可以使用 get（） 和 set（） 方法来获取默认值或将其值更改为当前线程所存的副本的值，从而避免了线程安全问题。

# ThreadLocal示例
```java

public class ThreadLocalDemo implements Runnable {

    ThreadLocal<Integer> threadLocal = ThreadLocal.withInitial(() -> 0); //声明ThreadLocal变量


    public static void main(String[] args) {
        ThreadLocalDemo threadLocalDemo = new ThreadLocalDemo();

        ExecutorService threadPoolExecutor = Executors.newFixedThreadPool(10); //线程池核心线程数量为10个
        for (int i = 0; i < 10; i++) {
            threadPoolExecutor.submit(threadLocalDemo); //线程提交任务
        }
    }


    @Override
    public void run() {
        String name = Thread.currentThread().getName();
        System.out.println("线程["+name+"]当前值为"+threadLocal.get());
        threadLocal.set(2);
        System.out.println("线程["+name+"]赋值后值为"+threadLocal.get());

    }
}
```
输出结果
```
线程[pool-1-thread-2]当前值为0
线程[pool-1-thread-3]当前值为0
线程[pool-1-thread-1]当前值为0
线程[pool-1-thread-3]赋值后值为2
线程[pool-1-thread-4]当前值为0
线程[pool-1-thread-4]赋值后值为2
线程[pool-1-thread-2]赋值后值为2
线程[pool-1-thread-6]当前值为0
线程[pool-1-thread-5]当前值为0
线程[pool-1-thread-6]赋值后值为2
线程[pool-1-thread-1]赋值后值为2
线程[pool-1-thread-5]赋值后值为2
线程[pool-1-thread-7]当前值为0
线程[pool-1-thread-7]赋值后值为2
线程[pool-1-thread-8]当前值为0
线程[pool-1-thread-8]赋值后值为2
线程[pool-1-thread-9]当前值为0
线程[pool-1-thread-9]赋值后值为2
线程[pool-1-thread-10]当前值为0
线程[pool-1-thread-10]赋值后值为2
```

从示例中可以看出,当线程1值已经更改为2时,其他线程值并没有变化

# ThreadLocal源码分析

先看Thread源码
```java
...

    // 与该线程相关的threadLocals 值,类型为ThreadLocalMap
    ThreadLocal.ThreadLocalMap threadLocals = null;

    // 存放该线程相关InheritableThreadLocal ,
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
```

ThreadLocal源码

```java
    //ThreadLocal#set() ThreadLocal设置值
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t); //获取当前线程的threadLocals,key为ThreadLocal类型,
        if (map != null)
            map.set(this, value); //赋值
        else
            createMap(t, value);// 第一次为当前线程创建并赋值
    }

        //ThreadLocal.ThreadLocalMap 构造方法,ThreadLocalMap为懒加载,有一条数据后才会创建
        ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
            table = new Entry[INITIAL_CAPACITY];
            int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
            table[i] = new Entry(firstKey, firstValue);
            size = 1;
            setThreshold(INITIAL_CAPACITY);
        }
        //ThreadLocal.ThreadLocalMap# set方法
        private void set(ThreadLocal<?> key, Object value) {

            // We don't use a fast path as with get() because it is at
            // least as common to use set() to create new entries as
            // it is to replace existing ones, in which case, a fast
            // path would fail more often than not.

            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len-1);

            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                ThreadLocal<?> k = e.get();

                if (k == key) {
                    e.value = value;
                    return;
                }

                if (k == null) {
                    replaceStaleEntry(key, value, i);
                    return;
                }
            }

            tab[i] = new Entry(key, value); // Entry 为弱引用,使用ThreadLocal时存在内存溢出的危险
            int sz = ++size;
            if (!cleanSomeSlots(i, sz) && sz >= threshold)
                rehash(); //扫描整个表,删除过期,如果表空间不足,扩容size为双倍
        }
    //ThreadLocal#set() ThreadLocal获取值
    public T get() {
        Thread t = Thread.currentThread(); //获取当前线程
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this); //获取Entry
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
```

# ThreadLocal内存泄露

THreadLocal使用不当会有内存泄露的危险,原因如下
当ThreadLocal变量没有了其他强依赖，而当前线程还存在的情况下，由于线程的ThreadLocalMap里面的key是弱依赖，则当前线程的ThreadLocalMap里面的ThreadLocal变量的弱引用会被在gc的时候回收，但是对应value还是会造成内存泄露，这时候ThreadLocalMap里面就会存在key为null但是value不为null的entry项。在ThreadLocal的set和get和remove方法里面有一些时机是会对这些key为null的entry进行清理的，但是这些清理不是必须发生的