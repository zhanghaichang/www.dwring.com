---
layout: post
title:  两道面试题，带你解析Java类加载机制
category: java
tags: [java]
---


在许多Java面试中，我们经常会看到关于Java类加载机制的考察，例如下面这道题：

```java
    class Grandpa
    {
        static
        {
            System.out.println("爷爷在静态代码块");
        }
    }    
    class Father extends Grandpa
    {
        static
        {
            System.out.println("爸爸在静态代码块");
        }
    
        public static int factor = 25;
    
        public Father()
        {
            System.out.println("我是爸爸~");
        }
    }
    class Son extends Father
    {
        static 
        {
            System.out.println("儿子在静态代码块");
        }
    
        public Son()
        {
            System.out.println("我是儿子~");
        }
    }
    public class InitializationDemo
    {
        public static void main(String[] args)
        {
            System.out.println("爸爸的岁数:" + Son.factor);  //入口
        }
    }
```

请写出最后的输出字符串。

正确答案是：

    爷爷在静态代码块
    爸爸在静态代码块
    爸爸的岁数:25

我相信很多同学看到这个题目之后，表情是崩溃的，完全不知道从何入手。有的甚至遇到了几次，仍然无法找到正确的解答思路。

**其实这种面试题考察的就是你对Java类加载机制的理解。**

如果你对Java加载机制不理解，那么你是无法解答这道题目的。

所以这篇文章，我先带大家学习Java类加载的基础知识，然后再实战分析几道题目让大家掌握思路。

下面我们先来学习下Java类加载机制的七个阶段。

Java类加载机制的七个阶段
--------------

当我们的Java代码编译完成后，会生成对应的 class 文件。接着我们运行`java Demo`命令的时候，我们其实是启动了JVM 虚拟机执行 class 字节码文件的内容。而 JVM 虚拟机执行 class 字节码的过程可以分为七个阶段：**加载、验证、准备、解析、初始化、使用、卸载。**

### 加载

下面是对于加载过程最为官方的描述。

> 加载阶段是类加载过程的第一个阶段。在这个阶段，JVM 的主要目的是将字节码从各个位置（网络、磁盘等）转化为二进制字节流加载到内存中，接着会为这个类在 JVM 的方法区创建一个对应的 Class 对象，这个 Class 对象就是这个类各种数据的访问入口。

其实加载阶段用一句话来说就是：把代码数据加载到内存中。这个过程对于我们解答这道问题没有直接的关系，但这是类加载机制的一个过程，所以必须要提一下。

### 验证

当 JVM 加载完 Class 字节码文件并在方法区创建对应的 Class 对象之后，JVM 便会启动对该字节码流的校验，只有符合 JVM 字节码规范的文件才能被 JVM 正确执行。这个校验过程大致可以分为下面几个类型：

*   **JVM规范校验。** JVM 会对字节流进行文件格式校验，判断其是否符合 JVM 规范，是否能被当前版本的虚拟机处理。例如：文件是否是以 `0x cafe bene`开头，主次版本号是否在当前虚拟机处理范围之内等。
*   **代码逻辑校验。** JVM 会对代码组成的数据流和控制流进行校验，确保 JVM 运行该字节码文件后不会出现致命错误。例如一个方法要求传入 int 类型的参数，但是使用它的时候却传入了一个 String 类型的参数。一个方法要求返回 String 类型的结果，但是最后却没有返回结果。代码中引用了一个名为 Apple 的类，但是你实际上却没有定义 Apple 类。

当代码数据被加载到内存中后，虚拟机就会对代码数据进行校验，看看这份代码是不是真的按照JVM规范去写的。这个过程对于我们解答问题也没有直接的关系，但是了解类加载机制必须要知道有这个过程。

### 准备（重点）

当完成字节码文件的校验之后，JVM 便会开始为类变量分配内存并初始化。这里需要注意两个关键点，即内存分配的对象以及初始化的类型。

*   **内存分配的对象。** Java 中的变量有「类变量」和「类成员变量」两种类型，「类变量」指的是被 static 修饰的变量，而其他所有类型的变量都属于「类成员变量」。在准备阶段，JVM 只会为「类变量」分配内存，而不会为「类成员变量」分配内存。「类成员变量」的内存分配需要等到初始化阶段才开始。

例如下面的代码在准备阶段，只会为 factor 属性分配内存，而不会为 website 属性分配内存。

    public static int factor = 3;
    public String website = "www.cnblogs.com/chanshuyi";

*   **初始化的类型。** 在准备阶段，JVM 会为类变量分配内存，并为其初始化。但是这里的初始化指的是为变量赋予 Java 语言中该数据类型的零值，而不是用户代码里初始化的值。

