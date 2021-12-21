> 本文来自廖雪峰，郎涯进行简单排版与补充



> Java使用擦拭法实现泛型，编译器内部永远把所有类型`T`视为`Object`处理

## 使用泛型

Java 泛型（generics）是 JDK 5 中引入的一个新特性, 泛型提供了编译时类型安全检测机制，该机制允许程序员在编译时检测非法的类型。**泛型的本质是参数化类型**，也就是说所操作的数据类型被指定为一个参数。

- 泛型就是编写**模板代码**来适应任意类型

- 泛型的好处是使用时不必对类型进行强制转换，它通过编译器对类型进行检查

- 注意泛型的继承关系：可以把`ArrayList`向上转型为`List`（**`T`不能变**！），但不能把`List`向下转型为`ArrayList`（`T`不能变成父类）

> ArrayList<Integer>和ArrayList<Number>两者完全没有继承关系



### **泛型接口**

`Arrays.sort(Object[])` 可以对任意数组进行排序，但待排序的元素必须实现`Comparable`这个泛型接口：

```java
public interface Comparable<T> {
    /**
     * 返回-1: 当前实例比参数o小
     * 返回0: 当前实例与参数o相等
     * 返回1: 当前实例比参数o大
     */
    int compareTo(T o);
}
```

> `String`等内置类型已经实现了`Comparable`接口



如果换成我们自定义的`Person`类型，需要让`Person`实现`Comparable`接口，否则会报告`ClassCastException`的错误：

```java
// sort
import java.util.Arrays;

public class Main {
    public static void main(String[] args) {
        Person[] ps = new Person[] {
            new Person("Bob", 61),
            new Person("Alice", 88),
            new Person("Lily", 75),
        };
        Arrays.sort(ps);
        System.out.println(Arrays.toString(ps));
    }
}

//实现泛型接口，指定类型：
class Person implements Comparable<Person> {
    String name;
    int score;
    Person(String name, int score) {
        this.name = name;
        this.score = score;
    }
    @Override
    public int compareTo(Person other) {
        return this.name.compareTo(other.name);
    }
    public String toString() {
        return this.name + "," + this.score;
    }
}

实现泛型接口，不指定类型：
class Person<T> implements Comparable<T> {
}    
```



**常用的通配符为： T，E，K，V，？**

- ？ 表示不确定的 java 类型
- T (type) 表示具体的一个 java 类型
- K V (key value) 分别代表 java 键值中的 Key Value
- E (element) 代表 Element



### **总结**

- 使用泛型时，把泛型参数`T`替换为需要的class类型，例如：`ArrayList<String>`，`ArrayList<Number>`等

- 可以省略编译器能自动推断出的类型，例如：`List<String> list = new ArrayList<>();`

- 不指定泛型参数类型时，编译器会给出警告，且只能将`T`视为`Object`类型

- 可以在接口中定义泛型类型，实现此接口的类必须实现正确的泛型类型



## 编写泛型

### 编写泛型

通常来说，泛型类一般用在集合类中，例如`ArrayList`，我们很少需要编写泛型类

- 编写泛型时，需要定义泛型类型`<T>`

- 静态方法不能引用泛型类型`<T>`，必须定义其他类型（例如`<K>`）来实现静态泛型方法

- 泛型可以同时定义多种类型，例如`Map<K, V>`



如何自定义一个泛型类：

- 先按照某种类型，例如：`String`，来编写类

- 再把特定类型`String`替换为`T`，并在类头部申明`<T>`

```java
public class Pair<T> {
    private T first;
    private T last;
    public Pair(T first, T last) {
        this.first = first;
        this.last = last;
    }
    public T getFirst() {
        return first;
    }
    public T getLast() {
        return last;
    }
}
```



**静态方法**

编写泛型类时，要特别注意，**泛型类型`<T>`不能用于静态方法**。

对于静态方法，我们可以单独改写为“泛型”方法，只需要使用另一个类型即可。这样将静态方法的泛型类型和实例类型的泛型类型区分开。

```java
    // 静态泛型方法应该使用其他类型区分:
    public static <K> Pair<K> create(K first, K last) {
        return new Pair<K>(first, last);
    }
```



**多个泛型类型**

