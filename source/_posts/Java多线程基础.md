title: Java多线程基础
author: 禾田
tags:
  - 多线程
categories:
  - java
date: 2018-03-25 10:21:00
description: java多线程基础知识
---
## 线程和进程的区别联系
**进程：**操作系统运行中的程序（进程和程序的区别是程序只是一个静态的指令集合，而进程则是系统中一个活动的指令集合，加入了时间）。  

**线程：**进程能够执行多项任务，而每一项任务就相当于一个线程。

**进程之间不能共享资源，但是线程之间可以共享资源。**

## 并发性和并行性区别  
**并行:** 有多条指令在多个处理器上同时执行。

**并发:** 同一时刻只有一条指令执行，但是多个进程指令会被快速轮换执行，使得看起来好像有多个进程在同时执行。

## 创建线程的方式
1） 继承Thread类来创建线程类，然后要重写run()方法。

2）实现Runnable接口来创建线程类，然后实现run()方法，将创建的实现Runnable接口的对象作为Thread的target。

3）实现Callable接口，功能是Runnable的增强版本（java5之后提供）。接口中定义的方法有返回值，可以抛出异常。

### 实现Runnable或Callable接口

**优点：** 
 
1）线程类只是实现了Runnable接口或Callable接口，它依然还可以继承其他类。

2）多个线程可以同时共享一个target对象，十分适合多个相同线程来处理同一份资源。

**缺点：**

编码可能较为复杂，如果要访问当前线程的话，则必须要使用Thread.currentThread()方法。

### 继承Thread类   
优点：  
- 程序编写起来简单，如果需要访问当前线程的话，不用使用Thread.currentThread()方法，直接使用this即可。

缺点：  
- 线程类以及继承了Thread类，不能再继承其他父类了。

