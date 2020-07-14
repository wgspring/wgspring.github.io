---
title: 你真的懂Kotlin中Lambda表达式吗？
date: 2020-07-14T23:13:35+08:00
coverImage: https://s1.ax1x.com/2020/07/14/Ua0vvV.jpg
categories: 
    - Android
    - Kotlin
tags: 
    - Android
    - Kotlin
---
<!-- toc -->
刚刚接触Lambda时，我们都觉得好用，简洁，大大减少模版代码，嗯不错。可让我们自己手写一个Lambda表达式时又突然发现无从下手，其实大多数已经用了很久 Kotlin 的人，对 Lambda 也只会简单使用而已，甚至相当一部分人不靠开发工具的自动补全功能，根本就完全不会写 Lambda。

<!-- more -->

## 1. 从高阶函数说起

函数这概念一开始时来源于数学的，数学中`y = f(x)`，对应到我们写的代码中，x就是参数，f就是函数名，y就是返回值。

高阶函数是指参数还是一个函数，高等数学中你一定写过`y = f(g(x))`，这就是高阶函数，函数y的参数是g(x)，是一个函数。对应到代码中，就是函数的参数是一个函数呗。

我们知道在Java代码中，函数的参数必须是一个变量，不能是函数。如果我们期望实现一个计算函数，函数名`compute`，接受三个参数，前两个是计算的值，第三个参数是计算的方法，例如加减乘除之类的。用Java怎么实现呢？

你可能会想到这样的代码：

```Java
int compute(int a, int b, String method){
    if (method.equal("add")) {
        return a + b;
    } else (method.equal("sub")) {
        return a - b;
    } else if (...)
      ...
    }
}
```

这样的代码耦合性太高，每添加一个新的计算方法就得改compute函数内部逻辑，开发中大概率肯定不会这样写的。一般是通过接口的方式来把方法包装起来，然后compute函数的第三个参数传入不同的实现类：

```Java
// 定义接口
public interface ComputeType {
  int invoke(int a, int b);
}

// 为ComputeType接口实现各种不同的具体计算方式的实现类
...

// 计算函数
int compute(int a, int b, ComputeType computeType) {
    return computeType.invoke(a, b)
}
```

以上是Java的做法，为了实现函数具体功能受参数影响，需要多写一个接口。

在Kotlin中是怎样解决这种问题的呢？

```Kotlin
fun compute(a: Int, b: Int, method: Fun): Int {
    return method(a, b)
}
```

上边的代码是不是很清晰，但是编译不过，😄 Kotlin中没有Fun这个数据类型哈，我只是方便你理解瞎写的一个。 method的数据类型是一个Fun，编译器不知道你这个Fun是要什么参数，有几个参数，会返回什么呀，所以我们得将这个Fun替换成`(Int, Int) -> Int`，这样之后，编译器就知道：哦，你是一个要两个Int参数，返回Int的Fun呀。所以上例代码正确写法就变成下面这样：

```Kotlin
fun compute(a: Int, b: Int, method: (Int, Int) -> Int): Int {
    return method(a, b)
}
```

这样的代码看上去有点阔怕，但是只有这样写编译器才知道你是需要怎么样的一个Fun呀。

同样的，函数类型不只可以作为函数的参数类型，还可以作为函数的返回值类型：

```Kotlin
fun f(param: Int): (Int) -> Unit {
  ...
}
```

这个函数f接受一个Int变量，然后返回一个Fun类型，这个Fun呢，接受一个Int，返回Unit

这种「参数或者返回值为函数类型的函数」，在 Kotlin 中就被称为「高阶函数」

既然提到了函数的参数可以是一个Fun，那么一个Fun类型的变量怎么定义呢，我们只有定义了一个Fun变量才可以更方便地作为参数传递呀。

```Kotlin
// 我们有一个add函数
fun add(a: Int, b: Int): Int{
    return a + b
}

// 将add函数赋值给一个变量
addFun = ::add

// 函数作为参数传递给compute函数
c = compute(1, 2, addFun)
// 或者直接传递
c = compute(1, 2, ::add)
```

## 2. 等等，等等，::add是什么鬼

这个双冒号的写法叫做函数引用 Function Reference，这是 Kotlin 官方的说法。但是这又表示什么意思？表示它指向上面的函数？那既然都是一个东西，为什么不直接写函数名，而要加两个冒号呢？

因为加了两个冒号，这个函数才变成了一个对象。

Kotlin 里「函数可以作为参数」这件事的本质，是函数在 Kotlin 里可以作为对象存在——因为只有对象才能被作为参数传递啊。赋值也是一样道理，只有对象才能被赋值给变量啊。但 Kotlin 的函数本身的性质又决定了它没办法被当做一个对象。那怎么办呢？Kotlin 的选择是，那就创建一个和函数具有相同功能的对象。怎么创建？使用双冒号。

在 Kotlin 里，一个函数名的左边加上双冒号，它就不表示这个函数本身了，而表示一个对象，或者说一个指向对象的引用，但，这个对象可不是函数本身，而是一个和这个函数具有相同功能的对象。

怎么个相同法呢？你可以怎么用函数，就能怎么用这个加了双冒号的对象：

```kotlin
// 直接调用add函数
c = add(1, 2)
// 通过对象调用add函数
c = (::add)(1, 2)

// 通过变量调用,这样addFun看上去是函数类型，实质上是add这个对象！
addFun = ::add
c = addFun(1, 2)
```

**包括双冒号加上函数名的这个写法，它是一个指向对象的引用，并不是指向函数本身，而是指向一个我们在代码里看不见的对象。这个对象复制了原函数的功能，它并不是原函数。**


## 3. 匿名函数

