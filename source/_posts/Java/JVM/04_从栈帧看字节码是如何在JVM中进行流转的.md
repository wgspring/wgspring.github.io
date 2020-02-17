---
title: 04_从栈帧看字节码是如何在JVM中进行流转的
date: 2020-02-15T19:35:17+08:00
coverImage: https://i.loli.net/2020/02/15/E6Z1aKqAfNDgFLi.png
categories: 
    - Java
    - JVM
tags: 
    - Java
    - JVM
---
<!-- toc -->
在前文多次提到字节码文件，那么：
- 怎么查看字节码文件？
- 字节码文件长什么样子？
- 对象初始化之后，具体的字节码又是怎么执行的？

本文将详细分析一个 Java 文件产生的字节码，并从栈帧层面看一下字节码的具体执行过程。
<!-- more -->


## 1. 准备代码源文件

首先，我们写一个最简单的 Java 程序 `B.java`。它有一个公共方法 test，还有一个静态成员变量C和动态成员变量a。`A.java` 有一个私有成员变量b和main函数，调用B的test方法。

``` Java
class B {
    private int a = 1234;

    static long C = 1111;

    public long test(long num) {
        long ret = this.a + num + C;
        return ret;
    }
}
```

``` Java
public class A {
    private B b = new B();

    public static void main(String[] args) {
        A a = new A();
        long num = 4321 ;

        long ret = a.b.test(num);

        System.out.println(ret);
    }
}
```

## 2. 查看字节码

### 2.1. 生成

生成字节码需要使用 javac 工具，常用的选项有：
- `javac -g:lines` 生成 LineNumberTable。
- `javac -g:vars ` 生成 LocalVariableTable。
- `javac -g` 生成所有的 debug 信息。

这里我们使用下面的命令生成本程序的字节码`A.class`和`B.class`：

`javac -g:lines -g:vars A.java`

### 2.2. 查看

javap 是 JDK 自带的反解析工具。它的作用是将 .class 字节码文件解析成可读的文件格式。在使用 javap 时我一般会添加 -v 参数，尽量多打印一些信息。打印本程序的字节码使用下面命令：

`javap -v A.class`  
`javap -v B.class`

## 3. 本例字节码概览

按照上面示例进行解析的字节码，以`javap -v B.class`为例大概类似下面这样：

```
Classfile /home/.../B.class
  ...
class B
  ...
Constant pool:
   #1 = Methodref          #7.#21         // java/lang/Object."<init>":()V
   #2 = Fieldref           #6.#22         // B.a:I
   ...
  #25 = Utf8               java/lang/Object
{
  static long C;
    descriptor: J
    flags: ACC_STATIC

  B();
    descriptor: ()V
    flags:
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         ...
        11: return
      LineNumberTable:
        line 1: 0
        line 2: 4
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      12     0  this   LB;


  public long test(long);
    descriptor: (J)J
    flags: ACC_PUBLIC
    Code:
      stack=4, locals=5, args_size=2
         0: aload_0
         1: getfield      #2                  // Field a:I
         ...
        13: lreturn
      LineNumberTable:
        line 7: 0
        line 8: 12
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      14     0  this   LB;
            0      14     1   num   J
           12       2     3   ret   J


  static {};
    descriptor: ()V
    flags: ACC_STATIC
    Code:
      stack=2, locals=0, args_size=0
         0: ldc2_w        #4                  // long 1111l
         3: putstatic     #3                  // Field C:J
         6: return
      LineNumberTable:
        line 4: 0
}
SourceFile: "B.java"
```

可以看到输出的字节码文件大概包含一下几个部分：
- Constant pool: 常量池，code块中的所有 `#x` 都可以在这里找到
- Code: 汇编代码部分
- LineNumberTable: javac的-g:lines选项生成的，描述了Code中的行号和源代码中的行号对应关系
- LocalVariableTable: javac的-g:vars选项生成的，描述了Code中的所有局部变量。

