---
title: Java单例模式之双重检查
date: 2020-04-15T22:56:30+08:00
coverImage: https://s1.ax1x.com/2020/04/15/JirOje.jpg
categories: 
    - Java
tags: 
    - Java
    - 设计模式
---
<!-- toc -->
单例模式的意图是保证一个类仅有一个实例，并提供一个访问它的全局访问点。Java中实现线程安全的单例模式有多种方法，例如getInstance方法加锁、双重检查、静态内部类和enum枚举实现。我们最常使用的是双重检查方式。

<!-- more -->

## 1. 代码

``` Java
public class Singleton {
    private static volatile Singleton singleton;
    private Singleton() {
        // ... 构造
    }

    public static Singleton getInstance() {
        if (singleton == null) {
            synchronized (Singleton.class) {
                if (singleton == null) {
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```

## 2. 双重检查用意

我们大部分调用`Singleton.getInstance()`时，singleton已经不为null了，所以第一个检查尽可能地避免了竞争锁的耗时。第二个检查是避免两个线程同时竞争锁时，其中一个new完Singleton之后，另一个再次重复`new Singleton()`。

## 3. volatile的作用

看上去已经很完美了，但是我们发现`Singleton singleton`这个变量是有`volatile`关键字修饰的，如果没有`volatile`修饰会发生什么呢？

我们知道`volatile`有两个作用：
1. 内存可见性
2. 禁止指令重排

这里主要用到其第二个作用——禁止指令重排。正常的对象创建过程是：
1. 堆内存分配空间
2. 初始化对象
3. 把对象指向堆内存空间

>注意，这里区分{% post_link Java/JVM/03_JVM的类加载机制 类加载过程 %}：
>1. 加载类
>2. 验证
>3. 准备阶段的初始化（默认值初始化）
>4. 解析（符号引用转直接引用）
>5. 初始化（静态变量赋值）

正常的对象创建过程是：1->2->3，但是JVM有时会进行指令优化重排，对象的创建过程就可能变成：1->3->2。

当A线程调用`Singleton.getInstance()`时，进入`new Singleton();`，如果构造函数很耗时，就可能发生 `singleton != null` 但是singleton没有完全初始化成功。此时若B线程也调用`Singleton.getInstance()`，就会通过第一个if判断，拿到一个初始化不完全的singleton。


---

系列：
上一篇：{% post_link  %}
下一篇：{% post_link  %}
