---
title: 03_JVM的类加载机制
date: 2020-02-14T19:13:38+08:00
coverImage: https://i.loli.net/2020/02/14/PXvcYlL4ORD5Kdp.png
categories: 
    - Java
    - JVM
tags: 
    - Java
    - JVM
---
<!-- toc -->
JVM 的类加载机制和 Java 的类加载机制类似，但 JVM 的类加载过程稍有些复杂。前面课时我们讲到，JVM 通过加载 .class 文件，能够将其中的字节码解析成操作系统机器码。那这些文件是怎么加载进来的呢？又有哪些约定？什么是双亲委派机制？如何打破这个机制？

<!-- more -->

## 1. 类加载过程

现实中并不是说，我把一个文件修改成 .class 后缀，就能够被 JVM 识别。类的加载过程非常复杂，主要有这几个过程：加载、链接（验证、准备、解析）、初始化。

![](/img/Java/JVM/03_JVM%E7%9A%84%E7%B1%BB%E5%8A%A0%E8%BD%BD%E6%9C%BA%E5%88%B6/%E7%B1%BB%E7%9A%84%E5%8A%A0%E8%BD%BD%E8%BF%87%E7%A8%8B.png)

如图所示。**大多数情况下**，类会按照图中给出的顺序进行加载。下面我们就来分别介绍下这个过程。

### 1.1. 加载

加载的主要作用是将外部的 .class 文件，加载到 Java 的方法区内，你可以回顾一下我们在上一课时讲的内存区域图。加载阶段主要是找到并加载类的二进制数据，比如从 jar 包里或者 war 包里找到它们。

### 1.2. 验证

肯定不能任何 .class 文件都能加载，那样太不安全了，容易受到恶意代码的攻击。验证阶段在虚拟机整个类加载过程中占了很大一部分，不符合规范的将抛出 java.lang.VerifyError 错误。像一些低版本的 JVM，是无法加载一些高版本的类库的，就是在这个阶段完成的。

### 1.3. 准备

从这部分开始，将**为一些类变量分配内存，并将其初始化为默认值**。此时，实例对象还没有分配内存，所以这些动作是在方法区上进行的。

下面两段代码，code-snippet 1 将会输出 0，而 code-snippet 2 将无法通过编译。

``` Java
code-snippet 1：
     public class A {
         static int a ;
         public static void main(String[] args) {
             System.out.println(a);
         }
     }
 code-snippet 2：
 public class A {
     public static void main(String[] args) {
         int a ;
         System.out.println(a);
     }
 }
```

为什么会有这种区别呢？

这是因为局部变量不像类变量那样存在准备阶段。类变量有两次赋初始值的过程，一次在准备阶段，赋予初始值（也可以是指定值）；另外一次在初始化阶段，赋予程序员定义的值。

因此，即使程序员没有为类变量赋值也没有关系，它仍然有一个默认的初始值。但局部变量就不一样了，如果没有给它赋初始值，是不能使用的。

### 1.4. 解析

解析在类加载中是非常非常重要的一环，是**将符号引用替换为直接引用**的过程。这句话非常的拗口，其实理解起来也非常的简单。

符号引用是一种定义，可以是任何字面上的含义，而直接引用就是直接指向目标的指针、相对偏移量。

直接引用的对象都存在于内存中，你可以把通讯录里的某号码，类比为符号引用，把某人所处的真实地址，类比为直接引用。

解析阶段负责把整个类激活，串成一个可以找到彼此的网，过程不可谓不重要。那这个阶段都做了哪些工作呢？大体可以分为：
- 类或接口的解析
- 类方法解析
- 接口方法解析
- 字段解析

我们来看几个经常发生的异常，就与这个阶段有关。
- java.lang.NoSuchFieldError 根据继承关系从下往上，找不到相关字段时的报错。
- java.lang.IllegalAccessError 字段或者方法，访问权限不具备时的错误。
- java.lang.NoSuchMethodError 找不到相关方法时的错误。

解析过程保证了相互引用的完整性，把继承与组合推进到运行时。

