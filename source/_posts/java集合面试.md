---
title: java集合面试
date: 2018-04-23 14:30:31
categories: java
tags: [java,面试]
description: 面试必问的java集合问题，包括：list set map
---
**在Java集合框架源码中有如下几点说明：**

1) 实现RandomAccess接口目的：说明支持快速访问，保证使用下标访问比用迭代器访问要快。否则，迭代器访问要快。

2) 实现Cloneable接口目的：保证对象能实现克隆。

3) AbstractList中的modCount变量（修改次数），通常会在迭代器中使用它，也就是说，在迭代器遍历的过程中，一旦发现这个对象的mcount和迭代器中存储的mcount不一样，那就抛异常。也就是Fail-Fast机制。对集合内容的修改都会改变modcount的值，那么在迭代器初始化过程中会将这个值赋给迭代器的expectedModCount。在迭代过程中，判断modCount跟expectedModCount是否相等，如果不相等就表示已经有其他线程修改了集合。  

4) Fail-Fast VS Fail-Safe

Fail-Safe（安全失败）：基于对底层集合做拷贝。因此，它不受源集合上修改的影响。java.util.concurrent包下面的所有的类都是安全失败的。  

Fail-Fast（快速失败）：java.util包下面的所有的集合类都是快速失败的，快速失败的迭代器会抛出ConcurrentModificationException异常，而安全失败的迭代器永远不会抛出这样的异常。

## List
有序可重复集合

### ArrayList 
**底层实现：** 基于动态数组

增：末尾添加元素，时间复杂度O(1); 任意位置添加元素，时间复杂度为O(N)。  
删：末尾删除元素，时间复杂度O(1); 任意位置删除元素，时间复杂度为O(N)。  
查：时间复杂度O(1)。 

非线程安全（性能高）

### Vector
功能基本同ArrayList。

线程安全（性能低）

### LinkedList  
**底层实现：** 双向链表

增：末尾添加元素，时间复杂度O(1); 任意位置添加元素，时间复杂度为O(N)。  
删：末尾删除元素，时间复杂度O(1); 任意位置删除元素，时间复杂度为O(N)。  
查：时间复杂度O(N)。 

## Map
### HashMap

参考：
[HashMap源码解析](https://zhuanlan.zhihu.com/p/21673805)  
[ConcurrentHashMap源码解析](https://www.cnblogs.com/chengxiao/p/6842045.html)

**底层实现：** 散列表+数组+链表+红黑树（JDK1.8增加了红黑树部分）  

**原理：** 它根据键的hashCode值存储数据，大多数情况下可以直接定位到它的值，因而具有很快的访问速度，但遍历顺序却是不确定的。HashMap最多只允许一条记录的键为null，允许多条记录的值为null。

**源码说明：**

HashMap在Map.Entry静态内部类实现中存储key-value对。HashMap使用哈希算法，在put和get方法中，它使用hashCode()和equals()方法。当我们通过传递key-value对调用put方法的时候，HashMap使用Key hashCode()和哈希算法来找出存储key-value对的索引。Entry存储在LinkedList中，所以如果存在entry，它使用equals()方法来检查传递的key是否已经存在，如果存在，它会覆盖value，如果不存在，它会创建一个新的entry然后保存。当我们通过传递key调用get方法时，它再次使用hashCode()来找到数组中的索引，然后使用equals()方法找出正确的Entry，然后返回它的值。下面的图片解释了详细内容。

其它关于HashMap比较重要的问题是容量、负荷系数和阀值调整。HashMap默认的初始容量是32，负荷系数是0.75。阀值是为负荷系数乘以容量，无论何时我们尝试添加一个entry，如果map的大小比阀值大的时候，HashMap会对map的内容进行重新哈希，且使用更大的容量。容量总是2的幂，所以如果你知道你需要存储大量的key-value对，比如缓存从数据库里面拉取的数据，使用正确的容量和负荷系数对HashMap进行初始化是个不错的做法。

**线程安全：** HashMap非线程安全，即任一时刻可以有多个线程同时写HashMap，可能会导致数据的不一致。如果需要满足线程安全，可以用Collections的synchronizedMap方法使HashMap具有线程安全的能力，或者使用ConcurrentHashMap。

**对比Hashtable：** Hashtable是遗留类，很多映射的常用功能与HashMap类似，不同的是它承自Dictionary类，并且是线程安全的，任一时间只有一个线程能写Hashtable，并发性不如ConcurrentHashMap，因为ConcurrentHashMap引入了分段锁。Hashtable不建议在新代码中使用，不需要线程安全的场合可以用HashMap替换，需要线程安全的场合可以用ConcurrentHashMap替换。（不允许键或值为空）

### weakHashMap
用法基本同HashMap。

**区别：**  
HashMap中的key保留了对实际对象的强引用，只要该HashMap对象不被销毁，该HashMap的所有key所引用的对象就不会被垃圾回收。

WeakHashMap中的key只保留了对于实际对象的弱引用，如果WeakHashMap对象的key所引用的对象没有被其他强引用变量所引用，则这些key所引用的对象可能被垃圾回收。


### LinkedHashMap

参考：[LinkedHashMap源码详解](http://www.cnblogs.com/whgk/p/6169622.html)  

**底层实现：** 散列表+数组+双向循环链表

**说明：** LinkedHashMap是HashMap的子类，实现的原理跟HashMap差不多，唯一的区别就是LinkedHashMap多了一个双向循环链表。因为有双向循环列表，所以LinkedHashMap能够记录插入元素的顺序，而HashMap不能，可以根据元素的插入顺序和访问顺序来遍历集合。

   
### TreeMap

参考：[TreeMap源码详解](http://cmsblogs.com/?p=1013)

**底层实现：** 散列表+红黑树

**说明：** TreeMap存储key-value节点对时，需要根据key对节点进行排序。它能保证所有的key-value对处于一种有序的状态。  

TreeMap有两种排序方式，自然排序和定制排序。  

自然排序：TreeMap所有key必须实现Comparable接口。  

定制排序：创建TreeMap时，传入一个Comparator对象，该对象负责对TreeMap中的所有key进行排序。使用定制排序的时候不需要Map的key来实现Comparable接口。

**适用场景：**

添加，查询元素：HashMap性能较好。(TreeMap需要红黑树算法来维护集合元素次序，性能低).    

保存排序的集合：TreeMap.    

遍历集合：LinkedHashMap.

## Set
相当于一个罐子，不允许包含重复元素

### HashSet
参考： [LinkedHashSet的实现原理](http://zhangshixi.iteye.com/blog/673319)  

**底层实现：** 基于HashMap实现。在它里面放置的元素对应到map里面的key部分，而在map中与key对应的value用一个固定Object()对象保存。

### LinkedHashSet
**说明：** 会根据元素的插入顺序来访问集合中的元素。

**底层实现：** 它继承于HashSet，又基于LinkedHashMap来实现的。

### TreeSet
**说明：** 确保集合处于排序状态。  

**底层实现：** TreeSet实际上是TreeMap实现的。

当构造TreeSet时；若使用不带参数的构造函数，则TreeSet的使用自然比较器；若用户需要使用自定义的比较器，则需要使用带比较器的参数。

**适用场景：**

添加，查询元素：HashSet性能较好。(TreeSet需要红黑树算法来维护集合元素次序，性能低).    

保存排序的集合：TreeSet.    

遍历集合：LinkedHashSet.  
