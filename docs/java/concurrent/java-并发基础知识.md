> 本文来自JavaGuide、廖雪峰，郎涯进行简单排版与补充



CPU 执行代码都是一条一条顺序执行的，但是即使是单核 cpu，也可以同时运行多个任务。因为操作系统执行多任务实际上就是让 CPU 对多个任务轮流交替执行。例如，让浏览器执行 0.001 秒，让 QQ 执行 0.001 秒，再让音乐播放器执行0.001 秒，在人看来，CPU 就是在同时执行多个任务。

即使是多核 CPU，因为通常任务的数量远远多于 CPU 的核数，所以任务也是交替执行的。多任务既可以由多进程实现，也可以由单进程内的多线程实现，还可以混合多进程＋多线程。



## 线程 vs 进程

一个进程可以包含一个或多个线程。

### 进程

进程是是操作系统运行程序的基本单位。在 Java 中，当我们启动 main 函数时其实就是启动了一个 JVM 的进程，而 main 函数所在的线程就是这个进程中的一个线程，也称主线程。



### 线程

线程是进程划分成的更小的运行单位。Java 语言内置了多线程支持：在`main()` 方法内部，可以启动多个线程。如负责垃圾回收的工作线程等。



### 优缺点

- 多进程的优点：

  多进程基本上是独立的，稳定性比多线程高，因为在多进程的情况下，一个进程崩溃不会影响其他进程

  同进程的多线程有可能相互影响，任何一个线程崩溃会直接导致整个进程崩溃

- 多进程的缺点：

  创建进程比创建线程开销大，尤其是在 Windows 系统上

  进程间通信比线程间通信要慢，因为线程间通信就是读写同一个变量，速度很快



### **JVM 层面分析进程 vs 线程**

#### 图解进程和线程的关系

下图是 Java 内存区域，通过下图我们从 JVM 的角度来说一下线程和进程之间的关系

