Java 不支持单独定义函数，但可以把静态方法视为独立的函数，把实例方法视为自带 `this` 参数的函数。

Java 平台从 **Java8** 开始，支持函数式编程。函数式编程（Functional Programming）就是一种抽象程度很高的编程范式，把函数作为基本运算单元，函数可以作为变量，还可以返回函数。Java 8 中的函数（方法）是“值”的一种新的形式，可以将方法作为参数进行传递。而作为参数进行传递的方法主要是 `Lambda表达式` 和 `方法引用`。

其中：

- 单方法接口被称为 `FunctionalInterface`

- 接收 `FunctionalInterface` 作为参数的时候，可以把实例化的匿名类改写为 Lambda 表达式，能大大简化代码

- Lambda 表达式的参数和返回值均可由编译器自动推断

> 历史上研究函数式编程的理论是 Lambda 演算，所以我们经常把支持函数式编程的编码风格称为 Lambda 表达式。

​    

## Lambda 表达式

> Lambda表达式是一种匿名函数，在函数式编程里，它可以作为参数进行传递。

在 Java 程序中，我们经常遇到一大堆单方法接口，即一个接口只定义了一个方法：

- Comparator
- Runnable
- Callable

以`Comparator`为例，我们想要调用`Arrays.sort()`时，可以传入一个`Comparator`实例

**匿名类**

以匿名类方式编写如下：

```java
String[] array = ...
Arrays.sort(array, new Comparator<String>() {
    public int compare(String s1, String s2) {
        return s1.compareTo(s2);
    }
});
```

**Lambda 表达式**

上述写法非常繁琐。从 Java 8 开始，我们可以用 Lambda 表达式替换单方法接口。改写上述代码如下：

```java
public class Main {
    public static void main(String[] args) {
        String[] array = new String[] { "Apple", "Orange", "Banana", "Lemon" };
        Arrays.sort(array, (s1, s2) -> {
            return s1.compareTo(s2);
        });
        System.out.println(String.join(", ", array));
    }
}
```

观察 Lambda 表达式的写法，它只需要写出方法定义：

即 Lambada表达式作为 sort 方法的参数，是 `Comparator` 函数接口的实现

```java
(s1, s2) -> {
    return s1.compareTo(s2);
}
```



## 函数接口

上面的例子中涉及的函数接口都标记了 `@FunctionalInterface` 注解。我们把只定义了单方法的接口称之为 `FunctionalInterface`，用注解 `@FunctionalInterface` 标记。例如，`Callable`接口：

```java
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

> 从 Java 8 开始，接口内不仅可以有抽象方法，**还可以有静态方法和默认方法**。只要符合定义，即使没有标记@FunctionalInterface，它也是函数接口。如果不符合函数接口的定义，那么即使标记了 @FunctionalInterface，编译器也会报错，这就是 @FunctionalInterface 的作用。



### 类型

函数接口主要位于 java.util.function 包下，可分成下面几类：

- `Predicate`

  有输入且只输出布尔值的函数。

- `Function`

  有输入有输出的函数。

- `Consumer`

  有输入无输出的函数。

- `Supplier`

  无输入有输出的函数。

- `Operator`

  输入和输出为相同类型的函数。



### Function

这里以 `Function` 为例详细说明，其余类型类似。Function（函数）的源码定义如下：

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);

    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }

    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }

	static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```

Lambda 表达式是 apply 方法的实现，apply 方法可接收任意类型的参数 T，返回值类型为 R。

例如：入参 T 类型为 String，返回值 R 类型为Integer

```java
Function<String, Integer> lenFunction = (str) -> {return str.length();};
//输出的返回值的长度为 5
System.out.println("apple length:" + lenFunction.apply("apple"));
```



**组合Function**

Function 接口函数还提供了 `andThen` 和 `compose` 方法来组合已有的 Function，组合 Function 的返回值仍为 Function

`andThen`：新的Function是把组合中第一个函数的返回值作为第二个函数的输入

`compose`：新的Function是把组合中第二个函数的返回值作为第一个函数的输入

例如：