例如下面的代码在准备阶段之后，sector 的值将是 0，而不是 3。

    public static int sector = 3;

但如果一个变量是常量（被 static final 修饰）的话，那么在准备阶段，属性便会被赋予用户希望的值。例如下面的代码在准备阶段之后，number 的值将是 3，而不是 0。

    public static final int number = 3;

之所以 static final 会直接被复制，而 static 变量会被赋予零值。其实我们稍微思考一下就能想明白了。

两个语句的区别是一个有 final 关键字修饰，另外一个没有。而 final 关键字在 Java 中代表不可改变的意思，意思就是说 number 的值一旦赋值就不会在改变了。既然一旦赋值就不会再改变，那么就必须一开始就给其赋予用户想要的值，因此被 final 修饰的类变量在准备阶段就会被赋予想要的值。而没有被 final 修饰的类变量，其可能在初始化阶段或者运行阶段发生变化，所以就没有必要在准备阶段对它赋予用户想要的值。

### 解析

当通过准备阶段之后，JVM 针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用点限定符 7 类引用进行解析。这个阶段的主要任务是将其在常量池中的符号引用替换成直接其在内存中的直接引用。

其实这个阶段对于我们来说也是几乎透明的，了解一下就好。

### 初始化（重点）

到了初始化阶段，用户定义的 Java 程序代码才真正开始执行。在这个阶段，JVM 会根据语句执行顺序对类对象进行初始化，一般来说当 JVM 遇到下面 5 种情况的时候会触发初始化：

*   遇到 new、getstatic、putstatic、invokestatic 这四条字节码指令时，如果类没有进行过初始化，则需要先触发其初始化。生成这4条指令的最常见的Java代码场景是：使用new关键字实例化对象的时候、读取或设置一个类的静态字段（被final修饰、已在编译器把结果放入常量池的静态字段除外）的时候，以及调用一个类的静态方法的时候。
*   使用 java.lang.reflect 包的方法对类进行反射调用的时候，如果类没有进行过初始化，则需要先触发其初始化。
*   当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。
*   当虚拟机启动时，用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机会先初始化这个主类。
*   当使用 JDK1.7 动态语言支持时，如果一个 java.lang.invoke.MethodHandle实例最后的解析结果 REF\_getstatic,REF\_putstatic,REF_invokeStatic 的方法句柄，并且这个方法句柄所对应的类没有进行初始化，则需要先出触发其初始化。

看到上面几个条件你可能会晕了，但是不要紧，不需要背，知道一下就好，后面用到的时候回到找一下就可以了。

### 使用

当 JVM 完成初始化阶段之后，JVM 便开始从入口方法开始执行用户的程序代码。这个阶段也只是了解一下就可以。

### 卸载

当用户程序代码执行完毕后，JVM 便开始销毁创建的 Class 对象，最后负责运行的 JVM 也退出内存。这个阶段也只是了解一下就可以。

看完了Java的类加载机智之后，是不是有点懵呢。不怕，我们先通过一个小例子来醒醒神。
```java
    public class Book {
        public static void main(String[] args)
        {
            System.out.println("Hello ShuYi.");
        }
    
        Book()
        {
            System.out.println("书的构造方法");
            System.out.println("price=" + price +",amount=" + amount);
        }
    
        {
            System.out.println("书的普通代码块");
        }
    
        int price = 110;
    
        static
        {
            System.out.println("书的静态代码块");
        }
    
        static int amount = 112;
    }
```

思考一下上面这段代码输出什么？

给你5分钟思考，5分钟后交卷，哈哈。

怎么样，想好了吗，公布答案了。

    书的静态代码块
    Hello ShuYi.

怎么样，你答对了吗？是不是和你想得有点不一样呢。

下面我们来简单分析一下，首先根据上面说到的触发初始化的5种情况的第4种（当虚拟机启动时，用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机会先初始化这个主类），我们会进行类的初始化。

那么类的初始化顺序到底是怎么样的呢？

重点来了！

重点来了！

重点来了！

在我们代码中，我们只知道有一个构造方法，但实际上Java代码编译成字节码之后，是没有构造方法的概念的，只有类初始化方法 和 对象初始化方法 。

那么这两个方法是怎么来的呢？

*   类初始化方法。编译器会按照其出现顺序，收集类变量的赋值语句、静态代码块，最终组成类初始化方法。**类初始化方法一般在类初始化的时候执行。**

上面的这个例子，其类初始化方法就是下面这段代码了：

```java
        static
        {
            System.out.println("书的静态代码块");
        }
        static int amount = 112;
```

