title: CountDownLatch使用
author: 禾田
tags:
  - java并发
categories:
  - java并发
date: 2018-06-13 20:20:00
---
> CountDownLatch允许一个或多个线程等待其他线程完成操作。

## 问题

> 我们需要解析一个Excel里多个sheet的数据，此时可以考虑使用多
> 线程，每个线程解析一个sheet里的数据，等到所有的sheet都解析完之后，程序需要提示解析完成。

## 使用join方法实现


测试代码如下：
```java
package EigthChapter;

public class JoinCountDownLatchTest {
    public static void main(String[] args) throws InterruptedException {
        Thread parser1 = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("parser1 finished");
            }
        });

        Thread parser2 = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("parser2 finished");
            }
        });

        parser1.start();
        parser2.start();
        parser1.join();
        parser2.join();

        System.out.println("parser all finished");
    }
}
```

运行结果：
```
parser2 finished
parser1 finished
parser all finished
```

join用于让当前执行线程等待join线程执行结束。其实现原理是不停检查join线程是否存
活，如果join线程存活则让当前线程永远等待。其中，wait（0）表示永远等待下去，代码片段如下

```
while (isAlive()) {
    wait(0);
}
```

## 使用CountDownLatch方法实现

测试代码：
```
package EigthChapter;

import java.util.concurrent.CountDownLatch;

public class CountDownLatchTest {
    public static void main(String[] args) throws InterruptedException{
        CountDownLatch c = new CountDownLatch(2);
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(1);
                c.countDown();
                System.out.println(2);
                c.countDown();
            }
        }).start();
        c.await();
        System.out.println(3);
    }
}
```

运行结果： 

```
1
2
3
```

CountDownLatch的构造函数接收一个int类型的参数作为计数器，如果你想等待N个点完成，这里就传入N。

当我们调用CountDownLatch的countDown方法时，N就会减1，CountDownLatch的await方法会阻塞当前线程，直到N变成零。由于countDown方法可以用在任何地方，所以这里说的N个点，可以是N个线程，也可以是1个线程里的N个执行步骤。用在多个线程时，只需要把这个CountDownLatch的引用传递到线程里即可。

如果有某个解析sheet的线程处理得比较慢，我们不可能让主线程一直等待，所以可以使用另外一个带指定时间的await方法——await（long time，TimeUnit unit），这个方法等待特定时间后，就会不再阻塞当前线程。join也有类似的方法。