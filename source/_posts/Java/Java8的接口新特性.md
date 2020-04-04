---
title: Java8的接口新特性
date: 2020-04-04T22:20:58+08:00
coverImage: https://s1.ax1x.com/2020/04/04/G06qoD.jpg
categories: 
    - Java
    - 基础
tags: 
    - Java
    - 基础
---
<!-- toc -->
自从Java8 推出之后，Java的接口变得更加灵活，增添了一些新特性，接口里可以有静态方法和方法体了。

<!-- more -->
## 1. 回顾

首先回顾一下Java8 之前的接口：

- 接口中每一个方法是隐式抽象的,接口中的方法会被隐式的指定为 public abstract（默认是 public abstract，指定其他修饰符都会报错）。
- 接口中可以含有变量，但是接口中的变量会被隐式的指定为 public static final 变量（并且只能是 public，用 private 修饰会报编译错误）。
- 接口中的方法是不能在接口中实现的，只能由实现接口的类来实现接口中的方法。

## 2. 静态方法

Java8 之后，在接口中定义的静态方法，不是抽象的，是具体实现的，可以直接使用接口名称调用。

``` Java
public interface MyInterface {
    public static void method() {
        /**
         * 1、定义一个静态的带有方法体的方法
         * 2、接口不能创建对象，调用静态方法不需要对象
         * 3、接口名调用
         */
        System.out.println("接口中静态方法");
    }
}

调用：
MyInterface.method();
```

## 3. 默认方法

Java8 之后，在接口中不仅仅是可以定义静态方法，还可以进行普通方法的定义，不过不是抽象，java8中，可以使用关键字default。

``` Java
public interface MyInterface {
    public default void  methodDefault(){
        /**
         * 不同于静态方法，默认方法是一个非静态方法
         * 对于非静态方法，只能通过对象进行调用
         * 但是接口是不能创建对象的名故而我们需要子类来实现接口
         */
        System.out.println("接口中的默认方法");
    }
}

调用：
Impl impl = new Impl();
impl.methodDefault();
```

注意：如果多个接口定义了同样的默认方法，实现类实现多个接口时，必须重写掉默认方法，否则编译失败。

---

系列：
上一篇：{% post_link  %}
下一篇：{% post_link  %}
