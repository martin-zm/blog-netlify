title: LinkedList源码解析
author: 禾田
tags:
  - java集合源码
categories:
  - 'java集合源码 '
date: 2018-07-12 19:57:00
---

### 对于LinkedList需要掌握的八点内容

- LinkedList的创建：即构造器
- 往LinkedList中添加对象：即add(E)方法
- 获取LinkedList中的单个对象：即get(int index)方法
- 修改LinkedList中的指定索引的节点的数据set(int index, E element)
- 删除LinkedList中的对象：即remove(E)，remove(int index)方法
- 遍历LinkedList中的对象：即iterator，在实际中更常用的是增强型的for循环去做遍历
- 判断对象是否存在于LinkedList中：contain(E)
- LinkedList中对象的排序：主要取决于所采取的排序算法

### 源码分析

#### LinkedList的创建

实现方式：
```
List<String> strList0 = new LinkedList<String>();
```

源代码：在读源代码之前，首先要知道什么是环形双向链表

```
// 链表大小0
transient int size = 0;
// 双向链表的头结点
transient Node<E> first;
// 双向链表的尾结点
transient Node<E> last;

/**
 * 构造一个空节点
 */
public LinkedList() {
}

/**
 * 链表节点，LinkedList的一个内部类：
 */
private static class Node<E> {
    // 链表存储的数据
    E item;
    // 链表的下一个节点
    Node<E> next;
    // 链表的上一个节点
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

#### 往LinkedList中添加对象（add(E e)）

实现方式：

```
strList0.add("hello");
```

源代码：

```
// 在链表的尾部添加新节点，添加成功则返回true
public boolean add(E e) {
    linkLast(e);
    return true;
}

void linkLast(E e) {
    // 临时保存尾节点
    final Node<E> l = last;
    // 把节点添加到双向链表中
    final Node<E> newNode = new Node<>(l, e, null);
    // 令链表尾指针指向新添加的节点
    last = newNode;
    // 如果是空链表
    if (l == null)
        // 则令头指针指向新添加的节点，相当于创建一个新链表头节点
        first = newNode;
    else
        l.next = newNode;
    size++;
    // 保证遍历时候检查链表是否有添加或删除节点
    modCount++;
}
```

添加过程如下图：

![image](http://owq01tqh9.bkt.clouddn.com/linkedList%E5%8F%8C%E5%90%91%E9%93%BE%E8%A1%A8.png)

这里，结合着代码注释与图片去看add(E)的源代码就好。

注意：在添加元素方面LinkedList不需要考虑数组扩容和数组复制，只需要新建一个对象。

#### 获取LinkedList中的单个对象（get(int index)）

实现方式：

```
// 注意：下标从0开始
strList.get(0);
```

源码：
```
/**
 * 返回索引值为index节点的数据，index从0开始计算
 */
public E get(int index) {
    checkElementIndex(index);
    return node(index).item;
}
/**
 * 判断index是否在合理范围内
 */
