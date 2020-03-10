---
title: Java多线程同步相关的语法概念
date: 2020-03-10T11:57:16+08:00
coverImage: https://s2.ax1x.com/2020/03/10/8PAZLD.jpg
categories: 
    - Java
    - 基础
tags: 
    - Java
---
<!-- toc -->
当多个线程同时共享同一个全局变量或静态变量，如果同步没有做好，可能会发生数据冲突问题。同步是保证数据原子性，原子性就是数据不能受到其他线程干扰。本文将先介绍多线程的创建，然后介绍各种保证同步的手段。

<!-- more -->
## 1. 线程启动方法

### 1.1. 继承Thread类创建线程

启动线程的唯一方法就是通过Thread类的start()实例方法。start()方法是一个native方法，它将启动一个新线程，并执行run()方法。这种方式实现多线程很简单，通过自己的类直接extend Thread，并复写run()方法，就可以启动新线程并执行自己定义的run()方法。

``` Java
public class MyThread extends Thread {  
　　public void run() {  
　　 System.out.println("MyThread.run()");  
　　}  
}  
 
MyThread myThread1 = new MyThread();  
MyThread myThread2 = new MyThread();  
myThread1.start();  
myThread2.start();
```

### 1.2. 实现Runnable接口创建线程

如果自己的类已经extends另一个类，就无法直接extends Thread，此时，可以实现一个Runnable接口。然后传给Thread构造函数来启动线程。实际上Thread类本质上就是实现了Runnable接口的一个实例。

```Java
public class MyRunnable extends OtherClass implements Runnable {  
　　public void run() {  
　　 System.out.println("MyThread.run()");  
　　}  
}

MyRunnable myRunnable = new MyRunnable();  
Thread thread = new Thread(myRunnable);  
thread.start();  
```

### 1.3. 其他

- 使用Callable和Future创建线程
- 线程池


## 2. synchronized -- 锁

synchronized锁的存在，保证一个函数/代码块同时只会有一个线程访问。

### 2.1. 同步代码块--对象锁

``` Java
synchronized(对象){  // 这个对象可以为任意对象, 一般new Object()即可
    可能会发生线程冲突的问题
}
```

对象如同锁，持有锁的线程可以在同步块中执行，没持有锁的线程，即使获取cup的执行权，也进不去。

使用synchronized必须有一些条件：

1. 必须要有两个或者两个以上的线程需要发生同步。
2. 多个线程想同步，必须使用同一把锁
3. 保证只有一个线程进行执行

### 2.2. 同步函数--this锁

``` Java
public synchronized void func(){
    函数体内容
}
```

问题：
- 一个线程使用同步函数，另一个线程使用同步代码块this锁，能够同步吗？可以同步。
- 一个线程使用同步函数，另一个线程使用同步代码块（非this锁），能够同步吗？不能同步。

### 2.3. 静态同步函数--字节码文件锁

``` Java
public static synchronized void func(){
    函数体内容
}
```

同步函数使用this锁，静态同步函数使用当前字节码文件（字节码文件就是当前class文件）。所以两个线程，一个线程使用同步函数，另一个线程使用静态同步函数不能实现同步。

## 3. Volatile--可见性

Volatile 关键字的作用是变量在多个线程之间可见，但不保证原子性。

线程可见性：线程1本地内存的变量值发生改变之后，立马通知给另一个线程。

```Java
public class ThreadVolatile {
    public static void main(String[] args) throws InterruptedException {
        ThreadVolatileDemo td = new ThreadVolatileDemo();
        td.start();
        Thread.sleep(3000);
        // 主线程修改了共享全局变量为false
        td.setFlg(false);
        System.out.println("flg值已经修改为false!");
        Thread.sleep(1000);
        System.out.println(td.flg);

    }
}

class ThreadVolatileDemo extends Thread{

    public boolean flg = true;

    @Override
    public void run() {
        System.out.println("子线程开始执行...");
        while(flg){
      // ThreadVolatileDemo线程依然读取的是本地线程内存值 
        }
        System.out.println("子线程执行结束...");
    }

    public void setFlg(boolean flg){
        this.flg = flg;
    }
}
```

问题，当flg值修改为false后，程序一直停止不了

![](/img/Java/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%90%8C%E6%AD%A5%E7%9B%B8%E5%85%B3%E7%9A%84%E8%AF%AD%E6%B3%95%E6%A6%82%E5%BF%B5/violate.png)

**解决方法：**使用volatile关键字强制刷新主内存

`public volatile boolean flg = true;`

如果没有添加休眠时间，在不添加volatile关键字时，修改flg值后，flg值是实时修改，程序执行完毕结束。  
主线程添加休眠时间后，在不添加volatile关键字时，修改flg值后，不会及时通知给子线程。

## 4. AtomicInteger--原子类

