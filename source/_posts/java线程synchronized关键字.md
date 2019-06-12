---
title: JAVA多线程synchronized关键字
description: 
date: 2018-04-28 20:40:00
comments: true
tags: 
    - Java
categories:
    - Java
---

# synchronized原理

Synchronized是通过对象内部的一个叫做监视器锁（monitor）来实现的。但是监视器锁本质又是依赖于底层的操作系统的互斥锁（Mutex Lock）来实现的。而操作系统实现线程之间的切换这就需要从用户态转换到核心态，这个成本非常高，状态之间的转换需要相对比较长的时间，这就是为什么Synchronized效率低的原因。这种依赖于操作系统互斥锁（Mutex Lock）所实现的锁我们称之为“重量级锁”

# synchronized用法

```java
/*获取对象锁*/
//这个用法就是使用了obj的锁，来锁定一个代码块。
synchronized(obj){
    // some code...
}

//对整个方法上锁
public synchronized void aMethod(){
    // some code...
}
//这个时候它使用的是当前实例this的锁，相当于下面的模式：

public void aMethod(){
    synchronized(this){
        // some code...
    }
}
/*获取类锁*/
public synchronized static void method(){
    // 
}

synchronized(Class.class){

}

```
# 两个代码块的互斥
```java
    public void do1() {
        synchronized(this) {
            for (int i=0; i < 4; i++) {
                System.out.println(Thread.currentThread().getName() + "-do1-" + i);
            }
        }
        
    }
    public void do2() {
        synchronized(this) {
            for (int i=0; i < 4; i++) {
                System.out.println(Thread.currentThread().getName() + "-do2-" + i);
                try{
                    Thread.sleep(1000);
                }catch(InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```
创建一个实例，开启两个线程，分别调用do1(),do2()方法，这两个方法也是互斥的，即使访问的是两个线程访问的是两个方法，也是互斥的，因为决定是否同时访问的是根据对象锁来的，这两个调用使用的是同一个对象锁

#总结

 synchronized关键字加在方法上还是对象上，如果它作用的对象是非静态的，则它取得的锁是对象；如果synchronized作用的对象是一个静态方法或一个类，则它取得的锁是对类，该类所有的对象同一把锁。 