private void checkElementIndex(int index) {
    if (!isElementIndex(index))
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
private boolean isElementIndex(int index) {
    return index >= 0 && index < size;
}

/**
 * 获取指定index索引位置的节点（需要遍历链表）
 */
Node<E> node(int index) {
    // 如果index小于链表大小的半，就从链表头开始遍历，否则从尾部开始遍历
    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```
注意：链表节点的按索引查找，需要遍历链表；而数组不需要。

#### 修改LinkedList中指定索引的节点的数据：set(int index, E element)

使用方式：

```
strList.set(0, "world");
```

源码：

```
/**
 * 修改指定索引位置index上的节点的数据为element
 */
public E set(int index, E element) {
    // 判断index是否在合理范围内
    checkElementIndex(index);
    // 获取指定index索引位置的节点（需要遍历链表）
    Node<E> x = node(index);
    // 保存节点旧值
    E oldVal = x.item;
    // 将新值赋给该节点的element属性
    x.item = element;
    // 返回旧值
    return oldVal;
}
```

#### 删除LinkedList中的对象

##### remove(Object o)

使用方式：

```
strList.remove("world");
```

源码：

```
/**
 * 删除第一个出现的指定元数据为o的节点
 */
public boolean remove(Object o) {
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}

/*
 * 删除节点逻辑（节点不为空）
 */
E unlink(Node<E> x) {
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;

    // 处理待删除节点的前面节点
    if (prev == null) {
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }
    // 处理待删除节点的后面节点
    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }
    x.item = null;
    size--;
    modCount++;
    return element;
}
```

##### remove(int index)

使用方式：

```
strList.remove(0);
```

源代码： 

```
// 删除指定位置的节点
public E remove(int index) {
    checkElementIndex(index);
    return unlink(node(index));
}
```

注意：

unlink(node(index))见上边

#### 判断对象是否存在于LinkedList中（contains(E)）

```
/**
 * 链表中是否包含指定数据o的节点
 */
public boolean contains(Object o) {
    return indexOf(o) != -1;
}

/**
 * 从链表头开始，查找第一个出现o的索引
 */
public int indexOf(Object o) {
    int index = 0;
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null)
                return index;
            index++;
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item))
                return index;
            index++;
        }
    }
    return -1;
}
```

注意：indexOf(Object o)返回第一个出现的元素o的索引

#### 遍历LinkedList中的对象（iterator()）

使用方式：

```
List<String> strList = new LinkedList<String>();
strList.add("jigang");
strList.add("nana");
strList.add("nana2");

Iterator<String> it = strList.iterator();
while (it.hasNext()) {
    System.out.println(it.next());
}
```

源代码：iterator()方法是在父类AbstractSequentialList中实现的

```
public Iterator<E> iterator() {
    return listIterator();
}
```

listIterator()方法是在父类AbstractList中实现的

```
public ListIterator<E> listIterator() {
    return listIterator(0);
}
```

listIterator(int index)方法是在父类AbstractList中实现的

```
public ListIterator<E> listIterator(final int index) {
    if (index < 0 || index > size())
        throw new IndexOutOfBoundsException("Index: " + index);

    return new ListItr(index);
}
```
该方法返回AbstractList的一个内部类ListItr对象

```
private class ListItr extends Itr implements ListIterator<E> {
    ListItr(int index) {
        cursor = index;
    }
```
上边这个类并不完整，它继承了内部类Itr，还扩展了一些其他方法（eg.向前查找方法hasPrevious()等），至于hasNext()/next()等方法还是来自于Itr的。

```
private class Itr implements Iterator<E> {
    //标记位：标记遍历到哪一个元素
    int cursor = 0;
    //标记位：用于判断是否在遍历的过程中，是否发生了add、remove操作
    int expectedModCount = modCount;
    //检测对象数组是否还有元素
    public boolean hasNext() {
        //如果cursor==size，说明已经遍历完了，上一次遍历的是最后一个元素
        return cursor != size();
    }

    //获取元素
    public E next() {
        //检测在遍历的过程中，是否发生了add、remove操作
        checkForComodification();
        try {
            E next = get(cursor++);
            return next;
            //捕获get(cursor++)方法的IndexOutOfBoundsException
        } catch (IndexOutOfBoundsException e) {
            checkForComodification();
            throw new NoSuchElementException();
        }
    }

    //检测在遍历的过程中，是否发生了add、remove等操作
    final void checkForComodification() {
        //发生了add、remove操作,这个我们可以查看add等的源代码，发现会出现modCount++
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
```

注：上述的Itr我去掉了一个此时用不到的方法和属性。这里的get(int index)方法参照2.3所示。

### 总结

- LinkedList基于环形双向链表方式实现，无容量的限制
- 添加元素时不用扩容（直接创建新节点，调整插入节点的前后节点的指针属性的指向即可）
- 线程不安全
- get(int index)：需要遍历链表
- remove(Object o)需要遍历链表
- remove(int index)需要遍历链表
- contains(E)需要遍历链表