创建10个线程，每个线程执行1000次，共享count变量。

```Java
public class VolatileNoAtomic extends Thread{
    // 需要10个线程同时共享count  static修饰关键字，存放在静态区，只会存放一次，所有线程都会共享
    private volatile static int count = 0;

    @Override
    public void run() {
        for (int i = 0; i < 1000; i++) {
            count++;
        }
        System.out.println(getName()+","+count);
    }

    public static void main(String[] args) {
        // 创建10个线程
        VolatileNoAtomic[] vaList = new VolatileNoAtomic[10];
        for (int i = 0; i < vaList.length; i++) {
            vaList[i] = new VolatileNoAtomic();
        }
        for (int i = 0; i < vaList.length; i++) {
            vaList[i].start();
        }
    }
}
```

结果：

![](/img/Java/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%90%8C%E6%AD%A5%E7%9B%B8%E5%85%B3%E7%9A%84%E8%AF%AD%E6%B3%95%E6%A6%82%E5%BF%B5/%E4%B8%A2%E5%A4%B1%E4%BF%AE%E6%94%B9.png)

发现即便加上volatile关键字数据也不准确。期望Thread-9的结果应该是10000，结果却不足10000，可以发现`count++;`没有实现原子操作。线程A线程B同时读取count,然后同时+1,导致其中一次+1没有起作用。

**解决方法：**

java并发包：java.util.concurrent  jdk1.5

![](/img/Java/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%90%8C%E6%AD%A5%E7%9B%B8%E5%85%B3%E7%9A%84%E8%AF%AD%E6%B3%95%E6%A6%82%E5%BF%B5/%E5%8E%9F%E5%AD%90%E7%B1%BB.png)

AtomicInteger原子类,用于计数

```Java
...
    private static AtomicInteger count = new AtomicInteger(0);

    @Override
    public void run() {
        for (int i = 0; i < 1000; i++) {
            //count++;
            // 以原子方式将当前值加 1
            count.incrementAndGet();
        }
        System.out.println(getName()+","+count.get());
    }
...
```

## 5. ThreadLocal

ThreadLocal就是一个Map, key是当前线程，value是存储的值

使用起来就是：get/set，只能操作自己线程对应的值

```Java
public static ThreadLocal<Integer> threadLocal = new ThreadLocal<Integer>(){
    protected Integer initialValue(){
        return 0;
    }
};

public int getNumber(){
    count = threadLocal.get()+1;
    threadLocal.set(count);
    return count;
}
```

**底层原理：**

![](/img/Java/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%90%8C%E6%AD%A5%E7%9B%B8%E5%85%B3%E7%9A%84%E8%AF%AD%E6%B3%95%E6%A6%82%E5%BF%B5/ThreadLocal.png)

## 6. Semaphore--信号量

**应用场景：**用于控制先后顺序、并发数量。和操作系统中的信号量意义一致。

``` Java
Semaphore semaphore = new Semaphore(资源数量);
...
各线程：
semaphore.acquire();  // 获取资源
其他工作
semaphore.release();  // 释放资源
```

## 7. CyclicBarrier--同步屏障

**应用场景：** 用于多线程计算数据，最后合并计算结果。例如所有人到齐了，一起做下一件事，做完再做各自的事情。

``` Java
final CyclicBarrier cb = new CyclicBarrier(等待的人数, runnable);   // runnable就是等待的人数都到齐后做的事情。
...
各线程：
cb.await();        // 等待屏障结束各自才执行后面的代码
```


## 8. ConutDownLatch--倒计时器

**应用场景：** 等待若干个其他线程执行完任务之后再执行

`ConutDownLatch` 与 `CyclicBarrier` 区别：
- 共同点：都能够实现线程之间的等待
- 不同点：
    - `ConutDownLatch` 一般用于某个线程A等待若干个其他线程执行完任务之后，它才能执行。`CyclicBarrier` 一般用于一组线程互相等待到某个状态，然后这一组线程在同时执行
    - `ConutDownLatch` `是不能重用的，CyclicBarrier` 可以重复使用

``` Java
final CountDownLatch latch = new CountDownLatch(等待人数);  
...
其他多个线程：
latch.countDown();           // 该线程做完了，计数器-1
...
主线程：
latch.await();               // 等待计数器为0才执行后面的代码
```

## 9. Exchanger--数据交换

**应用场景：**用于两个线程之间交换数据，例如一手交钱一手交货

```Java
Exchanger<T> exchanger = new Exchanger<>();        // T 交换的数据类型
...
A线程：
T fromB = exchanger.exchange(fromA);               // 将fromA交出去，得到fromB
B线程：
T fromA = exchanger.exchange(fromB);
```


---

系列：
上一篇：{% post_link  %}
下一篇：{% post_link  %}