*   对象初始化方法。编译器会按照其出现顺序，收集成员变量的赋值语句、普通代码块，最后收集构造函数的代码，最终组成对象初始化方法。**对象初始化方法一般在实例化类对象的时候执行。**

上面这个例子，其对象初始化方法就是下面这段代码了：

```java
        {
            System.out.println("书的普通代码块");
        }
        int price = 110;
        System.out.println("书的构造方法");
        System.out.println("price=" + price +",amount=" + amount);
```

类初始化方法 和 对象初始化方法 之后，我们再来看这个例子，我们就不难得出上面的答案了。

但细心的朋友一定会发现，其实上面的这个例子其实没有执行对象初始化方法。

因为我们确实没有进行 Book 类对象的实例化。如果你在 main 方法中增加 new Book() 语句，你会发现对象的初始化方法执行了！

感兴趣的朋友可以自己动手试一下，我这里就不执行了。

通过了上面的理论和简单例子，我们下面进入更加复杂的实战分析吧！

实战分析
----
```java
    class Grandpa
    {
        static
        {
            System.out.println("爷爷在静态代码块");
        }
    }    
    class Father extends Grandpa
    {
        static
        {
            System.out.println("爸爸在静态代码块");
        }
    
        public static int factor = 25;
    
        public Father()
        {
            System.out.println("我是爸爸~");
        }
    }
    class Son extends Father
    {
        static 
        {
            System.out.println("儿子在静态代码块");
        }
    
        public Son()
        {
            System.out.println("我是儿子~");
        }
    }
    public class InitializationDemo
    {
        public static void main(String[] args)
        {
            System.out.println("爸爸的岁数:" + Son.factor);  //入口
        }
    }
```
	
思考一下，上面的代码最后的输出结果是什么？

最终的输出结果是：

    爷爷在静态代码块
    爸爸在静态代码块
    爸爸的岁数:25

也许会有人问为什么没有输出「儿子在静态代码块」这个字符串？

**这是因为对于静态字段，只有直接定义这个字段的类才会被初始化（执行静态代码块）。**因此通过其子类来引用父类中定义的静态字段，只会触发父类的初始化而不会触发子类的初始化。

对面上面的这个例子，我们可以从入口开始分析一路分析下去：

*   首先程序到 main 方法这里，使用标准化输出 Son 类中的 factor 类成员变量，但是 Son 类中并没有定义这个类成员变量。于是往父类去找，我们在 Father 类中找到了对应的类成员变量，于是触发了 Father 的初始化。
*   但根据我们上面说到的初始化的 5 种情况中的第 3 种（当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化）。我们需要先初始化 Father 类的父类，也就是先初始化 Grandpa 类再初始化 Father 类。于是我们先初始化 Grandpa 类输出：「爷爷在静态代码块」，再初始化 Father 类输出：「爸爸在静态代码块」。
*   最后，所有父类都初始化完成之后，Son 类才能调用父类的静态变量，从而输出：「爸爸的岁数：25」。

怎么样，是不是觉得豁然开朗呢。

我们再来看一下一个更复杂点的例子，看看输出结果是啥。

```java
    class Grandpa
    {
        static
        {
            System.out.println("爷爷在静态代码块");
        }
    
        public Grandpa() {
            System.out.println("我是爷爷~");
        }
    }
    class Father extends Grandpa
    {
        static
        {
            System.out.println("爸爸在静态代码块");
        }
    
        public Father()
        {
            System.out.println("我是爸爸~");
        }
    }
    class Son extends Father
    {
        static 
        {
            System.out.println("儿子在静态代码块");
        }
    
        public Son()
        {
            System.out.println("我是儿子~");
        }
    }
    public class InitializationDemo
    {
        public static void main(String[] args)
        {
            new Son();  //入口
        }
    }
```

输出结果是：

    爷爷在静态代码块
    爸爸在静态代码块
    儿子在静态代码块
    我是爷爷~
    我是爸爸~
    我是儿子~


怎么样，是不是觉得这道题和上面的有所不同呢。

让我们仔细来分析一下上面代码的执行流程：

*   首先在入口这里我们实例化一个 Son 对象，因此会触发 Son 类的初始化，而 Son 类的初始化又会带动 Father 、Grandpa 类的初始化，从而执行对应类中的静态代码块。因此会输出：「爷爷在静态代码块」、「爸爸在静态代码块」、「儿子在静态代码块」。
*   当 Son 类完成初始化之后，便会调用 Son 类的构造方法，而 Son 类构造方法的调用同样会带动 Father、Grandpa 类构造方法的调用，最后会输出：「我是爷爷~」、「我是爸爸~」、「我是儿子~」。