### 1.5. 初始化

如果前面的流程一切顺利的话，接下来该**初始化成员变量**了，到了这一步，才真正开始执行一些字节码。

下面的代码，会输出什么？

``` Java
public class A {
     static int a = 0 ;
     static {
         a = 1;
         b = 1;
     }
     static int b = 0;
 
     public static void main(String[] args) {
         System.out.println(a);
         System.out.println(b);
     }
 }
```

结果是 1 0。a 和 b 唯一的区别就是它们的 static 代码块的位置。

这就引出一个规则：**static 语句块，只能访问到定义在 static 语句块之前的变量，定义之后的变量，只可以赋值，不能访问。**。所以下面的代码是无法通过编译的，而上面语句可以编译。不能访问是因为还没定义变量，也就没初始化，也就没有值。但是可以先赋值。

``` Java
static {
         b = b + 1;
 }
 static int b = 0;
```

我们再来看第二个规则：**JVM 会保证在子类的初始化方法执行之前，父类的初始化方法已经执行完毕**。

所以，JVM 第一个被执行的类初始化方法一定是 java.lang.Object。另外，也意味着父类中定义的 static 语句块要优先于子类的。

#### 1.5.1. 类的初始化和对象初始化

下面代码的输出是什么？

``` Java
public class A {
     static {
         System.out.println("1");
     }
     public A(){
         System.out.println("2");
     }
 }
 
 public class B extends A {
     static{
         System.out.println("a");
     }
     public B(){
         System.out.println("b");
     }
 
     public static void main(String[] args){
         A ab = new B();
         ab = new B();
     }
 }
```

答案是 
``` Java
1
a
2
b
2
b
```

你可以看下这张图。其中 static 字段和 static 代码块，是属于类的，在类的加载的初始化阶段就已经被执行。类信息会被存放在方法区，在同一个类加载器下，这些信息有一份就够了，所以上面的 static 代码块只会执行一次，它对应的是 `<cinit>` 方法。

而对象初始化就不一样了。通常，我们在 new 一个新对象的时候，都会调用它的构造方法，就是 `<init>`，用来初始化对象的属性。每次新建对象的时候，都会执行。

![](/img/Java/JVM/03_JVM%E7%9A%84%E7%B1%BB%E5%8A%A0%E8%BD%BD%E6%9C%BA%E5%88%B6/%E7%B1%BB%E7%9A%84%E5%88%9D%E5%A7%8B%E5%8C%96%E5%92%8C%E5%AF%B9%E8%B1%A1%E5%88%9D%E5%A7%8B%E5%8C%96.png)

所以，上面代码的 static 代码块只会执行一次，对象的构造方法执行两次。

## 2. 类加载器

类加载器做的就是上面 5 个步骤的事。

如果你在项目代码里，写一个 java.lang 的包，然后改写 String 类的一些行为，编译后，发现并不能生效。JRE 的类当然不能轻易被覆盖，否则会被别有用心的人利用，这就太危险了。

那类加载器是如何保证这个过程的安全性呢？其实，它是有着严格的等级制度的。

### 2.1. 几个类加载器

首先，我们介绍几个不同等级的类加载器。

#### 2.1.1. Bootstrap ClassLoader

这是加载器中的大 Boss，任何类的加载行为，都要经它过问。它的作用是加载核心类库，也就是 rt.jar、resources.jar、charsets.jar 等。当然这些 jar 包的路径是可以指定的，-Xbootclasspath 参数可以完成指定操作。

这个加载器是 C++ 编写的，随着 JVM 启动。

#### 2.1.2. Extention ClassLoader

扩展类加载器，主要用于加载 lib/ext 目录下的 jar 包和 .class 文件。同样的，通过系统变量 java.ext.dirs 可以指定这个目录。

这个加载器是个 Java 类，继承自 URLClassLoader。

#### 2.1.3. Application ClassLoader

这是我们写的 Java 类的默认加载器，有时候也叫作 System ClassLoader。一般用来加载 classpath 下的其他所有 jar 包和 .class 文件，我们写的代码，会首先尝试使用这个类加载器进行加载。

