> 本文来自JavaGuide、廖雪峰，郎涯进行简单排版与补充



## 面向对象

### 面向对象 vs 面向过程

- **面向过程** ：是把模型分解成一步一步的过程。当性能是最重要的考量因素的时候，比如单片机、嵌入式开发、Linux/Unix 等一般采用面向过程开发。
- **面向对象** ：是一种通过对象的方式，把现实世界映射到计算机模型的一种编程方法。**面向对象易维护、易复用、易扩展。** 因为面向对象有封装、继承、多态性的特性，所以可以设计出低耦合的系统，使系统更加灵活、更加易于维护。

参见 issue : [面向过程 ：面向过程性能比面向对象高？？](https://github.com/Snailclimb/JavaGuide/issues/431)

> 这个并不是根本原因，面向过程也需要分配内存，计算内存偏移量，Java 性能差的主要原因并不是因为它是面向对象语言，而是 Java 是半编译语言，最终的执行代码并不是可以直接被 CPU 执行的二进制机械码。
>
> 而面向过程语言大多都是直接编译成机械码在电脑上执行，并且其它一些面向过程的脚本语言性能也并不一定比 Java 好。



### 成员变量 vs 局部变量

- 语法形式

  成员变量是属于类的，而局部变量是在代码块或方法中定义的变量或是方法的参数；成员变量可以被 `public`,`private`,`static` 等修饰符所修饰，而局部变量不能被访问控制修饰符及 `static` 所修饰；但是，成员变量和局部变量都能被 `final` 所修饰。

- 内存中的存储方式

  如果成员变量是使用 `static` 修饰的，那么这个成员变量是属于类的，如果没有使用 `static` 修饰，这个成员变量是属于实例的。而对象存在于堆内存，局部变量则存在于栈内存。

- 内存中的生存时间

  成员变量是对象的一部分，它随着对象的创建而存在，而局部变量随着方法的调用而自动消失。

- 默认值

  成员变量如果没有被赋初，则会自动以类型的默认值而赋值（一种情况例外:被 `final` 修饰的成员变量也必须显式地赋值），而**局部变量则不会自动赋值**。



### 对象实体 vs 对象引用

- 对象实例在堆内存中

- 对象引用存放在栈内存中

一个对象引用可以指向 0 个或 1 个对象（一根绳子可以不系气球，也可以系一个气球）;一个对象可以有 n 个引用指向它（可以用 n 条绳子系住一个气球）。



### 构造方法

构造方法主要作用是完成对类对象的初始化工作。

如果一个类没有声明构造方法，也可以执行！因为一个类即使没有声明构造方法也会有默认的不带参数的构造方法。如果我们自己添加了类的构造方法（无论是否有参），Java 就不会再添加默认的无参数的构造方法了，这时候，就不能直接 new 一个对象而不传递参数了，所以我们一直在不知不觉地使用构造方法，这也是为什么我们在创建对象的时候后面要加一个括号（因为要调用无参的构造方法）。如果我们重载了有参的构造方法，记得都要把无参的构造方法也写出来（无论是否用到），因为这可以帮助我们在创建对象的时候少踩坑。

**特点：**

- 名字与类名相同

- 没有返回值，但不能用 void 声明构造函数

- 生成类的对象时自动执行，无需调用

> 构造方法不能被 override（重写）,  但是可以 overload（重载）,所以你可以看到一个类中有多个构造函数的情况



### Object 常见方法

Object 类是一个特殊的类，是所有类的父类。它主要提供了以下 11 个方法：

```java
public final native Class<?> getClass()//native方法，用于返回当前运行时对象的Class对象，使用了final关键字修饰，故不允许子类重写。

public native int hashCode() //native方法，用于返回对象的哈希码，主要使用在哈希表中，比如JDK中的HashMap。
public boolean equals(Object obj)//用于比较2个对象的内存地址是否相等，String类对该方法进行了重写用户比较字符串的值是否相等。

protected native Object clone() throws CloneNotSupportedException//naitive方法，用于创建并返回当前对象的一份拷贝。一般情况下，对于任何对象 x，表达式 x.clone() != x 为true，x.clone().getClass() == x.getClass() 为true。Object本身没有实现Cloneable接口，所以不重写clone方法并且进行调用的话会发生CloneNotSupportedException异常。

public String toString()//返回类的名字@实例的哈希码的16进制的字符串。建议Object所有的子类都重写这个方法。

public final native void notify()//native方法，并且不能重写。唤醒一个在此对象监视器上等待的线程(监视器相当于就是锁的概念)。如果有多个线程在等待只会任意唤醒一个。

public final native void notifyAll()//native方法，并且不能重写。跟notify一样，唯一的区别就是会唤醒在此对象监视器上等待的所有线程，而不是一个线程。

public final native void wait(long timeout) throws InterruptedException//native方法，并且不能重写。暂停线程的执行。注意：sleep方法没有释放锁，而wait方法释放了锁 。timeout是等待时间。

public final void wait(long timeout, int nanos) throws InterruptedException//多了nanos参数，这个参数表示额外时间（以毫微秒为单位，范围是 0-999999）。 所以超时的时间还需要加上nanos毫秒。

public final void wait() throws InterruptedException//跟之前的2个wait方法一样，只不过该方法一直等待，没有超时时间这个概念

protected void finalize() throws Throwable { }//实例被垃圾回收器回收的时候触发的操作
```



## 三大特征

### 封装

封装是指把一个对象的状态信息（也就是属性）隐藏在对象内部，不允许外部对象直接访问对象的内部信息。但是可以提供一些可以被外界访问的方法来操作属性。

```java
public class Student {
    private int id;//id属性私有化
    private String name;//name属性私有化

    //获取id的方法
    public int getId() {
        return id;
    }

    //设置id的方法
    public void setId(int id) {
        this.id = id;
    }

    //获取name的方法
    public String getName() {
        return name;
    }

    //设置name的方法
    public void setName(String name) {
        this.name = name;
    }
}
```



### 继承

继承是使用已存在的类的定义作为基础建立新类的技术，新类的定义可以增加新的数据或新的功能，也可以用父类的功能，但不能选择性地继承父类。通过使用继承，可以快速地创建新的类，可以提高代码的重用，程序的可维护性，节省大量创建新类的时间 ，提高我们的开发效率。

**关于继承如下 3 点请记住：**

- 子类拥有父类对象所有的属性和方法（包括私有属性和私有方法），但是父类中的私有属性和方法子类是无法访问，**只是拥有**
- 子类可以拥有自己属性和方法，即子类可以对父类进行扩展
- 子类可以用自己的方式实现父类的方法



**Java只允许一个class继承自一个类**，因此，一个类有且仅有一个父类。只有`Object`特殊，它没有父类。

```ascii
       ┌───────────┐
       │  Object   │
       └───────────┘
             ▲
             │
       ┌───────────┐
       │  Person   │
       └───────────┘
          ▲     ▲
          │     │
          │     │
┌───────────┐ ┌───────────┐
│  Student  │ │  Teacher  │
└───────────┘ └───────────┘
```



**向上转型 upcasting**

一个子类类型安全地变为父类类型的赋值，被称为向上转型

```java
Person p = new Student(); 
```



**向下转型 downcasting**

一个父类类型强制转型为子类类型

```java
Person p1 = new Student(); // upcasting, ok
Person p2 = new Person();

if (p instanceof Student) {
    // 只有判断成功才会向下转型:
    Student s = (Student) p; // 一定会成功
}

Student s2 = (Student) p2; // runtime error! ClassCastException!
```

> 为了避免向下转型出错，Java提供了`instanceof`操作符。`instanceof`实际上判断一个变量所指向的实例是否是指定类型，或者这个类型的子类



**区分继承和组合**

继承是is关系，组合是has关系

`Student`是`Person`的一种，它们是is关系，而`Student`并不是`Book`。`Student`和`Book`的关系是has关系。具有has关系不应该使用继承，而是使用组合，即`Student`可以持有一个`Book`实例。

```java
class Student extends Person {
    protected Book book;
    protected int score;
}
```



### 多态

表示一个对象具有多种的状态，**具体表现为父类的引用指向子类的实例**。

**多态的特点:**

- 对象类型和引用类型之间具有继承（类）/实现（接口）的关系
- 引用类型变量发出的方法调用的到底是哪个类中的方法，必须在程序运行期间才能确定
- 多态不能调用“只在子类存在但在父类不存在”的方法
- 如果子类重写了父类的方法，真正执行的是子类覆盖的方法，如果子类没有覆盖父类的方法，执行的是父类的方法



`@Override`可以让编译器帮助检查是否进行了正确的覆写。希望进行覆写，但是不小心写错了方法签名，编译器会报错

```java
class Person {
    public void run() {}
}

public class Student extends Person {
    @Override // Compile error!
    public void run(String s) {}
}
```



**覆写Object方法**

- `toString()`：把instance输出为`String`；

- `equals()`：判断两个instance是否逻辑相等；

- `hashCode()`：计算一个instance的哈希值。



## 抽象类与接口

### 抽象类

如果父类的方法本身不需要实现任何功能，仅仅是为了定义方法签名，目的是让子类去覆写它，那么可以把父类的方法声明为抽象方法。

因为抽象类本身被设计成只能用于被继承， 抽象方法实际上相当于定义了“规范”。

```java
abstract class Person {   
    public abstract void run();
}
```

> 定义了抽象方法的class必须被定义为抽象类，从抽象类继承的子类必须实现抽象方法

面向抽象编程的本质就是：

- 上层代码只定义规范（例如：`abstract class Person`）；
- 不需要子类就可以实现业务逻辑（正常编译）；
- 具体的业务逻辑由不同的子类实现，调用者并不关心。



### 接口

如果一个抽象类没有字段，所有方法全部都是抽象方法：

```java
abstract class Person {
    public abstract void run();
    public abstract String getName();
}
```

就可以把该抽象类改写为接口：`interface`。在Java中，使用`interface`可以声明一个接口：

```java
interface Person {
    void run();
    String getName();
}
```

`interface`，**就是比抽象类还要抽象的纯抽象接口，因为它连字段都不能有**。

当一个具体的`class`去实现一个`interface` 时，需要使用 `implements` 关键字

> 因为接口定义的所有方法默认都是`public abstract`的，所以这两个修饰符不需要写出来（写不写效果都一样）。



**接口继承**

一个`interface`可以继承自另一个`interface`。`interface`继承自`interface`使用`extends`，它相当于扩展了接口的方法。例如：

```java
interface Hello {
    void hello();
}

interface Person extends Hello {
    void run();
    String getName();
    
    //default 方法
    default void run() {
        System.out.println(getName() + " run");
    }
}
```

一般来说，公共逻辑适合放在`abstract class`中，具体逻辑放到各个子类，而接口层次代表抽象程度。

```ascii
┌───────────────┐
│   Iterable    │
└───────────────┘
        ▲                ┌───────────────────┐
        │                │      Object       │
┌───────────────┐        └───────────────────┘
│  Collection   │                  ▲
└───────────────┘                  │
        ▲     ▲          ┌───────────────────┐
        │     └──────────│AbstractCollection │
┌───────────────┐        └───────────────────┘
│     List      │                  ▲
└───────────────┘                  │
              ▲          ┌───────────────────┐
              └──────────│   AbstractList    │
                         └───────────────────┘
                                ▲     ▲
                                │     │
                                │     │
                     ┌────────────┐ ┌────────────┐
                     │ ArrayList  │ │ LinkedList │
                     └────────────┘ └────────────┘
```



**default方法**

在接口中，可以定义 `default` 方法。目的是当我们需要给接口新增一个方法时，会涉及到修改全部子类。如果新增的是`default` 方法，那么子类就不必全部修改，只需要在需要覆写的地方去覆写新增方法。



**接口的静态字段**

因为`interface`是一个纯抽象类，所以它**不能定义实例字段**。但是 `interface` 是可以有静态字段的，并且静态字段必须为`final`类型：

```java
public interface Person {
    public static final int MALE = 1;
    public static final int FEMALE = 2;
}
```

>  静态方法常用于工具类和辅助方法



## 深拷贝 vs 浅拷贝

- **浅拷贝**：浅拷贝会在堆上创建一个新的对象（区别于引用拷贝的一点），不过，如果原对象内部的属性是引用类型的话，浅拷贝会直接复制内部对象的引用地址，也就是说拷贝对象和原对象共用同一个内部对象。

- **深拷贝** ：深拷贝会完全复制整个对象，包括这个对象所包含的内部对象。

![deep and shallow copy](https://img-note.langyastudio.com/202110121158019.jpeg?x-oss-process=style/watermark)

上面的结论没有完全理解的话也没关系，我们来看一个具体的案例！

**浅拷贝**

浅拷贝的示例代码如下，我们这里实现了 `Cloneable` 接口，并重写了 `clone()` 方法。

 `clone()` 方法的实现很简单，直接调用的是父类 `Object` 的 `clone()` 方法。

```java
public class Address implements Cloneable{
    private final String name;
    // 省略构造函数、Getter&Setter方法
    @Override
    public Address clone() {
        try {
            return (Address) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}

public class Person implements Cloneable {
    private Address address;
    // 省略构造函数、Getter&Setter方法
    @Override
    public Person clone() {
        try {
            Person person = (Person) super.clone();
            return person;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

测试 ：

```java
Person person1 = new Person(new Address("武汉"));
Person person1Copy = person1.clone();
// true
System.out.println(person1.getAddress() == person1Copy.getAddress());
```

从输出结构就可以看出， `person1` 的克隆对象和 `person1`  使用的仍然是同一个 `Address` 对象。

**深拷贝**

这里我们简单对 `Person` 类的 `clone()` 方法进行修改，连带着要把 `Person` 对象内部的 `Address` 对象一起复制。

```java
@Override
public Person clone() {
    try {
        Person person = (Person) super.clone();
        person.setAddress(person.getAddress().clone());
        return person;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

测试 ：

```java
Person person1 = new Person(new Address("武汉"));
Person person1Copy = person1.clone();
// false
System.out.println(person1.getAddress() == person1Copy.getAddress());
```

从输出结构就可以看出，虽然 `person1` 的克隆对象和 `person1`  包含的 `Address` 对象已经是不同的了。

**那什么是引用拷贝呢？** 简单来说，引用拷贝就是两个不同的引用指向同一个对象。



我专门画了一张图来描述浅拷贝、深拷贝、引用拷贝：

![shallow&deep-copy](https://img-note.langyastudio.com/202111240912700.png?x-oss-process=style/watermark)



## Java 提供的机制

### package

解决名字冲突

```java
package mr.jun; // 申明包名mr.jun

public class Arrays {
}
```

java文件对应的目录层次要和包的层次一致

```ascii
package_sample
└─ src
    ├─ hong
    │  └─ Person.java
    │  ming
    │  └─ Person.java
    └─ mr
       └─ jun
          └─ Arrays.java
```



**包作用域**

位于同一个包的类，可以访问包作用域的字段和方法。不用`public`、`protected`、`private`修饰的字段和方法就是包作用域。

```java
package hello;

public class Person {
    // 包作用域:
    void hello() {
        System.out.println("Hello!");
    }
}
```

`Main`类也定义在`hello`包下面：

```java
package hello;

public class Main {
    public static void main(String[] args) {
        Person p = new Person();
        p.hello(); // 可以调用，因为Main和Person在同一个包
    }
}
```

>  要特别注意：包没有父子关系。`java.util`和`java.util.zip`是不同的包，两者没有任何继承关系。



**import**

```java
// 导入完整类名:
import mr.jun.Arrays;
```



**查找顺序**

当编译器遇到一个`class`名称时：

- 如果是完整类名，就直接根据完整类名查找这个`class`；
- 如果是简单类名，按下面的顺序依次查找：
  - 查找当前`package`是否存在这个`class`；
  - 查找`import`的包是否包含这个`class`；
  - 查找`java.lang`包是否包含这个`class`。

如果按照上面的规则还无法确定类名，则编译报错。



**最佳实践**

为了避免名字冲突，我们需要确定唯一的包名。**推荐的做法是使用倒置的域名来确保唯一性**。例如：

- org.apache
- org.apache.commons.log
- com.liaoxuefeng.sample

子包就可以根据功能自行命名。



- 一个`.java`文件只能包含一个`public`类，但可以包含多个非`public`类。

- 如果有`public`类，文件名必须和`public`类的名字相同。

- Java内建的访问权限包括`public`、`protected`、`private`和`package`权限；



### classpath

`classpath` 是 JVM 用到的一个环境变量，它用来指示 JVM 如何搜索 `class`。因为 JVM 需要知道，如果要加载一个`abc.xyz.Hello` 的类，应该去哪搜索对应的 `Hello.class` 文件

所以，`classpath` 就是一组目录的集合，它设置的搜索路径与操作系统相关。例如，在Windows系统上，用`;`分隔，带空格的目录用`""`括起来，可能长这样：

```java
C:\work\project1\bin;C:\shared;"D:\My Documents\project1\bin"
```

在Linux系统上，用`:`分隔，可能长这样：

```java
/usr/shared:/usr/local/bin:/home/liaoxuefeng/bin
```



现在我们假设 `classpath` 是 `.;C:\work\project1\bin;C:\shared`，当 JVM 在加载 `abc.xyz.Hello` 这个类时，会依次查找：

```
<当前目录>\abc\xyz\Hello.class
C:\work\project1\bin\abc\xyz\Hello.class
C:\shared\abc\xyz\Hello.class
```



**设置方法**

- 在系统环境变量中设置 `classpath` 环境变量，不推荐；

- 在启动 JVM 时设置 `classpath` 变量，**推荐**。

实际上就是给 `java` 命令传入 `-classpath` 或 `-cp` 参数：

```
java -classpath .;C:\work\project1\bin;C:\shared abc.xyz.Hello
```

或者使用 `-cp` 的简写：

```
java -cp .;C:\work\project1\bin;C:\shared abc.xyz.Hello
```

没有设置系统环境变量，也没有传入 `-cp` 参数，那么JVM默认的 `classpath` 为 `.`，即当前目录

> 不要把任何Java核心库添加到 `classpath` 中！JVM根本不依赖 `classpath` 加载核心库！



### jar

jar 包实际上就是一个 zip 格式的压缩文件，它可以把 `package` 组织的目录层级，以及各个目录下的所有文件（包括`.class`文件和其他文件）都打成一个 jar 文件，这样一来，无论是备份，还是发给客户，就简单多了。

```ascii
package_sample
└─ bin
   ├─ hong
   │  └─ Person.class
   │  ming
   │  └─ Person.class
   └─ mr
      └─ jun
         └─ Arrays.class
```

这里需要特别注意的是，jar 包里的第一层目录，不能是 `bin`，而应该是 `hong`、`ming`、`mr`

```java
java -cp ./hello.jar abc.xyz.Hello
```



jar 包还可以包含一个特殊的 `/META-INF/MANIFEST.MF `文件，`MANIFEST.MF`是纯文本，可以指定`Main-Class`和其它信息。JVM会自动读取这个`MANIFEST.MF`文件，如果存在`Main-Class`，我们就不必在命令行指定启动的类名，而是用更方便的命令：

```java
java -jar hello.jar
```



jar 包还可以包含其它 jar 包，这个时候，就需要在 `MANIFEST.MF` 文件里配置 `classpath` 了。

> 在大型项目中，不可能手动编写`MANIFEST.MF`文件，再手动创建zip包。Java社区提供了大量的开源构建工具，例如[Maven](https://www.liaoxuefeng.com/wiki/1252599548343744/1255945359327200)，可以非常方便地创建jar包



### 模块

jar 只是用于存放 class 的容器，它并不关心 class 之间的依赖。

如果是自己开发的程序，除了一个自己的`app.jar`以外，还需要一堆第三方的jar包，运行一个Java程序，一般来说，命令行写这样：

```bash
java -cp app.jar:a.jar:b.jar:c.jar com.liaoxuefeng.sample.Main
```



**从Java 9开始，JDK又引入了模块（Module）**，主要是为了解决“依赖”这个问题。如果`a.jar`必须依赖另一个`b.jar`才能运行，那我们应该给`a.jar`加点说明啥的，让程序在编译和运行的时候能自动定位到`b.jar`，这种自带“依赖关系”的class容器就是模块。

从Java 9开始，原有的Java标准库已经由一个单一巨大的`rt.jar`分拆成了几十个模块，这些模块以`.jmod`扩展名标识，可以在`$JAVA_HOME/jmods`目录下找到它们：

- java.base.jmod
- java.compiler.jmod
- java.datatransfer.jmod
- java.desktop.jmod
- ...

模块之间的依赖关系已经被写入到模块内的`module-info.class`文件了。所有的模块都直接或间接地依赖`java.base`模块，只有`java.base`模块不依赖任何模块，它可以被看作是“根模块”，好比所有的类都是从`Object`直接或间接继承而来。



**编写模块**

```java
oop-module
├── bin
├── build.sh
└── src
    ├── com
    │   └── itranswarp
    │       └── sample
    │           ├── Greeting.java
    │           └── Main.java
    └── module-info.java
```

仅仅在`src`目录下多了一个`module-info.java`这个文件

```java
module hello.world {
	requires java.xml;
}
```

当我们使用模块声明了依赖关系后，才能使用引入的模块。

```java
package com.itranswarp.sample;

// 必须引入java.xml模块后才能使用其中的类:
import javax.xml.XMLConstants;

public class Main {
	public static void main(String[] args) {
		Greeting g = new Greeting();
		System.out.println(g.hello(XMLConstants.XML_NS_PREFIX));
	}
}
```



**目的**

- Java 9 引入的模块目的是为了管理依赖；

- 使用模块可以按需打包JRE；

- 使用模块对类的访问权限有了进一步限制。



## 常用类

### String

字符串在`String`内部是通过一个`char[]`数组表示的。`String`和`char[]`类型可以互相转换，方法是：

```java
char[] cs = "Hello".toCharArray(); // String -> char[]
String s = new String(cs); // char[] -> String
```

> 特点是字符串**不可变**。这种不可变性是通过内部的`private final char[]`字段实现
>
> `String` 类被 `final` 修饰导致其不能被继承，进而避免了子类破坏 `String` 不可变
>
> 相关阅读：[如何理解 String 类型值的不可变？ - 知乎提问](https://www.zhihu.com/question/20618891/answer/114125846)



**字符串比较**

使用`equals()`方法，要忽略大小写比较，使用`equalsIgnoreCase()`方法。



**搜索子串**

```java
// 是否包含子串:
"Hello".contains("ll"); // true

"Hello".indexOf("l"); // 2
"Hello".lastIndexOf("l"); // 3
"Hello".startsWith("He"); // true
"Hello".endsWith("lo"); // true
```



**提取子串**

```java
"Hello".substring(2); // "llo"
"Hello".substring(2, 4); "ll"
```



**去除首尾空白字符**

使用`trim()`方法可以移除字符串首尾空白字符。空白字符包括空格，`\t`，`\r`，`\n`：

```java
"  \tHello\r\n ".trim(); // "Hello"
```

`strip()`方法也可以移除字符串首尾空白字符。它和`trim()`不同的是，类似中文的空格字符`\u3000`也会被移除：

```java
"\u3000Hello\u3000".strip(); // "Hello"
" Hello ".stripLeading(); // "Hello "
" Hello ".stripTrailing(); // " Hello"
```



**是否为空**

提供了`isEmpty()`和`isBlank()`来判断字符串是否为空和空白字符串：

```java
"".isEmpty(); // true，因为字符串长度为0
"  ".isEmpty(); // false，因为字符串长度不为0
"  \n".isBlank(); // true，因为只包含空白字符
" Hello ".isBlank(); // false，因为包含非空白字符
```



**替换子串**

要在字符串中替换子串，有两种方法。一种是根据字符或字符串替换：

```java
String s = "hello";
s.replace('l', 'w'); // "hewwo"，所有字符'l'被替换为'w'
s.replace("ll", "~~"); // "he~~o"，所有子串"ll"被替换为"~~"
```

另一种是通过正则表达式替换：

```java
String s = "A,,B;C ,D";
s.replaceAll("[\\,\\;\\s]+", ","); // "A,B,C,D"
```



**分割字符串**

要分割字符串，使用`split()`方法，并且传入的也是正则表达式：

```java
String s = "A,B,C,D";String[] ss = s.split(","); // {"A", "B", "C", "D"}
```



**拼接字符串**

拼接字符串使用静态方法`join()`，它用指定的字符串连接字符串数组：

```java
String[] arr = {"A", "B", "C"};
String s = String.join("***", arr); // "A***B***C"
```



**格式化**

字符串提供了`formatted()`方法和`format()`静态方法

```java
public class Main {
    public static void main(String[] args) {
        String s = "Hi %s, your score is %d!";
        System.out.println(s.formatted("Alice", 80));
        System.out.println(String.format("Hi %s, your score is %.2f!", "Bob", 59.5));
    }
}
```



**类型转换**

要把任意基本类型或引用类型转换为字符串，可以使用静态方法`valueOf()`

```java
String.valueOf(123); // "123"
String.valueOf(45.67); // "45.67"
String.valueOf(true); // "true"
String.valueOf(new Object()); // 类似java.lang.Object@636be97c
```



要把字符串转换为其他类型，就需要根据情况。例如，把字符串转换为`int`类型：

```java
int n1 = Integer.parseInt("123"); // 123
int n2 = Integer.parseInt("ff", 16); // 按十六进制转换，255
```



要特别注意，`Integer`有个`getInteger(String)`方法，它不是将字符串转换为`int`，而是把该**字符串对应的系统变量**转换为`Integer`：

```java
Integer.getInteger("java.version"); // 版本号，11
```



把字符串转换为`boolean`类型：

```java
boolean b1 = Boolean.parseBoolean("true"); // true
boolean b2 = Boolean.parseBoolean("FALSE"); // false
```



**延伸阅读**

对于不同版本的JDK，`String`类在内存中有不同的优化方式。具体来说，早期JDK版本的`String`总是以`char[]`存储，它的定义如下：

```java
public final class String {
    private final char[] value;
    private final int offset;
    private final int count;
}
```

**而较新的JDK版本的`String`则以`byte[]`存储：**如果`String`仅包含ASCII字符，则每个`byte`存储一个字符，否则，每两个`byte`存储一个字符，这样做的目的是为了节省内存，因为大量的长度较短的`String`通常仅包含ASCII字符：

```java
public final class String {
    private final byte[] value;
    private final byte coder; // 0 = LATIN1, 1 = UTF16
```

对于使用者来说，`String`内部的优化不影响任何已有代码，因为它的`public`方法签名是不变的。

- 转换编码就是将`String`和`byte[]`转换，需要指定编码；

- 转换为`byte[]`时，始终优先考虑`UTF-8`编码。

    

### StringBuilder

频繁的拼接字符串，不但浪费内存，还会影响GC效率。于是Java标准库提供了`StringBuilder`，它不会创建新的临时对象，是一个可变对象，可以预分配缓冲区。

注意：对于普通的字符串`+`操作，并不需要我们将其改写为`StringBuilder`，因为Java编译器在编译时就自动把多个连续的`+`操作编码为`StringConcatFactory`的操作。在运行期，`StringConcatFactory`会自动把字符串连接操作优化为数组复制或者`StringBuilder`操作。

> `StringBuffer`是Java早期的一个`StringBuilder`的线程安全版本，现在很少使用。
>
>  `StringBuilder` 与 `StringBuffer` 都继承自 `AbstractStringBuilder` 类，在 `AbstractStringBuilder` 中也是使用字符数组保存字符串，不过没有使用 `final`  和 `private` 关键字修饰，最关键的是这个 `AbstractStringBuilder` 类还提供了很多修改字符串的方法比如 `append` 方法。



### StringJoiner

类似用分隔符拼接数组的需求很常见，所以Java标准库还提供了一个`StringJoiner`来干这个事

```java
public class Main {
    public static void main(String[] args) {
        String[] names = {"Bob", "Alice", "Grace"};
        //指定分隔符、开头、结尾
        var sj = new StringJoiner(", ", "Hello ", "!");
        for (String name : names) {
            sj.add(name);
        }
        System.out.println(sj.toString());
    }
}
```

`String`还提供了一个静态方法`join()`，这个方法在内部使用了`StringJoiner`来拼接字符串，在不需要指定“开头”和“结尾”的时候，用`String.join()`更方便。



### String vs StringBuilder

**可变性**

简单的来说：`String` 类中使用 final 关键字修饰字符数组来保存字符串，`private final char value[]`，所以**`String` 对象是不可变的**。

> 补充（来自[issue 675](https://github.com/Snailclimb/JavaGuide/issues/675)）：在 Java 9 之后，String 、`StringBuilder` 与 `StringBuffer` 的实现改用 byte 数组存储字符串 `private final byte[] value`

而 `StringBuilder` 与 `StringBuffer` 都继承自 `AbstractStringBuilder` 类，在 `AbstractStringBuilder` 中也是使用字符数组保存字符串`char[]value` 但是没有用 `final` 关键字修饰，所以这两种对象都是可变的。



**线程安全性**

`String` 中的对象是不可变的，也就可以理解为常量，线程安全。`AbstractStringBuilder` 是 `StringBuilder` 与 `StringBuffer` 的公共父类，定义了一些字符串的基本操作，如 `expandCapacity`、`append`、`insert`、`indexOf` 等公共方法。`StringBuffer` 对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。`StringBuilder` 并没有对方法进行加同步锁，所以是非线程安全的。



**性能**

每次对 `String` 类型进行改变的时候，都会生成一个新的 `String` 对象，然后将指针指向新的 `String` 对象。`StringBuffer` 每次都会对 `StringBuffer` 对象本身进行操作，而不是生成新的对象并改变对象引用。相同情况下使用 `StringBuilder` 相比使用 `StringBuffer` 仅能获得 10%~15% 左右的性能提升，但却要冒多线程不安全的风险。



**对于三者使用的总结**

- 操作少量的数据: 适用 `String`

- 单线程操作字符串缓冲区下操作大量数据: 适用 `StringBuilder`

- 多线程操作字符串缓冲区下操作大量数据: 适用 `StringBuffer`



**格式化输出**

https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Formatter.html#syntax

| 占位符 | 说明                             |
| :----- | :------------------------------- |
| %d     | 格式化输出整数                   |
| %x     | 格式化输出十六进制整数           |
| %f     | 格式化输出浮点数                 |
| %e     | 格式化输出科学计数法表示的浮点数 |
| %s     | 格式化字符串                     |

注意，由于%表示占位符，因此，**连续两个%%表示一个%字符本身**。



### Array

**字符串数组**

字符串是引用类型，因此我们先定义一个字符串数组：

```
String[] names = {
    "ABC", "XYZ", "zoo"
};
```

对于`String[]`类型的数组变量`names`，它实际上包含3个元素，但每个元素都指向某个字符串对象：

```ascii
          ┌─────────────────────────┐
    names │   ┌─────────────────────┼───────────┐
      │   │   │                     │           │
      ▼   │   │                     ▼           ▼
┌───┬───┬─┴─┬─┴─┬───┬───────┬───┬───────┬───┬───────┬───┐
│   │░░░│░░░│░░░│   │ "ABC" │   │ "XYZ" │   │ "zoo" │   │
└───┴─┬─┴───┴───┴───┴───────┴───┴───────┴───┴───────┴───┘
      │                 ▲
      └─────────────────┘
```

对`names[1]`进行赋值，例如`names[1] = "cat";`，效果如下：

```ascii
          ┌─────────────────────────────────────────────────┐
    names │   ┌─────────────────────────────────┐           │
      │   │   │                                 │           │
      ▼   │   │                                 ▼           ▼
┌───┬───┬─┴─┬─┴─┬───┬───────┬───┬───────┬───┬───────┬───┬───────┬───┐
│   │░░░│░░░│░░░│   │ "ABC" │   │ "XYZ" │   │ "zoo" │   │ "cat" │   │
└───┴─┬─┴───┴───┴───┴───────┴───┴───────┴───┴───────┴───┴───────┴───┘
      │                 ▲
      └─────────────────┘
```

这里注意到原来`names[1]`指向的字符串`"XYZ"`并没有改变，仅仅是将`names[1]`的引用从指向`"XYZ"`改成了指向`"cat"`，其结果是字符串`"XYZ"`再也无法通过`names[1]`访问到了。



### Math

顾名思义，`Math`类就是用来进行数学计算的，它提供了大量的静态方法来便于我们实现数学计算：

取最大或最小值：

```java
Math.max(100, 99); // 100
Math.min(1.2, 2.3); // 1.2
```

生成一个随机数x，x的范围是`0 <= x < 1`：

```java
Math.random(); // 0.53907... 每次都不一样
```

计算xy次方：

```java
Math.pow(2, 10); // 2的10次方=1024
```

> Java标准库还提供了一个`StrictMath`，它提供了和`Math`几乎一模一样的方法。这两个类的区别在于，由于浮点数计算存在误差，不同的平台（例如x86和ARM）计算的结果可能不一致（指误差不同），因此，`StrictMath`保证所有平台计算结果都是完全相同的，而`Math`会尽量针对平台优化计算速度，所以，绝大多数情况下，使用`Math`就足够了。



### Random

`Random`用来创建伪随机数。

所谓伪随机数，是指只要给定一个初始的**种子**，产生的随机数序列是完全一样的。

要生成一个随机数，可以使用`nextInt()`、`nextLong()`、`nextFloat()`、`nextDouble()`：

```java
Random r = new Random();
r.nextInt(); // 2071575453,每次都不一样
r.nextInt(10); // 5,生成一个[0,10)之间的int
r.nextLong(); // 8811649292570369305,每次都不一样
r.nextFloat(); // 0.54335...生成一个[0,1)之间的float
r.nextDouble(); // 0.3716...生成一个[0,1)之间的double
```

每次运行程序，生成的随机数都是不同的，没看出伪随机数的特性来。这是因为我们创建`Random`实例时，如果不给定种子，就使用系统当前时间戳作为种子，因此每次运行时，种子不同

下面的例子为真正的伪随机数：

```java
 Random r = new Random(12345);
 for (int i = 0; i < 10; i++) {
     System.out.println(r.nextInt(100));
 }
 // 51, 80, 41, 28, 55...
```



### SecureRandom

真随机数，`SecureRandom`就是用来创建**安全的随机数**。

实际上真正的真随机数只能通过量子力学原理来获取，而我们想要的是一个不可预测的安全的随机数：

```java
SecureRandom sr = null;
try {
    sr = SecureRandom.getInstanceStrong(); // 获取高强度安全随机数生成器
} catch (NoSuchAlgorithmException e) {
    sr = new SecureRandom(); // 获取普通的安全随机数生成器
}
byte[] buffer = new byte[16];
sr.nextBytes(buffer); // 用安全随机数填充buffer
System.out.println(sr.nextInt(100));
System.out.println(Arrays.toString(buffer));
```



### Record

使用`String`、`Integer`等类型的时候，这些类型都是不变类，一个不变类具有以下特点：

1. 定义class时使用`final`，无法派生子类；
2. 每个字段使用`final`，保证创建实例后无法修改任何字段。

假设我们希望定义一个`Point`类，有`x`、`y`两个变量，同时它是一个不变类，可以这么写：

```java
public final class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int x() {
        return this.x;
    }

    public int y() {
        return this.y;
    }
}
```

为了保证不变类的比较，还需要正确覆写`equals()`和`hashCode()`方法，这样才能在集合类中正常使用。这些代码写起来都非常简单，但是很繁琐。

从 **Java 14** 开始，引入了新的`record`类。我们定义`Record`类时，使用关键字`record`。

```java
public record Point(int x, int y) {}
```



### JavaBean

JavaBean是特殊的、符合命名规范的 Java 类，主要用来传递数据

- 所有属性为private

- 通过`getter`和`setter`来定义属性

- 属性（`property`）是一种通用的叫法，并非Java语法规定

- 使用`Introspector.getBeanInfo()`可以获取属性列表



如果读写方法符合以下这种命名规范，那么这种`class`被称为`JavaBean`

```java
// 读方法:
public Type getXyz()
// 写方法:
public void setXyz(Type value)
```

> `getter`和`setter`也是一种数据封装的方法



**枚举JavaBean属性**

要枚举一个JavaBean的所有属性，可以直接使用Java核心库提供的`Introspector`：

```java
public class Main {
    public static void main(String[] args) throws Exception {
        BeanInfo info = Introspector.getBeanInfo(Person.class);
        for (PropertyDescriptor pd : info.getPropertyDescriptors()) {
            System.out.println(pd.getName());
            System.out.println("  " + pd.getReadMethod());
            System.out.println("  " + pd.getWriteMethod());
        }
    }
}

class Person {
    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```
