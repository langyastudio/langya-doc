> 本文来自JavaGuide，郎涯进行简单排版与补充



如果没有特殊说明，都是针对的是 HotSpot 虚拟机

## 写在前面

### 基本问题

- 介绍下 Java 内存区域（运行时数据区）
- Java 对象的创建过程（五步，建议能默写出来并且要知道每一步虚拟机做了什么）
- 对象的访问定位的两种方式（句柄和直接指针两种方式）



### 拓展问题

- String 类和常量池
- 8 种基本类型的包装类和常量池



## 运行时数据区域

一旦出现内存泄漏和溢出方面的问题，**如果不了解虚拟机是怎样使用内存的，那么排查错误将会是一个非常艰巨的任务**。

Java 虚拟机在执行 Java 程序的过程中会把它管理的内存划分成若干个不同的数据区域。JDK 1.8 和之前的版本略有不同，下面会介绍到。

**JDK 1.8 之前：**

![](https://img-note.langyastudio.com/202111091446197.png?x-oss-process=style/watermark)



**JDK 1.8 ：**

![](https://img-note.langyastudio.com/202111091446616.png?x-oss-process=style/watermark)

**线程私有的：**

- 虚拟机栈
- 本地方法栈
- 程序计数器

**线程共享的：**

- 堆
- 方法区
- 直接内存 (非运行时数据区的一部分)



### 程序计数器

程序计数器是一块较小的内存空间，可以看作是当前线程所执行的**字节码的行号指示器**。字节码解释器工作时通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等功能都需要依赖这个计数器来完成。

另外，为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各线程之间计数器互不影响，独立存储，我们称这类内存区域为**“线程私有”**的内存。

程序计数器主要有两个作用：

- 字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制，如：顺序执行、选择、循环、异常处理。

- 在多线程的情况下，程序计数器用于记录当前线程执行的位置，从而当线程被切换回来的时候能够知道该线程上次运行到哪儿了。

注意：**程序计数器是唯一一个不会出现 `OutOfMemoryError` 的内存区域**，它的生命周期随着线程的创建而创建，随着线程的结束而死亡。



### 虚拟机栈

与程序计数器一样，Java 虚拟机栈也是**线程私有**的，它的生命周期和线程相同，描述的是 Java **方法执行的内存模型**，每次方法调用的数据都是通过栈传递的。

Java 内存可以粗糙的区分为堆内存（Heap）和栈内存 (Stack)，其中栈就是现在说的虚拟机栈，或者说是虚拟机栈中局部变量表部分。 局部变量表主要存放了编译期可知的**各种数据类型**（boolean、byte、char、short、int、float、long、double）、**对象引用**（reference 类型，它不同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置）。



Java 虚拟机栈会出现两种错误：`StackOverFlowError` 和 `OutOfMemoryError`。

- `StackOverFlowError`

  若 Java 虚拟机栈的内存大小不允许动态扩展，那么当线程请求**栈的深度超过当前虚拟机栈的最大深度**的时候，就抛出 StackOverFlowError 错误

- `OutOfMemoryError`

  Java 虚拟机栈的内存大小可以动态扩展， 如果虚拟机在动态扩展栈时**无法申请到足够的内存**空间，则抛出`OutOfMemoryError`异常

![](https://img-note.langyastudio.com/202111091446901.png?x-oss-process=style/watermark)



**扩展：那么方法/函数如何调用？**

Java 栈可以类比数据结构中栈，Java 栈中保存的主要内容是栈帧，每一次函数调用都会有一个对应的栈帧被压入 Java 栈，每一个函数调用结束后，都会有一个栈帧被弹出。

Java 方法有两种返回方式：

- return 语句

- 抛出异常

不管哪种返回方式都会导致栈帧被弹出。



### 本地方法栈

和虚拟机栈所发挥的作用非常相似，区别是： 虚拟机栈为虚拟机执行 Java 方法 （也就是字节码）服务，而本地方法栈则为虚拟机使用到的 **Native 方法服务**。 在 HotSpot 虚拟机中和 Java 虚拟机栈合二为一。

本地方法被执行的时候，在本地方法栈也会创建一个栈帧，用于存放该本地方法的局部变量表、操作数栈、动态链接、出口信息。方法执行完毕后相应的栈帧也会出栈并释放内存空间，也会出现 `StackOverFlowError` 和 `OutOfMemoryError` 两种错误。



### 堆

Java 虚拟机所管理的内存中最大的一块，Java 堆是所有线程共享的一块内存区域，在虚拟机启动时创建。此内存区域的唯一目的就是**存放对象实例**，几乎所有的对象实例以及数组都在这里分配内存。

Java 世界中“几乎”所有的对象都在堆中分配，但是，随着 JIT 编译器的发展与逃逸分析技术逐渐成熟，栈上分配、标量替换优化技术将会导致一些微妙的变化，所有的对象都分配到堆上也渐渐变得不那么“绝对”了。从 JDK 1.7 开始已经默认开启逃逸分析，如果某些方法中的对象引用没有被返回或者未被外面使用（也就是未逃逸出去），那么对象可以直接在栈上分配内存。

Java **堆是垃圾收集器管理的主要区域**，因此也被称作 GC 堆（Garbage Collected Heap）。从垃圾回收的角度，由于现在收集器基本都采用分代垃圾收集算法，所以 Java 堆还可以细分为：新生代和老年代；再细致一点有：Eden 空间、From Survivor、To Survivor 空间等。**进一步划分的目的是更好地回收内存，或者更快地分配内存。**



在 JDK 7 版本及 JDK 7 版本之前，堆内存被通常分为下面三部分：

- 新生代内存(Young Generation)

- 老年代(Old Generation)

- 永生代(Permanent Generation)

![JVM堆内存结构-JDK7](https://img-note.langyastudio.com/202111091446780.png?x-oss-process=style/watermark)



JDK 8 版本之后方法区（HotSpot 的永久代）被彻底移除了（JDK1.7 就已经开始了），取而代之是**元空间，元空间使用的是直接内存。**

![JVM堆内存结构-JDK8](https://img-note.langyastudio.com/202111091446667.png?x-oss-process=style/watermark)

上图所示的 Eden 区、两个 Survivor 区都属于新生代（为了区分，这两个 Survivor 区域按照顺序被命名为 from 和 to），中间一层属于老年代。



大部分情况，对象都会首先在 Eden 区域分配，在一次新生代垃圾回收后，如果对象还存活，则会进入 s0 或者 s1，并且对象的年龄还会加 1(Eden 区->Survivor 区后对象的初始年龄变为 1)，当它的年龄增加到一定程度（默认为 15 岁），就会被晋升到老年代中。对象晋升到老年代的年龄阈值，可以通过参数 `-XX:MaxTenuringThreshold` 来设置。

> **🐛 修正（参见：[issue552](https://github.com/Snailclimb/JavaGuide/issues/552)）** ：“Hotspot 遍历所有对象时，按照年龄从小到大对其所占用的大小进行累积，当累积的某个年龄大小超过了 survivor 区的一半时，取这个年龄和 MaxTenuringThreshold 中更小的一个值，作为新的晋升年龄阈值”。
>
> **动态年龄计算的代码如下**
>
> ```c++
> uint ageTable::compute_tenuring_threshold(size_t survivor_capacity) {
> 	//survivor_capacity是survivor空间的大小
> size_t desired_survivor_size = (size_t)((((double) survivor_capacity)*TargetSurvivorRatio)/100);
> size_t total = 0;
> uint age = 1;
> while (age < table_size) {
> total += sizes[age];//sizes数组是每个年龄段对象大小
> if (total > desired_survivor_size) break;
> age++;
> }
> uint result = age < MaxTenuringThreshold ? age : MaxTenuringThreshold;
> 	...
> }
> ```

堆这里最容易出现的就是 OutOfMemoryError 错误，并且出现这种错误之后的表现形式还会有几种，比如：

- `java.lang.OutOfMemoryError: GC Overhead Limit Exceeded`

  当 JVM 花太多时间执行垃圾回收并且只能回收很少的堆空间时，就会发生此错误

- `java.lang.OutOfMemoryError: Java heap space`

  假如在创建新的对象时, 堆内存中的空间不足以存放新创建的对象, 就会引发此错误。(和配置的最大堆内存有关，且受制于物理内存大小。最大堆内存可通过`-Xmx` 参数配置，若没有特别配置，将会使用默认值，详见：[Default Java 8 max heap size](https://stackoverflow.com/questions/28272923/default-xmxsize-in-java-8-max-heap-size))

  ......

  

### 方法区

方法区与 Java 堆一样，是各个线程共享的内存区域，它用于**存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据**。虽然 Java 虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫做 Non-Heap（非堆），目的应该是与 Java 堆区分开来。



方法区也被称为永久代。很多人都会分不清方法区和永久代的关系，为此我也查阅了文献。

#### 方法区和永久代的关系

> 《Java 虚拟机规范》只是规定了有方法区这么个概念和它的作用，并没有规定如何去实现它。那么，在不同的 JVM 上方法区的实现肯定是不同的了。 方法区和永久代的关系很像 Java 中接口和类的关系，类实现了接口，而**永久代就是 HotSpot 虚拟机对虚拟机规范中方法区的一种实现方式**。 也就是说，永久代是 HotSpot 的概念，方法区是 Java 虚拟机规范中的定义，是一种规范，而永久代是一种实现，一个是标准一个是实现，其他的虚拟机实现并没有永久代这一说法。



#### 常用参数

JDK 1.8 之前永久代还没被彻底移除的时候通常通过下面这些参数来调节方法区大小

```java
-XX:PermSize=N //方法区 (永久代) 初始大小
-XX:MaxPermSize=N //方法区 (永久代) 最大大小,超过这个值将会抛出 OutOfMemoryError 异常:java.lang.OutOfMemoryError: PermGen
```

相对而言，垃圾收集行为在这个区域是比较少出现的，但并非数据进入方法区后就“永久存在”了。



JDK 1.8 的时候，方法区（HotSpot 的永久代）被彻底移除了（JDK1.7 就已经开始了），取而代之是元空间，元空间使用的是直接内存。

下面是一些常用参数：

```java
-XX:MetaspaceSize=N //设置 Metaspace 的初始（和最小大小）
-XX:MaxMetaspaceSize=N //设置 Metaspace 的最大大小
```

与永久代很大的不同就是，如果不指定大小的话，随着更多类的创建，虚拟机会耗尽所有可用的系统内存。



#### 为什么要将永久代 (PermGen) 替换为元空间 (MetaSpace) 

下图来自《深入理解 Java 虚拟机》第 3 版 2.2.5

![](https://img-note.langyastudio.com/202111091458796.png?x-oss-process=style/watermark)

- 整个永久代有一个 **JVM 本身设置的固定大小上限**，无法进行调整，而元空间使用的是直接内存，受本机可用内存的限制，虽然元空间仍旧可能溢出，但是比原来出现的几率会更小

  当元空间溢出时会得到如下错误： `java.lang.OutOfMemoryError: MetaSpace`

  你可以使用 `-XX：MaxMetaspaceSize` 标志设置最大元空间大小，默认值为 unlimited，这意味着它只受系统内存的限制。`-XX：MetaspaceSize` 调整标志定义元空间的初始大小如果未指定此标志，则 Metaspace 将根据运行时的应用程序需求动态地重新调整大小。

- 元空间里面存放的是类的元数据，这样加载多少类的元数据就不由 `MaxPermSize` 控制了, 而由系统的实际可用空间来控制，这样能加载的类就更多了

- 在 JDK8，合并 HotSpot 和 JRockit 的代码时, **JRockit 从来没有一个叫永久代的东西**, 合并之后就没有必要额外的设置这么一个永久代的地方了



### 运行时常量池

运行时常量池是方法区的一部分。Class 文件中除了有类的版本、字段、方法、接口等描述信息外，还有常量池表（用于存放编译期生成的各种字面量和符号引用）

既然运行时常量池是方法区的一部分，自然受到方法区内存的限制，当常量池无法再申请到内存时会抛出 OutOfMemoryError 错误。

~~**JDK1.7 及之后版本的 JVM 已经将运行时常量池从方法区中移了出来，在 Java 堆（Heap）中开辟了一块区域存放运行时常量池。**~~

> **🐛 修正（参见：[issue747](https://github.com/Snailclimb/JavaGuide/issues/747)，[reference](https://blog.csdn.net/q5706503/article/details/84640762)）** ：
>
> 1. JDK1.7 之前运行时常量池逻辑包含字符串常量池存放在方法区, 此时 hotspot 虚拟机对方法区的实现为永久代
> 2. JDK1.7 字符串常量池被从方法区拿到了堆中, 这里没有提到运行时常量池,也就是说字符串常量池被单独拿到堆,运行时常量池剩下的东西还在方法区, 也就是 hotspot 中的永久代 。
> 3. JDK1.8 hotspot 移除了永久代用元空间(Metaspace)取而代之, 这时候字符串常量池还在堆, 运行时常量池还在方法区, 只不过方法区的实现从永久代变成了元空间(Metaspace)

相关问题：JVM 常量池中存储的是对象还是引用呢？： https://www.zhihu.com/question/57109429/answer/151717241 by RednaxelaFX



### 直接内存

直接内存并不是虚拟机运行时数据区的一部分，也不是虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用。而且也可能导致 OutOfMemoryError 错误出现。

JDK1.4 中新加入的 **NIO(New Input/Output) 类**，引入了一种基于**通道（Channel）**与 **缓存区（Buffer）**的 I/O 方式，它可以直接使用 Native 函数库直接分配堆外内存，然后通过一个存储在 Java 堆中的 DirectByteBuffer 对象作为这块内存的引用进行操作。这样就能在一些场景中显著提高性能，因为**避免了在 Java 堆和 Native 堆之间来回复制数据**。

本机直接内存的分配不会受到 Java 堆的限制，但是，既然是内存就会受到本机总内存大小以及处理器寻址空间的限制。



## HotSpot 虚拟机对象探秘

通过上面的介绍我们大概知道了虚拟机的内存情况，下面我们来详细的了解一下 HotSpot 虚拟机在 Java 堆中对象分配、布局和访问的全过程。



### 对象的创建

下图便是 Java 对象的创建过程，我建议最好是能默写出来，并且要掌握每一步在做什么。
![Java创建对象的过程](https://img-note.langyastudio.com/202111091446037.png?x-oss-process=style/watermark)

#### 类加载检查

虚拟机遇到一条 new 指令时，首先将去检查这个指令的参数是否能在常量池中定位到这个类的符号引用，并且检查这个符号引用代表的类是否已被加载过、解析和初始化过。如果没有，那必须先执行相应的类加载过程。



#### 分配内存

在**类加载检查**通过后，接下来虚拟机将为新生对象**分配内存**。对象所需的内存大小在类加载完成后便可确定，为对象分配空间的任务等同于把一块确定大小的内存从 Java 堆中划分出来。**分配方式**有 **“指针碰撞”** 和 **“空闲列表”** 两种，选择哪种分配方式由 Java 堆是否规整决定，而 Java 堆是否规整又由所采用的垃圾收集器是否带有压缩整理功能决定。



**内存分配的两种方式：**

选择以上两种方式中的哪一种，取决于 Java 堆内存是否规整。而 Java 堆内存是否规整，取决于 GC 收集器的算法是"标记-清除"，还是"标记-整理"（也称作"标记-压缩"），值得注意的是，复制算法内存也是规整的

![内存分配的两种方式](https://img-note.langyastudio.com/202111091446939.png?x-oss-process=style/watermark)



**内存分配并发问题**

在创建对象的时候有一个很重要的问题，就是线程安全，因为在实际开发过程中，创建对象是很频繁的事情，作为虚拟机来说，必须要保证线程是安全的，通常来讲，虚拟机采用两种方式来保证线程安全：

- **CAS+失败重试** 

  CAS 是乐观锁的一种实现方式。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。**虚拟机采用 CAS 配上失败重试的方式保证更新操作的原子性。**

- **TLAB** 

  为每一个线程预先在 Eden 区分配一块儿内存，JVM 在给线程中的对象分配内存时，首先在 TLAB 分配，当对象大于 TLAB 中的剩余内存或 TLAB 的内存已用尽时，再采用上述的 CAS 进行内存分配



#### 初始化零值

内存分配完成后，虚拟机需要将分配到的内存空间都初始化为零值（不包括对象头），这一步操作保证了对象的实例字段在 Java 代码中可以不赋初始值就直接使用，程序能访问到这些字段的数据类型所对应的零值。



#### 设置对象头

初始化零值完成之后，**虚拟机要对对象进行必要的设置**，例如这个对象是哪个类的实例、如何才能找到类的元数据信息、对象的哈希码、对象的 GC 分代年龄等信息。 **这些信息存放在对象头中。** 另外，根据虚拟机当前运行状态的不同，如是否启用偏向锁等，对象头会有不同的设置方式。



#### 执行 init 方法

在上面工作都完成之后，从虚拟机的视角来看，一个新的对象已经产生了，但从 Java 程序的视角来看，对象创建才刚开始，`<init>` 方法还没有执行，所有的字段都还为零。所以一般来说，执行 new 指令之后会接着执行 `<init>` 方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完全产生出来。



### 对象的内存布局

在 Hotspot 虚拟机中，对象在内存中的布局可以分为 3 块区域：**对象头**、**实例数据**和**对齐填充**。

Hotspot 虚拟机的对象头包括两部分信息，第一部分用于存储对象自身的运行时数据（哈希码、GC 分代年龄、锁状态标志等等），另一部分是类型指针，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是那个类的实例。

实例数据部分是对象真正存储的有效信息，也是在程序中所定义的各种类型的字段内容。

对齐填充部分不是必然存在的，也没有什么特别的含义，仅仅起占位作用。 因为 Hotspot 虚拟机的自动内存管理系统要求对象起始地址必须是 8 字节的整数倍，换句话说就是对象的大小必须是 8 字节的整数倍。而对象头部分正好是 8 字节的倍数（1 倍或 2 倍），因此，当对象实例数据部分没有对齐时，就需要通过对齐填充来补全。



### 对象的访问定位

建立对象就是为了使用对象，我们的 Java 程序通过栈上的 reference 数据来操作堆上的具体对象。对象的访问方式由虚拟机实现而定，目前主流的访问方式有 **① 使用句柄** 和 **② 直接指针** 两种：

**句柄：**

如果使用句柄的话，那么 Java 堆中将会划分出一块内存来作为句柄池，reference 中存储的就是对象的句柄地址，而句柄中包含了对象实例数据与类型数据各自的具体地址信息；

![对象的访问定位-使用句柄](https://img-note.langyastudio.com/202111091446587.png?x-oss-process=style/watermark)



**直接指针：** 

如果使用直接指针访问，那么 Java 堆对象的布局中就必须考虑如何放置访问类型数据的相关信息，而 reference 中存储的直接就是对象的地址。

![对象的访问定位-直接指针](https://img-note.langyastudio.com/202111091446700.png?x-oss-process=style/watermark)



这两种对象访问方式各有优势。使用句柄来访问的最大好处是 reference 中存储的是稳定的句柄地址，在对象被移动时只会改变句柄中的实例数据指针，而 reference 本身不需要修改。使用直接指针访问方式最大的好处就是速度快，它**节省了一次指针定位的时间开销**。



## 重点补充内容

### 字符串常量池常见问题

我们先来看一个非常常见的面试题：**String 类型的变量和常量做“+”运算时发生了什么？** 

先来看字符串不加 `final` 关键字拼接的情况（JDK1.8）：

```java
String str1 = "str";
String str2 = "ing";
String str3 = "str" + "ing";//常量池中的对象
String str4 = str1 + str2; //在堆上创建的新的对象
String str5 = "string";//常量池中的对象
System.out.println(str3 == str4);//false
System.out.println(str3 == str5);//true
System.out.println(str4 == str5);//false
```

> **注意** ：比较 String 字符串的值是否相等，可以使用 `equals()` 方法。 `String` 中的 `equals` 方法是被重写过的。 `Object` 的 `equals` 方法是比较的对象的内存地址，而 `String` 的 `equals` 方法比较的是字符串的值是否相等。如果你使用 `==` 比较两个字符串是否相等的话，IDEA 还是提示你使用 `equals()` 方法替换。

![](https://img-note.langyastudio.com/202111091446878.png?x-oss-process=style/watermark)

> 对于基本数据类型来说，== 比较的是值。对于引用数据类型来说，==比较的是对象的内存地址。



**对于编译期可以确定值的字符串，也就是常量字符串 ，jvm 会将其存入字符串常量池。**

字符串常量池是 JVM 为了**提升性能和减少内存消耗**针为字符串（String 类）专门开辟的一块区域，主要目的是为了避免字符串的重复创建。

 ```java
 String aa = "ab"; // 放在常量池中
 String bb = "ab"; // 从常量池中查找
 System.out.println(aa==bb);// true
 ```

JDK1.7 之前运行时常量池逻辑包含字符串常量池存放在方法区。JDK1.7 的时候，字符串常量池被从方法区拿到了堆中。



并且，字符串常量拼接得到的字符串常量在编译阶段就已经被存放字符串常量池，这个得益于编译器的优化。

> 在编译过程中，Javac 编译器（下文中统称为编译器）会进行一个叫做 **常量折叠(Constant Folding)** 的代码优化。《深入理解 Java 虚拟机》中是也有介绍到：
>
> ![](https://img-note.langyastudio.com/202111091446497.png?x-oss-process=style/watermark)
>
> 常量折叠会把常量表达式的值求出来作为常量嵌在最终生成的代码中，这是 Javac 编译器会对源代码做的极少量优化措施之一(代码优化几乎都在即时编译器中进行)。
>
> 对于 `String str3 = "str" + "ing";` 编译器会给你优化成 `String str3 = "string";` 。
>
> 
>
> 并不是所有的常量都会进行折叠，只有编译器在程序编译期就可以确定值的常量才可以：
>
> 1. 基本数据类型(byte、boolean、short、char、int、float、long、double)以及字符串常量
> 2. `final` 修饰的基本数据类型和字符串变量
> 3. 字符串通过 “+”拼接得到的字符串、基本数据类型之间算数运算（加减乘除）、基本数据类型的位运算（<<、\>>、\>>> ）

因此，`str1` 、 `str2` 、 `str3` 都属于字符串常量池中的对象。

引用的值在程序编译期是无法确定的，编译器无法对其进行优化。



对象引用和“+”的字符串拼接方式，实际上是通过 `StringBuilder` 调用 `append()` 方法实现的，拼接完成之后调用 `toString()` 得到一个 `String` 对象 。

```java
String str4 = new StringBuilder().append(str1).append(str2).toString();
```

因此，`str4` 并不是字符串常量池中存在的对象，属于堆上的新对象。

我画了一个图帮助理解：

![](https://img-note.langyastudio.com/202111171116354.png?x-oss-process=style/watermark)





我们在平时写代码的时候，尽量避免多个字符串对象拼接，因为这样会重新创建对象。如果需要改变字符串的话，可以使用 `StringBuilder` 或者 `StringBuffer`。

不过，字符串使用 `final` 关键字声明之后，可以让编译器当做常量来处理。

```java
final String str1 = "str";
final String str2 = "ing";

// 下面两个表达式其实是等价的
String c = "str" + "ing";// 常量池中的对象
String d = str1 + str2; // 在堆上创建的新的的对象
System.out.println(c == d);// true
```

**被 `final` 关键字修改之后的 `String` 会被编译器当做常量来处理**，编译器在程序编译期就可以确定它的值，其效果就想到于访问常量。



如果 ，编译器在运行时才能知道其确切值的话，就无法对其优化。

示例代码如下（`str2` 在运行时才能确定其值）：

```java
final String str1 = "str";
final String str2 = getStr();
String c = "str" + "ing";// 常量池中的对象
String d = str1 + str2; // 常量池中的对象
System.out.println(c == d);// false
public static String getStr() {
      return "ing";
}
```



**我们再来看一个类似的问题！**

```java
String str1 = "abcd";
String str2 = new String("abcd");
String str3 = new String("abcd");
System.out.println(str1==str2);
System.out.println(str2==str3);
```

上面的代码运行之后会输出什么呢？

答案是：

```
false
false
```

**这是为什么呢？**

我们先来看下面这种创建字符串对象的方式：

```java
// 从字符串常量池中拿对象
String str1 = "abcd";
```

这种情况下，jvm 会先检查字符串常量池中有没有"abcd"，如果字符串常量池中没有，则创建一个，然后 str1 指向字符串常量池中的对象，如果有，则直接将 str1 指向"abcd""；

因此，`str1` 指向的是字符串常量池的对象。

我们再来看下面这种创建字符串对象的方式：

```java
// 直接在堆内存空间创建一个新的对象。
String str2 = new String("abcd");
String str3 = new String("abcd");
```



**只要使用 new 的方式创建对象，便需要创建新的对象** 。

使用 new 的方式创建对象的方式如下，可以简单概括为 3 步：

1. 在堆中创建一个字符串对象
2. 检查字符串常量池中是否有和 new 的字符串值相等的字符串常量
3. 如果没有的话需要在字符串常量池中也创建一个值相等的字符串常量，如果有的话，就直接返回堆中的字符串实例对象地址。

因此，`str2` 和 `str3` 都是在堆中新创建的对象。



**字符串常量池比较特殊，它的主要使用方法有两种：**

1. 直接使用双引号声明出来的 `String` 对象会直接存储在常量池中。
2. 如果不是用双引号声明的 `String` 对象，使用 `String` 提供的 `intern()` 方法也有同样的效果。`String.intern()` 是一个 Native 方法，它的作用是：如果运行时常量池中已经包含一个等于此 String 对象内容的字符串，则返回常量池中该字符串的引用；如果没有，JDK1.7 之前（不包含 1.7）的处理方式是在常量池中创建与此 `String` 内容相同的字符串，并返回常量池中创建的字符串的引用，JDK1.7 以及之后，字符串常量池被从方法区拿到了堆中，jvm 不会在常量池中创建该对象，而是将堆中这个对象的引用直接放到常量池中，减少不必要的内存开销。



示例代码如下（JDK 1.8） :

```java
String s1 = "Javatpoint";
String s2 = s1.intern();
String s3 = new String("Javatpoint");
String s4 = s3.intern();
System.out.println(s1==s2); // True
System.out.println(s1==s3); // False
System.out.println(s1==s4); // True
System.out.println(s2==s3); // False
System.out.println(s2==s4); // True
System.out.println(s3==s4); // False
```



**总结** ：

- 对于基本数据类型来说，比较的是值。对于引用数据类型来说，比较的是对象的内存地址。

- 在编译过程中，Javac 编译器（下文中统称为编译器）会进行一个叫做 **常量折叠(Constant Folding)** 的代码优化。常量折叠会把常量表达式的值求出来作为常量嵌在最终生成的代码中，这是 Javac 编译器会对源代码做的极少量优化措施之一(代码优化几乎都在即时编译器中进行)。

- 一般来说，我们要尽量避免通过 new 的方式创建字符串。使用双引号声明的 `String` 对象（ `String s1 = "java"` ）更利于让编译器有机会优化我们的代码，同时也更易于阅读。

- 被 `final` 关键字修改之后的 `String` 会被编译器当做常量来处理，编译器程序编译期就可以确定它的值，其效果就想到于访问常量。



### String s1 = new String("abc");创建了几个字符串对象？

会创建 1 或 2 个字符串：

- 如果字符串常量池中已存在字符串常量“abc”，则只会在堆空间创建一个字符串常量“abc”。
- 如果字符串常量池中没有字符串常量“abc”，那么它将首先在字符串常量池中创建，然后在堆空间中创建，因此将创建总共 2 个字符串对象。

**验证：**

```java
String s1 = new String("abc");// 堆内存的地址值
String s2 = "abc";
System.out.println(s1 == s2);// 输出 false,因为一个是堆内存，一个是常量池的内存，故两者是不同的。
System.out.println(s1.equals(s2));// 输出 true
```

**结果：**

```
false
true
```



### 8 种基本类型的包装类和常量池

 Java 基本类型的包装类的大部分都实现了常量池技术。

**`Byte`,`Short`,`Integer`,`Long` 这 4 种包装类默认创建了数值 [-128，127] 的相应类型的缓存数据，`Character` 创建了数值在 [0,127] 范围的缓存数据，`Boolean` 直接返回 `True` Or `False`。**

两种浮点数类型的包装类 `Float`,`Double` 并没有实现常量池技术。

```java
Integer i1 = 33;
Integer i2 = 33;
System.out.println(i1 == i2);// 输出 true
Integer i11 = 333;
Integer i22 = 333;
System.out.println(i11 == i22);// 输出 false
Double i3 = 1.2;
Double i4 = 1.2;
System.out.println(i3 == i4);// 输出 false
```



**Integer 缓存源代码：**

```java
/**
*此方法将始终缓存-128 到 127（包括端点）范围内的值，并可以缓存此范围之外的其他值。
*/
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
      return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];
}
```



**`Character` 缓存源码:**

```java
public static Character valueOf(char c) {
    if (c <= 127) { // must cache
      return CharacterCache.cache[(int)c];
    }
    return new Character(c);
}

private static class CharacterCache {
    private CharacterCache(){}

    static final Character cache[] = new Character[127 + 1];
    static {
        for (int i = 0; i < cache.length; i++)
            cache[i] = new Character((char)i);
    }
}
```



**`Boolean` 缓存源码：**

```java
public static Boolean valueOf(boolean b) {
    return (b ? TRUE : FALSE);
}
```

如果超出对应范围仍然会去创建新的对象，缓存的范围区间的大小只是在性能和资源之间的权衡。



下面我们来看一下问题。下面的代码的输出结果是 `true` 还是 `flase` 呢？

```java
Integer i1 = 40;
Integer i2 = new Integer(40);
System.out.println(i1==i2);
```

`Integer i1=40` 这一行代码会发生装箱，也就是说这行代码等价于 `Integer i1=Integer.valueOf(40)` 。因此，`i1` 直接使用的是常量池中的对象。而`Integer i1 = new Integer(40)` 会直接创建新的对象。

因此，答案是 `false` 。你答对了吗？



记住：**所有整型包装类对象之间值的比较，全部使用 equals 方法比较**。

![](https://img-note.langyastudio.com/202111171116361.png?x-oss-process=style/watermark)



**Integer 比较更丰富的一个例子:**

```java
Integer i1 = 40;
Integer i2 = 40;
Integer i3 = 0;
Integer i4 = new Integer(40);
Integer i5 = new Integer(40);
Integer i6 = new Integer(0);

System.out.println(i1 == i2);// true
System.out.println(i1 == i2 + i3);//true
System.out.println(i1 == i4);// false
System.out.println(i4 == i5);// false
System.out.println(i4 == i5 + i6);// true
System.out.println(40 == i5 + i6);// true
```

`i1` , `i2 ` , `i3` 都是常量池中的对象，`i4` , `i5` , `i6` 是堆中的对象。

 `i4 == i5 + i6` 为什么是 true 呢？因为， `i5` 和 `i6` 会进行自动拆箱操作，进行数值相加，即 `i4 == 40` 。 `Integer` 对象无法与数值进行直接比较，所以 `i4` 自动拆箱转为 int 值 40，最终这条语句转为 `40 == 40` 进行数值比较。



## 参考

- 《深入理解 Java 虚拟机：JVM 高级特性与最佳实践（第二版》
- 《实战 java 虚拟机》
- <https://docs.oracle.com/javase/specs/index.html>
- <http://www.pointsoftware.ch/en/under-the-hood-runtime-data-areas-javas-memory-model/>
- <https://dzone.com/articles/jvm-permgen-%E2%80%93-where-art-thou>
- <https://stackoverflow.com/questions/9095748/method-area-and-permgen>
- 深入解析 String#intern<https://tech.meituan.com/2014/03/06/in-depth-understanding-string-intern.html>
- R 大（RednaxelaFX）关于常量折叠的回答：https://www.zhihu.com/question/55976094/answer/147302764