#### 2.1.4. Custom ClassLoader

自定义加载器，支持一些个性化的扩展功能。

### 2.2. 双亲委派机制

双亲委派机制的意思是除了顶层的启动类加载器以外，其余的类加载器，在加载之前，都会委派给它的父加载器进行加载。这样一层层向上传递，直到祖先们都无法胜任，它才会真正的加载。

打个比方。有一个家族，都是一些听话的孩子。孙子想要买一块棒棒糖，最终都要经过爷爷过问，如果力所能及，爷爷就直接帮孙子买了。

![](/img/Java/JVM/03_JVM%E7%9A%84%E7%B1%BB%E5%8A%A0%E8%BD%BD%E6%9C%BA%E5%88%B6/%E5%8F%8C%E4%BA%B2%E5%A7%94%E6%B4%BE%E6%9C%BA%E5%88%B6.png)

另外可以看到，除了启动类加载器，每一个加载器都有一个parent，并没有所谓的双亲。但是由于翻译的问题，这个叫法已经非常普遍了，一定要注意背后的差别。

使用双亲委派模型的好处在于**Java类随着它的类加载器一起具备了一种带有优先级的层次关系**。例如类java.lang.Object，它存在在rt.jar中，无论哪一个类加载器要加载这个类，最终都是委派给处于模型最顶端的Bootstrap ClassLoader进行加载，因此Object类在程序的各种类加载器环境中都是同一个类。

相反，如果没有双亲委派模型而是由各个类加载器自行加载的话，如果用户编写了一个java.lang.Object的同名类并放在ClassPath中，那系统中将会出现多个不同的Object类，程序将混乱。因此，如果开发者尝试编写一个与rt.jar类库中重名的Java类，可以正常编译，但是永远无法被加载运行。

#### 2.2.1. 双亲委派模型的系统实现

``` Java
protected synchronized Class<?> loadClass(String name,boolean resolve)throws ClassNotFoundException{
    //check the class has been loaded or not
    Class c = findLoadedClass(name);
    if(c == null){
        try{
            if(parent != null){
                c = parent.loadClass(name,false);
            }else{
                c = findBootstrapClassOrNull(name);
            }
        }catch(ClassNotFoundException e){
            //if throws the exception ,the father can not complete the load
        }
        if(c == null){
            c = findClass(name);
        }
    }
    if(resolve){
        resolveClass(c);
    }
    return c;
}
```

注意，双亲委派模型是Java设计者推荐给开发者的类加载器的实现方式，并不是强制规定的。大多数的类加载器都遵循这个模型，但是JDK中也有较大规模破坏双亲模型的情况。下面列举了一些特殊情况。

### 2.3. 一些自定义加载器

#### 2.3.1. 案例一：tomcat

![](/img/Java/JVM/03_JVM%E7%9A%84%E7%B1%BB%E5%8A%A0%E8%BD%BD%E6%9C%BA%E5%88%B6/Tomcat%E7%9A%84%E7%B1%BB%E5%8A%A0%E8%BD%BD%E5%99%A8%E6%9C%BA%E5%88%B6.png)

对于一些需要加载的非基础类，会由一个叫作 WebAppClassLoader 的类加载器优先加载。并不会传递给父类的加载器。等它加载不到的时候，再交给上层的 ClassLoader 进行加载。这个加载器用来隔绝不同应用的 .class 文件，比如你的两个应用，可能会依赖同一个第三方的不同版本，它们是相互没有影响的。

（**注**：对于任意一个类，都需要由加载它的类加载器和这个类本身共同确立其在Java虚拟机中的唯一性。）


另外Tomcat却可以使用 SharedClassLoader 所加载的类，实现了共享和分离的功能。

但是你自己写一个 ArrayList，放在应用目录里，tomcat 依然不会加载。它只是自定义的加载器顺序不同，但对于顶层来说，还是一样的（都是启动类加载器，ArrayList由其加载）。

#### 2.3.2. 案例二：SPI