## 线程的生命周期
![图片飞走了~](http://owq01tqh9.bkt.clouddn.com/11.png)
**线程5种状态：**   

1）new(创建)：线程通过new方法创建。

2）Runnable(就绪)：当线程调用start()方法，线程进入就绪状态，等待系统调度。

3）Running(运行): 当系统调度时，线程进入运行状态。

4）Blocked(阻塞):  一个正在运行的线程因某些原因不能继续运行时，它就进入阻塞状态。  

5）Dead(死亡)：线程正常结束或异常退出。

**线程阻塞的几种情形：**

1）处于运行状态的线程若遇到sleep()方法，则线程进入睡眠状态，不会让出资源锁，sleep()方法结束，线程转为就绪状态，等待系统重新调度。  
2）处于运行状态的线程可能在等待io，也可能进入挂起状态。io完成，转为就绪状态。  
3）处于运行状态的线程调用yield()方法，线程转为就绪状态。（yield只让给权限比自己高的）。
4）处于运行状态的线程遇到wait()方法（object的方法），线程处于等待状态，需要notify()/notifyALL()来唤醒线程，唤醒后的线程处于锁定状态，获取了“同步锁”，之后，线程才转为就绪状态。  
5）处于运行的线程synchronized，加上后变成同步操作。处于锁定状态，获取了“同步锁”，之后，线程才转为就绪状态。

## 线程控制

1）join()方法（Thread类提供的静态方法） [java 线程方法join的简单总结](https://www.cnblogs.com/lcplcpjava/p/6896904.html)  
说明：让一个线程等待另一个线程执行完成。当前线程必须等待调用join()方法的线程执行完成后，才能继续向下执行。

2）sleep()方法（Thread类提供的静态方法）   
说明：暂停当前线程，并进入阻塞状态，这时，如果有其它线程，则其它线程会执行。

3）yield()方法（Thread类提供的静态方法）   
说明：暂停当前线程，将当前线程转入就绪状态，不是阻塞状态，这时只有大于等于当前线程优先级状态的线程才能获得从就绪状态转入运行状态的机会。

## 线程同步

**同步代码块**

```
//格式
synchronized(obj) {
    ......
    //此处代码为同步代码块代码
}
```
**说明：**上面格式中的obj就是同步监视器。当线程开始执行同步代码块之前，必须先得到同步监视器的锁定。这样可以保证同一时刻只有一个线程能执行当前代码，从而保证线程安全。

**同步方法**

```
//格式
public synchronized void draw() {
    ......
}
```
**说明：**对synchronized修饰的实例方法（非static方法）来说，不需要显示的指定同步监视器，同步方法的同步监视器是this，也就是调用该方法的对象。

**释放同步监视器的时机：**

1.）当前线程的同步方法、同步代码块执行结束，当前线程就会释放同步监视器。  
2）当前线程的同步方法、同步代码块出现了未处理的Error和Exception，导致代码块异常结束了。
3）当前线程执行同步方法、同步代码块时，程序执行了同步监视器对象的wait()方法，则当前线程暂停，并释放同步监视器。

[ java中的synchronized（同步代码块和同步方法的区别）](https://blog.csdn.net/h_gao/article/details/52266950)

**同步锁Lock**  

>通过显示定义同步锁对象来实现同步，此时同步锁是由Lock对象来充当。（java5之后实现）

**常用：** ReentrantLock（可重入锁）  
```Java
//格式
class X {
    //定义锁对象
    private final ReentrantLock lock = new ReentrantLock(); 
    //...
    //定义需要保证线程安全的方法
    public void m() {
        //加锁
        lock.lock(); 
        try {
            //需要保证线程安全的代码
            //...method body
        } finally {
            lock.unlock(); 
        }
        
    }
}
```
**优点：**使用同步锁Lock比同步方法和同步代码块更加灵活！

## 死锁    
>两个线程相互等待对方释放同步监视器时就会发生死锁。 

死锁很容易发生，特别是在系统中出现了多个同步监视器的情况下。


## 线程间通讯

### 传统线程通讯（对应同步代码块和同步方法情况）

调用Object类的wait()、notify()、notifyAll()方法来实现线程间通讯，这三个方法是jvm中的native方法。

**wait():**

调用某个对象的wait()方法能让当前线程阻塞，并且当前线程必须拥有此对象的monitor（即锁）

**notify():**

调用某个对象的notify()方法能够唤醒一个正在等待这个对象的monitor的线程，如果有多个线程都在等待这个对象的monitor，则只能唤醒其中一个线程；。

**notifyAll():**

调用notifyAll()方法能够唤醒所有正在等待这个对象的monitor的线程；

**注意：** 一个线程被唤醒不代表立即获取了对象的monitor，只有等调用完notify()或者notifyAll()并退出synchronized块，释放对象锁后，其余线程才可获得锁执行。

### Condition（对应Lock的情况）（JDK1.5之后出现）

Condition类中提供了await()、signal()、signalAll()方法来对应wait()、notify()、notifyAll()方法，使用方法基本相同。

```Java
//格式
class X {
    //定义锁对象
    private final Lock lock = new ReentrantLock(); 
    private final Condition cond = lock.newCondition();
    
    ....
    
    cond.await();
    cond.signal();
    cond.signalAll();
}
```

相比使用Object的wait()、notify()，使用Condition的await()、signal()这种方式实现线程间协作更加安全和高效。

### BlockingQueue（java5提供）  

它是Queue的子接口，作用不是作为容器，而是作为一个线程同步的工具。  

**特征：**   

（经典的生产者消费者模型）

当生产者线程尝试向BlockingQueue中放入元素时，如果队列已经满了，那么线程就会被阻塞；

当消费者线程尝试向BlockingQueue中取出元素时，如果队列已空，那么线程也会被阻塞；

对应put()方法和take()方法。

## 线程池
[线程池原理](http://cmsblogs.com/?s=%E7%BA%BF%E7%A8%8B%E6%B1%A0)

1）线程池能够提高系统性能 

**说明：** 系统启动线程成本高，因为涉及到和操作系统的交互。这种情况下，使用线程池能够提高系统性能。特别是需要创建大量生存期比较短的线程时候，才更需要使用线程池。

2）线程池的最大线程数参数可以控制系统中并发线程数目不会超过设置的线程数  

**说明：** 当系统中包含大量并发线程时，会导致系统性能剧烈下降，甚至导致JVM崩溃。

**过程：** 线程池在系统启动时就会创建大量空闲的线程，程序将一个Runnable对象或Callable对象传给线程池，则线程池就会启动一个线程来执行它们的run()或call()方法，当run()或call()方法执行结束之后，线程不会死亡，它会再次返回到线程池中从而成为空闲状态，等待下一个run()或call()方法。

Executors工厂类来产生线程池。

```Java
public class ThreadPoolTest {
    public static void main(String[] args) throws Exception {
        //创建一个具有固定线程数(6)的线程池
        ExecutorService pool = Executors.newFixedThreadPool(6);
        //使用Lambda表达式创建Runnable对象
        Runnable target = () -> {
            for (int i = 0; i < 100; i++) {
                System.out.println(Thread.currentThread().getName() + "的i值为：" + i);
            }
        };
        //向线程池中提交两个线程
        pool.submit(target);
        pool.submit(target);
        //关闭线程池
        pool.shutdown();
    }  
}
```

## ThreadLocal类
>线程局部变量（ThreadLocal）为每一个使用该变量的线程都提供一个变量值的副本，使每一个线程都可以独立地改变自己的副本，而不会和其他线程的副本冲突。

**ThreadLocal类方法 :** 

T get()  :  取得当前线程副本数据值  

void remove()  ：删除当前线程副本数据值  

void set(T value)  ：设置当前线程副本数据的值  

ThreadLocal类是从另一个角度来解决多线程的并发访问，ThreadLocal将需要并发访问的资源复制多份，每个线程拥有一份资源，每个线程都拥有自己的资源副本，从而也就没有必要对该变量进行同步了。

**实用场景：** 

1）如果多个线程之间需要共享资源，来达到线程之间的通信功能，就使用同步机制。  

2）如果只需要隔离多个线程之间的共享冲突，可以使用ThreadLocal。

## Volatile关键字

[深入分析volatile的实现原理](http://cmsblogs.com/?p=2092)