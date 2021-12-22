> 本文由 JavaGuide 翻译
>
> 原文地址：https://www.baeldung.com/foreach-java



在 Java 8 中引入的 *forEach* 循环为程序员提供了一种新的，简洁而有趣的迭代集合的方式。

## 基础知识

```Java
public interface Collection<E> extends Iterable<E>
```

Collection 接口实现了 Iterable 接口，而 Iterable 接口在 Java 8 开始具有一个新的 API：

```java
//对 Iterable的每个元素执行给定的操作，直到所有元素都被处理或动作引发异常
void forEach(Consumer<? super T> action)
```



使用 *forEach*，我们可以迭代一个集合并对每个元素执行给定的操作，就像任何其他*迭代器一样。*

例如，迭代和打印字符串集合的  for 循环版本：

```java
for (String name : names) {
    System.out.println(name);
}
```

我们可以使用 *forEach* 写这个 ：

```java
names.forEach(name -> {
    System.out.println(name);
});
```



## forEach

### 匿名类

我们使用  *forEach* 迭代集合并对每个元素执行特定操作。**要执行的操作包含在实现 Consumer 接口的类中，并作为参数传递给 forEach 。**

所述 *消费者* 接口是一个功能接口(具有单个抽象方法的接口）。它接受输入并且不返回任何结果。

Consumer 接口定义如下：

```java
@FunctionalInterface
public interface Consumer {
    void accept(T t);
}
```
任何实现，例如，只是打印字符串的消费者：

```java
Consumer<String> printConsumer = new Consumer<String>() {
    public void accept(String name) {
        System.out.println(name);
    };
};
```

可以作为参数传递给 *forEach*：

```java
names.forEach(printConsumer);
```

但这不是通过消费者和使用 *forEach* API 创建操作的唯一方法。让我们看看我们将使用 *forEach* 方法的另外2种最流行的方式：



### Lambda

Java 8 功能接口的主要优点是我们可以使用 Lambda 表达式来实例化它们，并避免使用庞大的匿名类实现。

由于 Consumer 接口属于函数式接口，我们可以通过以下形式在Lambda中表达它：

```java
(argument) -> { body }
name -> System.out.println(name)
names.forEach(name -> System.out.println(name));
```



### 方法参考

我们可以使用方法引用语法而不是普通的 Lambda 语法，其中已存在一个方法来对类执行操作：

```java
names.forEach(System.out::println);
```



## forEach在集合中的使用

### 迭代集合

**任何类型 Collection 的可迭代  - 列表，集合，队列 等都具有使用 forEach 的相同语法。**

List：

```java
List<String> names = Arrays.asList("Larry", "Steve", "James");
 
names.forEach(System.out::println);
```



Set：

```java
Set<String> uniqueNames = new HashSet<>(Arrays.asList("Larry", "Steve", "James"));
 
uniqueNames.forEach(System.out::println);
```



Queue：

```java
Queue<String> namesQueue = new ArrayDeque<>(Arrays.asList("Larry", "Steve", "James"));
 
namesQueue.forEach(System.out::println);
```



### 迭代Map

Map没有实现 Iterable 接口，但它**提供了自己的 forEach 变体，它接受 BiConsumer**。* 

```java
Map<Integer, String> namesMap = new HashMap<>();
namesMap.put(1, "Larry");
namesMap.put(2, "Steve");
namesMap.put(3, "James");

namesMap.forEach((key, value) -> System.out.println(key + " " + value));

namesMap.entrySet().forEach(entry -> System.out.println(entry.getKey() + " " + entry.getValue()));
```