你会注意到 :I 这样特殊的字符。它们也是有意义的，如果你经常使用 jmap 这种命令，应该不会陌生。大体包括：
- B 基本类型 byte
- C 基本类型 char
- D 基本类型 double
- F 基本类型 float
- I 基本类型 int
- J 基本类型 long
- S 基本类型 short
- Z 基本类型 boolean
- V 特殊类型 void
- L 对象类型，以分号结尾，如 Ljava/lang/Object;
- [Ljava/lang/String; 数组类型，每一位使用一个前置的"["字符来描述

我们注意到 code 区域，有非常多的二进制指令。如果你接触过汇编语言，会发现它们之间其实有一定的相似性。但这些二进制指令，并不是操作系统能够认识的，它们是提供给 JVM 运行的源材料。

## 4. test 函数执行过程及其Code 区域介绍

test 函数同时使用了成员变量 a、静态变量 C，以及输入参数 num。我们此时说的函数执行，内存其实就是在虚拟机栈上分配的。下面这些内容，就是 test 方法的字节码。

```
public long test(long);
   descriptor: (J)J
   flags: ACC_PUBLIC
   Code:
     stack=4, locals=5, args_size=2
        0: aload_0
        1: getfield      #2                  // Field a:I
        4: i2l
        5: lload_1
        6: ladd
        7: getstatic     #3                  // Field C:J
       10: ladd
       11: lstore_3
       12: lload_3
       13: lreturn
     LineNumberTable:
       line 13: 0
       line 14: 12
     LocalVariableTable:
       Start  Length  Slot  Name   Signature
           0      14     0  this   LB;
           0      14     1   num   J
          12       2     3   ret   J
```

其中`stack=4, locals=5, args_size=2`：
- stack： 它此时的数值为 4，表明了 test 方法的最大操作数栈深度为 4。JVM 运行时，会根据这个数值，来分配栈帧中操作栈的深度。
- locals：存储了局部变量的存储空间。它的单位是 Slot（槽），可以被重用。其中存放的内容，包括：
    - this
    - 方法参数
    - 异常处理器的参数
    - 方法体中定义的局部变量
- args_size: 指的是方法的参数个数，因为每个方法都有一个隐藏参数 this，所以这里的数字是 2。

### 4.1. 字节码执行过程

在系列文章02介绍过main 线程会拥有两个主要的运行时区域：Java 虚拟机栈和程序计数器。其中，虚拟机栈中的每一项内容叫作栈帧，栈帧中包含四项内容：局部变量报表、操作数栈、动态链接和完成出口。

![](/img/Java/JVM/04_%E4%BB%8E%E6%A0%88%E5%B8%A7%E7%9C%8B%E5%AD%97%E8%8A%82%E7%A0%81%E6%98%AF%E5%A6%82%E4%BD%95%E5%9C%A8%20JVM%20%E4%B8%AD%E8%BF%9B%E8%A1%8C%E6%B5%81%E8%BD%AC%E7%9A%84/%E5%AD%97%E8%8A%82%E7%A0%81%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B.png)

我们的字节码指令，就是靠操作这些数据结构运行的。下面我们看一下具体的字节码指令。

**0: aload_0**

把第 0 个引用型局部变量推到操作数栈，这里的意思是把 this 装载到了操作数栈中。

对于 static 方法，aload_0 表示对方法的第0个参数的操作。

![](/img/Java/JVM/04_%E4%BB%8E%E6%A0%88%E5%B8%A7%E7%9C%8B%E5%AD%97%E8%8A%82%E7%A0%81%E6%98%AF%E5%A6%82%E4%BD%95%E5%9C%A8%20JVM%20%E4%B8%AD%E8%BF%9B%E8%A1%8C%E6%B5%81%E8%BD%AC%E7%9A%84/%E8%BF%87%E7%A8%8B1.png)

**1: getfield      #2**

将栈顶的指定的对象的第 2 个实例域（Field）的值，压入栈顶。#2 就是指的我们的成员变量 a。

所有#x都可以在Constant pool中找到：
```
#2 = Fieldref           #6.#27         // B.a:I
...
#6 = Class             #29           // B
#27 = NameAndType       #8:#9         // a:I
```

![](/img/Java/JVM/04_%E4%BB%8E%E6%A0%88%E5%B8%A7%E7%9C%8B%E5%AD%97%E8%8A%82%E7%A0%81%E6%98%AF%E5%A6%82%E4%BD%95%E5%9C%A8%20JVM%20%E4%B8%AD%E8%BF%9B%E8%A1%8C%E6%B5%81%E8%BD%AC%E7%9A%84/%E8%BF%87%E7%A8%8B2.png)

**4: i2l**

将栈顶 int 类型的数据转化为 long 类型，这里就涉及我们的隐式类型转换了。图中的信息没有变动，不再详解介绍。

**5: lload_1**

将第一个局部变量入栈。也就是我们的参数 num。这里的 l 表示 long，同样用于局部变量装载。你会看到这个位置的局部变量，一开始就已经有值了。

![](/img/Java/JVM/04_%E4%BB%8E%E6%A0%88%E5%B8%A7%E7%9C%8B%E5%AD%97%E8%8A%82%E7%A0%81%E6%98%AF%E5%A6%82%E4%BD%95%E5%9C%A8%20JVM%20%E4%B8%AD%E8%BF%9B%E8%A1%8C%E6%B5%81%E8%BD%AC%E7%9A%84/%E8%BF%87%E7%A8%8B4.png)

**6: ladd**

把栈顶两个 long 型数值出栈后相加，并将结果入栈。

![](/img/Java/JVM/04_%E4%BB%8E%E6%A0%88%E5%B8%A7%E7%9C%8B%E5%AD%97%E8%8A%82%E7%A0%81%E6%98%AF%E5%A6%82%E4%BD%95%E5%9C%A8%20JVM%20%E4%B8%AD%E8%BF%9B%E8%A1%8C%E6%B5%81%E8%BD%AC%E7%9A%84/%E8%BF%87%E7%A8%8B5.png)

**7: getstatic #3**

根据偏移获取静态属性的值，并把这个值 push 到操作数栈上。

![](/img/Java/JVM/04_%E4%BB%8E%E6%A0%88%E5%B8%A7%E7%9C%8B%E5%AD%97%E8%8A%82%E7%A0%81%E6%98%AF%E5%A6%82%E4%BD%95%E5%9C%A8%20JVM%20%E4%B8%AD%E8%BF%9B%E8%A1%8C%E6%B5%81%E8%BD%AC%E7%9A%84/%E8%BF%87%E7%A8%8B6.png)

**10: ladd**

再次执行 ladd。

![](/img/Java/JVM/04_%E4%BB%8E%E6%A0%88%E5%B8%A7%E7%9C%8B%E5%AD%97%E8%8A%82%E7%A0%81%E6%98%AF%E5%A6%82%E4%BD%95%E5%9C%A8%20JVM%20%E4%B8%AD%E8%BF%9B%E8%A1%8C%E6%B5%81%E8%BD%AC%E7%9A%84/%E8%BF%87%E7%A8%8B7.png)

**11: lstore_3**

把栈顶 long 型数值存入第 3 个局部变量。

在LocalVariableTable 中有以下代码：
```
       Start  Length  Slot  Name   Signature
           0      14     0  this   LB;
           0      14     1   num   J
          12       2     3   ret   J
```

slot 为 3，就是 ret 变量。

![](/img/Java/JVM/04_%E4%BB%8E%E6%A0%88%E5%B8%A7%E7%9C%8B%E5%AD%97%E8%8A%82%E7%A0%81%E6%98%AF%E5%A6%82%E4%BD%95%E5%9C%A8%20JVM%20%E4%B8%AD%E8%BF%9B%E8%A1%8C%E6%B5%81%E8%BD%AC%E7%9A%84/%E8%BF%87%E7%A8%8B8.png)

**12: lload_3**

正好与上面相反。上面是变量存入，我们现在要做的，就是把这个变量 ret，压入虚拟机栈中。

![](/img/Java/JVM/04_%E4%BB%8E%E6%A0%88%E5%B8%A7%E7%9C%8B%E5%AD%97%E8%8A%82%E7%A0%81%E6%98%AF%E5%A6%82%E4%BD%95%E5%9C%A8%20JVM%20%E4%B8%AD%E8%BF%9B%E8%A1%8C%E6%B5%81%E8%BD%AC%E7%9A%84/%E8%BF%87%E7%A8%8B9.png)

**13: lreturn**

从当前方法返回 long。

到此为止，我们的函数就完成了相加动作，执行成功了。JVM 为我们提供了非常丰富的字节码指令。详细的字节码指令列表，可以参考以下网址：

https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html

---

JVM系列：
上一篇：{% post_link Java/JVM/03_JVM的类加载机制 %}
下一篇：{% post_link Java/JVM/05_JVM垃圾回收-上 %}