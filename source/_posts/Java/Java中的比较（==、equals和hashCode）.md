---
title: Java中的比较 —— ==、equals和hashCode
date: 2020-03-08T18:39:15+08:00
coverImage: https://s2.ax1x.com/2020/03/08/3z9T9H.jpg
categories: 
    - Java
    - 基础
tags: 
    - Java
---
<!-- toc -->
在编写Java代码的时候，我们常常涉及到两个对象的比较。这其中最长用到的是`==`和`equals`。在Hash相关的对象中，检测两个key是否相同，则是使用hashCode。这其中究竟有什么区别和联系呢？

<!-- more -->

## 1. ==

java中的数据类型，可分为两类：
- 基本数据类型，他们之间的比较，用`==`比较的是他们的值。
- 引用类型(对象)，当他们用`==`进行比较的时候，比较的是他们的引用。引用本身存在于栈中，引用的值就是对象在堆中的地址。所以本质上相当于比较的是对象的地址。

我们知道每个对象肯定是有唯一的地址的，所以如果要两个对象的内容是否相同，那么就要重写equals方法了。

## 2. equals

1. 默认情况（没有覆盖equals方法）下：equals方法都是调用Object类的equals方法

    下面是Object类中equals方法：

    ``` Java
    public boolean equals(Object obj) {
        return (this == obj);
    }
    ```

    可以看到，默认情况下`equals`等价于`==`

2. 若类中覆盖了equals方法：那么就要根据具体的代码来确定equals方法的作用了，覆盖后一般都是通过对象的内容是否相等来判断对象是否相等。比如String中重写了equals方法比较的就是对象的内容，我们自己写的类如果不重写调用的就是object类中的equals方法，也就是相当于==，比较的是地址。

## 3. hashCode

我们知道hash表本质上是一个数组，Java是如何根据key来找到数组下标的呢？

**路径：** `key-->hashCode-->下标`

hashCode是通过一定算法将一个对象转换成一个整形数字。通常来说对象不同，hashCode也不同（但不一定，hash算法是有可能冲突的）。如果hashCode 不同，则两个对象一定不是同一个对象（这里是指地址，不是equals）。

`hashCode-->下标`的过程，HashMap会有一次再hash的过程（高16位与低16位相与），然后再对表长取余。HashTable则直接将hashCode对表长取余。

HashMap中的key和HashSet中的元素都要唯一。所以这里面就涉及到key或者元素的比较。HashMap和HashSet就是比较hashCode的。

我们通常认为两个元素相同，指的是`equals`方法，所以当我们重写了`equals`方法就一定要重写`hashCode`方法。否则HashMap可能出现两个key相同的情况，HashSet出现元素重复的情况。

## 4. 总结

我们可以发现：
- `==`比较地址，`equals`通常比较地址，但有可能被重写。所以 `==` => `equals`
- `hashCode`是根据对象内容计算得到的。所以 `==` => `hashCode`
- `hashCode`和`equals`都可能被重写，没有可比性


![](/img/Java/Java%E4%B8%AD%E7%9A%84%E6%AF%94%E8%BE%83%EF%BC%88==%E3%80%81equals%E5%92%8ChashCode%EF%BC%89/%E6%80%BB%E7%BB%93.png)

---

系列：
上一篇：{% post_link  %}
下一篇：{% post_link  %}