看完了两个例子之后，相信大家都胸有成足了吧。

下面给大家看一个特殊点的例子，有点难哦！

```java
    public class Book {
        public static void main(String[] args)
        {
            staticFunction();
        }
    
        static Book book = new Book();
    
        static
        {
            System.out.println("书的静态代码块");
        }
    
        {
            System.out.println("书的普通代码块");
        }
    
        Book()
        {
            System.out.println("书的构造方法");
            System.out.println("price=" + price +",amount=" + amount);
        }
    
        public static void staticFunction(){
            System.out.println("书的静态方法");
        }
    
        int price = 110;
        static int amount = 112;
    }
```

上面这个例子的输出结果是：

    
    书的普通代码块
    书的构造方法
    price=110,amount=0
    书的静态代码块
    书的静态方法

下面我们一步步来分析一下代码的整个执行流程。

在上面两个例子中，因为 main 方法所在类并没有多余的代码，我们都直接忽略了 main 方法所在类的初始化。

但在这个例子中，main 方法所在类有许多代码，我们就并不能直接忽略了。

*   当 JVM 在准备阶段的时候，便会为类变量分配内存和进行初始化。此时，我们的 book 实例变量被初始化为 null，amount 变量被初始化为 0。
*   当进入初始化阶段后，因为 Book 方法是程序的入口，根据我们上面说到的类初始化的五种情况的第四种（当虚拟机启动时，用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机会先初始化这个主类）。所以JVM 会初始化 Book 类，即执行类构造器 。
*   JVM 对 Book 类进行初始化首先是执行类构造器（按顺序收集类中所有静态代码块和类变量赋值语句就组成了类构造器 ），后执行对象的构造器（按顺序收集成员变量赋值和普通代码块，最后收集对象构造器，最终组成对象构造器 ）。

对于 Book 类，其类构造方法（）可以简单表示如下：
```java
    static Book book = new Book();
    static
    {
        System.out.println("书的静态代码块");
    }
    static int amount = 112;
```

于是首先执行`static Book book = new Book();`这一条语句，这条语句又触发了类的实例化。于是 JVM 执行对象构造器 ，收集后的对象构造器 代码：

```java
    {
        System.out.println("书的普通代码块");
    }
    int price = 110;
    Book()
    {
        System.out.println("书的构造方法");
        System.out.println("price=" + price +", amount=" + amount);
    }
```

于是此时 price 赋予 110 的值，输出：「书的普通代码块」、「书的构造方法」。而此时 price 为 110 的值，而 amount 的赋值语句并未执行，所以只有在准备阶段赋予的零值，所以之后输出「price=110,amount=0」。

当类实例化完成之后，JVM 继续进行类构造器的初始化：
```java
    static Book book = new Book();  //完成类实例化
    static
    {
        System.out.println("书的静态代码块");
    }
    static int amount = 112;
```

即输出：「书的静态代码块」，之后对 amount 赋予 112 的值。

*   到这里，类的初始化已经完成，JVM 执行 main 方法的内容。
```java
    public static void main(String[] args)
    {
        staticFunction();
    }
```
即输出：「书的静态方法」。

方法论
---

从上面几个例子可以看出，分析一个类的执行顺序大概可以按照如下步骤：

*   **确定类变量的初始值。** 在类加载的准备阶段，JVM 会为类变量初始化零值，这时候类变量会有一个初始的零值。如果是被 final 修饰的类变量，则直接会被初始成用户想要的值。
*   **初始化入口方法。** 当进入类加载的初始化阶段后，JVM 会寻找整个 main 方法入口，从而初始化 main 方法所在的整个类。当需要对一个类进行初始化时，会首先初始化类构造器（），之后初始化对象构造器（）。
*   **初始化类构造器。** JVM 会按顺序收集类变量的赋值语句、静态代码块，最终组成类构造器由 JVM 执行。
*   **初始化对象构造器。** JVM 会按照收集成员变量的赋值语句、普通代码块，最后收集构造方法，将它们组成对象构造器，最终由 JVM 执行。

如果在初始化 main 方法所在类的时候遇到了其他类的初始化，那么就先加载对应的类，加载完成之后返回。如此反复循环，最终返回 main 方法所在类。

树义有话说
-----

看完了上面的解析之后，再去看看开头那道题是不是觉得简单多了呢。很多东西就是这样，掌握了一定的方法和知识之后，原本困难的东西也变得简单许多了。

一时没有看懂也不要灰心，毕竟我也是用了不少的时间才弄懂的。不懂的话可以多看几遍，或者加入树义的技术交流群，和小伙们一起交流。
