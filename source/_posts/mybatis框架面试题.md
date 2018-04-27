title: mybatis框架面试题
author: 禾田
tags:
  - mybatis
categories:
  - mybatis
date: 2018-04-25 10:04:00
---
参考博客：[Mybatis常见面试题](https://my.oschina.net/zudajun/blog/747682)

> 对于简单语句来说，使用注解代码会更加清晰，然而Java注解对于复杂语句来说就会混乱，应该限制使用。因此，如果你不得不做复杂的事情，那么最好使用XML来映射语句。

持久是相对于瞬时来说的，其实就是可以把数据固化在硬盘或者磁带一类可以保存很长时间的设备上，不像放在内存中一样断电就消失了。企业应用中数据很重要(各种订单数据、客户数据、库存数据之类的)，比应用程序本身更重要，所以需要把数据持久化。持久化可以通过很多方式，写文件和数据库都可以。只是现在企业一般都会选择把数据持久化到数据库中，因为可以很方便的查询统计分析，但数据库的数据最终还是会写到磁盘上的。

### #、$区别     

参考博客：[#、$区别](http://www.importnew.com/25791.html)

答：\${}是Properties文件中的变量占位符，它可以用于标签属性值和sql内部，属于静态文本替换，比如${driver}会被静态替换为com.mysql.jdbc.Driver。#{}是sql的参数占位符，Mybatis会将sql中的#{}替换为?号，在sql执行前会使用PreparedStatement的参数设置方法，按序给sql的?号占位符设置参数值，比如ps.setInt(0, parameterValue)，#{item.name}的取值方式为使用反射从参数对象中获取item对象的name属性值，相当于param.getItem().getName()。   

### Xml映射文件中，除了常见的select|insert|updae|delete标签之外，还有哪些标签？

答：还有很多其他的标签：
```
<resultMap>
<parameterMap>
<sql>
<include>
<selectKey>
```
加上动态sql的9个标签：
```
trim|where|set|foreach|if|choose|when|otherwise|bind
```
其中<sql>为sql片段标签，通过<include>标签引入sql片段，<selectKey>为不支持自增的主键生成策略标签。

### 一级、二级缓存  

1）一级缓存: 基于 PerpetualCache 的 HashMap 本地缓存，其存储作用域为 Session，当 Session flush 或 close 之后，该 Session 中的所有 Cache 就将清空。

2）二级缓存与一级缓存其机制相同，默认也是采用 PerpetualCache，HashMap 存储，不同在于其存储作用域为 Mapper(Namespace)，并且可自定义存储源，如 Ehcache。要开启二级缓存，你需要在你的 SQL 映射文件中添加一行：
```xml
<cache/>
```

3）对于缓存数据更新机制，当某一个作用域(一级缓存 Session/二级缓存Namespaces)的进行了C/U/D 操作后，默认该作用域下所有 select 中的缓存将被 clear。