泛型还可以定义多种类型。例如，我们希望`Pair`不总是存储两个类型一样的对象，就可以使用类型`<T, K>`



#### 常用的通配符

- ？ 表示不确定的 Java 类型
- T (type) 表示具体的一个 Java 类型
- K V (key value) 分别代表 Java 键值中的 Key Value
- E (element) 代表 Element



#### 项目中哪里用到了泛型

对于基本数据类型来说，== 比较的是值。对于引用数据类型来说，==比较的是对象的内存地址。
- 可用于定义通用返回结果 `CommonResult<T>` 通过参数 `T` 可根据具体的返回类型动态指定结果的数据类型
- 定义 `Excel` 处理类 `ExcelUtil<T>` 用于动态指定 `Excel` 导出的数据类型
- 用于构建集合工具类。参考 `Collections` 中的 `sort`, `binarySearch` 方法
- ......



### 擦拭法

Java语言的泛型实现方式是擦拭法（Type Erasure），即虚拟机对泛型其实一无所知，所有的工作都是编译器做的。

例如上例是编译器看到的代码，而虚拟机根本不知道泛型。这是虚拟机执行的代码：

```java
public class Pair {
    private Object first;
    private Object last;
    public Pair(Object first, Object last) {
        this.first = first;
        this.last = last;
    }
    public Object getFirst() {
        return first;
    }
    public Object getLast() {
        return last;
    }
}
```

因此，**Java使用擦拭法实现泛型，编译器内部永远把所有类型`T`视为`Object`处理**，导致了：

- 编译器把类型`<T>`视为`Object`
- 编译器根据`<T>`实现安全的强制转型



例如如下非安全代码：

```java
List<Integer> list = new ArrayList<>();

list.add(12);
//这里直接添加会报错
list.add("a");

//但是通过反射添加，是可以的
Class<? extends List> clazz = list.getClass();
Method add = clazz.getDeclaredMethod("add", Object.class);
add.invoke(list, "kl");

System.out.println(list);
```



#### **Java泛型的局限**

局限一：`<T>`不能是基本类型，例如`int`，因为实际类型是`Object`，`Object`类型无法持有基本类型：

```java
Pair<int> p = new Pair<>(1, 2); // compile error!
```



局限二：无法取得带泛型的`Class`

```java
Pair<String> p1 = new Pair<>("Hello", "world");
Pair<Integer> p2 = new Pair<>(123, 456);
Class c1 = p1.getClass();
Class c2 = p2.getClass();
System.out.println(c1==c2); // true
System.out.println(c1==Pair.class); // true
```

所有泛型实例，无论`T`的类型是什么，`getClass()`返回同一个`Class`实例，因为编译后它们全部都是`Pair<Object>`。



局限三：无法判断带泛型的`Class`：

```java
Pair<Integer> p = new Pair<>(123, 456);
// Compile error:
if (p instanceof Pair<String>.class) {
}
```

原因和前面一样，并不存在Pair<String>.class，而是只有唯一的`Pair.class`。

 

局限四：不能实例化`T`类型：

```java
public class Pair<T> {
    private T first;
    private T last;
    public Pair() {
        // Compile error:
        first = new T();
        last = new T();
    }
}
```

上述代码无法通过编译，因为构造方法的两行语句：

```java
first = new T();
last = new T();
```

擦拭后实际上变成了：

```java
first = new Object();
last = new Object();
```

这样一来，创建`new Pair()`和创建`new Pair()`就全部成了`Object`，显然编译器要阻止这种类型不对的代码。

要实例化`T`类型，我们必须借助额外的`Class`参数：

```java
public class Pair<T> {
    private T first;
    private T last;
    public Pair(Class<T> clazz) {
        first = clazz.newInstance();
        last = clazz.newInstance();
    }
}
```

上述代码借助`Class`参数并通过反射来实例化`T`类型，使用的时候，也必须传入`Class`。例如：

```java
Pair<String> pair = new Pair<>(String.class);
```



#### **不恰当的覆写方法**

```java
public class Pair<T> {
    public boolean equals(T t) {
        return this == t;
    }
}
```

定义的`equals(T t)`方法实际上会被擦拭成`equals(Object t)`，而这个方法是继承自`Object`的，编译器会阻止一个实际上会变成覆写的泛型方法定义。