![](https://img-note.langyastudio.com/202110191146474.png?x-oss-process=style/watermark)


从上图可以看出：一个进程中可以有多个线程，多个线程共享进程的**堆**和**方法区 (JDK1.8 之后的元空间)**资源，但是每个线程有自己的**程序计数器**、**虚拟机栈** 和 **本地方法栈**。



#### 程序计数器为什么是私有的

程序计数器主要有下面两个作用：

- 字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制，如：顺序执行、选择、循环、异常处理

- 在多线程的情况下，**程序计数器用于记录当前线程执行的位置**，从而当线程被切换回来的时候能够知道该线程上次运行到哪儿了

需要注意的是，如果执行的是 native 方法，那么程序计数器记录的是 undefined 地址，只有执行的是 Java 代码时程序计数器记录的才是下一条指令的地址。

所以，程序计数器私有主要是为了**线程切换后能恢复到正确的执行位置**。



#### 虚拟机栈和本地方法栈为什么是私有的

- **虚拟机栈：**

  每个 Java 方法在执行的同时会创建一个栈帧用于存储局部变量表、操作数栈、常量池引用等信息。从方法调用直至执行完成的过程，就对应着一个栈帧在 Java 虚拟机栈中入栈和出栈的过程。

- **本地方法栈：**

  和虚拟机栈所发挥的作用非常相似，区别是： **虚拟机栈为虚拟机执行 Java 方法 （也就是字节码）服务，而本地方法栈则为虚拟机使用到的 Native 方法服务。** 在 HotSpot 虚拟机中和 Java 虚拟机栈合二为一。

所以，为了**保证线程中的局部变量不被别的线程访问到**，虚拟机栈和本地方法栈是线程私有的。



#### 堆和方法区

堆和方法区是所有线程共享的资源

- 堆是进程中最大的一块内存，主要用于**存放新创建的对象** (几乎所有对象都在这里分配内存)

- 方法区主要用于存放已被加载的类信息、常量、静态变量、即时编译器编译后的代码等**数据**



## 并发 vs 并行

- 并发

  同一时间段，多个任务都在执行 (单位时间内不一定同时执行)

- 并行

  单位时间内，多个任务同时执行



## 为什么要使用多线程

- 从计算机底层来说

  线程间的切换和调度的成本远远小于进程

  多核 CPU 时代意味着多个线程可以同时运行，这减少了线程上下文切换的开销

- 从当代互联网发展趋势来说

  现在的系统动不动就要求百万级甚至千万级的并发量，而多线程并发编程正是开发高并发系统的基础，利用好多线程机制可以大大提高系统整体的并发能力以及性能



## 多线程带来什么问题

和单线程相比，多线程编程的特点在于：

多线程经常需要**读写共享数据，并且需要同步**。例如，播放电影时，就必须由一个线程播放视频，另一个线程播放音频，两个线程需要协调运行，否则画面和声音就不同步。因此，多线程编程的**复杂度高、调试更困难**，还可能会遇到很多问题，比如：**内存泄漏、死锁、线程不安全**等等。

Java 多线程编程的特点又在于：

- 多线程模型是 Java 程序最基本的**并发模型**

- 后续读写网络、数据库、Web 开发等都依赖 Java 多线程模型




## 上下文切换

线程在执行过程中会有自己的**运行条件和状态**（也称上下文），比如上文所说到过的程序计数器，栈信息等。当出现如下情况的时候，线程会从占用 CPU 状态中退出。

- 主动让出 CPU，比如调用了 `sleep()`, `wait()` 等
- 时间片用完，因为操作系统要防止一个线程或者进程长时间占用CPU导致其他线程或者进程饿死
- 调用了阻塞类型的系统中断，比如请求 IO，线程被阻塞
- 被终止或结束运行

这其中前三种都会发生线程切换，线程切换意味着需要保存当前线程的上下文，留待线程下次占用 CPU 的时候恢复现场。并加载下一个将要占用 CPU 的线程上下文。这就是所谓的**上下文切换**。

上下文切换是现代操作系统的基本功能，因其**每次需要保存信息恢复信息**，这将会占用 CPU，内存等系统资源进行处理，也就意味着效率会有一定损耗，**如果频繁切换就会造成整体效率低下**。



## 创建多线程

Java 语言内置了多线程支持。Java 用 `Thread` 对象表示一个线程，通过调用 `start()` 启动一个新线程

- 一个线程对象只能调用一次 `start()` 方法
- 线程的执行代码写在 `run()` 方法中
- 线程调度由操作系统决定，程序本身无法决定调度顺序
- `Thread.sleep()` 可以把当前线程暂停一段时间



### 创建多线程

线程能执行指定的代码，有以下几种方法：

- 从 `Thread` 派生一个自定义类，然后覆写 `run()` 方法

- 创建 `Thread` 实例时，传入一个 `Runnable` 实例：

  ```java
  public class Main {
      public static void main(String[] args) {
          Thread t = new Thread(new MyRunnable());
          t.start(); // 启动新线程
      }
  }
  
  class MyRunnable implements Runnable {
      @Override
      public void run() {
          System.out.println("start new thread!");
      }
  }
  ```

- Java 8 引入的 `lambda` 语法进一步简写为：

  ```java
  public class Main {
      public static void main(String[] args) {
          Thread t = new Thread(() -> {
              System.out.println("start new thread!");
          });
          t.start(); // 启动新线程
      }
  }
  ```

- Executor 线程池

  


### 线程的优先级

可以对线程设定优先级，设定优先级的方法是：

```java
// 1~10, 默认值5
Thread.setPriority(int n) 
```

优先级高的线程被操作系统调度的优先级较高，操作系统对高优先级线程可能调度更频繁，但我们**决不能通过设置优先级来确保高优先级的线程一定会先执行**。



## 线程状态

Java 线程的状态有以下几种：

- New：新创建的线程，尚未执行
- Runnable：运行中的线程，正在执行 `run()` 方法的 Java 代码
- Blocked：运行中的线程，因为某些操作被阻塞而挂起
- Waiting：运行中的线程，因为某些操作在等待中
- Timed Waiting：运行中的线程，因为执行 `sleep()` 方法正在计时等待
- Terminated：线程已终止，因为 `run()` 方法执行完毕

用一个状态转移图表示如下：

```ascii
         ┌─────────────┐
         │     New     │
         └─────────────┘
                │
                ▼
┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐
 ┌─────────────┐ ┌─────────────┐
││  Runnable   │ │   Blocked   ││
 └─────────────┘ └─────────────┘
│┌─────────────┐ ┌─────────────┐│
 │   Waiting   │ │Timed Waiting│
│└─────────────┘ └─────────────┘│
 ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─
                │
                ▼
         ┌─────────────┐
         │ Terminated  │
         └─────────────┘
```



线程在生命周期中并不是固定处于某一个状态而是随着代码的执行在不同状态之间切换。Java 线程状态变迁如下图所示（图源《Java 并发编程艺术》4.1.4 节）：

![Java 线程状态变迁 ](https://img-note.langyastudio.com/202110191233018.png?x-oss-process=style/watermark)

> 订正(来自[issue736](https://github.com/Snailclimb/JavaGuide/issues/736))：原图中 wait 到 runnable 状态的转换中，`join`实际上是`Thread`类的方法，但这里写成了`Object`。



> 在操作系统中层面线程有 READY 和 RUNNING 状态，而在 JVM 层面只能看到 RUNNABLE 状态（图源：[HowToDoInJava](https://howtodoinJava.com/ "HowToDoInJava")：[Java Thread Life Cycle and Thread States](https://howtodoinJava.com/Java/multi-threading/Java-thread-life-cycle-and-thread-states/ "Java Thread Life Cycle and Thread States")），所以 Java 系统一般将这两个状态统称为 **RUNNABLE（运行中）** 状态 。
>
> 
>
> **为什么 JVM 没有区分这两种状态呢？** （摘自：[java线程运行怎么有第六种状态？ - Dawell的回答](https://www.zhihu.com/question/56494969/answer/154053599) ） 现在的<b>时分</b>（time-sharing）<b>多任务</b>（multi-task）操作系统架构通常都是用所谓的“<b>时间分片</b>（time quantum or time slice）”方式进行<b>抢占式</b>（preemptive）轮转调度（round-robin式）。这个时间分片通常是很小的，一个线程一次最多只能在 CPU 上运行比如 10-20ms 的时间（此时处于 running 状态），也即大概只有 0.01 秒这一量级，时间片用后就要被切换下来放入调度队列的末尾等待再次调度。（也即回到 ready 状态）。线程切换的如此之快，区分这两种状态就没什么意义了。

![RUNNABLE-VS-RUNNING](https://img-note.langyastudio.com/202110191251340.png?x-oss-process=style/watermark)

当线程执行 `wait()`方法之后，线程进入 **WAITING（等待）** 状态。进入等待状态的线程需要依靠其他线程的通知才能够返回到运行状态，而 **TIMED_WAITING(超时等待)** 状态相当于在等待状态的基础上增加了超时限制，比如通过 `sleep（long millis）`方法或 `wait（long millis）`方法可以将 Java 线程置于 TIMED_WAITING 状态。当超时时间到达后 Java 线程将会返回到 RUNNABLE 状态。当线程调用同步方法时，在没有获取到锁的情况下，线程将会进入到 **BLOCKED（阻塞）** 状态。线程在执行 Runnable 的`run()`方法之后将会进入到 **TERMINATED（终止）** 状态。



**线程终止的原因有：**

- 线程正常终止：`run()` 方法执行到 `return` 语句返回
- 线程意外终止：`run()` 方法因为未捕获的异常导致线程终止
- 对某个线程的 `Thread` 实例调用 `stop()` 方法强制终止（**强烈不推荐使用**）



## 中断线程

我们举个栗子：假设从网络下载一个 100M 的文件，如果网速很慢，用户不耐烦，就可能在下载过程中点“取消”，这时，程序就需要中断下载线程的执行。



### interrupt

线程需要执行一个长时间任务，就可能需要能中断线程。中断线程就是其他线程给该线程发一个信号，该线程收到信号后结束执行 `run()` 方法，使得自身线程能立刻结束运行。

- 对目标线程调用 `interrupt()` 方法可以请求中断一个线程，目标线程通过检测 `isInterrupted()` 标志获取自身是否已中断。如果目标线程处于等待状态，该线程会捕获到 `InterruptedException`

  目标线程检测到 `isInterrupted()` 为 `true` 或者捕获了 `InterruptedException` 都应该立刻结束自身线程

- 通过标志位判断需要正确使用 `volatile` 关键字

  `volatile` 关键字解决了共享变量在线程间的可见性问题

> interrupt 一般用于清理，但很多程序都是直接 kill 退出，懒得清理



### volatile

另一个常用的中断线程的方法是设置标志位。我们通常会用一个 `running` 标志位来标识线程是否应该继续运行，在外部线程中，通过把 `HelloThread.running` 置为 `false`，就可以让线程结束：

```java
public class Main {
    public static void main(String[] args)  throws InterruptedException {
        HelloThread t = new HelloThread();
        t.start();
        Thread.sleep(1);
        
        t.running = false; // 标志位置为false
    }
}

class HelloThread extends Thread {
    public volatile boolean running = true;
    
    public void run() {
        int n = 0;
        while (running) {
            n ++;
            System.out.println(n + " hello!");
        }
        System.out.println("end!");
    }
}
```

**线程间共享变量需要使用 `volatile` 关键字标记，确保每个线程都能读取到更新后的变量值**



为什么要对线程间共享的变量用关键字 `volatile` 声明？这涉及到 Java 的内存模型。在 Java 虚拟机中，**变量的值保存在主内存中**，但是，当线程访问变量时，它会先获取一个副本，并保存在自己的工作内存中。如果线程修改了变量的值，虚拟机会在某个时刻把修改后的值回写到主内存，但是，这个时间是不确定的！

```ascii
┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐
           Main Memory
│                               │
   ┌───────┐┌───────┐┌───────┐
│  │ var A ││ var B ││ var C │  │
   └───────┘└───────┘└───────┘
│     │ ▲               │ ▲     │
 ─ ─ ─│─│─ ─ ─ ─ ─ ─ ─ ─│─│─ ─ ─
      │ │               │ │
┌ ─ ─ ┼ ┼ ─ ─ ┐   ┌ ─ ─ ┼ ┼ ─ ─ ┐
      ▼ │               ▼ │
│  ┌───────┐  │   │  ┌───────┐  │
   │ var A │         │ var C │
│  └───────┘  │   │  └───────┘  │
   Thread 1          Thread 2
└ ─ ─ ─ ─ ─ ─ ┘   └ ─ ─ ─ ─ ─ ─ ┘
```

这会导致如果一个线程更新了某个变量，另一个线程读取的值可能还是更新前的。

因此，`volatile ` 关键字的目的是告诉虚拟机：

- 每次访问变量时，总是获取主内存的最新值
- 每次修改变量后，立刻回写到主内存

`volatile` 关键字解决的是可见性问题：**当一个线程修改了某个共享变量的值，其他线程能够立刻看到修改后的值**。



## ThreadLocal 

Java 标准库提供了一个特殊的 `ThreadLocal`，它**可以在一个线程中传递同一个对象**。需要传递的对象，我们通常称之为上下文（Context），它是一种**状态**，可以是用户身份、任务信息等。



### ThreadLocal 示例

`ThreadLocal ` 实例通常总是以静态字段初始化，例如：

```java
static ThreadLocal<User> threadLocalUser = new ThreadLocal<>();
```



可以把 `ThreadLocal` 看成一个全局 `Map<Thread, Object>`：每个线程获取 `ThreadLocal` 变量时，总是使用 `Thread` 自身作为 key：

```java
Object threadLocalValue = threadLocalMap.get(Thread.currentThread());
```

- `ThreadLocal` 表示线程的“局部变量”，它确保每个线程的 `ThreadLocal` 变量都是各自独立的

- `ThreadLocal` 适合在一个线程的处理流程中保持上下文（避免了同一参数在所有方法中传递）

- 使用 `ThreadLocal` 要用 `try ... finally` 结构，**并在 `finally` 中清除**



它的典型使用方式如下：

```java
void processUser(user) {
    try {
        threadLocalUser.set(user);
        
        step1();
        step2();
        
    } finally {
        threadLocalUser.remove();
    }
}
```

通过设置一个 `User` 实例关联到 `ThreadLocal` 中，在移除之前，所有方法都可以随时获取到该 `User` 实例：

```java
void step1() {
    User u = threadLocalUser.get();
    log();
    printUser();
}

void log() {
    User u = threadLocalUser.get();
    println(u.name);
}

void step2() {
    User u = threadLocalUser.get();
    checkUser(u.id);
}
```



![ThreadLocal数据结构](https://img-note.langyastudio.com/202110170733898.png?x-oss-process=style/watermark)

`ThreadLocalMap` 是 `ThreadLocal` 的静态内部类：

![ThreadLocal内部类](https://img-note.langyastudio.com/202110170733580.png?x-oss-process=style/watermark)



### ThreadLocal 内存泄露

`ThreadLocalMap` 中使用的 key 为 `ThreadLocal` 的弱引用，而 value 是强引用。所以如果 `ThreadLocal` 没有被外部强引用的情况下，在垃圾回收的时候，key 会被清理掉，而 value 不会被清理掉。这样一来 `ThreadLocalMap` 中就会出现 key 为 null 的 Entry。

**假如我们不做任何措施的话，value 永远无法被 GC 回收，这个时候就可能会产生内存泄露。**

ThreadLocalMap 实现中已经考虑了这种情况，在调用 `set()`、`get()`、`remove()` 方法的时候，会清理掉 key 为 null 的记录。使用完 `ThreadLocal`方法后最好手动调用 `remove()` 方法

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```



**弱引用介绍：**

> 如果一个对象只具有弱引用，那就类似于**可有可无的生活用品**。弱引用与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期。在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过由于垃圾回收器是一个优先级很低的线程， 因此不一定会很快发现那些只具有弱引用的对象。
>
> 弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被垃圾回收，Java 虚拟机就会把这个弱引用加入到与之关联的引用队列中。



## 守护线程

有一种线程的目的就是无限循环，例如一个定时触发任务的线程：

```java
class TimerThread extends Thread {
    @Override
    public void run() {
        while (true) {
            System.out.println(LocalTime.now());
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                break;
            }
        }
    }
}
```

如果这个线程不结束，JVM 进程就无法结束。但是当其他线程结束时，JVM 进程又必须要结束，怎么办？

答案是使用守护线程（Daemon Thread）

- 守护线程是为其他线程服务的线程
- 所有非守护线程都执行完毕后，无论有没有守护线程，虚拟机都会自动退出



**创建守护线程**

```java
Thread t = new MyThread();
t.setDaemon(true);
t.start();
```

**守护线程不能持有任何需要关闭的资源**，例如打开文件等，因为虚拟机退出时，守护线程没有任何机会来关闭文件，这会导致数据丢失。



## wait and notify

**多线程协调运行的原则就是：**

- 当条件不满足时，线程进入等待状态

- 当条件满足时，线程被唤醒，继续执行任务



`wait` 和 `notify` 用于多线程协调运行：

- 在 `synchronized` 内部可以调用 `wait()` 使线程进入等待状态

  必须在已获得的锁对象上调用 `wait()` 方法

  因为 `wait()` 方法调用时，会**释放**线程获得的锁，`wait()` 方法返回后，线程又会重新试图获得锁

- 在 `synchronized` 内部可以调用 `notify()` 或 `notifyAll()` 唤醒其他等待线程

  必须在已获得的锁对象上调用 `notify()` 或 `notifyAll()` 方法

- 已唤醒的线程还需要**重新**获得锁后才能继续执行

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        var q = new TaskQueue();
        var ts = new ArrayList<Thread>();
        
        for (int i=0; i<5; i++) {
            var t = new Thread() {
                public void run() {                    
                    // 执行task:
                    while (true) {
                        try {
                            String s = q.getTask();
                            System.out.println("execute task: " + s);
                        } catch (InterruptedException e) {
                            return;
                        }
                    }
                }
            };
            
            t.start();
            ts.add(t);
        }
        
        var add = new Thread(() -> {
            for (int i=0; i<10; i++) {
                // 放入task:
                String s = "t-" + Math.random();
                System.out.println("add task: " + s);
                q.addTask(s);
                
                try { Thread.sleep(100); } catch(InterruptedException e) {}
            }
        });        
        add.start();
        add.join();
        
        Thread.sleep(100);
        for (var t : ts) {
            t.interrupt();
        }
    }
}

class TaskQueue {
    Queue<String> queue = new LinkedList<>();

    public synchronized void addTask(String s) {
        this.queue.add(s);
        this.notifyAll();
    }

    public synchronized String getTask() throws InterruptedException {
        while (queue.isEmpty()) {
            this.wait();
        }
        
        return queue.remove();
    }
}
```

我们在 `while()` 循环中调用 `wait()`，而不是 `if `语句：

```java
public synchronized String getTask() throws InterruptedException {
    if (queue.isEmpty()) {
        this.wait();
    }
    return queue.remove();
}
```

这种写法实际上是错误的，因为线程被唤醒时，需要再次获取 `this` 锁。多个线程被唤醒后，只有一个线程能获取 `this` 锁，此刻该线程执行 `queue.remove()` 可以获取到队列的元素，然而剩下的线程如果获取 `this` 锁后执行`queue.remove()`，此刻队列可能已经没有任何元素了，所以**要始终在`while`循环中`wait()`**，并且每次被唤醒后拿到`this` 锁就必须再次判断：

```java
while (queue.isEmpty()) {
    this.wait();
}
```

所以，正确编写多线程代码是非常困难的，需要仔细考虑的条件非常多，任何一个地方考虑不周，都会导致多线程运行时不正常。



## 线程死锁

### 可重入锁

Java 的线程锁 `synchronized` 是可重入的锁。

什么是可重入的锁？我们还是来看例子：

```java
public class Counter {
    private int count = 0;

    public synchronized void add(int n) {
        if (n < 0) {
            dec(-n);
        } else {
            count += n;
        }
    }

    public synchronized void dec(int n) {
        count += n;
    }
}
```

JVM 允许同一个线程重复获取同一个锁，这种能**被同一个线程反复获取的锁，就叫做可重入锁**。

由于 Java 的线程锁是可重入锁，所以获取锁的时候，不但要判断是否是第一次获取，还要记录这是第几次获取。每获取一次锁，记录+1，每退出 `synchronized` 块，记录-1，减到0的时候，才会真正释放锁。



### 死锁

两个线程各自持有不同的锁，然后各自试图获取对方手里的锁，造成了双方无限等待下去，这就是死锁。

**死锁发生后，没有任何机制能解除死锁，只能强制结束 JVM 进程。**

![线程死锁示意图 ](https://img-note.langyastudio.com/202110191251366.png?x-oss-process=style/watermark)



下面通过一个例子来说明线程死锁

```java
public void dec(int m) {
     // 获得lockA的锁
    synchronized(lockA) {
        this.value -= m;
        
        // 获得lockB的锁
        synchronized(lockB) { 
            this.another -= m;
        } // 释放lockB的锁
        
    } // 释放lockA的锁
}
```



**产生死锁必须具备以下四个条件：**

- 互斥条件：该资源任意一个时刻只由一个线程占用

- 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放

- 不剥夺条件: 线程已获得的资源在未使用完之前不能被其他线程强行剥夺，只有自己使用完毕后才释放资源

- 循环等待条件: 若干进程之间形成一种头尾相接的循环等待资源关系



### 如何预防死锁

- 破坏循环等待条件

  **按顺序申请资源**来预防。按某一顺序申请资源，释放资源则反序释放

- 破坏请求与保持条件

  一次性申请所有的资源

- 破坏不剥夺条件

  占用部分资源的线程进一步申请其他资源时，如果申请不到，可以主动释放它占有的资源



## sleep() vs wait() 

- 两者都可以暂停线程的执行

- 两者最主要的区别在于

  **`sleep()` 方法没有释放锁，而 `wait()` 方法释放了锁** 

- `wait()` 通常被用于线程间交互/通信

  `sleep() `通常被用于暂停执行

- `wait()` 方法被调用后，线程不会自动苏醒，需要别的线程调用同一个对象上的 `notify() `或者 `notifyAll()` 方法。

  `sleep() `方法执行完成后，线程会自动苏醒。或者可以使用 `wait(long timeout)` 超时后线程会自动苏醒。



## 为什么不能直接调用 run() 方法

new  一个  Thread，线程进入了新建状态。调用 `start()` 方法，会启动一个线程并使线程进入了就绪状态，当分配到时间片后就可以开始运行了。 `start()` 会执行线程的相应准备工作，然后自动执行 `run()` 方法的内容，这是真正的多线程工作。 

但是直接执行 `run()` 方法，会把 `run()` 方法当成一个 main 线程下的普通方法去执行，并不会在某个线程中执行它，所以这并不是多线程工作。

**即调用 `start()` 方法方可启动线程并使线程进入就绪状态，直接执行 `run()` 方法的话不会以多线程的方式执行**

