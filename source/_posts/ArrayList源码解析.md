title: ArrayList源码解析
author: 禾田
tags:
  - java集合源码
categories:
  - java集合源码
date: 2018-07-11 15:00:00
---
## 对于ArrayList需要掌握的七点内容

- ArrayList的创建：即构造器  
- 往ArrayList中添加对象：即add(E)方法  
- 获取ArrayList中的单个对象：即get(int index)方法  
- 删除ArrayList中的对象：即remove(E)方法  
- 遍历ArrayList中的对象：即iterator，在实际中更常用的是增强型的for循环去做遍历  
- 判断对象是否存在于ArrayList中：contain(E)  
- ArrayList中对象的排序：主要取决于所采取的排序算法  

## 源码分析

### ArrayList的创建（常见的两种方式）

```java
List<String> strList = new ArrayList<String>();
List<String> strList2 = new ArrayList<String>(5);
```

ArrayList源代码：

基本属性：
```java
//对象数组：ArrayList的底层数据结构
private transient Object[] elementData;
//elementData中已存放的元素的个数，注意：不是elementData的容量
private int size;
```
注意：

- transient关键字的作用：在采用Java默认的序列化机制的时候，被该关键字修饰的属性不会被序列化。  
- ArrayList类实现了java.io.Serializable接口，即采用了Java默认的序列化机制  
- 上面的elementData属性采用了transient来修饰，表明其不使用Java默认的序列化机制来实例化，但是该属性是ArrayList的底层数据结构，在网络传输中一定需要将其序列化，之后使用的时候还需要反序列化，那不采用Java默认的序列化机制，那采用什么呢？直接翻到源码的最下边有两个方法，发现ArrayList自己实现了序列化和反序列化的方法

```java
/**
 * Save the state of the <tt>ArrayList</tt> instance to a stream (that
 * is, serialize it).
 *
 * @serialData The length of the array backing the <tt>ArrayList</tt>
 *             instance is emitted (int), followed by all of its elements
 *             (each an <tt>Object</tt>) in the proper order.
 */
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();

    // Write out size as capacity for behavioural compatibility with clone()
    s.writeInt(size);

    // Write out all elements in the proper order.
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }

    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}

/**
 * Reconstitute the <tt>ArrayList</tt> instance from a stream (that is,
 * deserialize it).
 */
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    elementData = EMPTY_ELEMENTDATA;

    // Read in size, and any hidden stuff
    s.defaultReadObject();

    // Read in capacity
    s.readInt(); // ignored

    if (size > 0) {
        // be like clone(), allocate array based upon size not capacity
        int capacity = calculateCapacity(elementData, size);
        SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
        ensureCapacityInternal(size);

        Object[] a = elementData;
        // Read in all elements in the proper order.
        for (int i=0; i<size; i++) {
            a[i] = s.readObject();
        }
    }
}
```

构造器：

```
/**
 * 创建一个容量为initialCapacity的空的（size==0）对象数组
 */
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: " + initialCapacity);
    }
}

/**
 * 默认初始化一个容量为10的对象数组(java8之前)
 */
public ArrayList() {
    //即上边的public ArrayList(int initialCapacity){}构造器
    this(10);
}

/**
 * 默认初始化一个空数组(java8)，在java8中，在第一次add方法中会调用ensureExplicitCapac * ity方法把容量初始化为10，效果同上面一样  
 */
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

在我们执行new ArrayList\<String\>()时，会调用上边的无参构造器，创造一个容量为10的对象数组。
在我们执行new ArrayList\<String\>(5)时，会调用上边的public ArrayList(int initialCapacity)，创造一个容量为5的对象数组。

在实际使用中，如果我们能对所需的ArrayList的大小进行判断，有两个好处：
- 节省内存空间（eg.我们只需要放置两个元素到数组，new ArrayList<String>(2)）
- 避免数组扩容（下边会讲）引起的效率下降（eg.我们只需要放置大约37个元素到数组，new ArrayList<String>(40)）


### 往ArrayList中添加对象（常见的两个方法add(E)和addAll(Collection<? extends E> c)）

#### add(E)

```
strList.add("hello");
```

ArrayList源代码: 
```
/**
 * 向elementData中添加元素
 */
