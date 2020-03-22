---
title: 线程池ThreadPoolExecutor
date: 2020-03-22T21:34:45+08:00
coverImage: https://s1.ax1x.com/2020/03/22/8IHptO.jpg
categories: 
    - Java
    - 基础
tags: 
    - Java
    - 基础
---
<!-- toc -->
线程池是为了避免线程频繁的创建和销毁带来的性能消耗，而建立的一种池化技术，它是把已创建的线程放入“池”中，当有任务来临时就可以重用已有的线程，无需等待创建的过程，这样就可以有效提高程序的响应速度。线程池不建议使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式，这样的处理方式让写的读者更加明确线程池的运行规则，规避资源耗尽的风险。

<!-- more -->
## 1. ThreadPoolExecutor 的核心参数

```Java
public ThreadPoolExecutor(int corePoolSize,                    // 线程池的常驻核心线程数
                          int maximumPoolSize,                 // 最大可以创建的线程数
                          long keepAliveTime,                  // 线程空闲时的存活时间
                          TimeUnit unit,                       // 存活时间的单位
                          BlockingQueue<Runnable> workQueue,   // 线程池执行的任务队列
                          ThreadFactory threadFactory,         // 线程的创建工厂
                          RejectedExecutionHandler handler);
```

第 1 个参数：**corePoolSize** 表示线程池的常驻核心线程数。如果设置为 0，则表示在没有任何任务时，销毁线程池；如果大于 0，即使没有任务时也会保证线程池的线程数量等于此值。但需要注意，此值如果设置的比较小，则会频繁的创建和销毁线程（创建和销毁的原因会在本课时的下半部分讲到）；如果设置的比较大，则会浪费系统资源，所以开发者需要根据自己的实际业务来调整此值。

第 2 个参数：**maximumPoolSize** 表示线程池在任务最多时，最大可以创建的线程数。官方规定此值必须大于 0，也必须大于等于 corePoolSize，此值只有在任务比较多，且不能存放在任务队列时，才会用到。

第 3 个参数：**keepAliveTime** 表示线程的存活时间，当线程池空闲时并且超过了此时间，多余的线程就会销毁，直到线程池中的线程数量销毁的等于 corePoolSize 为止，如果 maximumPoolSize 等于 corePoolSize，那么线程池在空闲的时候也不会销毁任何线程。

第 4 个参数：**unit** 表示存活时间的单位，它是配合 keepAliveTime 参数共同使用的。

第 5 个参数：**workQueue** 表示线程池执行的任务队列，当线程池的所有线程都在处理任务时，如果来了新任务就会缓存到此任务队列中排队等待执行。

第 6 个参数：**threadFactory** 表示线程的创建工厂，此参数一般用的比较少，我们通常在创建线程池时不指定此参数，它会使用默认的线程创建工厂的方法来创建线程。我们也可以自定义一个线程工厂，通过实现 ThreadFactory 接口来完成，这样就可以自定义线程的名称或线程执行的优先级了。

第 7 个参数：**RejectedExecutionHandler** 表示指定线程池的拒绝策略，当线程池的任务已经在缓存队列 workQueue 中存储满了之后，并且不能创建新的线程来执行此任务时，就会用到此拒绝策略，它属于一种限流保护的机制。

## 2. 线程池任务执行的主要流程

![](/img/Java/%E7%BA%BF%E7%A8%8B%E6%B1%A0ThreadPoolExecutor/%E6%89%A7%E8%A1%8C%E4%BB%BB%E5%8A%A1%E7%9A%84%E6%B5%81%E7%A8%8B.png)

## 3. execute() VS submit()

execute() 和 submit() 都是用来执行线程池任务的，它们最主要的区别是，submit() 方法可以接收线程池执行的返回值，而 execute() 不能接收返回值。

```Java
ThreadPoolExecutor executor = new ThreadPoolExecutor(2, 10, 10L,
        TimeUnit.SECONDS, new LinkedBlockingQueue(20));
// execute 使用
executor.execute(new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello, execute.");
    }
});
// submit 使用
Future<String> future = executor.submit(new Callable<String>() {
    @Override
    public String call() throws Exception {
        System.out.println("Hello, submit.");
        return "Success";
    }
});
System.out.println(future.get());
```