```java
//functional
Function<String, Integer> lenFunction = (str) -> {return str.length();};
Function<Integer, Integer> multiFunction = x -> x*x;

//第一个函数的返回值作为第二个函数的输入
Function<String, Integer> andFunction = lenFunction.andThen(multiFunction);
//打印 25
System.out.println("andFunction:" + andFunction.apply("apple"));
```



**数据类型 Function**

为了解决包装类型的性能损失问题，针对原始数据类型提供了一些特殊定义的Function

- 入参固定类型

  IntFunction、LongFunction、DoubleFunction

- 返回值固定类型

  ToIntFunction、ToLongFunction、ToDoubleFunction

- 入参和返回值都固定类型

  IntToLongFunction、IntToDoubleFunction、LongToIntFunction等



**多参数 Function**

同时还提供了 2 个入参的 `BiFunction`



### 自定义函数接口

函数接口的定义主要是看入参和返回值，例如定义3个入参+1个返回值的接口：

```java
/**
 * 接收三个入参
 */
@FunctionalInterface
public interface TriFunction <T, U, K, R>
{
    R apply(T t, U u, K k);
}
```

调用示例：

```java
//自定义Function
TriFunction<String, String, String, Integer> allLenFunction =
        (str1, str2, str3) -> str1.length() + str2.length() + str3.length();
System.out.println("all length:" + allLenFunction.apply("Apple", "Orange", "Banana"));
```



## 方法引用

可以把方法引用作为方法的参数使用，在Java中，方法引用使用 `::` 表示。

`FunctionalInterface`允许传入：

- 接口的实现类（很繁琐）

- Lambda 表达式

- 静态方法

  类名::静态方法

- 实例方法

  实例对象名::实例方法。**实例类型this隐式被看做第一个参数类型**

- 构造方法

  类名::new。实例类型被看做返回类型

  

静态方式、构造方法比较好理解，这里以**实例方法**举例说明：

````java
Arrays.sort(array, String::compareTo);	
````

因为实例方法本质上有一个隐含的 `this` 参数，`String`类的`compareTo()`方法在实际调用的时候，第一个隐含参数总是传入`this`，相当于静态方法：

```java
public static int compareTo(this, String o);
```

所以，`String.compareTo()`方法也可作为方法引用传入。

> 在调用现有类的已有方法时，方法引用比 Lambda 表达式更自然，可读性更强



## Stream

Java 8 开始，引入了一个全新的流式 Stream API，特点是：

- 提供了一套新的流式处理的抽象序列
- 支持函数式编程和链式操作
- 可以表示无限序列，并且大多数情况下是 **惰性求值** 的

不同于`java.io`的`InputStream`和`OutputStream`，它代表的是任意Java对象的序列

|      | java.io                  | java.util.stream             |
| :--- | :----------------------- | :--------------------------- |
| 存储 | 顺序读写的`byte`或`char` | 顺序输出的任意 Java 对象实例 |
| 用途 | 序列化至文件或网络       | 内存计算／业务逻辑           |

不同于List，`List`存储的每个元素都是已经存储在内存中的某个 Java 对象，而`Stream`输出的元素可能并没有预先存储在内存中，而是实时计算出来的

|      | java.util.List           | java.util.stream     |
| :--- | :----------------------- | -------------------- |
| 元素 | 已分配并存储在内存       | 可能未分配，实时计算 |
| 用途 | 操作一组已存在的Java对象 | 惰性计算             |



`Stream`提供的常用操作有：

转换操作：`map()`，`filter()`，`sorted()`，`distinct()`；

合并操作：`concat()`，`flatMap()`；

并行处理：`parallel()`；

聚合操作：`reduce()`，`collect()`，`count()`，`max()`，`min()`，`sum()`，`average()`；

其他操作：`allMatch()`, `anyMatch()`, `forEach()`，`findAny`、`findFirst`



一般来说，可以从数据源（集合类、数组）获得 Stream，而 Stream 就是数据序列，我们可以对数据序列进行各种数据处理操作（过滤、转换、排序、查询等）。

在进行 Stream 开发时只需以下三步：

- 从数据源获得Stream
- 组成处理管道
- 从管道中产生处理结果