此时换个方法名即可。



#### **泛型继承**

子类可以获取父类的泛型类型`<T>`

```java
public class IntPair extends Pair<Integer> {
}
```

在父类是泛型类型的情况下，编译器就必须把类型`T`（对`IntPair`来说，也就是`Integer`类型）保存到子类的class文件中，不然编译器就不知道`IntPair`只能存取`Integer`这种类型。

在继承了泛型类型的情况下，子类可以获取父类的泛型类型。例如：`IntPair`可以获取到父类的泛型类型`Integer`。

```java
Class<IntPair> clazz = IntPair.class;
Type t = clazz.getGenericSuperclass();
if (t instanceof ParameterizedType) {
   ParameterizedType pt = (ParameterizedType) t;
   Type[] types = pt.getActualTypeArguments(); // 可能有多个泛型类型
   Type firstType = types[0]; // 取第一个泛型类型
   Class<?> typeClass = (Class<?>) firstType;
   System.out.println(typeClass); // Integer
}
```

```ascii
                      ┌────┐
                      │Type│
                      └────┘
                         ▲
                         │
   ┌────────────┬────────┴─────────┬───────────────┐
   │            │                  │               │
┌─────┐┌─────────────────┐┌────────────────┐┌────────────┐
│Class││ParameterizedType││GenericArrayType││WildcardType│
└─────┘└─────────────────┘└────────────────┘└────────────┘
```



### extends通配符

因为`Pair<Integer>`不是`Pair<Number>`的子类，因此，`add(Pair<Number>)`不接受参数类型`Pair<Integer>`。

使用`Pair<? extends Number>`使得方法接收所有泛型类型为`Number`或`Number`子类的`Pair`类型。

```java
static int add(Pair<? extends Number> p) {
    Number first = p.getFirst();
    Number last = p.getLast();
    return first.intValue() + last.intValue();
}
```

这种使用`<? extends Number>`的泛型定义称之为**上界通配符**（Upper Bounds Wildcards），即把泛型类型`T`的上界限定在`Number`了。



#### **重要限制**

泛型内部的方法参数签名`setFirst(? extends Number)`无法传递任何`Number`类型给`setFirst(? extends Number)`

原因还在于擦拭法。如果我们传入的`p`是`Pair<Double>`，显然它满足参数定义`Pair<? extends Number>`，然而，`Pair<Double>`的`setFirst()`显然无法接受`Integer`类型。



#### **extends通配符的作用**

- 允许调用`get()`方法获取`Integer`的引用；
- 不允许调用`set(? extends Integer)`方法并传入任何`Integer`的引用（`null`除外）。

```java
int sumOfList(List<? extends Integer> list) {
    int sum = 0;
    for (int i=0; i<list.size(); i++) {
        Integer n = list.get(i);
        sum = sum + n;
    }
    return sum;
}
```

因此，方法参数类型`List`表明了该方法内部只会读取`List`的元素，不会修改`List`的元素（因为无法调用`add(? extends Integer)`、`remove(? extends Integer)`这些方法。

> 即使用`extends`通配符表示可以读，不能写。（恶意调用`set(null)`除外）。



#### **使用extends限定T类型**

在定义泛型类型`Pair`的时候，也可以使用`extends`通配符来限定`T`的类型：

```java
public class Pair<T extends Number> { ... }
```



### super通配符

使用类似`<? super Integer>`通配符作为方法参数时表示：

- 方法内部可以调用传入`Integer`引用的方法，例如：`obj.setFirst(Integer n);`；
- 方法内部无法调用获取`Integer`引用的方法（`Object`除外），例如：`Integer n = obj.getFirst();`。

即使用`super`通配符表示只能写不能读。唯一例外是可以获取`Object`的引用：`Object o = p.getFirst()`。

使用`extends`和`super`通配符要遵循PECS原则。

无限定通配符`<?>`很少使用，可以用`<T>`替换，同时它是所有`<T>`类型的超类。

```java
public class Main {
    public static void main(String[] args) {
        Pair<Number> p1 = new Pair<>(12.3, 4.56);
        Pair<Integer> p2 = new Pair<>(123, 456);
        setSame(p1, 100);
        setSame(p2, 200);
        System.out.println(p1.getFirst() + ", " + p1.getLast());
        System.out.println(p2.getFirst() + ", " + p2.getLast());
    }

    static void setSame(Pair<? super Integer> p, Integer n) {
        p.setFirst(n);
        p.setLast(n);
    }
}
```