以上程序执行结果如下：

```Java
Hello, submit.
Hello, execute.
Success
```

从以上结果可以看出 submit() 方法可以配合 Futrue 来接收线程执行的返回值。它们的另一个区别是 execute() 方法属于 Executor 接口的方法，而 submit() 方法则是属于 ExecutorService 接口的方法，它们的继承关系如下图所示：

![](/img/Java/%E7%BA%BF%E7%A8%8B%E6%B1%A0ThreadPoolExecutor/Executor%E5%92%8CExecutorService%E7%9A%84%E7%BB%A7%E6%89%BF%E5%85%B3%E7%B3%BB.png)

## 4. 线程池的拒绝策略

当线程池中的任务队列已经被存满，再有任务添加时会先判断当前线程池中的线程数是否大于等于线程池的最大值，如果是，则会触发线程池的拒绝策略。

Java 自带的拒绝策略有 4 种：
1. AbortPolicy，终止策略，线程池会**抛出异常**并终止执行，它是默认的拒绝策略；
2. CallerRunsPolicy，把任务交给当前线程来执行；
3. DiscardPolicy，忽略此任务（最新的任务）；
4. DiscardOldestPolicy，忽略最早的任务（最先加入队列的任务）。

### 4.1. 自定义拒绝策略

自定义拒绝策略只需要新建一个 RejectedExecutionHandler 对象，然后重写它的 rejectedExecution() 方法即可，如下代码所示：

```Java
ThreadPoolExecutor executor = new ThreadPoolExecutor(1, 3, 10,
        TimeUnit.SECONDS, new LinkedBlockingQueue<>(2),
        new RejectedExecutionHandler() {  // 添加自定义拒绝策略
            @Override
            public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
                // 业务处理方法
                System.out.println("执行自定义拒绝策略");
            }
        });
for (int i = 0; i < 6; i++) {
    executor.execute(() -> {
        System.out.println(Thread.currentThread().getName());
    });
}
```

以上代码执行的结果如下：

``` Java
执行自定义拒绝策略
pool-1-thread-2
pool-1-thread-3
pool-1-thread-1
pool-1-thread-1
pool-1-thread-2
```

可以看出线程池执行了自定义的拒绝策略，我们可以在 rejectedExecution 中添加自己业务处理的代码。

## 5. ThreadPoolExecutor 扩展

ThreadPoolExecutor 的扩展主要是通过重写它的 beforeExecute() 和 afterExecute() 方法实现的，我们可以在扩展方法中添加日志或者实现数据统计，比如统计线程的执行时间。

```Java
class MyThreadPoolExecutor extends ThreadPoolExecutor {

    public MyThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime,
                        TimeUnit unit, BlockingQueue<Runnable> workQueue) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue);
    }
    /**
     * 开始执行之前
     * @param t 线程
     * @param r 任务
     */
    @Override
    protected void beforeExecute(Thread t, Runnable r) {
        // 自定义线程执行之前的逻辑
        super.beforeExecute(t, r);
    }
    /**
     * 执行完成之后
     * @param r 任务
     * @param t 抛出的异常
     */
    @Override
    protected void afterExecute(Runnable r, Throwable t) {
        // 自定义线程执行之后的逻辑
        super.afterExecute(r, t);
    }
}
```

## 6. 小结

最后我们总结一下：线程池的使用必须要通过 ThreadPoolExecutor 的方式来创建，这样才可以更加明确线程池的运行规则，规避资源耗尽的风险。同时，也介绍了 ThreadPoolExecutor 的七大核心参数，包括核心线程数和最大线程数之间的区别，当线程池的任务队列没有可用空间且线程池的线程数量已经达到了最大线程数时，则会执行拒绝策略，Java 自动的拒绝策略有 4 种，用户也可以通过重写 rejectedExecution() 来自定义拒绝策略，我们还可以通过重写 beforeExecute() 和 afterExecute() 来实现 ThreadPoolExecutor 的扩展功能。


---

系列：
上一篇：{% post_link  %}
下一篇：{% post_link  %}