```java
int result = createNaturalStream() // 从数据源获得Stream
             .filter(n -> n % 2 == 0) // 组成处理管道
             .map(n -> n * n) // 组成处理管道
             .limit(100) // 组成处理管道
             .sum(); // 从管道中产生处理结果
```



### 创建 Stream

- 通过指定元素 or 数组、`Collection` 创建 `Stream`
- 通过 `Supplier` 创建 `Stream`，可以是无限序列
- 通过其他类的相关方法创建
- 基本类型的`Stream`有`IntStream`、`LongStream`和`DoubleStream`



**基于Stream.of**

```java
Stream<String> stream = Stream.of("A", "B", "C", "D");
```



**基于数组Arrays.stream或Collection**

```java
public class Main {
    public static void main(String[] args) {
        Stream<String> stream1 = Arrays.stream(new String[] { "A", "B", "C" });
        Stream<String> stream2 = List.of("X", "Y", "Z").stream();
        stream1.forEach(System.out::println);
        stream2.forEach(System.out::println);
    }
}
```

> 对于`Collection`（`List`、`Set`、`Queue`等），直接调用`stream()`方法就可以获得`Stream`。



**基于Supplier**

创建`Stream`还可以通过`Stream.generate()`方法，它需要传入一个`Supplier`对象：

```java
Stream<String> s = Stream.generate(Supplier<String> sp);
```

基于`Supplier`创建的`Stream`会不断调用`Supplier.get()`方法来不断产生下一个元素

```java
public class Main {
    public static void main(String[] args) {
        Stream<Integer> natual = Stream.generate(new NatualSupplier());
        
        // 注意：无限序列必须先变成有限序列再打印:
        natual.limit(20).forEach(System.out::println);
    }
}

class NatualSupplier implements Supplier<Integer> {
    int n = 0;
    public Integer get() {
        n++;
        return n;
    }
}
```



**其他方法**

创建`Stream`的第三种方法是通过类提供的接口，直接获得`Stream`。

例如，`Files`类的`lines()`方法可以把一个文件变成一个`Stream`，每个元素代表文件的一行内容：

```java
try (Stream<String> lines = Files.lines(Paths.get("/path/to/file.txt"))) {
    ...
}
```



**基本类型**

`因为 Java 的范型不支持基本类型，所以我们无法用 Stream<int> 这样的类型，会发生编译错误。` 为了保存`int`，只能使用`Stream<Integer>`，但这样会产生频繁的装箱、拆箱操作。为了提高效率，Java 标准库提供了`IntStream`、`LongStream`和`DoubleStream`这三种使用基本类型的`Stream`：

```java
// 将int[]数组变为IntStream:
IntStream is = Arrays.stream(new int[] { 1, 2, 3 });

// 将Stream<String>转换为LongStream:
LongStream ls = List.of("1", "2", "3")
    .stream()
    .mapToLong(Long::parseLong);
```



### 中间操作

**map**

可以将一种元素类型转换成另一种元素类型

```java
public class Main {
    public static void main(String[] args) {
        List.of("Apple", "Orange", "Banana")
            .stream()
            .map(String::trim)
            .map(String::toLowerCase)
            .filter((str) -> str.length() > 5)
            .forEach(System.out::println);
    }
}
```



**flatMap**

把`Stream`的每个元素（例如`List`）映射为`Stream`，然后合并成一个新的`Stream`。

例如`Stream`的元素是集合：

```java
Stream<List<Integer>> s = Stream.of(
        Arrays.asList(1, 2, 3),
        Arrays.asList(4, 5, 6),
        Arrays.asList(7, 8, 9));
```

而我们希望把上述`Stream`转换为`Stream<Integer>`，就可以使用`flatMap()`：

```java
Stream<Integer> i = s.flatMap(list -> list.stream());
```

```ascii
┌─────────────┬─────────────┬─────────────┐
│┌───┬───┬───┐│┌───┬───┬───┐│┌───┬───┬───┐│
││ 1 │ 2 │ 3 │││ 4 │ 5 │ 6 │││ 7 │ 8 │ 9 ││
│└───┴───┴───┘│└───┴───┴───┘│└───┴───┴───┘│
└─────────────┴─────────────┴─────────────┘
                     │
                     │flatMap(List -> Stream)
                     │
                     ▼
   ┌───┬───┬───┬───┬───┬───┬───┬───┬───┐
   │ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │ 7 │ 8 │ 9 │
   └───┴───┴───┴───┴───┴───┴───┴───┴───┘
```



