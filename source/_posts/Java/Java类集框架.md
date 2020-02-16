---
title: Java类集框架
date: 2020-02-07T06:52:07.015Z
coverImage: https://i.loli.net/2020/02/13/LeDjEckTYG7AyWf.jpg
categories: 
    - Java
    - 基础
tags: 
    - Java
---
<!-- toc -->

早在 Java 2 中之前，Java 就提供一些类用来存储和操作对象组。比如：Dictionary, Vector, Stack, 和 Properties。

虽然这些类都非常有用，但是它们缺少一个核心的，统一的主题。由于这个原因，使用 Vector 类的方式和使用 Properties 类的方式有着很大不同。

<!-- more -->

集合框架被设计成要满足以下几个目标。

- 该框架必须是高性能的。基本集合（动态数组，链表，树，哈希表）的实现也必须是高效的。
- 该框架允许不同类型的集合，以类似的方式工作，具有高度的互操作性。
- 对一个集合的扩展和适应必须是简单的。

为此，整个集合框架就围绕一组标准接口而设计。你可以直接使用这些接口的标准实现，诸如： LinkedList, HashSet, 和 TreeSet 等,除此之外你也可以通过这些接口实现自己的集合。

## 1. 框架图概览

![](/img/Java/Java%E7%B1%BB%E9%9B%86%E6%A1%86%E6%9E%B6/%E6%A1%86%E6%9E%B6%E5%9B%BE%E6%A6%82%E8%A7%88.png)

上图初看可能有点懵，别急，下面一点点细分。

## 2. 从Collection接口开始

`Collection`**接口**是构造类集框架的基础。是类集框架中的最大父接口，它声明了所有类集都将拥有的核心方法。所有类集均实现`Collection`。

`Collection`接口之下有两大子接口，分别为`List`和`Set`。`Set` 强调存储的是无序的(有特殊情况，后面介绍)，不重复的数据。`List` 接口强调存储的是有序的，可以重复的元素。

`Collection`接口之下除了两大接口，还有一个抽象类`AbstractCollection`，其中实现了大多数Collection接口。但是

![](/img/Java/Java%E7%B1%BB%E9%9B%86%E6%A1%86%E6%9E%B6/Collection%E5%BC%80%E5%A7%8B.png)

## 3. List接口

`AbstractList`继承`AbstractCollection`并实现大多数`List`接口。类集框架中所有List的具体实现类均是从`AbstractList`这个抽象类继承而来。

![](/img/Java/Java%E7%B1%BB%E9%9B%86%E6%A1%86%E6%9E%B6/AbstractList.png)
`AbstractList`的具体实现子类有：
- `ArrayList` -- 数据结构中有数组实现的动态列表
- `Vector` -- 同上类似，但是是多线程安全的

另外 `AbstractList`还有一个抽象子类 `AbstractSequentialList` ，提供了对数据元素的链式访问而不是随机访问。

![](/img/Java/Java%E7%B1%BB%E9%9B%86%E6%A1%86%E6%9E%B6/AbstractList2.png)

在`AbstractSequentialList`抽象类下面有具体实现类`LinkedList`。`LinkedList`就是数据结构中用指针实现的链表。

另外在`Vector`下面还有一个数据结构子类`Stack`。

![](/img/Java/Java%E7%B1%BB%E9%9B%86%E6%A1%86%E6%9E%B6/List.png)


## 4. Queue接口

说起`Stack`，你可能会迷惑`Queue`去哪了。在类集框架中，`Stack`是一个具体的实现类，继承于`Vector`，而`Queue`却仅仅是一个接口，继承于总接口`Collection`，并没有具体实现。如果需要实例化一个普通的`Queue`可以使用`LinkedList`来实例化。例如：`Queue<Integer> q = new LinkedList<>();`

![](/img/Java/Java%E7%B1%BB%E9%9B%86%E6%A1%86%E6%9E%B6/Queue.png)

## 5. Set接口

类似`List`接口，`Set`接口下有`AbstractSet`这个抽象类，然后所有`Set`具体实现类都继承于`AbstractSet`。同样`AbstractSet`也继承于`AbstractCollection`这个抽象类。

