title: HashMap源码解析
author: 禾田
tags:
  - java集合源码
categories:
  - java集合源码
date: 2018-07-15 16:28:00
---
### 对于HashMap需要掌握以下几点

- Map的创建：HashMap()
- 往Map中添加键值对：即put(Object key, Object value)方法
- 获取Map中的单个对象：即get(Object key)方法

下面结合源码看看hashmap的原理：

### 构建HashMap

源码中关键的一些属性：

```
// 默认的初始化容量（必须是2的次方） java8中都改成了位运算，提高运算效率
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;   
// 最大指定容量为2的30次方
static final int MAXIMUM_CAPACITY = 1 << 30; 
// 默认的加载因子（用于resize）
static final float DEFAULT_LOAD_FACTOR = 0.75f;    
// Node数组（数组容量必须是2的多少次方，若不足必要会扩容resize）--这就是HashMap的底层数据结构
transient Node<K,V>[] table;
// 该map中存放的key-value对个数，该个数决定了数组的扩容（而非table中的所占用的桶的个数来决定是否扩容）
transient int size;        
// 扩容resize的条件:eg.capacity=16,load_factor=0.75,threshold=capacity*load_factor=12,即当该map中存放的key-value对个数size>=12时，就resize）
int threshold;        
// 负载因子（用于resize）
final float loadFactor;    
// 标志位，用于标识并发问题，主要用于迭代的快速失败（在迭代过程中，如果发生了put（添加而不是更新的时候）、remove操作，该值发生变化，快速失败）
transient volatile int modCount;
```

注意：
- map中存放的key-value对个数size，该个数决定了数组的扩容（size>=threshold时，扩容），而非table中的所占用的桶的个数来决定是否扩容
- 标志位modCount采用volatile实现该变量的线程可见性（之后会在"Java并发"章节中去讲）
- 数组中的桶，指的就是table[i]
- threshold默认为0.75，这是综合时间和空间的利用率来考虑的，通常不要变，如果该值过大，可能会造成链表太长，导致get、put等操作缓慢；如果太小，空间利用率不足。
- 容量（必须是2的次方）是为了方便后面定位hash桶的位置的时候进行取模运算

无参构造器（也是当下最常用的构造器）

```
/**
 * 构造一个空的hashMap，设置默认初始化容量为16和默认负载因子为0.75
 */
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}
```

对于hashmap而言，还有两个比较常用的构造器，一个双参，一个单参。

```
public HashMap(int initialCapacity, float loadFactor) {
    // 一些边界条件判断
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}

// 获取cap的下一个2的n次幂的数（这个算法有兴趣可以研究下，涉及到位运算的一些技巧）
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}

/**
 * 指定初始容量
 */
public HashMap(int initialCapacity) {
    // 会调用上边的双参构造器
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
```
注意：
- 利用上述两个构造器构造出的数组容量不一定是指定的初始化容量，而是一个刚刚大于指定初始化容量的2的几次方的一个值。
- 在实际使用中，若我们能预判所要存储的元素的多少，最好使用上述的单参构造器来指定初始容量，这样的话，就可以避免就来扩容时带来的消耗（这一点与ArrayList一样）。
- table的初始化是在resize中进行的，这个和java7中的初始化逻辑不一样。后面会具体说。

从结构实现来讲，HashMap是数组+链表+红黑树（JDK1.8增加了红黑树部分）实现的，如下如所示：

