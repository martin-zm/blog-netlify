title: Memcached基本操作
author: 禾田
tags:
  - memcached
categories:
  - memcached
date: 2018-04-10 10:23:00
description: memcached基本操作，让你对memcached有个直观的理解
---
## Memcached简介
Memcached是一个自由开源的，高性能，分布式内存对象缓存系统。  

Memcached是一种基于内存的key-value存储，用来存储小块的任意数据（字符串、对象）。这些数据可以是数据库调用、API调用或者是页面渲染的结果。   

本质上，它是一个简洁的key-value存储系统。

一般的使用目的是，通过缓存数据库查询结果，减少数据库访问次数，以提高动态Web应用的速度、提高可扩展性。

## Memcached安装配置
[Memcached安装配置---菜鸟教程](http://www.runoob.com/memcached/window-install-memcached.html)
```
//注意：路径对应修改为自己的安装路径
//安装和卸载memcached服务
c:\memcached\memcached.exe -d install
c:\memcached\memcached.exe -d uninstall
//启动和关闭memcached服务
c:\memcached\memcached.exe -d start
c:\memcached\memcached.exe -d stop
//-m 512 意思是设置 memcached 最大的缓存配置为512M。
c:\memcached\memcached.exe -d runservice -m 512
//该命令可以用来查看命令帮助和参数配置
c:\memcached\memcached.exe -h
```

## Memcached 连接
```
//执行任何命令时需要先启动memcached服务(路径对应修改为自己的安装路径)
c:\memcached\memcached.exe -d start
//telnet HOST PORT --------- HOST：主机名 PORT：端口
telnet 127.0.0.1 11211
```

## Memcached 存储命令
<br>1. set命令
```
//语法
set key flags exptime bytes [noreply] 
value 
```
参数说明：
key：键值 key-value 结构中的 key，用于查找缓存值。  
flags：可以包括键值对的整型参数，客户机使用它存储关于键值对的额外信息。  
exptime：在缓存中保存键值对的时间长度（以秒为单位，0 表示永远）  
bytes：在缓存中存储的字节数  
noreply（可选）： 该参数告知服务器不需要返回数据  
value：存储的值（始终位于第二行）（可直接理解为key-value结构中的value）  

例子：
```
set runoob 0 900 9
memcached
STORED

get runoob
VALUE runoob 0 9
memcached

END
```
说明：
```
key → runoob
flag → 0
exptime → 900 (以秒为单位)
bytes → 9 (数据存储的字节数)
value → memcached
```
输出：
```
//数据设置成功
STORED
```
输出信息说明：
```
STORED：保存成功后输出。
ERROR：在保持失败后输出。
```
---
<br>2. add命令
```
//语法
add key flags exptime bytes [noreply]
value
```
参数说明：
key：键值 key-value 结构中的 key，用于查找缓存值。  
flags：可以包括键值对的整型参数，客户机使用它存储关于键值对的额外信息。  
exptime：在缓存中保存键值对的时间长度（以秒为单位，0 表示永远）  
bytes：在缓存中存储的字节数  
noreply（可选）： 该参数告知服务器不需要返回数据  
value：存储的值（始终位于第二行）（可直接理解为key-value结构中的value）  

例子：
```
add new_key 0 900 10
data_value
STORED
get new_key
VALUE new_key 0 10
data_value
END
```
说明：
```
key → new_key
flag → 0
exptime → 900 (以秒为单位)
bytes → 10 (数据存储的字节数)
value → data_value
```
输出：
```
//数据设置成功
STORED
```
输出信息说明：
```
STORED：保存成功后输出。
NOT_STORED ：在保持失败后输出。
```
---
<br>3. replace命令
```
//语法
replace key flags exptime bytes [noreply]
value
```
参数说明：
key：键值 key-value 结构中的 key，用于查找缓存值。  
flags：可以包括键值对的整型参数，客户机使用它存储关于键值对的额外信息。  
exptime：在缓存中保存键值对的时间长度（以秒为单位，0 表示永远）  
bytes：在缓存中存储的字节数  
noreply（可选）： 该参数告知服务器不需要返回数据  
value：存储的值（始终位于第二行）（可直接理解为key-value结构中的value）  

例子：
```
add mykey 0 900 10
data_value
STORED
get mykey
VALUE mykey 0 10
data_value
END
replace mykey 0 900 16
some_other_value
get mykey
VALUE mykey 0 16
some_other_value
END
```
说明：
```
key → mykey
flag → 0
exptime → 900 (以秒为单位)
bytes → 10 (数据存储的字节数)
value → data_value
```
输出：
```
//数据设置成功
STORED
```
输出信息说明：
```
STORED：保存成功后输出。
NOT_STORED ：在保持失败后输出。
```
---
<br>4. append命令(prepend在键值的前面追加数据)
```
//语法
append key flags exptime bytes [noreply]
value
```
参数说明：
key：键值 key-value 结构中的 key，用于查找缓存值。  
flags：可以包括键值对的整型参数，客户机使用它存储关于键值对的额外信息。  
exptime：在缓存中保存键值对的时间长度（以秒为单位，0 表示永远）  
bytes：在缓存中存储的字节数  
noreply（可选）： 该参数告知服务器不需要返回数据  
value：存储的值（始终位于第二行）（可直接理解为key-value结构中的value）  

例子：
```
set runoob 0 900 9
memcached
STORED
get runoob
VALUE runoob 0 9
memcached
END
append runoob 0 900 5
redis
STORED
get runoob
VALUE runoob 0 14
memcachedredis
END
```
说明：

首先我们在 Memcached 中存储一个键 runoob，其值为 memcached。
然后，我们使用 get 命令检索该值。
然后，我们使用 append 命令在键为 runoob 的值后面追加 "redis"。
最后，我们再使用 get 命令检索该值。

输出：
```
//数据设置成功
STORED
```
输出信息说明：
```
STORED：保存成功后输出。
NOT_STORED：该键在 Memcached 上不存在。
CLIENT_ERROR：执行错误。
```
---
<br>5. CAS命令
说明：Memcached CAS（Check-And-Set 或 Compare-And-Swap） 命令用于执行一个"检查并设置"的操作
它仅在当前客户端最后一次取值后，该key 对应的值没有被其他客户端修改的情况下， 才能够将值写入。
检查是通过cas_token参数进行的，这个参数是Memcach指定给已经存在的元素的一个唯一的64位值。

```
//语法
cas key flags exptime bytes unique_cas_token [noreply]
value
```
参数说明：
key：键值 key-value 结构中的 key，用于查找缓存值。  
flags：可以包括键值对的整型参数，客户机使用它存储关于键值对的额外信息。  
exptime：在缓存中保存键值对的时间长度（以秒为单位，0 表示永远）  
bytes：在缓存中存储的字节数 
unique_cas_token通过 gets 命令获取的一个唯一的64位值。
noreply（可选）： 该参数告知服务器不需要返回数据  
value：存储的值（始终位于第二行）（可直接理解为key-value结构中的value）  

例子：  
要在 Memcached 上使用 CAS 命令，你需要从 Memcached 服务商通过 gets 命令获取令牌（token）。
gets 命令的功能类似于基本的 get 命令。两个命令之间的差异在于，gets 返回的信息稍微多一些：64 位的整型值非常像名称/值对的 "版本" 标识符。
```
cas tp 0 900 9
ERROR             <− 缺少 token

cas tp 0 900 9 2
memcached
NOT_FOUND         <− 键 tp 不存在

set tp 0 900 9
memcached
STORED

gets tp
VALUE tp 0 9 1
memcached
END

cas tp 0 900 5 1
redis
STORED

get tp
VALUE tp 0 5
redis
END
```
说明：

如果没有设置唯一令牌，则 CAS 命令执行错误。
如果键 key 不存在，执行失败。
添加键值对。
通过 gets 命令获取唯一令牌。
使用 cas 命令更新数据。
使用 get 命令查看数据是否更新。

输出：
```
//数据设置成功
STORED
```
输出信息说明：
```
STORED：保存成功后输出。
ERROR：保存出错或语法错误。
EXISTS：在最后一次取值后另外一个用户也在更新该数据。
NOT_FOUND：Memcached 服务上不存在该键值。
```
---
## Memcached 查找命令
<br>1. get命令
说明：Memcached get 命令获取存储在 key(键) 中的 value(数据值) ，如果 key 不存在，则返回空。
```
//语法
get key
//多个 key 使用空格隔开
get key1 key2 key3
```
参数说明：
key：键值 key-value 结构中的 key，用于查找缓存值。 

例子：
```
set runoob 0 900 9
memcached
STORED
get runoob
VALUE runoob 0 9
memcached
END
```
说明：
使用 runoob 作为 key，过期时间设置为 900 秒。

---
<br>2. gets命令
Memcached gets 命令获取带有 CAS 令牌存 的 value(数据值) ，如果 key 不存在，则返回空。
```
//语法
get key
//多个 key 使用空格隔开
get key1 key2 key3
```
参数说明：
key：键值 key-value 结构中的 key，用于查找缓存值。 

例子：
```
set runoob 0 900 9
memcached
STORED
gets runoob
VALUE runoob 0 9 1
memcached
END
```
说明：
在使用gets命令的输出结果中，在最后一列的数字1代表了key为runoob的CAS令牌。

---
<br>3. delete命令
Memcached delete 命令用于删除已存在的 key(键)。
```
//语法
delete key [noreply]
```
参数说明：
key：键值 key-value 结构中的 key，用于查找缓存值。 
noreply（可选）： 该参数告知服务器不需要返回数据。

例子：
```
set runoob 0 900 9
memcached
STORED
get runoob
VALUE runoob 0 9
memcached
END
delete runoob
DELETED
get runoob
END
delete runoob
NOT_FOUND
```
说明：
使用runoob作为key，过期时间设置为900秒。之后我们使用delete命令删除该key。

输出信息说明：
```
DELETED：删除成功。
ERROR：语法错误或删除失败。
NOT_FOUND：key不存在。
```
---
<br>3. incr / decr 命令
Memcached incr 与 decr 命令用于对已存在的 key(键) 的数字值进行自增或自减操作。
incr与decr命令操作的数据必须是十进制的32位无符号整数。
如果key不存在返回NOT_FOUND，如果键的值不为数字，则返回 CLIENT_ERROR，其他错误返回 ERROR。
```
//语法
incr key increment_value
```
参数说明：
key：键值 key-value 结构中的 key，用于查找缓存值。 
increment_value： 增加的数值。

例子：
```
set visitors 0 900 2
10
STORED
get visitors
VALUE visitors 0 2
10
END
incr visitors 5
15
get visitors
VALUE visitors 0 2
15
END
```
说明：
使用 visitors 作为 key，初始值为 10，之后进行加 5 操作。

输出信息说明：
```
NOT_FOUND：key 不存在。
CLIENT_ERROR：自增值不是对象。
ERROR其他错误，如语法错误等。
```

decr命令与此类似

---
## Memcached 统计命令
<br>1. Memcached stats 命令
说明：Memcached stats 命令用于返回统计信息例如 PID(进程号)、版本号、连接数等。
```
//语法
stats
```

<br>2. Memcached stats items命令
说明：Memcached stats items 命令用于显示各个slab中 item的数目和存储时长(最后一次访问距离现在的秒数)。
```
//语法
stats items
```

<br>3. Memcached stats slabs命令
说明：Memcached stats slabs 命令用于显示各个slab的信息，包括chunk的大小、数目、使用情况等。
```
//语法
stats slabs
```

3. Memcached stats sizes命令
说明：Memcached stats sizes 命令用于显示所有item的大小和个数。该信息返回两列，第一列是item的大小，第二列是item的个数。
```
//语法
stats sizes
```

例子：
```
stats sizes
STAT 96 1
END
```

<br>4. Memcached flush_all 命令
说明：Memcached flush_all 命令用于用于清理缓存中的所有 key=>value(键=>值) 对。
该命令提供了一个可选参数 time，用于在制定的时间后执行清理缓存操作。
```
//语法
flush_all [time] [noreply]
```

例子：
```
set runoob 0 900 9
memcached
STORED
get runoob
VALUE runoob 0 9
memcached
END
flush_all
OK
get runoob
END
```

## Java连接Memcached服务
[Java连接Memcached服务---菜鸟教程](http://www.runoob.com/memcached/java-memcached.html)