回到上文讲到的compute函数，我们通过将add函数(对象)赋值给addFun这个变量，然后传递给compete函数的：

```Kotlin
fun compute(a: Int, b: Int, method: (Int, Int) -> Int): Int {
    return method(a, b)
}

fun add(a: Int, b: Int): Int{
    return a + b
}

var addFun = ::add

c = compute(1, 2, addFun)
```

实际上，除了用双冒号来拿现成的函数使用，我们还可以直接把这个函数挪过来写，既然整个函数都挪过来了，函数名也就不起作用了，可以直接省略：

```Kotlin
fun compute(a: Int, b: Int, method: (Int, Int) -> Int): Int {
    return method(a, b)
}

// 将整个add函数作为compute的参数传递
c = compute(1, 2, fun(a: Int, b: Int): Int{
    return a + b
})

// 将整个add函数赋值给addFun这个变量
var addFun =  fun(a: Int, b: Int): Int{
    return a + b
}
```

这种函数名字都省略的写法就叫匿名函数

## 4. Lambda表达式

上文我们讲解高阶函数和匿名函数是以compute为例，包含两个普通参数和一个函数参数。现在为了更方便解释Lambda表达式，我们以安卓中非常常用的`View.setOnClickListener`为例介绍Lambda表达式。

```Kotlin
// 设置点击监听的这个函数需要接受一个fun(v: View): Unit类型的函数参数，或者说接受一个Lambda表达式
fun setOnClickListener(onClick: (View) -> Unit) {
    this.onClick = onClick
}

// 为view设置监听器，并通过匿名函数的形式传入一个fun(v: View): Unit类型参数
view.setOnClickListener(fun(v: View): Unit{
    v.xxx
})


```

有了上文介绍的高阶函数和匿名函数，这个代码应该很容易看懂了。现在我们开始简化为view设置监听器的过程：

1. 首先简化返回值类型，简化返回值类型之后，fun这个关键字也就不需要了，参数类型也可以挪到大括号里面，然后变成下面这样，变成一个Lambda表达式

```Kotlin
view.setOnClickListener({ v: View ->
    v.xxx
})
```

为什么可以简化返回值类型？是因为`fun setOnClickListener(onClick: (View) -> Unit) {`这句话，定义这个函数时，已经指定了返回类型是Unit，使用这个函数时就没必要声明返回类型了，编译器能够自动推断。

2. 如果 Lambda 是函数的最后一个参数，你可以把 Lambda 写在大括号的外面：

```kotlin
view.setOnClickListener() { v: View ->
    v.xxx
}
```

3. 如果 Lambda 是函数唯一的参数，你还可以直接把括号去了：

```kotlin
view.setOnClickListener { v: View ->
    v.xxx
}
```

4. 如果这个 Lambda 是单参数的，它的这个参数也省略掉不写，如果需要用到这个参数怎么办？用`it`代替：

```kotlin
view.setOnClickListener {
    it.xxx
}
```

到此是不是变得很清爽？？

前面我们介绍的是将Lambda作为函数参数传递，如果想将Lambda传递给一个变量再传给函数怎么办呢？

```Kotlin
// 匿名函数写法
var listener = fun(v: View): Unit{
    v.xxx
}

// Lambda表达式写法
var listener: (v: View) -> Unit = {
    v.xxx
}

// 或者省略参数类型，自动推断，用it代替
var listener: (View) -> Unit = {
    it.xxx
}
```

最后，补充一点需要注意的：Lambda 的返回值别写 return，如果你写了，它会把这个作为它外层的函数的返回值来直接结束外层函数。当然如果你就是想这么做那没问题啊，但如果你是只是想返回 Lambda，这么写就出错了。

另外因为 Lambda 是个代码块，它总能根据最后一行代码来推断出返回值类型，所以它的返回值类型确实可以不写。实际上，Kotlin 的 Lambda 也是写不了返回值类型的，语法上就不支持。

## 5. Kotlin 里匿名函数和 Lambda 表达式的本质

我们先看匿名函数。它可以作为参数传递，也可以赋值给变量，对吧？

但是我们刚才也说过了函数是不能作为参数传递，也不能赋值给变量的，对吧？

那为什么匿名函数就这么特殊呢？

因为 Kotlin 的匿名函数不——是——函——数。它是个对象。匿名函数虽然名字里有「函数」两个字，包括英文的原名也是 Anonymous Function，但它其实不是函数，而是一个对象，一个函数类型的对象。它和双冒号加函数名是一类东西，和函数不是。

所以，你才可以直接把它当做函数的参数来传递以及赋值给变量。

同理，Lambda 其实也是一个函数类型的对象而已。你能怎么使用双冒号加函数名，就能怎么使用匿名函数，以及怎么使用 Lambda 表达式。

这，就是 Kotlin 的匿名函数和 Lambda 表达式的本质，它们都是函数类型的对象。Kotlin 的 Lambda 跟 Java 8 的 Lambda 是不一样的，Java 8 的 Lambda 只是一种便捷写法，本质上并没有功能上的突破，而 Kotlin 的 Lambda 是实实在在的对象。

在你知道了在 Kotlin 里「函数并不能传递，传递的是对象」和「匿名函数和 Lambda 表达式其实都是对象」这些本质之后，你以后去写 Kotlin 的高阶函数会非常轻松非常舒畅。

Kotlin 官方文档里对于双冒号加函数名的写法叫 Function Reference 函数引用，故意引导大家认为这个引用是指向原函数的，这是为了简化事情的逻辑，让大家更好上手 Kotlin；但这种逻辑是有毒的，一旦你信了它，你对于匿名函数和 Lambda 就怎么也搞不清楚了。