#### **extends vs super**

我们再回顾一下`extends`通配符。作为方法参数，`<? extends T>`类型和`<? super T>`类型的区别在于：

- `<? extends T>`**允许调用读方法**`T get()`获取`T`的引用，但不允许调用写方法`set(T)`传入`T`的引用（传入`null`除外）；
- `<? super T>`**允许调用写方法**`set(T)`传入`T`的引用，但不允许调用读方法`T get()`获取`T`的引用（获取`Object`除外）。

一个是允许读不允许写，另一个是允许写不允许读。

先记住上面的结论，我们来看Java标准库的`Collections`类定义的`copy()`方法：

```java
public class Collections {
    // 把src的每个元素复制到dest中:
    public static <T> void copy(List<? super T> dest, List<? extends T> src) {
        for (int i=0; i<src.size(); i++) {
            T t = src.get(i);
            dest.add(t);
        }
    }
}
```



#### **PECS原则**

Producer Extends Consumer Super。

如果需要返回`T`，它是生产者（Producer），要使用`extends`通配符；

如果需要写入`T`，它是消费者（Consumer），要使用`super`通配符。



#### **无限定通配符**

```java
void sample(Pair<?> p) {
}
```

既不能读，也不能写，那只能做一些`null`判断



`<?>`通配符有一个独特的特点，就是：`Pair<?>`是所有`Pair<T>`的超类：

```java
public static void main(String[] args) {
   Pair<Integer> p = new Pair<>(123, 456);
   Pair<?> p2 = p; // 安全地向上转型
   System.out.println(p2.getFirst() + ", " + p2.getLast());
}
```

上述代码是可以正常编译运行的，因为`Pair<Integer>`是`Pair<?>`的子类，可以安全地向上转型。



## 泛型与反射

- 可以声明带泛型的数组，但不能直接创建带泛型的数组，必须强制转型。因为泛型数组`T[]`擦拭后代码变为`Object[]`

- 可以通过`Array.newInstance(Class, int)`创建`T[]`数组，需要强制转型



Java的部分反射API也是泛型。例如：`Class`就是泛型，构造方法`Constructor`也是泛型：

```java
Class<Integer> clazz = Integer.class;
Constructor<Integer> cons = clazz.getConstructor(int.class);
Integer i = cons.newInstance(123);
```

我们可以声明带泛型的数组，但不能用`new`操作符创建带泛型的数组：

```java
Pair<String>[] ps = null; // ok
Pair<String>[] ps = new Pair<String>[2]; // compile error!
```



使用泛型数组要特别小心，因为数组实际上在运行期没有泛型，编译器可以强制检查变量`ps`，因为它的类型是泛型数组，但编译器不会检查变量`arr`，因为它不是泛型数组。

因为这两个变量实际上指向同一个数组，所以，操作`arr`可能导致从`ps`获取元素时报错，例如，以下代码演示了不安全地使用带泛型的数组：

```java
Pair[] arr = new Pair[2];
Pair<String>[] ps = (Pair<String>[]) arr;

ps[0] = new Pair<String>("a", "b");
arr[1] = new Pair<Integer>(1, 2);

// ClassCastException:
Pair<String> p = ps[1];
String s = p.getFirst();
```



要安全地使用泛型数组，必须扔掉`arr`的引用。必须通过强制转型实现带泛型的数组：

```java
@SuppressWarnings("unchecked")
Pair<String>[] ps = (Pair<String>[]) new Pair[2];
```



内部必须借助`Class`来创建泛型数组：

```java
T[] createArray(Class<T> cls) {
    return (T[]) Array.newInstance(cls, 5);
}
```



我们还可以利用可变参数创建泛型数组`T[]`：

```java
public class ArrayHelper {
    @SafeVarargs
    static <T> T[] asArray(T... objs) {
        return objs;
    }
}

String[] ss = ArrayHelper.asArray("a", "b", "c");
Integer[] ns = ArrayHelper.asArray(1, 2, 3);
```

>  如果在方法内部创建了泛型数组，最好不要将它返回给外部使用
