title: Redis基本操作
author: 禾田
tags:
  - redis
categories:
  - redis
date: 2018-04-20 10:24:00
description: redis缓存基础知识
---
## Redis简介
Redis是一个速度非常快的非关系数据库（non-relational database），它可以存储键（key）与5种不同类型的值（value）之间的映射（mapping），可以将存储在内存的键值对数据持久化到硬盘，可以使用复制特性来扩展读性能，还可以使用客户端分片来扩展写性能
<br>
1. 与关系数据库对比  
Redis属于人们常说的NoSQL数据库或者非关系数据库：Redis不使用表，它的数据库也不会预定义或者强制去要求用户对Redis存储的不同数据进行关联。
2. 与memcached对比
这两者都可用于存储键值映射，彼此的性能也相差无几，但是Redis能够自动以两种不同的方式将数据写入硬盘，并且Redis除了能存储普通的字符串键之外，还可以存储其他4种数据结构，而memcached只能存储普通的字符串键。这些不同之处使得Redis可以用于解决更为广泛的问题，并且既可以用作主数据库（primary database）使用，又可以作为其他存储系统的辅助数据库（auxiliary database）使用。

## Redis安装配置(windows版本)
[下载地址](https://github.com/MSOpenTech/redis/releases) (Redis-x64-xxx.zip)
<br>
1. 打开cmd窗口
必须切换到存放Redis原文件的目录下（可以将目录设置到环境变量中），并运行如下命令来启动服务端
```
 redis-server.exe redis.windows.conf
```
<br>
2. 打开另一个cmd窗口
切换到对应目录下运行如下命令（若设置了环境变量则不需要切换目录），就可以访问服务端了。
```
redis-cli.exe -h 127.0.0.1 -p 6379
```

## Redis命令
<br>
1. 连接远程redis服务
```
//host:主机名 port:端口名 password:密码
$ redis-cli -h host -p port -a password
//默认连接本机，等同于下一条命令
$ redis-cli
$ redis-cli -h 127.0.0.1 -p 6379 -a "mypass"
redis 127.0.0.1:6379>
redis 127.0.0.1:6379> PING
PONG
```
PING命令用于检测redis服务是否启动。

## 数据类型
<br>
1. Redis字符串(String)（[命令手册](https://redis.io/commands)）
```
//语法
redis 127.0.0.1:6379> COMMAND KEY_NAME
//实例
redis 127.0.0.1:6379> SET runoobkey redis
OK
redis 127.0.0.1:6379> GET runoobkey
"redis"
```
<br>
2. Redis哈希(Hash)（[命令手册](https://redis.io/commands)）

说明：Redis hash 是一个string类型的field和value的映射表，hash特别适合用于存储对象。Redis 中每个 hash 可以存储 232 - 1 键值对（40多亿）。
```
//runoobkey为键名，后面的数据都是一对一对存储的，比如name和"redis tutorial"是一对键值对
127.0.0.1:6379>  HMSET runoobkey name "redis tutorial" description "redis basic commands for caching" likes 20 visitors 23000
OK

//显示所有hash中的所有数据
127.0.0.1:6379>  HGETALL runoobkey
1) "name"
2) "redis tutorial"
3) "description"
4) "redis basic commands for caching"
5) "likes"
6) "20"
7) "visitors"
8) "23000"
```
<br>
3. Redis列表(List)（[命令手册](https://redis.io/commands)）  
说明： Redis列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素导列表的头部（左边）或者尾部（右边）一个列表最多可以包含 232 - 1 个元素 (4294967295, 每个列表超过40亿个元素)。

```
//在runoobkey头插入值（LPUSH在头部插入，RPUSH在尾部插入）
redis 127.0.0.1:6379> LPUSH runoobkey redis
(integer) 1
redis 127.0.0.1:6379> LPUSH runoobkey mongodb
(integer) 2
redis 127.0.0.1:6379> LPUSH runoobkey mysql
(integer) 3
//返回第1个元素到第11个元素区间内的值
redis 127.0.0.1:6379> LRANGE runoobkey 0 10

1) "mysql"
2) "mongodb"
3) "redis"
```
<br>
4. Redis集合(Set)（[命令手册](https://redis.io/commands)）  
说明：Redis的Set是string类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据。Redis中集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。集合中最大的成员数为 232 - 1 (4294967295, 每个集合可存储40多亿个成员)。

```
//向集合添加一个成员
redis 127.0.0.1:6379> SADD runoobkey redis
(integer) 1
redis 127.0.0.1:6379> SADD runoobkey mongodb
(integer) 1
redis 127.0.0.1:6379> SADD runoobkey mysql
(integer) 1
redis 127.0.0.1:6379> SADD runoobkey mysql
(integer) 0
//返回集合中的所有成员
redis 127.0.0.1:6379> SMEMBERS runoobkey

1) "mysql"
2) "mongodb"
3) "redis"
```
<br>
5.  Redis有序集合(sorted set)（[命令手册](https://redis.io/commands)）  
说明：Redis 有序集合和集合一样也是string类型元素的集合,且不允许重复的成员。不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。有序集合的成员是唯一的,但分数(score)却可以重复。集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。 集合中最大的成员数为 232 - 1 (4294967295, 每个集合可存储40多亿个成员)。

```
redis 127.0.0.1:6379> ZADD runoobkey 1 redis
(integer) 1
redis 127.0.0.1:6379> ZADD runoobkey 2 mongodb
(integer) 1
redis 127.0.0.1:6379> ZADD runoobkey 3 mysql
(integer) 1
redis 127.0.0.1:6379> ZADD runoobkey 3 mysql
(integer) 0
redis 127.0.0.1:6379> ZADD runoobkey 4 mysql
(integer) 0
redis 127.0.0.1:6379> ZRANGE runoobkey 0 10 WITHSCORES

1) "redis"
2) "1"
3) "mongodb"
4) "2"
5) "mysql"
6) "4"
```
<br>
6. Redis HyperLogLog
1) Redis 在 2.8.9 版本添加了 HyperLogLog 结构。
2) Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。
在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。
3) 因为HyperLogLog只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。

说明：
基数：集合中不同元素的数量。比如 {'apple', 'banana', 'cherry', 'banana', 'apple'} 的基数就是3。  
估算值：算法给出的基数并不是精确的，可能会比实际稍微多一些或者稍微少一些，但会控制在合理的范围之内。

```
//向HyperLogLog中添加元素
redis 127.0.0.1:6379> PFADD runoobkey "redis"
1) (integer) 1
redis 127.0.0.1:6379> PFADD runoobkey "mongodb"
1) (integer) 1
redis 127.0.0.1:6379> PFADD runoobkey "mysql"
1) (integer) 1
//统计基数的数量
redis 127.0.0.1:6379> PFCOUNT runoobkey
(integer) 3
```

## Redis发布订阅
[发布订阅---菜鸟教程](http://www.runoob.com/redis/redis-pub-sub.html)

## Redis事务
[Redis事务---菜鸟教程](http://www.runoob.com/redis/redis-transactions.html)

## Java使用Redis
[Java使用Redis---菜鸟教程](http://www.runoob.com/redis/redis-java.html)