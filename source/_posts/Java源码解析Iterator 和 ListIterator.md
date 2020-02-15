---
title: Java源码解析Iterator 和 ListIterator
date: 2020-02-15T14:32:41+08:00
coverImage: https://i.loli.net/2020/02/15/v3Vckb7SlZI1mKU.png
categories: 
    - Java
    - 基础
tags: 
---
<!-- toc -->
在使用java集合的时候，都需要使用Iterator。但是java集合中还有一个迭代器ListIterator,在使用List、ArrayList、LinkedList和Vector的时候可以使用。这两种迭代器有什么区别呢？
<!-- more -->

## 1. 相同点
都是迭代器，当需要对集合中元素进行遍历不需要干涉其遍历过程时，这两种迭代器都可以使用。

## 2. 不同点
- ListIterator相较Iterator而言是双向遍历，多了一个previous()方法，只有List及其子类可以获取ListIterator。
- Iterator不可以修改值，而**ListIterator可以修改list中的值**
- 只有ListIterator具有add方法
- ListIterator可以定位当前索引的位置，nextIndex()和previousIndex()可以实现。Iterator没有此功能。

## 3. add/remove/next/previous代码逻辑
代码逻辑：
``` Java
cursor = 0;
lastRet = -1;

next(){
    tmp = get(cursor);
    cursor++;
    lastRet = cursor;
    return tmp;
}

previous(){
    cursor--;
    lastRet = cursor;
    tmp = get(cursor);
    return tmp;
}

remove(){
    remove(lastRet);
    cursor--;
    lastRet = -1;
}

add(){
    add(lastRet);
    cursor++;
    lastRet = -1;
}
```

从代码逻辑可以看到，next是先获取值再移动指针，previous是先移动指针再获取值。所以先执行next再执行previous返回的结果是一样的。

源码中有一个cursor记录当前所在位置
- next是获取当前位置元素并下标增加
- previous是获取前一个元素并下标减少
- add 是在上次操作的位置插入元素，并将lastRet置1
- remvoe 是删除上次操作的位置的元素，并将lastRet置1

由此可见，反复调用next() previous()会返回同一个元素，不可以连续add/remove