**limit**

截取操作常用于把一个无限的`Stream`转换成有限的`Stream`，`skip()`用于跳过当前`Stream`的前N个元素，`limit()`用于截取当前`Stream`最多前N个元素：

```java
List.of("A", "B", "C", "D", "E", "F")
    .stream()
    .skip(2) // 跳过A, B
    .limit(3) // 截取C, D, E
    .collect(Collectors.toList()); // [C, D, E]
```



**filter**

使用`filter()`方法可以对一个`Stream`的每个元素进行测试，通过测试的元素被过滤后生成一个新的`Stream`。

例如，从一组给定的`LocalDate`中过滤掉工作日，以便得到休息日：

```java
public class Main {
    public static void main(String[] args) {
        Stream.generate(new LocalDateSupplier())
                .limit(31)
                .filter(ldt -> ldt.getDayOfWeek() == DayOfWeek.SATURDAY || ldt.getDayOfWeek() == DayOfWeek.SUNDAY)
                .forEach(System.out::println);
    }
}

class LocalDateSupplier implements Supplier<LocalDate> {
    LocalDate start = LocalDate.of(2020, 1, 1);
    int n = -1;
    public LocalDate get() {
        n++;
        return start.plusDays(n);
    }
}
```



**foreach**

```java
List<String> list = List.of("Apple", "Orange", "Banana");
list.stream().forEach(System.out::println);
```



### 输出操作

**输出为List**

```java
Stream<String> stream = Stream.of("Apple", "", null, "Pear", "  ", "Orange");
List<String> list = stream.filter(s -> s != null && !s.isBlank()).collect(Collectors.toList());
System.out.println(list);
```

类似的，`collect(Collectors.toSet())`可以把`Stream`的每个元素收集到`Set`中。



**输出为Map**

因为对于每个元素，添加到Map时需要key和value，因此，我们要指定两个映射函数，分别把元素映射为key和value：

```java
public class Main {
    public static void main(String[] args) {
        Stream<String> stream = Stream.of("APPL:Apple", "MSFT:Microsoft");
        Map<String, String> map = stream
                .collect(Collectors.toMap(
                        // 把元素s映射为key:
                        s -> s.substring(0, s.indexOf(':')),
                        // 把元素s映射为value:
                        s -> s.substring(s.indexOf(':') + 1)));
        System.out.println(map);
    }
}
```



**分组输出**

```java
public class Main {
    public static void main(String[] args) {
        List<String> list = List.of("Apple", "Banana", "Blackberry", "Coconut", "Avocado", "Cherry", "Apricots");
        Map<String, List<String>> groups = list.stream()
                .collect(Collectors.groupingBy(s -> s.substring(0, 1), Collectors.toList()));
        System.out.println(groups);
    }
}
```



**输出为数组**

```java
List<String> list = List.of("Apple", "Banana", "Orange");
String[] array = list.stream().toArray(String[]::new);
```



**使用reduce**

`reduce()`方法将一个`Stream`的每个元素依次计算并将结果合并，例如：

```java
public class Main {
    public static void main(String[] args) {
        int sum = Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9).reduce(0, (acc, n) -> acc + n);
        System.out.println(sum); // 45
    }
}
```



## Optional

Optional 类是可以解决空指针异常 `NullPointException` 的问题，可以作为任意类型的容器，在对象值不为空的时候返回值。当值为空时，可以预先做处理，而不是抛出空指针异常。

主要有以下方法：

- Optional.of

  包含非 null 值的 Optional

- Optional.ofNullable

  包含 null 值的 Optional。

  若参数不为 null，则返回包含参数的 Optional；若参数为 null，则返回空的 Optional

- isPresent

  存在检查使用

- isEmpty

  为空检查使用

```java
String str = null;
Optional<String> stringOptional = Optional.ofNullable(str);
if(stringOptional.isPresent())
{
    //此时下面的代码不会执行
    System.out.println(stringOptional);
}
```



> 参考文档：廖雪峰等
