---
title: java IO 基础
date: 2018-04-23 17:29:01
categories: java
tags: [java,面试]
description: java IO 基础知识总结
---
参考：[Java IO技术博客](http://blog.csdn.net/hxpjava1/article/details/56282385)

## BIO（同步并阻塞）

 | 字节输入流 | 字节输出流 | 字符输入流 | 字符输出流
:---:|:---:|:---:|:---:|:---:
抽象基类 | InputStream | OutputStream | Reader | Writer
访问文件 | FileInputStream | FileOutputStream | FileReader | FileWriter
访问数组 | ByteArrayInputStream | ByteArrayOutputStream | CharArrayReader | CharArrayWriter
缓冲流 | BufferedInputStream | BufferedOutputStream | BufferedReader | BufferedWriter
打印流 |  | PrintStream |  | PrintWriter


<br>
**说明：** 上面是一些比较常用的输入输出流，缓冲流可以一次处理一行文本，以换行符为标志，打印流的输出功能非常强大，方便处理打印操作。

**注意：** PrintWriter 的close()方法通常自带flush()。

**使用场景：**   

输入输出内容为文本： 使用字符流 
 
输入输出内容为二进制： 使用字节流


>**java使用处理流来包装节点流是一种典型的装饰器设计模式。**  

## RandomAccessFile类  

**说明：** RandomAccessFile是java输入/输出流体系中功能最丰富的文件内容访问类，可以读取和向文件写入数据。  

**优点：**支持“随机访问”，可以直接跳转到文件的任意位置读写数据。 （注意：在向文件指定位置插入内容时，会覆盖掉插入点之后原有的内容） 

**缺点：**只能读写文件，不能读写其它IO节点。

**使用场景：**  

1）只需要访问文件部分内容，而不是把文件从头读到尾。

2）向已存在的文件后面追加内容，而不是从文件开始的地方直接输出。

## 对象序列化  

**说明：**将实现序列化的Java对象转换成字节序列，这些字节序列可以保存在磁盘上，或者能够通过网络传输，方便以后重新再恢复成原来的对象。

**注意：**定义的类必须实现Serializable接口

## NIO（同步非阻塞）（JDK1.4）

>新IO采用了内存映射文件的方式来处理输入/输出，新IO将文件或文件的一段区域映射到内存中，这样就可以像访问内存一样来访问文件了（模拟了操作系统虚拟内存的概念），相比传统的输入/输出要快！

**Channels and Buffers（通道和缓冲区）：** 标准的IO基于字节流和字符流进行操作的，而NIO是基于通道（Channel）和缓冲区（Buffer）进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。  

**Selectors（选择器）：** Java NIO引入了选择器的概念，选择器用于监听多个通道的事件（比如：连接打开，数据到达）。因此，单个的线程可以监听多个数据通道。

**Asynchronous IO（异步IO）：**  Java NIO可以让你异步的使用IO，例如：当线程从通道读取数据到缓冲区时，线程还是可以进行其他事情。当数据被写入到缓冲区时，线程可以继续处理它。从缓冲区写入通道也类似。





