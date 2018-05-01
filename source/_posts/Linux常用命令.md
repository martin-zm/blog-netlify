title: Linux常用命令
author: 禾田
tags:
  - linux
categories:
  - linux
date: 2018-02-05 17:15:00
description: Linux面试常考命令
---
### 竖杠 |   

管道就像水管一样，将前面命令的执行结果输送给后面的命令。

```
//'ls -l'负责收集当前目录下的文件信息，然后将这些文件名作为结果输送到管道，
//wc这个命令接着就从管道中把它们读取出来，并计算出行数，单词个数和总字符数。
ls -l | wc
```
---
### 查看日志命令(tail)

tail的功能就是输出文件的尾部内容。  

Linux文件的日志文件通常存储在'/var/log'目录下。

```
//输出syslog文件的尾部几行
tail /var/log/syslog
//tail会继续监视日志文件，输出写入到文件的下一行。这意味着你可以在终端窗口里面实时关注什么写入到syslog
tail -f /var/log/syslog
//只想查看写入到syslog的尾部5行
tail -f -n 5 /var/log/syslog 
```
---
### CPU相关命令  

ps：查看当前瞬间系统的进程信息

pstree：以树状方式查看当前系统的进程

top：持续跟踪系统的进程情况
```
//动态跟踪所有进程（<向左翻页，>向右翻页）
top
//动态跟踪PID为1234的进程
top -p 1234
```

kill：给一个指定的进程发送一个信号
```
//列出当前系统所支持的所有信号
kill -l
//给PID为1234的进程发送9号信号
kill -9 1234    
//给PID为1234的进程发送SIGINT信号
kill -s SIGINT 1234
```

nice：以某一个指定的NICE值启动进程
```
//以NICE值为5的起始状态来启动程序./example
nice -n 5 ./example
```

renice：动态修改一个进程的NICE值
```
//将PID为1234的进程的NICE值调整为15
renice -n 15 1234
```
---
### 网络相关命令

ifconfig：查看系统当前活跃的网络接口
```
//查看系统当前所有的活跃的网络接口信息
ifconfig 
//查看eth0相关的信息
ifconfig eth0 
//将网络接口eth0的IP地址设置为192.168.1.5
ifconfig eth0 192.168.1.5 
```
ping：给某主机发送ICMP数据包以检测网络

```
//给百度服务器发送ICMP数据包以检测网络是否连通
ping www.baidu.com
```

netstat：查看系统网络连接的相关信息

```
//查看系统中所有状态的网络连接信息
netstat -a
//查看系统中处于监听状态的网络连接信息
netstat -l
//查看系统中所有状态的TCP（或者UDP 或者UNIX域）的网络连接信息
netstat -at （或者 netstat -au 或者 netstat -ax）
```

ifdown：禁用网络接口

```
//禁用网络接口eth0
ifdown eth0
```

ifup：启用网络接口

```
//启用网络接口eth0
ifup eth0
```

host：查看域名所对应的IP地址
```
//查看域名www.baidu.com所对应的IP地址，以此来检测本机的DNS服务设置正确与否
host www.baidu.com
```

route：查看、设置路由和网关相关信息

```
//查看网关地址
route -n
//添加默认网关为192.168.1.1
route add default gw 192.168.1.1
```

ln：创建一个连接文件

```
//为文件file创建一个硬连接（别名）,叫file1
ln file file1 
//为文件file创建一个软连接，叫file2 
ln -s file file2
```






