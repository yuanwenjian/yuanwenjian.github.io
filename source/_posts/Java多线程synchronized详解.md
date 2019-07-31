---
title: Java synchronized关键字
description: 在Java中，synchronized关键字是用来控制线程同步的，就是在多线程的环境下，控制synchronized代码段不被多个线程同时执行。synchronized既可以加在一段代码上，也可以加在方法上
date: 2018-05-24 20:40:00
comments: true
tags: 
    - Java
categories:
    - Java
---

syncchronized  是java关键字，只要在代码块(方法)添加synchronized ,即可实现同步功能，synchronized是一种互斥锁，每次只能允许单个线程进入被锁定的代码块

1. 使用

synchronized使用非常简单，只要在想同步的代码块添加synchronized关键词即可，可以修饰静态方法，非静态方法，对象
```java
public class SyncTest {

    //修饰this 对象
    public  void syncDemo() {
        synchronized (this){
            System.out.println("synchronized demo");
        }
    }
    //修饰静态方法
    public synchronized static void staticSycnDemo() {
        System.out.println("staticSycnDemo");
    }

    //修饰普通方法
    public synchronized   void syncDemoMethod() {
        System.out.println("staticSyncDemo");
    }

    //修饰Class
    public void syncClassDemo() {
        synchronized (SyncTest.class) {
            System.out.println("syncClassDemo");
        }
    }
}
```

2. 原理
使用javap -v SyncTest 查看字节码文件，以上代码下图所示

修饰this syncDemo()
![syncDemo][syncDemo]

修饰静态方法 staticSycnDemo()
![staticSycnDemo][staticSycnDemo]

修饰普通方法 syncDemoMethod()
![syncDemoMethod][syncDemoMethod]

修饰class 
![syncClassDemo][syncClassDemo]

从上图可知，分为两种情况，一是通过monitorenter，monitorexit指令实现，另一种为ACC_SYNCHRONIZED标识


- monitorenter 指令
1、如果monitor的进入数为0，则该线程进入monitor，然后将进入数设置为1，该线程即为monitor的所有者。

2、如果线程已经占有该monitor，只是重新进入，则进入monitor的进入数加1.

3.如果其他线程已经占用了monitor，则该线程进入阻塞状态，直到monitor的进入数为0，再重新尝试获取monitor的所有权。

- monitorexit

执行monitorexit的线程必须是objectref所对应的monitor的所有者。

指令执行时，monitor的进入数减1，如果减1后进入数为0，那线程退出monitor，不再是这个monitor的所有者。其他被这个monitor阻塞的线程可以尝试去获取这个 monitor 的所有权。 

通过这两段描述，我们应该能很清楚的看出Synchronized的实现原理，Synchronized的语义底层是通过一个monitor的对象来完成，其实wait/notify等方法也依赖于monitor对象，这就是为什么只有在同步的块或者方法中才能调用wait/notify等方法，否则会抛出java.lang.IllegalMonitorStateException的异常的原因。

ACC_SYNCHRONIZED标识

JVM就是根据该标示符来实现方法的同步的：当方法调用时，调用指令将会检查方法的 ACC_SYNCHRONIZED 访问标志是否被设置，如果设置了，执行线程将先获取monitor，获取成功之后才能执行方法体，方法执行完后再释放monitor。在方法执行期间，其他任何线程都无法再获得同一个monitor对象。 其实本质上没有区别，只是方法的同步是一种隐式的方式来实现，无需通过字节码来完成。

### 总结
synchronized 修改普通类及对象时，获取的是对象的monitor,修饰静态类及Class获取的是类对象的monitor。所以多个不同对象实例，访问同步普通方法时，会出问题

[syncDemo]:/images/sync/syncDemo.png
[staticSycnDemo]:/images/sync/staticSycnDemo.png
[syncDemoMethod]:/images/sync/syncDemoMethod.png
[syncClassDemo]:/images/sync/syncClassDemo.png