![image](http://owq01tqh9.bkt.clouddn.com/hashmap%E7%BB%93%E6%9E%84.jpg)

HashMap的底层数据结构是一个Node[]，Node是HashMap的一个内部类，源代码如下：
```
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    // 该Node的下一个Node（hash冲突时，形成链表）
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}

// 在hashmap中可以存放null键和null值
public static boolean equals(Object a, Object b) {
    return (a == b) || (a != null && a.equals(b));
}
```
注意： 
- Node是一个节点，在其中还保存了下一个Node的引用（用来解决put时的hash冲突问题），这样的话，我们可以把hashmap看作是"一个链表数组"
- Node类中的equals()方法会在get(Object key)中使用

### put(Object key, Object value)

put流程图：

![image](http://owq01tqh9.bkt.clouddn.com/hashmap%E7%9A%84put%E6%B5%81%E7%A8%8B)

1. 判断键值对数组table[i]是否为空或为null，否则执行resize()进行扩容；

2. 根据键值key计算hash值得到插入的数组索引i，如果table[i]==null，直接新建节点添加，转向6，如果table[i]不为空，转向3；

3. 判断table[i]的首个元素是否和key一样，如果相同直接覆盖value，否则转向4，这里的相同指的是hashCode以及equals；

4. 判断table[i] 是否为treeNode，即table[i] 是否是红黑树，如果是红黑树，则直接在树中插入键值对，否则转向5；

5. 遍历table[i]，判断链表长度是否大于8，大于8的话把链表转换为红黑树，在红黑树中执行插入操作，否则进行链表的插入操作；遍历过程中若发现key已经存在直接覆盖value即可；

6. 插入成功后，判断实际存在的键值对数量size是否超多了最大容量threshold，如果超过，进行扩容。

源代码：

```
public V put(K key, V value) {
      // 对key的hashCode()做hash
      return putVal(hash(key), key, value, false, true);
  }
  
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // 步骤1：tab为空则创建
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 步骤2：计算index，并对null做处理 
    if ((p = tab[i = (n - 1) & hash]) == null) 
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        // 步骤3：节点key存在，直接覆盖value
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 步骤4：判断该链为红黑树
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 步骤5：该链为链表
        else {
            for (int binCount = 0; ; ++binCount) {
                // 如果没找到则将节点添加到链表最后面
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key,value,null);
                    //链表长度大于8转换为红黑树进行处理
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st  
                        treeifyBin(tab, hash);
                    break;
                }
                // key已经存在直接覆盖value
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))                                          
                    break;
                p = e;
            }
        }
         
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }

    ++modCount;
    // 步骤6：超过最大容量就扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

hash运算：
```
static final int hash(Object key) {
    int h;
    // h = key.hashCode() 为第一步 取hashCode值
    // h ^ (h >>> 16)  为第二步 高位参与运算
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```
在JDK1.8的实现中，优化了高位运算的算法，通过hashCode()的高16位异或低16位实现的：(h = k.hashCode()) ^ (h >>> 16)，主要是从速度、功效、质量来考虑的，这么做可以在数组table的length比较小的时候，也能保证考虑到高低Bit都参与到Hash的计算中，同时不会有太大的开销。

说明：在上述的步骤1中判断tab为空时候，调用了扩容函数resize()

下面举个例子说明下扩容过程。假设了我们的hash算法就是简单的用key表示。其中的哈希桶数组table的size=2， 所以key = 3、7、5，put顺序依次为3、7、5 。在mod 2以后都冲突在table[1]这里了。这里假设负载因子 loadFactor=1，即当键值对的实际大小size 大于 table的实际大小时进行扩容。接下来的三个步骤是哈希桶数组 resize成4，然后所有的Node重新rehash的过程。

![image](http://owq01tqh9.bkt.clouddn.com/hashmap%E6%89%A9%E5%AE%B91.jpg)

注意：java8在添加新节点到链表时候是使用的尾插，所以插入顺序和链表遍历顺序是一样的。而java7使用的是头插方式，插入顺序和链表遍历的顺序相反。


java8对扩容做了些优化。经过观测可以发现，我们使用的是2次幂的扩展(指长度扩为原来2倍)，所以，元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置。看下图可以明白这句话的意思，n为table的长度，图（a）表示扩容前的key1和key2两种key确定索引位置的示例，图（b）表示扩容后key1和key2两种key确定索引位置的示例，其中hash1是key1对应的哈希与高位运算结果。

![image](http://owq01tqh9.bkt.clouddn.com/hashmap%E6%89%A9%E5%AE%B93.jpg)

元素在重新计算hash之后，因为n变为2倍，那么n-1的mask范围在高位多1bit(红色)，因此新的index就会发生这样的变化：

![image](http://owq01tqh9.bkt.clouddn.com/hashmap%E6%89%A9%E5%AE%B94.jpg)

因此，我们在扩充HashMap的时候，不需要像JDK1.7的实现那样重新计算hash，只需要看看原来的hash值新增的那个bit是1还是0就好了，是0的话索引没变，是1的话索引变成“原索引+oldCap”，可以看看下图为16扩充为32的resize示意图：

![image](http://owq01tqh9.bkt.clouddn.com/hashmap%E6%89%A9%E5%AE%B95.jpg)

这个设计确实非常的巧妙，既省去了重新计算hash值的时间，而且同时，由于新增的1bit是0还是1可以认为是随机的，因此resize的过程，均匀的把之前的冲突的节点分散到新的bucket了。

```
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        // 如果超过最大值就不再扩充了，就只好随你碰撞去吧
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 没超过最大值，就扩充为原来的2倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 计算新的resize上限
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        // 把每个bucket都移动到新的buckets中
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { 
                    // 链表优化重hash的代码块，保证链表节点顺序
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // 原索引
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        // 原索引+oldCap
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 原索引放到bucket里
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

### get(Object key)

源代码：

```
public V get(Object key) {
    Node<K,V> e;
    // 在hashmap的结构中查找key值，如果没找到返回null，找到了就返回对应的value值
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    // 根据hash(key)定位桶的位置
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 判断第一个节点是否相同，相同直接返回
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // 遍历链表
        if ((e = first.next) != null) {
            // 如果已经调整成红黑树，则遍历红黑树
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

总结：

- HashMap底层就是一个Node数组，Node又包含next，事实上，可以看成是一个"链表数组",java8还在此基础上增加了红黑树结构
- 扩容：map中存放的key-value对个数size，该个数决定了数组的扩容（size>=threshold时，扩容），而非table中的所占用的桶的个数来决定是否扩容
- 扩容过程，不会重新计算hash值，只会重新按位与
- 在实际使用中，若我们能预判所要存储的元素的多少，最好使用上述的单参构造器来指定初始容量
- HashMap可以插入null的key和value
- HashMap线程不安全（多线程情况下会导致环形链表产生），若想要线程安全，最好使用ConcurrentHashMap

ConcurrentHashMap源码参考：    
[ConcurrentHashMap源码](https://www.cnblogs.com/chengxiao/p/6842045.html)
