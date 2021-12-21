> 来自掘金用户：[说出你的愿望吧丷](https://juejin.im/user/5c2400afe51d45451758aa96)投稿
>
> 原文地址：https://juejin.im/post/5e1505d0f265da5d5d744050#heading-28



如果在文中用词或者理解方面出现问题，欢迎指出。**此文旨在提及而不深究**，但会尽量效率地把知识点都抛出来

## JVM的基本介绍

JVM 是 Java Virtual Machine 的缩写，它是一个虚构出来的计算机，一种规范。通过在实际的计算机上仿真模拟各类计算机功能实现···

好，其实抛开这么专业的句子不说，就知道 JVM 其实就类似于一台小电脑运行在 windows 或者 linux 这些操作系统环境下即可。**它直接和操作系统进行交互，与硬件不直接交互**，而操作系统可以帮我们完成和硬件进行交互的工作。

![](https://img-note.langyastudio.com/202111091449798.png?x-oss-process=style/watermark)

### Java文件是如何被运行的

比如我们现在写了一个 HelloWorld.java 好了，那这个 HelloWorld.java 抛开所有东西不谈，那是不是就类似于一个文本文件，只是这个文本文件它写的都是英文，而且有一定的缩进而已。

那我们的 **JVM** 是不认识文本文件的，所以它需要一个 **编译** ，让其成为一个它会读二进制文件的 **HelloWorld.class** 

#### ① 类加载器

如果 **JVM** 想要执行这个 **.class** 文件，我们需要将其装进一个 **类加载器** 中，它就像一个搬运工一样，会把所有的 **.class** 文件全部搬进 JVM 里面来。

![](https://img-note.langyastudio.com/202111091449491.png?x-oss-process=style/watermark)



#### ② 方法区

**方法区** 是用于存放类似于**元数据信息**方面的数据的，比如类信息，常量，静态变量，编译后代码···等

类加载器将 .class 文件搬过来就是先丢到这一块上



#### ③ 堆

**堆** 主要放了一些**存储的数据**，比如对象实例，数组···等，它和方法区都同属于 **线程共享区域** 。也就是说它们都是 **线程不安全** 的



#### ④ 栈

**栈** 这是我们的**代码运行空间**。我们编写的每一个方法都会放到 **栈** 里面运行。

我们会听说过 本地方法栈 或者 本地方法接口 这两个名词，不过我们基本不会涉及这两块的内容，它俩底层是使用C来进行工作的，和 Java 没有太大的关系。



#### ⑤ 程序计数器

主要就是完成一个**加载工作**，类似于一个指针一样的，指向下一行我们需要执行的代码。和栈一样，都是 **线程独享** 的，就是说每一个线程都会有自己对应的一块区域而不会存在并发和多线程的问题。

![](https://img-note.langyastudio.com/202111091449620.png?x-oss-process=style/watermark)



#### 小总结

- Java 文件经过编译后变成 .class 字节码文件

- 字节码文件通过类加载器被搬运到 JVM 虚拟机中

- 虚拟机主要的5大块：方法区，堆都为线程共享区域，有线程安全问题；栈和本地方法栈和计数器都是独享区域，不存在线程安全问题，而 **JVM 的调优主要就是围绕堆，栈两大块进行**



### 简单的代码例子

一个简单的学生类

![](https://img-note.langyastudio.com/202111091449366.png?x-oss-process=style/watermark)

一个main方法

![](https://img-note.langyastudio.com/202111091449024.png?x-oss-process=style/watermark)



执行main方法的步骤如下:

1.  编译好 App.java 后得到 App.class 后，执行 App.class，系统会启动一个 JVM 进程，从 classpath 路径中找到一个名为 App.class 的二进制文件，将 App 的类信息加载到**运行时数据区**的方法区内，这个过程叫做 App 类的加载
2.  JVM 找到 App 的主程序入口，执行 main 方法
3.  这个 main 中的第一条语句为 Student student = new Student("tellUrDream") ，就是让 JVM 创建一个 Student 对象，但是这个时候方法区中是没有 Student 类的信息的，所以 JVM 马上加载 Student 类，把 Student 类的信息放到方法区中
4.  加载完 Student 类后，JVM 在堆中为一个新的 Student 实例分配内存，然后调用构造函数初始化 Student 实例，这个 Student 实例持有 **指向方法区中的 Student 类的类型信息** 的引用
5.  执行 student.sayName(); 时，JVM 根据 student 的引用找到 student 对象，然后根据 student 对象持有的引用定位到方法区中 student 类的类型信息的方法表，获得 sayName() 的字节码地址。
6.  执行sayName()

其实也不用管太多，只需要知道对象实例初始化时会去方法区中找类信息，完成后再到栈那里去运行方法。找方法就在方法表中找。



## 类加载器

之前也提到了它是负责加载 .class 文件的，它们在文件开头会有特定的文件标示，将 class 文件字节码内容加载到内存中，并将这些内容转换成方法区中的运行时数据结构，并且 **ClassLoader 只负责 class 文件的加载，而是否能够运行则由 Execution Engine 来决定**



### 类加载器的流程

从类被加载到虚拟机内存中开始，到释放内存总共有7个步骤：加载、验证&准备&解析、初始化、使用、卸载。其中**验证，准备，解析三个部分统称为链接**

#### 加载

1.  将 class 文件加载到内存
2.  将静态数据结构转化成方法区中运行时的数据结构
3.  在堆中生成一个代表这个类的 java.lang.Class 对象作为数据访问的入口



#### 链接

1.  验证：确保加载的类符合 JVM 规范和安全，保证被校验类的方法在运行时不会做出危害虚拟机的事件，其实就是一个安全检查
2.  准备：为 static 变量在方法区中分配内存空间，设置变量的初始值，例如 static int a = 3 （注意：准备阶段只设置类中的静态变量（方法区中），不包括实例变量（堆内存中），实例变量是对象初始化时赋值的）
3.  解析：虚拟机将常量池内的符号引用替换为直接引用的过程（符号引用比如我现在 import java.util.ArrayList 这就算符号引用，直接引用就是指针或者对象地址，注意引用对象一定是在内存进行）



#### 初始化

初始化其实就是执行类构造器方法的 `<clinit>()` 的过程，而且要保证执行前父类的 `<clinit>()` 方法执行完毕。这个方法由编译器收集，顺序执行所有类变量（static 修饰的成员变量）显式初始化和静态代码块中语句。此时准备阶段时的那个 `static int a` 由默认初始化的 0 变成了显式初始化的 3。 由于执行顺序缘故，初始化阶段类变量如果在静态代码块中又进行了更改，会覆盖类变量的显式初始化，最终值会为静态代码块中的赋值。
>注意：字节码文件中初始化方法有两种，非静态资源初始化的 `<init>` 和静态资源初始化的 `<clinit>` ，类构造器方法 `<clinit>()` 不同于类的构造器，这些方法都是字节码文件中只能给 JVM 识别的特殊方法。



#### 卸载

GC 将无用对象从内存中卸载



### 类加载器的加载顺序

加载一个 Class 类的顺序也是有优先级的，类加载器从最底层开始往上的顺序是这样的

1.  BootStrap ClassLoader：rt.jar
2.  Extension ClassLoader: 加载扩展的jar包
3.  App ClassLoader：指定的 classpath 下面的jar包
4.  Custom ClassLoader：自定义的类加载器



### 双亲委派机制

**当一个类收到了加载请求时，它是不会先自己去尝试加载的，而是委派给父类去完成**，比如我现在要 new 一个 Person，这个 Person 是我们自定义的类，如果我们要加载它，就会先委派 App ClassLoader ，只有当父类加载器都反馈自己无法完成这个请求（也就是父类加载器都没有找到加载所需的 Class）时，子类加载器才会自行尝试加载。

这样做的好处是，加载位于 rt.jar 包中的类时不管是哪个加载器加载，最终都会委托到 BootStrap ClassLoader 进行加载，这样保证了使用不同的类加载器得到的都是同一个结果。

其实这个也是一个**隔离**的作用，避免了我们的代码影响了 JDK 的代码，比如我现在自己定义一个 `java.lang.String` ：

```java
package java.lang;
public class String {
    public static void main(String[] args) {
        System.out.println();
    }
}
```

尝试运行当前类的 `main` 函数的时候，我们的代码肯定会报错。这是因为在加载的时候其实是找到了 rt.jar 中的`java.lang.String`，然而发现这个里面并没有 `main` 方法。



## 运行时数据区

### 本地方法栈

比如说我们现在点开 Thread 类的源码，会看到它的 start0 方法带有一个 native 关键字修饰，而且不存在方法体，这种用 **native 修饰的方法就是本地方法**，这是使用 C 来实现的，然后一般这些方法都会放到一个叫做本地方法栈的区域。



### 程序计数器

程序计数器其实就是一个指针，它指向了我们程序中下一句需要执行的指令，它也是内存区域中唯一一个不会出现OutOfMemoryError 的区域，而且占用内存空间小到基本可以忽略不计。这个内存仅代表当前线程所执行的字节码的行号指示器，字节码解析器通过改变这个计数器的值选取下一条需要执行的字节码指令。

如果执行的是 native 方法，那这个指针就不工作了。



### 方法区

方法区主要的作用是存放类的元数据信息，常量和静态变量···等。当它存储的信息过大时，会在无法满足内存分配时报错。



### 栈 vs 堆

一句话便是：**栈管运行，堆管存储**。虚拟机栈负责运行代码，而虚拟机堆负责存储数据。

#### 虚拟机栈的概念

它是 Java 方法执行的内存模型。里面会对**局部变量，动态链表，方法出口，栈的操作（入栈和出栈）进行存储**，且线程独享。同时如果我们听到局部变量表，那也是在说虚拟机栈

```java
public class Person{
    int a = 1;
    
    public void doSomething(){
        int b = 2;
    }
}
```



#### 栈存在的异常

如果线程请求的栈的深度大于虚拟机栈的最大深度，就会报 **StackOverflowError** （这种错误经常出现在递归中）。Java虚拟机也可以动态扩展，但随着扩展会不断地申请内存，当无法申请足够内存时就会报错 **OutOfMemoryError**。



#### 栈的生命周期

对于栈来说，不存在垃圾回收。**只要程序运行结束，栈的空间自然就会释放了**。栈的生命周期和所处的线程是一致的。

这里补充一句：8种基本类型的变量+对象的引用变量+实例方法都是在栈里面分配内存。



#### 虚拟机栈的执行

我们经常说的栈帧数据，说白了在 JVM 中叫栈帧，放到 Java 中其实就是方法，它也是存放在栈中的。

栈中的数据都是以栈帧的格式存在，它是一个关于方法和运行期数据的数据集。比如我们执行一个方法 a，就会对应产生一个栈帧 A1，然后 A1 会被压入栈中。同理方法 b 会有一个 B1，方法 c 会有一个 C1，等到这个线程执行完毕后，栈会先弹出 C1，后 B1,A1。它是一个先进后出，后进先出原则。



#### 局部变量的复用

局部变量表用于存放方法参数和方法内部所定义的局部变量。它的容量是以 Slot 为最小单位，一个 slot 可以存放 32 位以内的数据类型。

**虚拟机通过索引定位的方式使用局部变量表**，范围为[0, 局部变量表的slot的数量]。方法中的参数就会按一定顺序排列在这个局部变量表中，至于怎么排的我们可以先不关心。而为了节省栈帧空间，这些 slot 是可以复用的，当方法执行位置超过了某个变量，那么这个变量的 slot 可以被其它变量复用。当然如果需要复用，那我们的垃圾回收自然就不会去动这些内存。



#### 堆的概念

JVM 内存会划分为堆内存和非堆内存，堆内存中也会划分为**年轻代**和**老年代**，而非堆内存则为**永久代**。年轻代又会分为**Eden** 和 **Survivor** 区。Survivor 也会分为 **FromPlace** 和 **ToPlace**，toPlace 的 survivor 区域是空的。Eden，FromPlace 和 ToPlace 的默认占比为 **8:1:1**。当然这个东西其实也可以通过一个 -XX:+UsePSAdaptiveSurvivorSizePolicy 参数来根据生成对象的速率动态调整

**堆内存中存放的是对象，垃圾收集就是收集这些对象然后交给 GC 算法进行回收**。非堆内存其实我们已经说过了，就是方法区。在 1.8 中已经移除永久代，替代品是一个元空间(MetaSpace)，最大区别是 metaSpace 是不存在于 JVM 中的，它使用的是本地内存。并有两个参数

```ini
MetaspaceSize：初始化元空间大小，控制发生GC
MaxMetaspaceSize：限制元空间大小上限，防止占用过多物理内存。
```

移除的原因可以大致了解一下：融合 HotSpot JVM 和 JRockit VM 而做出的改变，因为 JRockit 是没有永久代的，不过这也间接性地解决了永久代的 OOM 问题。



#### Eden 年轻代

当我们 new 一个对象后，会先放到 Eden 划分出来的一块作为存储空间的内存，但是我们知道对堆内存是线程共享的，所以有可能会出现两个对象共用一个内存的情况。这里 JVM 的处理是每个线程都会预先申请好一块连续的内存空间并规定了对象存放的位置，而如果空间不足会再申请多块内存空间。这个操作我们会称作 TLAB，有兴趣可以了解一下。

当 Eden 空间满了之后，会触发一个叫做 Minor GC（就是一个发生在年轻代的GC）的操作，存活下来的对象移动到Survivor0 区。Survivor0 区满后触发 Minor GC，就会将存活对象移动到 Survivor1 区，此时还会把 from 和 to 两个指针交换，这样保证了一段时间内总有一个 survivor 区为空且 to 所指向的 survivor 区为空。经过多次的 Minor GC 后仍然存活的对象（**这里的存活判断是15次，对应到虚拟机参数为 -XX:MaxTenuringThreshold 。为什么是15，因为HotSpot会在对象投中的标记字段里记录年龄，分配到的空间仅有4位，所以最多只能记录到15**）会移动到老年代。老年代是存储长期存活的对象的，占满时就会触发我们最常听说的 Full GC，期间会停止所有线程等待 GC 的完成。所以对于响应要求高的应用应该尽量去减少发生 Full GC 从而避免响应超时的问题。

而且当老年区执行了 full gc 之后仍然无法进行对象保存的操作，就会产生 OOM，这时候就是虚拟机中的堆内存不足，原因可能会是堆内存设置的大小过小，这个可以通过参数 **-Xms、-Xmx** 来调整。也可能是代码中创建的对象大且多，而且它们一直在被引用从而长时间垃圾收集无法收集它们。

![](https://img-note.langyastudio.com/202111091449071.png?x-oss-process=style/watermark)

**补充说明：**

关于 -XX:TargetSurvivorRatio 参数的问题。其实也不一定是要满足 -XX:MaxTenuringThreshold 才移动到老年代。可以举个例子：如对象年龄5的占30%，年龄6的占36%，年龄7的占34%，加入某个年龄段（如例子中的年龄6）后，总占用超过 Survivor空间*TargetSurvivorRatio 的时候，从该年龄段开始及大于的年龄对象就要进入老年代（即例子中的年龄6对象，就是年龄6和年龄7晋升到老年代），这时候无需等到 MaxTenuringThreshold 中要求的15



#### 如何判断一个对象需要被干掉

![](https://img-note.langyastudio.com/202111091450730.png?x-oss-process=style/watermark)



垃圾收集器所关注的都是堆和方法这部分内存。在进行回收前就要判断哪些对象还存活，哪些已经死去。下面介绍两个基础的计算方法

1.**引用计数器计算**

给对象添加一个引用计数器，每次引用这个对象时计数器加一，引用失效时减一，计数器等于 0 时就是不会再次使用的。不过这个方法有一种情况就是出现对象的循环引用时 GC 没法回收。

2.**可达性分析计算**

这是一种类似于二叉树的实现，将一系列的 GC ROOTS 作为起始的存活对象集，从这个节点往下搜索，搜索所走过的路径成为引用链，把能被该集合引用到的对象加入到集合中。搜索当一个对象到 GC Roots 没有使用任何引用链时，则说明该对象是不可用的。主流的商用程序语言，例如 Java，C# 等都是靠这招去判定对象是否存活的。

在 Java 语言汇总能作为 GC Roots 的对象分为以下几种（了解一下即可）：

1.  虚拟机栈（栈帧中的本地方法表）中引用的对象（局部变量）
2.  方法区中静态变量所引用的对象（静态变量）
3.  方法区中常量引用的对象
4.  本地方法栈（即 native 修饰的方法）中 JNI 引用的对象（JNI 是 Java 虚拟机调用对应的 C 函数的方式，通过 JNI 函数也可以创建新的 Java 对象。且 JNI 对于对象的局部引用或者全局引用都会把它们指向的对象都标记为不可回收）
5.  已启动的且未终止的 Java 线程


这种方法的优点是能够解决循环引用的问题，可它的实现需要耗费大量资源和时间，也需要 GC（它的分析过程引用关系不能发生变化，所以需要停止所有进程）



#### 如何宣告一个对象的真正死亡

首先必须要提到的是一个名叫 **finalize()** 的方法。finalize() 是 Object 类的一个方法、一个对象的 finalize() 方法只会被系统自动调用一次，经过 finalize() 方法逃脱死亡的对象，第二次不会再调用。

补充一句：并不提倡在程序中调用 finalize() 来进行自救。建议忘掉 Java 程序中该方法的存在。因为它执行的时间不确定，甚至是否被执行也不确定（Java 程序的不正常退出），而且运行代价高昂，无法保证各个对象的调用顺序（甚至有不同线程中调用）。在 Java9 中已经被标记为 **deprecated** ，且 java.lang.ref.Cleaner（也就是强、软、弱、幻象引用的那一套）中已经逐步替换掉它，会比 finalize 来的更加的轻量及可靠。
　　
![](https://img-note.langyastudio.com/202111091450104.png?x-oss-process=style/watermark)



判断一个对象的死亡至少需要两次标记

1.  如果对象进行可达性分析之后没发现与 GC Roots 相连的引用链，那它将会第一次标记并且进行一次筛选。判断的条件是决定这个对象是否有必要执行 finalize() 方法。如果对象有必要执行 finalize() 方法，则被放入 **F-Queue** 队列中。
2. GC 对 F-Queue 队列中的对象进行二次标记。如果对象在 finalize() 方法中重新与引用链上的任何一个对象建立了关联，那么二次标记时则会将它移出“即将回收”集合。如果此时对象还没成功逃脱，那么只能被回收了。

如果确定对象已经死亡，我们又该如何回收这些垃圾呢



### 垃圾回收算法

不会非常详细的展开，常用的有标记清除，复制，标记整理和分代收集算法



#### 标记清除算法

标记清除算法就是分为“标记”和“清除”两个阶段。标记出所有需要回收的对象，标记结束后统一回收。这个套路很简单，也存在不足，后续的算法都是根据这个基础来加以改进的。

其实它就是把已死亡的对象标记为空闲内存，然后记录在一个空闲列表中，当我们需要new一个对象时，内存管理模块会从空闲列表中寻找空闲的内存来分给新的对象。

不足的方面就是标记和清除的**效率比较低下**。且这种做法会让**内存中的碎片**非常多。这个导致了如果我们需要使用到较大的内存块时，无法分配到足够的连续内存。比如下图

![](https://img-note.langyastudio.com/202111091450409.png?x-oss-process=style/watermark)

此时可使用的内存块都是零零散散的，导致了刚刚提到的大内存对象问题



#### 复制算法

为了解决效率问题，复制算法就出现了。它将可用内存按容量划分成两等分，每次只使用其中的一块。和 survivor 一样也是用 from 和 to 两个指针这样的玩法。fromPlace 存满了，就把存活的对象 copy 到另一块 toPlace 上，然后交换指针的内容。这样就解决了碎片的问题。

这个算法的代价就是把内存缩水了，这样堆内存的使用效率就会变得十分低下了

![](https://img-note.langyastudio.com/202111091450199.png?x-oss-process=style/watermark)

不过它们分配的时候也不是按照 1:1 这样进行分配的，就类似于 Eden 和 Survivor 也不是等价分配是一个道理。



#### 标记整理算法

复制算法在对象存活率高的时候会有一定的效率问题，标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，然后直接清理掉边界以外的内存

![](https://img-note.langyastudio.com/202111091452211.png?x-oss-process=style/watermark)



#### 分代收集算法

这种算法并没有什么新的思想，只是**根据对象存活周期的不同将内存划分为几块**。一般是把 Java 堆分为新生代和老年代，这样就可以根据各个年代的特点采用最适当的收集算法。在新生代中，每次垃圾收集时都发现有大批对象死去，只有少量存活，那就选用复制算法，只需要付出少量存活对象的复制成本就可以完成收集。而老年代中因为对象存活率高、没有额外空间对它进行分配担保，就必须使用“标记-清理”或者“标记-整理”算法来进行回收。

说白了就是八仙过海各显神通，具体问题具体分析了而已。



### 各种各样的垃圾回收器（了解）

HotSpot VM 中的垃圾回收器，以及适用场景

![](https://img-note.langyastudio.com/202111091452914.png?x-oss-process=style/watermark)

到 jdk8 为止，默认的垃圾收集器是 Parallel Scavenge  和 Parallel Old。

从 jdk9 开始，**G1 收集器**成为默认的垃圾收集器。目前来看，G1 回收器停顿时间最短而且没有明显缺点，非常适合 Web应用。在 jdk8 中测试 Web 应用，堆内存 6G，新生代 4.5G 的情况下，Parallel Scavenge 回收新生代停顿长达 1.5 秒。G1 回收器回收同样大小的新生代只停顿 0.2 秒。



### JVM的常用参数（了解）

JVM 的参数非常之多，这里只列举比较重要的几个，通过各种各样的搜索引擎也可以得知这些信息。

| 参数名称 | 含义 | 默认值 | 说明 |
|------|------------|------------|------|
| -Xms  | 初始堆大小          | 物理内存的1/64(<1GB)         |默认(MinHeapFreeRatio 参数可以调整) 空余堆内存小于 40% 时，JVM 就会增大堆直到-Xmx的最大限制.|
| -Xmx  | 最大堆大小        | 物理内存的1/4(<1GB)        | 默认( MaxHeapFreeRatio 参数可以调整) 空余堆内存大于 70% 时，JVM会减少堆直到 -Xms 的最小限制 |
| -Xmn  | 年轻代大小(1.4 or later)     |        |注意：此处的大小是（eden+ 2 survivor space)与jmap -heap中显示的New gen是不同的。整个堆大小=年轻代大小 + 老年代大小 + 持久代（永久代）大小.增大年轻代后,将会减小年老代大小.此值**对系统性能影响较大**,Sun官方推荐配置为整个堆的 3/8|
| -XX:NewSize  | 年轻代大小(for 1.3/1.4)          |          ||
| -XX:MaxNewSize  | 年轻代最大值(for 1.3/1.4)        |         ||
| -XX:PermSize  | 设置持久代(perm gen)初始值     | 物理内存的1/64       ||
| -XX:MaxPermSize  | 设置持久代最大值          | 物理内存的1/4         ||
| -Xss  | 每个线程的栈大小        |         | JDK5.0 以后每个线程堆栈大小为1M,以前每个线程堆栈大小为256K.根据应用的线程所需内存大小进行调整.在相同物理内存下,减小这个值能生成更多的线程.但是操作系统对一个进程内的线程数还是有限制的,不能无限生成,经验值在3000~5000左右一般小的应用，如果栈不是很深， 应该是 128k 够用的 大的应用建议使用256k。**这个选项对性能影响比较大**，需要严格的测试。（校长）和 threadstacksize 选项解释很类似,官方文档似乎没有解释,在论坛中有这样一句话:-Xss is translated in a VM flag named ThreadStackSize”一般设置这个值就可以了 |
| -XX:NewRatio  | 年轻代(包括Eden和两个Survivor区)与年老代的比值(除去持久代)       |        |-XX:NewRatio=4表示年轻代与年老代所占比值为1:4,年轻代占整个堆栈的1/5Xms=Xmx并且设置了Xmn的情况下，该参数不需要进行设置。|
| -XX:SurvivorRatio  | Eden区与Survivor区的大小比值          |          |设置为8, 则两个Survivor区与一个Eden区的比值为2:8, 一个Survivor区占整个年轻代的1/10|
| -XX:+DisableExplicitGC  | 关闭System.gc()        |         |这个参数需要严格的测试|
| -XX:PretenureSizeThreshold  | 对象超过多大是直接在旧生代分配       | 0      |单位字节 新生代采用Parallel ScavengeGC时无效另一种直接在旧生代分配的情况是大的数组对象,且数组中无外部引用对象.|
| -XX:ParallelGCThreads  | 并行收集器的线程数         |          |此值最好配置与处理器数目相等，同样适用于CMS|
| -XX:MaxGCPauseMillis  | 每次年轻代垃圾回收的最长时间(最大暂停时间)        |         |如果无法满足此时间,JVM会自动调整年轻代大小,以满足此值.|

其实还有一些打印及 CMS 方面的参数，这里就不以一一列举了



## 关于JVM调优

根据刚刚涉及的 jvm 的知识点，我们可以尝试对 JVM 进行调优，主要就是堆内存那块



### 调整最大堆内存和最小堆内存

-Xmx –Xms：指定 java 堆最大值（默认值是物理内存的1/4(<1GB)）和初始 java 堆最小值（默认值是物理内存的1/64(<1GB))

默认( MinHeapFreeRatio 参数可以调整) 空余堆内存小于 40% 时，JVM 就会增大堆直到 -Xmx 的最大限制，默认( MaxHeapFreeRatio 参数可以调整) 空余堆内存大于 70% 时，JVM 会减少堆直到 -Xms 的最小限制。简单点来说，你不停地往堆内存里面丢数据，等它剩余大小小于 40% 了，JVM 就会动态申请内存空间不过会小于 -Xmx，如果剩余大小大于70%，又会动态缩小不过不会小于 –Xms。就这么简单

开发过程中，通常会将 -Xms 与 -Xmx 两个参数配置成相同的值，其目的是为了能够在 java 垃圾回收机制清理完堆区后不需要重新分隔计算堆区的大小而浪费资源。

我们执行下面的代码

```java
System.out.println("Xmx=" + Runtime.getRuntime().maxMemory() / 1024.0 / 1024 + "M");    //系统的最大空间
System.out.println("free mem=" + Runtime.getRuntime().freeMemory() / 1024.0 / 1024 + "M");  //系统的空闲空间
System.out.println("total mem=" + Runtime.getRuntime().totalMemory() / 1024.0 / 1024 + "M");  //当前可用的总空间
```

注意：此处设置的是 Java 堆大小，也就是新生代大小 + 老年代大小

![](https://img-note.langyastudio.com/202111091451599.png?x-oss-process=style/watermark)



设置一个 VM options 的参数

```shell
-Xmx20m -Xms5m -XX:+PrintGCDetails
```

![](https://img-note.langyastudio.com/202111091451083.png?x-oss-process=style/watermark)



再次启动main方法

![](https://img-note.langyastudio.com/202111091451900.png?x-oss-process=style/watermark)

这里 GC 弹出了一个 Allocation Failure 分配失败，这个事情发生在 PSYoungGen，也就是年轻代中

这时候申请到的内存为 18M，空闲内存为 4.214195251464844M



我们此时创建一个字节数组看看，执行下面的代码

```java
byte[] b = new byte[1 * 1024 * 1024];
System.out.println("分配了1M空间给数组");
System.out.println("Xmx=" + Runtime.getRuntime().maxMemory() / 1024.0 / 1024 + "M");  //系统的最大空间
System.out.println("free mem=" + Runtime.getRuntime().freeMemory() / 1024.0 / 1024 + "M");  //系统的空闲空间
System.out.println("total mem=" + Runtime.getRuntime().totalMemory() / 1024.0 / 1024 + "M");
```


![](https://img-note.langyastudio.com/202111091451938.png?x-oss-process=style/watermark)

此时 free memory 就又缩水了，不过 total memory 是没有变化的。Java 会尽可能将 total mem 的值维持在最小堆内存大小




```java
byte[] b = new byte[10 * 1024 * 1024];
System.out.println("分配了10M空间给数组");
System.out.println("Xmx=" + Runtime.getRuntime().maxMemory() / 1024.0 / 1024 + "M");  //系统的最大空间
System.out.println("free mem=" + Runtime.getRuntime().freeMemory() / 1024.0 / 1024 + "M");  //系统的空闲空间
System.out.println("total mem=" + Runtime.getRuntime().totalMemory() / 1024.0 / 1024 + "M");  //当前可用的总空间
```

![](https://img-note.langyastudio.com/202111091451836.png?x-oss-process=style/watermark)

这时候我们创建了一个 10M 的字节数据，这时候最小堆内存是顶不住的。我们会发现现在的 total memory 已经变成了15M，这就是已经申请了一次内存的结果。



此时我们再跑一下这个代码

```java
System.gc();
System.out.println("Xmx=" + Runtime.getRuntime().maxMemory() / 1024.0 / 1024 + "M");    //系统的最大空间
System.out.println("free mem=" + Runtime.getRuntime().freeMemory() / 1024.0 / 1024 + "M");  //系统的空闲空间
System.out.println("total mem=" + Runtime.getRuntime().totalMemory() / 1024.0 / 1024 + "M");  //当前可用的总空间
```

![](https://img-note.langyastudio.com/202111091451068.png?x-oss-process=style/watermark)
    
此时我们手动执行了一次 fullgc，此时 total memory 的内存空间又变回 5.5M 了，此时又是把申请的内存释放掉的结果。



### 调整新生代和老年代的比值

-XX:NewRatio --- 新生代（eden+2*Survivor）和老年代（不包含永久区）的比值

例如：-XX:NewRatio=4，表示新生代:老年代=1:4，即新生代占整个堆的1/5。在Xms=Xmx并且设置了Xmn的情况下，该参数不需要进行设置。



### 调整Survivor区和Eden区的比值

-XX:SurvivorRatio（幸存代）--- 设置两个Survivor区和eden的比值

例如：8，表示两个Survivor:eden=2:8，即一个Survivor占年轻代的1/10



### 设置年轻代和老年代的大小

-XX:NewSize --- 设置年轻代大小

-XX:MaxNewSize --- 设置年轻代最大值

可以通过设置不同参数来测试不同的情况，反正最优解当然就是官方的 Eden 和 Survivor 的占比为 8:1:1，然后在刚刚介绍这些参数的时候都已经附带了一些说明，感兴趣的也可以看看。反正最大堆内存和最小堆内存如果数值不同会导致多次的 gc，需要注意。



### 小总结

根据实际事情调整新生代和老年代的大小，官方推荐新生代占 java 堆的3/8，老年代占新生代的1/10

在 OOM 时，记得 Dump 出堆，确保可以排查现场问题，通过下面命令你可以输出一个 .dump 文件，这个文件可以使用VisualVM 或者 Java 自带的 Java VisualVM 工具。

```shell
-Xmx20m -Xms5m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=你要输出的日志路径
```

一般我们也可以通过编写脚本的方式来让 OOM 出现时给我们报个信，可以通过发送邮件或者重启程序等来解决。



### 永久区的设置

```shell
-XX:PermSize -XX:MaxPermSize
```

初始空间（默认为物理内存的1/64）和最大空间（默认为物理内存的1/4）。也就是说，jvm 启动时，永久区一开始就占用了 PermSize 大小的空间，如果空间还不够，可以继续扩展，但是不能超过 MaxPermSize，否则会 OOM。

tips：如果堆空间没有用完也抛出了 OOM，有可能是永久区导致的。堆空间实际占用非常少，但是永久区溢出 一样抛出OOM。



### JVM的栈参数调优

#### 调整每个线程栈空间的大小

可以通过 -Xss：调整每个线程栈空间的大小

JDK5.0 以后每个线程堆栈大小为1M，以前每个线程堆栈大小为256K。在相同物理内存下,减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000~5000左右



#### 设置线程栈的大小

```shell
-XXThreadStackSize：
设置线程栈的大小(0 means use default stack size)
```

这些参数都是可以通过自己编写程序去简单测试的，这里碍于篇幅问题就不再提供demo了



### JVM其他参数介绍(可以直接跳过了)

形形色色的参数很多，就不会说把所有都扯个遍了，因为大家其实也不会说一定要去深究到底。

#### 设置内存页的大小

```shell
-XXThreadStackSize：
    设置内存页的大小，不可设置过大，会影响Perm的大小
```



#### 设置原始类型的快速优化

```shell
-XX:+UseFastAccessorMethods：
    设置原始类型的快速优化
```



#### 设置关闭手动GC

```shell
-XX:+DisableExplicitGC：
    设置关闭System.gc()(这个参数需要严格的测试)
```



#### 设置垃圾最大年龄

```shell
-XX:MaxTenuringThreshold
    设置垃圾最大年龄。如果设置为0的话,则年轻代对象不经过Survivor区,直接进入年老代.
    对于年老代比较多的应用,可以提高效率。如果将此值设置为一个较大值,
    则年轻代对象会在Survivor区进行多次复制,这样可以增加对象再年轻代的存活时间,
    增加在年轻代即被回收的概率。该参数只有在串行GC时才有效.
```



#### 加快编译速度

```shell
-XX:+AggressiveOpts
```
加快编译速度



#### 改善锁机制性能

```shell
-XX:+UseBiasedLocking
```



#### 禁用垃圾回收

```shell
-Xnoclassgc
```



#### 设置堆空间存活时间

```shell
-XX:SoftRefLRUPolicyMSPerMB
    设置每兆堆空闲空间中SoftReference的存活时间，默认值是1s。
```



#### 设置对象直接分配在老年代

```shell
-XX:PretenureSizeThreshold
    设置对象超过多大时直接在老年代分配，默认值是0。
```



#### 设置TLAB占eden区的比例

```shell
-XX:TLABWasteTargetPercent
    设置TLAB占eden区的百分比，默认值是1% 。 
```



#### 设置是否优先YGC

```shell
-XX:+CollectGen0First
    设置 FullGC 时是否先 YGC，默认值是 false。
```



## finally

真的扯了很久这东西，参考了多方的资料，有极客时间的《深入拆解虚拟机》和《Java核心技术面试精讲》，也有百度，也有自己在学习的一些线上课程的总结。希望对你有所帮助，谢谢。
