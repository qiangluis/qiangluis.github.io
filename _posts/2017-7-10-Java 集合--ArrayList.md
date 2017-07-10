---
layout:     post
title:      "Java 集合"
subtitle:   "ArrayList"
date:       2017-07-10 18:25:00
author:     "LQ"
header-img: "img/in-post/sleepAndWait.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Java
---

> 对于集合，关注的点主要有四点：

1、是否允许空

2、是否允许重复数据

3、是否有序，有序的意思是读取数据的顺序和存放数据的顺序是否一致

4、是否线程安全 
# ArrayList
Arraylist 是我们使用的最为频繁的集合之一了，ArrayList 是一个数组队列，相当于动态数组。与Java中的数组相比，它的容量能动态增长。它继承于AbstractList，实现了List, RandomAccess, Cloneable, java.io.Serializable这些接口。
ArrayList包含了两个重要的对象：elementData 和 size。

名称 | 作用
---|---
private transient Object[] elementData; |它保存了添加到ArrayList中的元素
private int size | 动态数组的实际大小。size是按照调用add\remove方法的次数进行自增或自减，如果add了一个null,size也会增加1

## ArrayList的特点

关注点 | 结果
---|---
是否为空 | 可以
重读数据 | 可以
是否有序 | 有序
线程安全 | 非线程安全

## 添加元素

```
public boolean add(E e) {
     ensureCapacity(size + 1);  // Increments modCount!!
     elementData[size++] = e;
     return true;
}
```
ensureCapacity方法是扩容用的，底层实际上在调用add方法的时候只是给elementData的某个位置添加了一个数据而已。elementData中存储的应该是堆内存中元素的引用，而不是实际的元素

## 扩容

ArrayList 默认底层大小为10
```
public ArrayList() {
    this(10);
}
```
当ArrayList容量不足以容纳全部元素时，ArrayList会重新设置容量：新的容量=“(原始容量x3)/2 + 1”。

```
public void ensureCapacity(int minCapacity) {
modCount++;
int oldCapacity = elementData.length;
if (minCapacity > oldCapacity) {
    Object oldData[] = elementData;
    int newCapacity = (oldCapacity * 3)/2 + 1;
        if (newCapacity < minCapacity)
    newCapacity = minCapacity;
           // minCapacity is usually close to size, so this is a win:
           elementData = Arrays.copyOf(elementData, newCapacity);
}
}
```

接下来是调用到的是Arrays的copyOf方法，将元素组里面的内容复制到新的数组里面去

```
public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
       T[] copy = ((Object)newType == (Object)Object[].class)
           ? (T[]) new Object[newLength]
           : (T[]) Array.newInstance(newType.getComponentType(), newLength);
       System.arraycopy(original, 0, copy, 0,
                        Math.min(original.length, newLength));
       return copy;
}
```
## 删除元素

```
int numMoved = size - index - 1;
if (numMoved > 0)
//把指定元素后面位置的所有元素，利用System.arraycopy方法整体向前移动一个位置
    System.arraycopy(elementData, index+1, elementData, index,numMoved);
//最后一个位置的元素指定为null，这样让gc可以去回收它
elementData[--size] = null; // Let gc do its work
```
## 插入元素


```
public void add(int index, E element) {
    if (index > size || index < 0)throw new IndexOutOfBoundsException(
    "Index: "+index+", Size: "+size);
         ensureCapacity(size+1);  // Increments modCount!!
    System.arraycopy(elementData, index, elementData, index + 1,size - index);
    elementData[index] = element;
    size++;
}
```
按照指定位置，把从指定位置开始的所有元素利用System,arraycopy方法做一个整体的复制，向后移动一个位置（当然先要用ensureCapacity方法进行判断，加了一个元素之后数组会不会不够大），然后指定位置的元素设置为需要插入的元素，完成了一次插入的操作。
## ArrayList的优缺点
> 从上面的几个过程总结一下ArrayList的优缺点。ArrayList的优点如下：
> 
> 1、ArrayList底层以数组实现，是一种随机访问模式，再加上它实现了RandomAccess接口，因此查找也就是get的时候非常快
> 
> 2、ArrayList在顺序添加一个元素的时候非常方便，只是往数组里面添加了一个元素而已
> 
> 不过ArrayList的缺点也十分明显：
> 
> 1、删除元素的时候，涉及到一次元素复制，如果要复制的元素很多，那么就会比较耗费性能
> 
> 2、插入元素的时候，涉及到一次元素复制，如果要复制的元素很多，那么就会比较耗费性能
> 
> 因此，ArrayList比较适合顺序添加、随机访问的场景。
> 
## ArrayList和Vector的区别
ArrayList是线程非安全的，这很明显，因为ArrayList中所有的方法都不是同步的，在并发下一定会出现线程安全问题。那么我们想要使用ArrayList并且让它线程安全怎么办？一个方法是用Collections.synchronizedList方法把你的ArrayList变成一个线程安全的List，比如：

```
List<String> synchronizedList = Collections.synchronizedList(list);
synchronizedList.add("aaa");
synchronizedList.add("bbb");
for (int i = 0; i < synchronizedList.size(); i++)
{
    System.out.println(synchronizedList.get(i));
}
```
另一个方法就是Vector，它是ArrayList的线程安全版本，其实现90%和ArrayList都完全一样，区别在于：

1、Vector是线程安全的，ArrayList是线程非安全的

2、Vector可以指定增长因子，如果该增长因子指定了，那么扩容的时候会每次新的数组大小会在原数组的大小基础上加上增长因子；如果不指定增长因子，那么就给原数组大小*2，源代码是这样的：

```
int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
 capacityIncrement : oldCapacity);
```
## 为什么ArrayList的elementData是用transient修饰的？
看到ArrayList实现了Serializable接口，这意味着ArrayList是可以被序列化的，用transient修饰elementData意味着我不希望elementData数组被序列化。这是为什么？因为序列化ArrayList的时候，ArrayList里面的elementData未必是满的，比方说elementData有10的大小，但是我只用了其中的3个，那么是否有必要序列化整个elementData呢？显然没有这个必要，因此ArrayList中重写了writeObject方法：

```
private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        int expectedModCount = modCount;
        s.defaultWriteObject();
       s.writeInt(elementData.length);
        for (int i=0; i<size; i++)
           s.writeObject(elementData[i]);
            if (modCount != expectedModCount) {
           throw new ConcurrentModificationException();
        }
    }
```
每次序列化的时候调用这个方法，先调用defaultWriteObject()方法序列化ArrayList中的非transient元素，elementData不去序列化它，然后遍历elementData，只序列化那些有的元素，这样：

1、加快了序列化的速度

2、减小了序列化之后的文件大小

不失为一种聪明的做法，如果以后开发过程中有遇到这种情况，也是值得学习、借鉴的一种思路。











