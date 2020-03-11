---
title: HashMap、HashTable、ConcurrentHashMap浅析
date: 2020-03-11T10:32:17+08:00
coverImage: https://s2.ax1x.com/2020/03/11/8kQkJf.jpg
categories: 
    - Java
    - 基础
tags: 
    - Java
---
<!-- toc -->
我们知道Map是用来存放键值对的，在我们开发时，使用的最多的是HashMap —— 一种将键值对存于hash表中实现快速存取的数据结构。HashTable和ConcurrentHashMap也是类似的作用，那么他们之间又有什么区别和联系呢？

<!-- more -->
## 1. 线程同步差异

**HashMap**是未经同步的，所以在多线程中使用，需要手动同步，否则可能会出现数据冲突问题。

**HashTable**就是为了解决多线程时的需求，HashTable就是在对应的方法上添加了synchronized关键字进行修饰，使得所有操作只能一个线程执行，不可并发执行。但是HashTable这种操作效率太低下，在高并发的环境下都要线程竞争同一把锁，于是有了后来的ConcurrentHashMap。

**ConcurrentHashMap**也是线程安全的。ConcurrentHashMap里面有很多锁，ConcurrentHashMap将数据分段（segment）,每一段数据有单独的一个锁。当某一个线程占用某一个数据段的锁的时候，其他线程还是可以并发访问其他数据段的。

ConcurrentHashMap的put操作会先定位到Segment，然后获取锁，然后在Segment里面执行插入操作。两个步骤：1. 判断是否需要扩容。2. 插入适当位置。

ConcurrentHashMap的get操作是不需要加锁的。除非读到值为空的情况才会加锁重读。get方法将要使用的共享变量都设置为volatile，使得他们能够在多线程之间保持可见性，能够同时读而不出现读到过期的值。

## 2. 遍历方式不同

遍历方式不同主要指 HashMap和HashTable不同，HashMap和ConcurrentHashMap都是类集框架里面的内容，接口是一致的。

HashTable的遍历采用Enumeration(枚举)。HashMap和ConcurrentHashMap则是和类集框架保持统一，采用Iterator(迭代器)。

## 3. 对空处理不同

HashTable不允许`null`值，不论key还是Value都不行。HashMap和ConcurrentHashMap却是允许的，key为null时，hashCode就是0。

## 4. 默认表长不同

HashTable默认表长大小为11，增加方式是`2 * old + 1`。HashMap和ConcurrentHashMap默认表长16，增长方式`2 * old`，这也就保证了表长始终是2的n次幂。这样做的好处是对表长取余时，直接与`（表长-1）`相与就可以了。

## 5. 哈希值的使用方式不同

HashTable直接将对象的hashCode对表长取模就直接使用了。而HashMap和ConcurrentHashMap会将hashCode进行一次再hash过程，然后与`表长-1`相与得到下标。

再hash过程：`(key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);` 将高16位与整个hashCode进行相异或

这么做的目的是为了降低hash冲突的概率。注意在JDK不同版本中，右移的次数和异或的次数可能不同，但思想一致。

## 6. 总结

|         | HhashTable  | HashMap  | ConcurrentHashMap |
| ------- | ----------- | -------- | ----------------- |
| 线程同步 | 是          | 否       | 是                |
| 遍历方式 | Enumeration | Iterator | Iterator          |
| 对空支持 | 否          | 是       | 是                |
| 默认长度 | 11          | 16       | 16                |
| 再hash   | 否          | 是       | 是                |



---

系列：
上一篇：{% post_link  %}
下一篇：{% post_link  %}