public boolean add(E e) {
    //确保对象数组elementData有足够的容量，可以将新加入的元素e加进去
    ensureCapacityInternal(size + 1);
    //加入新元素e，size加1
    elementData[size++] = e;
    return true;
}

//确保对象数组elementData有足够的容量
private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

// 计算数组容量，保证初始化容量为10
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}

// 保证有足够容量
private void ensureExplicitCapacity(int minCapacity) {
    // modCount变量用于在遍历集合（iterator()）时，检测是否发生了add、remove操作。
    modCount++;
    // 如果容量不够则扩容
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

// 扩容逻辑
private void grow(int minCapacity) {
    // 原始容量大小
    int oldCapacity = elementData.length;
    // 容量扩大1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    // 如果超出最大值，则设置为最大值
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```
在上述代码的扩容结束后，调用了Arrays.copyOf(elementData, newCapacity)方法，这个方法中：对于我们这里而言，先创建了一个新的容量为newCapacity的对象数组，然后使用System.arraycopy()方法将旧的对象数组复制到新的对象数组中去了。

注意：modCount变量用于在遍历集合（iterator()）时，检测是否发生了add、remove操作。

#### addAll(Collection<? extends E> c)

使用方式：

```
List<String> strList = new ArrayList<String>();
strList.add("jigang");
strList.add("nana");
strList.add("nana2");

List<String> strList2 = new ArrayList<String>(2);
strList2.addAll(strList);
```

源代码：

```
/**
 * 将c全部加入elementData
 */
public boolean addAll(Collection<? extends E> c) {
    // 将c集合转化为对象数组a
    Object[] a = c.toArray();
    // 获取a对象数组的容量
    int numNew = a.length;
    // 确保对象数组elementData有足够的容量，可以将新加入的a对象数组加进去
    ensureCapacityInternal(size + numNew);
    //将对象数组a拷贝到elementData中去
    System.arraycopy(a, 0, elementData, size, numNew);
    // 重新设置elementData中已加入的元素的个数
    size += numNew;
    // 若加入的是空集合则返回false
    return numNew != 0;
}
```

注意:

- 从上述代码可以看出，若加入的c是空集合，则返回false
- ensureCapacity(size + numNew);这个方法在上边用过
- System.arraycopy()方法定义如下：
```
public static native void arraycopy(Object src,  int  srcPos, Object dest, int destPos,  int length);
```
将数组src从下标为srcPos开始拷贝，一直拷贝length个元素到dest数组中，在dest数组中从destPos开始加入先的srcPos数组元素。

除了以上两种常用的add方法外，还有如下两种：

#### add(int index, E element)

```
/**
 * 在特定位置（只能是已有元素的数组的特定位置）index插入元素E
 */
public void add(int index, E element) {
    // 检查index是否在已有的数组中
    rangeCheckForAdd(index);
    // 确保对象数组elementData有足够的容量，可以将新加入的元素e加进去
    ensureCapacityInternal(size + 1); 
    // 将index及其后边的所有的元素整块后移，空出index位置
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    // 插入元素
    elementData[index] = element;
    // 已有数组元素个数+1
    size++;
}

/**
 * 检查index是否在已有的数组中
 */
private void rangeCheckForAdd(int index) {
    if (index > size || index < 0)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```

注意：index<=size才行，并不是index<elementData.length

#### set(int index, E element)

```
/**
 * 更换特定位置index上的元素为element，返回该位置上的旧值
 */
public E set(int index, E element) {
    // 检查index是否在已有的数组中
    rangeCheck(index);
    // 旧值
    E oldValue = elementData(index);
    // 该位置替换为新值
    elementData[index] = element;
    // 返回旧值 
    return oldValue;
}
```

### 获取ArrayList中的单个对象（get(int index)）

实现方式：

```
ArrayList<String> strList2 = new ArrayList<String>(2);
strList2.add("hello");
strList2.add("nana");
strList2.add("nana2");
System.out.println(strList2.get(0));
```

源代码：

```
/**
 * 按照索引查询对象E
 */
public E get(int index) {
    RangeCheck(index);//检查索引范围
    return (E) elementData[index];//返回元素，并将Object转型为E
}

/**
 * 检查索引index是否超出size-1
 */
private void RangeCheck(int index) {
    if (index >= size)
        throw new IndexOutOfBoundsException("Index:"+index+",Size:"+size);
}
```

注意：这里对index进行了索引检查，是为了将异常内容写的详细一些并且将检查的内容缩小（index < 0 || index >= size，注意这里的size是已存储元素的个数）；  
事实上不检查也可以，因为对于数组而言，如果index不满足要求（index < 0 || index > = length，注意这里的length是数组的容量）,都会直接抛出数组越界异常，而假设数组的length为10，当前的size是2，你去计算array[9]，这时候得出是null，这也是上边get为什么减小检查范围的原因。

### 删除ArrayList中的对象

#### remove(Object o)

使用方式： 

```
strList2.remove("hello");
```

源代码：

```
/**
 * 从前向后移除第一个出现的元素o
 */
public boolean remove(Object o) {
    //移除对象数组elementData中的第一个null
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    //移除对象数组elementData中的第一个o
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}

/*
 * 删除单个位置的元素，是ArrayList的私有方法
 */
private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    //删除的不是最后一个元素
    if (numMoved > 0)
        //删除的元素到最后的元素整块前移
        System.arraycopy(elementData, index + 1, elementData, index,numMoved);
    //将最后一个元素设为null，在下次gc的时候就会回收掉了
    elementData[--size] = null; 
}
```

#### remove(int index)

使用方式：

```
strList2.remove(0);
```

源代码：

```
/**
 * 删除指定索引index下的元素，返回被删除的元素
 */
public E remove(int index) {
    rangeCheck(index);
    modCount++;
    E oldValue = elementData(index);

    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; 
    return oldValue;
}
```

注意：
remove(Object o)需要遍历数组，remove(int index)不需要，只需要判断索引符合范围即可，所以，通常：后者效率更高。

### 判断对象是否存在于ArrayList中（contains(E)）

源代码：
```
/**
 * 判断动态数组是否包含元素o
 */
public boolean contains(Object o) {
    return indexOf(o) >= 0;
}

/**
 * 返回第一个出现的元素o的索引位置
 */
public int indexOf(Object o) {
    if (o == null) {//返回第一个null的索引
        for (int i = 0; i < size; i++)
            if (elementData[i] == null)
                return i;
    } else {//返回第一个o的索引
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;//若不包含，返回-1
}

/**
 * 返回最后一个出现的元素o的索引位置
 */
public int lastIndexOf(Object o) {
    if (o == null) {
        for (int i = size - 1; i >= 0; i--)
            if (elementData[i] == null)
                return i;
    } else {
        for (int i = size - 1; i >= 0; i--)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```
注意：
indexOf(Object o)返回第一个出现的元素o的索引；lastIndexOf(Object o)返回最后一个o的索引

### 遍历ArrayList中的对象（iterator()）

使用方式：

```
List<String> strList = new ArrayList<String>();
strList.add("jigang");
strList.add("nana");
strList.add("nana2");

Iterator<String> it = strList.iterator();
while (it.hasNext()) {
    System.out.println(it.next());
}
```

源代码：iterator()方法是在AbstractList中实现的，该方法返回AbstractList的一个内部类Itr对象

```
public Iterator<E> iterator() {
    return new Itr();//返回一个内部类对象
}

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
遍历的整个流程结合"使用方式"与"Itr的注释"来看。注：上述的Itr我去掉了一个此时用不到的方法和属性。

## 总结

- ArrayList基于数组方式实现，无容量的限制（会扩容）
- 添加元素时可能要扩容（所以最好预判一下），删除元素时不会减少容量（若希望减少容量，trimToSize()），删除元素时，将删除掉的位置元素置为null，下次gc就会回收这些元素所占的内存空间。  
- 线程不安全
- add(int index, E element)：添加元素到数组中指定位置的时候，需要将该位置及其后边所有的元素都整块向后复制一位
- get(int index)：获取指定位置上的元素时，可以通过索引直接获取（O(1)）
- remove(Object o)需要遍历数组
- remove(int index)不需要遍历数组，只需判断index是否符合条件即可，效率比remove(Object o)高
- contains(E)需要遍历数组
做以上总结，主要是为了与后边的LinkedList作比较。