SPI 全称为 (Service Provider Interface) ，是JDK内置的一种服务提供发现机制。SPI是一种动态替换发现的机制， 比如有个接口，想运行时动态的给它添加实现，你只需要添加一个实现。我们经常遇到的就是java.sql.Driver接口，其他不同厂商可以针对同一接口做出不同的实现，mysql和postgresql都有不同的实现提供给用户，而Java的SPI机制可以为某个接口寻找服务实现。

SPI 实际上是“基于接口的编程＋策略模式＋配置文件”组合实现的动态加载机制，主要使用 java.util.ServiceLoader 类进行动态装载。这种方式，同样打破了双亲委派的机制。

DriverManager 类和 ServiceLoader 类都是属于 rt.jar 的。它们的类加载器是 Bootstrap ClassLoader，也就是最上层的那个。以加载数据库驱动为例`Class.forName("com.mysql.jdbc.Driver")`，具体的数据库驱动，属于业务代码，这个启动类加载器是无法加载的。这就比较尴尬了，虽然凡事都要祖先过问，但祖先没有能力去做这件事情，怎么办？

```Java
//part1:DriverManager::loadInitialDrivers
  //jdk1.8 之后，变成了lazy的ensureDriversInitialized
  ...
  ServiceLoader <Driver> loadedDrivers = ServiceLoader.load(Driver.class);
  Iterator<Driver> driversIterator = loadedDrivers.iterator();
  ...
  
  //part2:ServiceLoader::load
  public static <T> ServiceLoader<T> load(Class<T> service) {
      ClassLoader cl = Thread.currentThread().getContextClassLoader();
      return ServiceLoader.load(service, cl);
  }
```

跟踪源码可以发现Java**把当前的类加载器，设置成了线程的上下文类加载器**。那么，对于一个刚刚启动的线程来说（例如main），它当前的加载器是谁呢？也就是说，启动 main 方法的那个加载器，到底是哪一个？

所以我们继续跟踪代码。找到 Launcher 类，就是 jre 中用于启动入口函数 main 的类。我们在 Launcher 中找到以下代码。

``` Java
public Launcher() {
 Launcher.ExtClassLoader var1;
 try {
     var1 = Launcher.ExtClassLoader.getExtClassLoader();
 } catch (IOException var10) {
     throw new InternalError("Could not create extension class loader", var10);
 }
 
 try {
     this.loader = Launcher.AppClassLoader.getAppClassLoader(var1);
 } catch (IOException var9) {
     throw new InternalError("Could not create application class loader", var9);
 }
 Thread.currentThread().setContextClassLoader(this.loader);
 ...
 }
```

可以发现当前线程**上下文的类加载器，是应用程序类加载器**。使用它来加载第三方驱动，是没有什么问题的。

#### 2.3.3. 案例三：OSGi

OSGi 曾经非常流行，Eclipse 就使用 OSGi 作为插件系统的基础。OSGi 是服务平台的规范，旨在用于需要长运行时间、动态更新和对运行环境破坏最小的系统。它实现了模块化，每个模块可以独立安装、启动、停止、卸载，

### 2.4. 如何替换 JDK 的类

因为我们上面提到的双亲委派机制，是无法直接在应用中替换 JDK 的原生类的。但是，有时候又不得不进行一下增强、替换，比如你想要调试一段代码，或者比 Java 团队早发现了一个 Bug。所以，Java 提供了 endorsed 技术，用于替换这些类。这个目录下的 jar 包，会比 rt.jar 中的文件，优先级更高，可以被最先加载到。

比如我们要修改 HashMap 类，就必须要使用到 Java 的 endorsed 技术。我们需要将自己的 HashMap 类，打包成一个 jar 包，然后放到 -Djava.endorsed.dirs 指定的目录中。注意类名和包名，应该和 JDK 自带的是一样的。但是，java.lang 包下面的类除外，因为这些都是特殊保护的。

另外根据官方文档描述：如果不想添加-D参数，如果我们希望基于这个JDK下的都统一改变，那么我们可以将我们修改的jar放到：

`$JAVA_HOME/jre/lib/endorsed`