另外前文提到`Set`强调无序，但有个例外，`Set`接口下还有一个`SortedSet`接口，该接口要求实现类必须实现有序的Set。

`AbstractSet`抽象类有以下几个具体实现类：
- `HashSet` -- hash表实现，故而无序
    - 其下还有一个子类`LinkedHashSet`，采用hash链表实现
- `TreeSet` -- 用红黑树实现，实现了`SortedSet`接口

![](/img/Java/Java%E7%B1%BB%E9%9B%86%E6%A1%86%E6%9E%B6/Set.png)


## 6. Map接口

`Map`接口并非直接继承于`Collection`接口，因为`Map`不是集合，但是它们完全整合在集合中。`Map` 里存储的是键/值对，一般我们要求其`key`必须唯一，这就和`Set`及其类似。因此`Map`也和`Set`一样具有类似的继承结构。

![](/img/Java/Java%E7%B1%BB%E9%9B%86%E6%A1%86%E6%9E%B6/Map.png)

## 7. 数组操作类--Arrays

本质上，类集本身是一个对象数组。java.util.Arrays类是可以操作数组的。Arrays类数组操作类，可用来操作数组（如数组元素排序，搜索和填充等）的各种方法。Arrays类常用如下方法表示：

|                      方法名称                      | 类型 |                                                  功能简述                                                  |
| ------------------------------------------------- | ---- | --------------------------------------------------------------------------------------------------------- |
| `static<T> List<T> asList(T...a)`                    | 静态 | 将多个元素变为List集合                                                                                      |
| `static int binarySearch(int[] a,int key)`         | 静态 | 使用二分搜索法，查询key元素值是否在a数组中，若不存在则返回负数，调用次方法前要求数组以升序排序。次方法可以被多次重载 |
| `static int[] copyOf(int[] original,int newLenggh)` | 静态 | 复制指定的数组。original表示原数组，newLength表示需要复制的长度，默认从第一个元素开始赋值。次方发可以被多次重载。   |
| `static boolean equals(int[] a,int[] a2)`           | 静态 | 比较两个数组是否相等，若相等则返回true，否则返回false。此方发可以被多次重载                                     |
| `static void fill(int[] a,int val)`                 | 静态 | 将指定的int val分配给指定的int型数组的每个元素。此方发可以被多次重载                                            |
| `static void sort(int[] a)`                         | 静态 | 对指定的数组按升序排序。此方发可以被多次重载                                                                  |
| `static String toString(int[] a)`                    | 静态 | 返回指定数组内容的字符串表示形式。此方发可以被多次重载                                                         |

## 8. 迭代器

通常情况下，你会希望遍历一个集合中的元素。例如，显示集合中的每个元素。

一般遍历数组都是采用for循环或者增强for，这两个方法也可以用在集合框架，但是还有一种方法是采用迭代器遍历集合框架，它是一个对象，实现了`Iterator` 接口或`ListIterator`接口。

迭代器，使你能够通过循环来得到或删除集合的元素。`ListIterator` 继承了`Iterator`，以允许双向遍历列表和修改元素。

![](/img/Java/Java%E7%B1%BB%E9%9B%86%E6%A1%86%E6%9E%B6/%E8%BF%AD%E4%BB%A3%E5%99%A8.png)

## 9. 比较器

需要为多个对象排序时必须设置排序规则，而排序规则就可以通过比较器进行设置，Java中比较器提供了两种：`Comparable`接口 和 `Comparator`类

`Comparable` 是一个要进行多个对象比较的类需要默认实现的一个接口，这个接口定义如下：
```Java
public interface Comparable<T>{
    public int comparableTo(T o); //T是对象类型，o是对象类型所定义得对象
};
```

从接口定义的格式来看，可以发现要想实现对象得排序功能，要实现Comparable接口，并覆写comparable(T o)方法。此方法返回得是一个int类型数据，该返回值只能有以下三种情况之一
- 相等：0；
- 大于：1；
- 小于：-1；
