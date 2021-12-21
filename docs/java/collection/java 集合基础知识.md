> 本文来自JavaGuide、廖雪峰，郎涯进行简单排版与补充



## 集合概述

### 为什么要使用集合

数组有如下限制：

- 数组初始化后大小不可变
- 数组只能按索引顺序存取

而集合提高了数据存储的灵活性，Java 集合不仅可以用来存储不同类型不同数量的对象，还可以保存具有映射关系的数据



### Java 集合概览

Java 集合也叫作容器，定义在 `java.util` 包中，支持泛型。主要是由两大接口派生而来：一个是 `Collection` 接口，主要用于存放单一元素；另一个是 `Map` 接口，主要用于存放键值对。

Java 集合使用统一的 `Iterator` 遍历，尽量不要使用遗留接口。

![](https://img-note.langyastudio.com/202110131537391.png?x-oss-process=style/watermark)


注：图中只列举了主要的继承派生关系，并没有列举所有关系。比方省略了 `AbstractList`,  `NavigableSet` 等抽象类以及其他的一些辅助类，如想深入了解，可自行查看源码。



Java 集合的设计有几个特点：

- 实现了接口和实现类相分离，例如有序表的接口是 `List`，具体的实现类有 `ArrayList`，`LinkedList` 等

- 支持泛型，我们可以限制在一个集合中只能放入同一种数据类型的元素



### List Set Queue Map 的区别

- `List`

  对付顺序的好帮手。存储的元素是有序的、可重复的、允许元素为null

- `Set`

  注重独一无二的性质。存储的元素是无序的、不可重复的

- `Queue`

  实现排队功能的叫号机。按特定的排队规则来确定先后顺序，存储的元素是有序的、可重复的

- `Map`

  用 key 来搜索的专家。 使用键值对（key-value）存储，key 是无序的、不可重复的



### 集合框架底层数据结构

#### List

- `Arraylist`： `Object[]` 数组
- `Vector`：`Object[]` 数组，线程安全
- `LinkedList`： 双向链表 (JDK1.6 之前为循环链表，JDK1.7 取消了循环)

#### Set

- `HashSet`(无序，唯一): 基于 `HashMap` 实现的，底层采用 `HashMap` 来保存元素
- `LinkedHashSet`: `LinkedHashSet` 是 `HashSet` 的子类，并且其内部是通过 `LinkedHashMap` 来实现的。有点类似于我们之前说的 `LinkedHashMap` 其内部是基于 `HashMap` 实现一样，不过还是有一点点区别的
- `TreeSet`(有序，唯一): 红黑树(自平衡的排序二叉树)

#### Queue

- `PriorityQueue`: `Object[]` 数组来实现二叉堆
- `ArrayQueue`: `Object[]` 数组 + 双指针

#### Map

- `HashMap`

  JDK1.8 之前 `HashMap` 由数组+链表组成的，数组是 `HashMap` 的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）

  JDK1.8 以后在解决哈希冲突时有了较大的变化，散列表容量大于64且链表大于8时，转成红黑树，以减少搜索时间

  允许为null，存储无序

- `LinkedHashMap`

  底层是散列表+红黑树+双向链表，父类是HashMap。提供插入顺序和访问顺序两种，访问顺序是符合LRU算法的，一般用于扩展(默认是插入顺序)，详细可以查看：[《LinkedHashMap 源码详细分析（JDK1.8）》](https://www.imooc.com/article/22931)

  迭代与初始容量无关(迭代的是维护的双向链表)

  允许为null，插入有序

- `Hashtable`

  线程安全。数组+链表组成的，数组是 `Hashtable` 的主体，链表则是主要为了解决哈希冲突而存在的

- `TreeMap`

  红黑树（自平衡的排序二叉树），保证了时间复杂度为log(n)

  可以对其进行排序，使用Comparator或者Comparable

  元素不能为null

- `ConcurrentHashMap`

    JDK1.8 以后散列表+红黑树，线程安全

    元素不能为null

    在高并发环境下，统计数据(计算size…等等)其实是无意义的，因为在下一时刻size值就变化了



由于 Java 的集合设计非常久远，中间经历过大规模改进，我们要注意到有一小部分集合类是遗留类，不应该继续使用：

- `Vector`：一种线程安全的`List`实现
- `Stack`：基于`Vector`实现的`LIFO`的栈
- `Hashtable`：一种线程安全的`Map`实现

还有一小部分接口是遗留接口，也不应该继续使用：

- `Enumeration`：已被 `Iterator` 取代



### 如何选用集合

主要根据集合的特点来选用

- 需要根据键值获取到元素值时就选用 `Map` 接口下的集合

  需要排序时选择 `TreeMap`，不需要排序时就选择 `HashMap`，需要保证线程安全就选用 `ConcurrentHashMap`。

- 只需要存放元素值时，就选择实现 `Collection` 接口的集合

  需要保证元素唯一时选择实现 `Set` 接口的集合比如 `TreeSet` 或 `HashSet`，不需要就选择实现 `List` 接口的比如 `ArrayList` 或 `LinkedList`，然后再根据实现这些接口的集合的特点来选用。



### `hashCode()`与 `equals()` 的相关规定

- 如果`equals()`返回`true`，则`hashCode()`返回值必须相等
- 如果`equals()`返回`false`，则`hashCode()`返回值尽量不要相等

综上，`equals()` 方法被覆盖过，则 `hashCode()` 方法也必须被覆盖

`hashCode()` 的默认行为是对堆上的对象产生独特值。如果没有重写 `hashCode()`，则该 class 的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）。

> `equals()`用到的用于比较的每一个字段，都必须在`hashCode()`中用于计算；`equals()`中没有使用到的字段，绝不可放在`hashCode()`中计算



`equals()` 方法要求我们必须满足以下条件：

- 自反性（Reflexive）：对于非`null`的`x`来说，`x.equals(x)`必须返回`true`
- 对称性（Symmetric）：对于非`null`的`x`和`y`来说，如果`x.equals(y)`为`true`，则`y.equals(x)`也必须为`true`
- 传递性（Transitive）：对于非`null`的`x`、`y`和`z`来说，如果`x.equals(y)`为`true`，`y.equals(z)`也为`true`，那么`x.equals(z)`也必须为`true`
- 一致性（Consistent）：对于非`null`的`x`和`y`来说，只要`x`和`y`状态不变，则`x.equals(y)`总是一致地返回`true`或者`false`
- 对`null`的比较：即`x.equals(null)`永远返回`false`



## List

`List` 的行为和数组几乎完全相同：`List `内部按照放入元素的先后顺序存放，每个元素都可以通过索引确定自己的位置，`List` 的索引和数组一样，从 `0` 开始。在实际应用中，需要增删元素的有序列表，使用最多的是 `ArrayList`，因为数组实现很麻烦。



### Arraylist vs Vector

- `ArrayList` 是 `List` 的主要实现类，底层使用 `Object[ ]`存储，适用于频繁的查找工作，线程不安全 
- `Vector` 是 `List` 的古老实现类，底层使用`Object[ ]` 存储，**线程安全**的



### Arraylist vs LinkedList

- 是否保证线程安全

  `ArrayList` 和 `LinkedList` 都是不同步的，也就是不保证线程安全

- 底层数据结构

  `Arraylist` 底层使用的是 **`Object` 数组**；`LinkedList` 底层使用的是 **双向链表** 数据结构（JDK1.6 之前为循环链表，JDK1.7 取消了循环。注意双向链表和双向循环链表的区别，下面有介绍到！）

- 插入和删除是否受元素位置的影响

  `ArrayList` 采用数组存储，所以插入和删除元素的时间复杂度受元素位置的影响。 比如：执行`add(E e)`方法的时候， `ArrayList` 会默认在将指定的元素追加到此列表的末尾，这种情况时间复杂度就是 O(1)。但是如果要在指定位置 i 插入和删除元素的话（`add(int index, E element)`）时间复杂度就为 O(n-i)。因为在进行上述操作的时候集合中第 i 和第 i 个元素之后的(n-i)个元素都要执行向后位/向前移一位的操作。

  `LinkedList` 采用链表存储，所以，如果是在头尾插入或者删除元素不受元素位置的影响（`add(E e)`、`addFirst(E e)`、`addLast(E e)`、`removeFirst()` 、 `removeLast()`），近似 O(1)，如果是要在指定位置 `i` 插入和删除元素的话（`add(int index, E element)`，`remove(Object o)`） 时间复杂度近似为 O(n) ，因为需要先移动到指定位置再插入。

- 是否支持快速随机访问

  `LinkedList` 不支持高效的随机元素访问，而 `ArrayList` 支持。快速随机访问就是通过元素的序号快速获取元素对象(对应于`get(int index)`方法)。

- 内存空间占用

  ArrayList 的空间浪费主要体现在在 list 列表的结尾会预留一定的容量空间，而 LinkedList 的空间花费则体现在它的每一个元素都需要消耗比 ArrayList 更多的空间（因为要存放直接后继和直接前驱以及数据）。

|                     | ArrayList    | LinkedList           |
| :------------------ | :----------- | :------------------- |
| 获取指定元素        | 速度很快     | 需要从头开始查找元素 |
| 添加元素到末尾      | 速度很快     | 速度很快             |
| 在指定位置添加/删除 | 需要移动元素 | 不需要移动元素       |
| 内存占用            | 少           | 较大                 |



### 双向链表和双向循环链表

**双向链表：** 包含两个指针，一个 prev 指向前一个节点，一个 next 指向后一个节点。

> 另外推荐一篇把双向链表讲清楚的文章：[https://juejin.cn/post/6844903648154271757](https://juejin.cn/post/6844903648154271757)

![双向链表](https://img-note.langyastudio.com/202110141616381.png?x-oss-process=style/watermark)

**双向循环链表：** 最后一个节点的 next 指向 head，而 head 的 prev 指向最后一个节点，构成一个环。

![双向循环链表](https://img-note.langyastudio.com/202110141616443.png?x-oss-process=style/watermark)



### RandomAccess 接口

```java
public interface RandomAccess {
}
```

查看源码我们发现实际上 `RandomAccess` 接口中什么都没有定义。所以，在我看来 `RandomAccess` 接口不过是一个标识罢了。标识什么？ 标识实现这个接口的类具有随机访问功能。

在 `binarySearch（)` 方法中，它要判断传入的 list 是否 `RamdomAccess` 的实例，如果是，调用`indexedBinarySearch()`方法，如果不是，那么调用`iteratorBinarySearch()`方法

```java
    public static <T>
    int binarySearch(List<? extends Comparable<? super T>> list, T key) {
        if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
            return Collections.indexedBinarySearch(list, key);
        else
            return Collections.iteratorBinarySearch(list, key);
    }
```

`ArrayList` 实现了 `RandomAccess` 接口， 而 `LinkedList` 没有实现。为什么呢？我觉得还是和底层数据结构有关！`ArrayList` 底层是数组，而 `LinkedList` 底层是链表。数组天然支持随机访问，时间复杂度为 O(1)，所以称为快速随机访问。链表需要遍历到特定位置才能访问特定位置的元素，时间复杂度为 O(n)，所以不支持快速随机访问。，`ArrayList` 实现了 `RandomAccess` 接口，就表明了他具有快速随机访问功能。 `RandomAccess` 接口只是标识，并不是说 `ArrayList` 实现 `RandomAccess` 接口才具有快速随机访问功能的！



### 说一说 ArrayList 的扩容机制吧

详见笔主的这篇文章:[通过源码一步一步分析 ArrayList 扩容机制](https://snailclimb.gitee.io/javaguide/#/docs/java/collection/arraylist-source-code?id=_2-arraylist-%e6%a0%b8%e5%bf%83%e6%ba%90%e7%a0%81%e8%a7%a3%e8%af%bb)



### 使用说明

考察`List`接口，可以看到几个主要的接口方法：

- 在末尾添加一个元素：`void add(E e)`
- 在指定索引添加一个元素：`void add(int index, E e)`
- 删除指定索引的元素：`int remove(int index)`
- 删除某个元素：`int remove(Object e)`
- 获取指定索引的元素：`E get(int index)`
- 获取链表大小（包含元素的个数）：`int size()`



**常用子类**

- ArrayList（数组）

- LinkedList（双向链表）

- CopyOnWriteArrayList（写加锁，读不加锁；只能保证数据的最终一致性，不能保证数据的实时一致性）



#### 创建 List

除了使用 `ArrayList` 和 `LinkedList`

我们还可以通过 `List` 接口提供的 `of()` 方法，根据给定元素快速创建 `List`：

```java
List<Integer> list = List.of(1, 2, 5);
```

但是 `List.of()` 方法不接受 `null` 值，如果传入 `null`，会抛出 `NullPointerException`异常。



#### 遍历 List

采用 `for` 方式实现不推荐，一是代码复杂，二是因为 `get(int)` 方法只有 `ArrayList` 的实现是高效的，换成`LinkedList` 后，索引越大访问速度越慢。

所以我们要始终坚持使用迭代器 `Iterator` 来访问 `List`，通过 `Iterator` 遍历 `List` 永远是最高效的方式：

```java
public class Main {
    public static void main(String[] args) {
        List<String> list = List.of("apple", "pear", "banana");
        for (Iterator<String> it = list.iterator(); it.hasNext(); ) {
            String s = it.next();
            System.out.println(s);
        }
    }
}
```

Java的 `for each` 循环本身就可以帮我们使用 `Iterator` 遍历，自动把 `for each` 循环变成 `Iterator` 的调用。把上面的代码再改写如下：

```java
public class Main {
    public static void main(String[] args) {
        List<String> list = List.of("apple", "pear", "banana");
        for (String s : list) {
            System.out.println(s);
        }
    }
}
```



#### List <=> Array 

把 `List` 变为 `Array` 有三种方法：

第一种是调用 `toArray()` 方法直接返回一个 `Object[]` 数组

第二种方式是给 `toArray(T[])` 传入一个类型相同的 `Array`，`List` 内部自动把元素复制到传入的 `Array` 中

```java
public class Main {
    public static void main(String[] args) {
        List<Integer> list = List.of(12, 34, 56);
        Integer[] array = list.toArray(new Integer[3]);
        for (Integer n : array) {
            System.out.println(n);
        }
    }
}
```

如果传入的数组不够大，那么 `List` 内部会创建一个新的刚好够大的数组，填充后返回；如果传入的数组比 `List` 元素还要多，那么填充完元素后，剩下的数组元素一律填充 `null`。

最后一种**更简洁的写法**是通过 `List` 接口定义的 `T[] toArray(IntFunction generator)`方法

```java
Integer[] array = list.toArray(Integer[]::new);
```



反过来，把 `Array` 变为 `List` 就简单多了，通过 `List.of(T...) `方法最简单：

```java
Integer[] array = { 1, 2, 3 };
List<Integer> list = List.of(array);
```

如果我们调用 `List.of()`，它返回的是一个 **只读`List`**



## Set

`Set` 用于存储无序、不重复的元素集合（底层大多数是Map结构的实现），我们经常用`Set`用于去除重复元素。



### comparable vs Comparator

- `comparable`

  接口实际上是出自 `java.lang` 包，它有一个 `compareTo(Object obj)`方法用来排序

  用于集合类自身的排序，如 TreeList

- `comparator` 

  接口实际上是出自 java.util 包，它有一个`compare(Object obj1, Object obj2)`方法用来排序
  
  用于两个参数版的，用于第三方类排序如 `Collections.sort()`



#### compareTo 排序

```java
// person对象没有实现Comparable接口，所以必须实现，这样才不会出错，才可以使treemap中的数据按顺序排列
// 前面一个例子的String类已经默认实现了Comparable接口，详细可以查看String类的API文档，另外其他
// 像Integer类等都已经实现了Comparable接口，所以不需要另外实现了
public  class Person implements Comparable<Person> {
    private String name;
    private int age;

    public Person(String name, int age) {
        super();
        this.name = name;
        this.age = age;
    }

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

    /**
     * T重写compareTo方法实现按年龄来排序
     */
    @Override
    public int compareTo(Person o) {
        if (this.age > o.getAge()) {
            return 1;
        }
        if (this.age < o.getAge()) {
            return -1;
        }
        return 0;
    }
}

```

```java
    public static void main(String[] args) {
        TreeMap<Person, String> pdata = new TreeMap<Person, String>();
        pdata.put(new Person("张三", 30), "zhangsan");
        pdata.put(new Person("李四", 20), "lisi");
        pdata.put(new Person("王五", 10), "wangwu");
        pdata.put(new Person("小红", 5), "xiaohong");
        
        // 得到key的值的同时得到key所对应的值
        Set<Person> keys = pdata.keySet();
        for (Person key : keys) {
            System.out.println(key.getAge() + "-" + key.getName());
        }
    }
```

Output：

```
5-小红
10-王五
20-李四
30-张三
```



#### Comparator 排序

```java
        ArrayList<Integer> arrayList = new ArrayList<Integer>();
        arrayList.add(-1);
        arrayList.add(3);
        arrayList.add(3);
        arrayList.add(-5);
        arrayList.add(7);
        arrayList.add(4);
        arrayList.add(-9);
        arrayList.add(-7);
        System.out.println("原始数组:");
        System.out.println(arrayList);

        // void reverse(List list)：反转
        Collections.reverse(arrayList);
        System.out.println("Collections.reverse(arrayList):");
        System.out.println(arrayList);

        // void sort(List list),按自然排序的升序排序
        Collections.sort(arrayList);
        System.out.println("Collections.sort(arrayList):");
        System.out.println(arrayList);

        // 定制排序的用法
        Collections.sort(arrayList, new Comparator<Integer>() {
            @Override
            public int compare(Integer o1, Integer o2) {
                return o2.compareTo(o1);
            }
        });
        System.out.println("定制排序后：");
        System.out.println(arrayList);
```

Output:

```
原始数组:
[-1, 3, 3, -5, 7, 4, -9, -7]
Collections.reverse(arrayList):
[-7, -9, 4, 7, -5, 3, 3, -1]
Collections.sort(arrayList):
[-9, -7, -5, -1, 3, 3, 4, 7]
定制排序后：
[7, 4, 3, 3, -1, -5, -7, -9]
```



### 无序性 不可重复性

- 无序性

  无序性不等于随机性 ，无序性是指存储的数据在底层数组中并非按照数组索引的顺序添加 ，而是根据数据的哈希值决定的

- 不可重复性

  不可重复性是指添加的元素按照 equals() 判断时 ，返回 false，需要同时重写 equals() 方法和 HashCode() 方法



### HashSet LinkedHashSet TreeSet 异同

- `HashSet`、`LinkedHashSet` 和 `TreeSet` 都是 `Set` 接口的实现类，都能保证元素唯一，并且都不是线程安全的

- `HashSet`、`LinkedHashSet` 和 `TreeSet` 的主要区别在于底层数据结构不同

  `HashSet` 的底层数据结构是哈希表（基于 `HashMap` 实现）

  `LinkedHashSet` 的底层数据结构是链表和哈希表，元素的插入和取出顺序满足 FIFO

  `TreeSet` 底层数据结构是红黑树，元素是有序的，排序的方式有自然排序和定制排序

- 底层数据结构不同又导致这三者的应用场景不同

  `HashSet` 用于不需要保证元素插入和取出顺序的场景

  `LinkedHashSet` 用于保证元素的插入和取出顺序满足 FIFO 的场景

  `TreeSet` 用于支持对元素自定义排序规则的场景



### HashSet 如何检查重复

以下内容摘自我的 Java 启蒙书《Head first java》第二版：

当你把对象加入`HashSet`时，`HashSet` 会先计算对象的 `hashcode` 值来判断对象加入的位置，同时也会与其他加入的对象的 `hashcode` 值作比较，如果没有相符的 `hashcode`，`HashSet` 会假设对象没有重复出现。但是如果发现有相同 `hashcode` 值的对象，这时会调用 `equals()` 方法来检查  `hashcode` 相等的对象是否真的相同。如果两者相同，`HashSet` 就不会让加入操作成功。

在 openjdk8 中，`HashSet `的 `add()` 方法只是简单的调用了 `HashMap` 的 `put()` 方法，并且判断了一下返回值以确保是否有重复元素。直接看一下 `HashSet` 中的源码：  

```java
// Returns: true if this set did not already contain the specified element
// 返回值：当set中没有包含add的元素时返回真
public boolean add(E e) {
        return map.put(e, PRESENT) == null;
}
```

而在 `HashMap` 的 `putVal()` 方法中也能看到如下说明：  

```java
// Returns : previous value, or null if none
// 返回值：如果插入位置没有元素返回 null，否则返回上一个元素
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
...
}
```

也就是说，在 openjdk8 中，实际上无论 `HashSet` 中是否已经存在了某元素，`HashSet` 都会直接插入，只是会在 `add()`方法的返回值处告诉我们插入前是否存在相同元素。



### 使用说明

**常用子类**

- HashSet（封装了HashMap、元素可以为null）
- LinkedHashSet（封装了LinkedHashMap、元素可以为null）
- TreeSet（封装了TreeMap、元素不能为null）



`Set`主要提供以下几个方法：

- 将元素添加进`Set`：`boolean add(E e)`
- 将元素从`Set`删除：`boolean remove(Object e)`
- 判断是否包含元素：`boolean contains(Object e)`

因为放入 `Set` 的元素和 `Map` 的 key 类似，都要正确实现 `equals()` 和 `hashCode()` 方法，否则该元素无法正确地放入`Set`。



把`HashSet`换成`TreeSet`，在遍历`TreeSet`时，输出就是有序的，这个顺序是元素的排序顺序：

```java
public class Main {
    public static void main(String[] args) {
        Set<String> set = new TreeSet<>();
        set.add("apple");
        set.add("banana");
        set.add("pear");
        set.add("orange");
        
        for (String s : set) {
            System.out.println(s);
        }
    }
}
```



## Queue

### Queue vs Deque

**Queue**

`Queue` 是单端队列，只能从一端插入元素，另一端删除元素，实现上一般遵循 **先进先出（FIFO）** 规则。

`Queue` 扩展了 `Collection` 的接口，根据 **因为容量问题而导致操作失败后处理方式的不同** 可以分为两类方法: 一种在操作失败后会抛出异常，另一种则会返回特殊值。

| `Queue` 接口| 抛出异常  | 返回特殊值 |
| ------------ | --------- | ---------- |
| 插入队尾     | add(E e)  | offer(E e) |
| 删除队首     | remove()  | poll()     |
| 查询队首元素 | element() | peek()     |

它和`List`的区别在于，`List`可以在任意位置添加和删除元素，而`Queue`只有两个操作：

- 把元素添加到队列末尾；
- 从队列头部取出元素。

超市的收银台就是一个队列：

![queue](https://img-note.langyastudio.com/20210323151259.jpeg?x-oss-process=style/watermark)

````java
public class Main {
    public static void main(String[] args) {
        Queue<String> q = new LinkedList<>();
        
        // 添加3个元素到队列:
        q.offer("apple");
        q.offer("pear");
        q.offer("banana");
        
        // 从队列取出元素:
        System.out.println(q.poll()); // apple
        System.out.println(q.poll()); // pear
        System.out.println(q.poll()); // banana
        System.out.println(q.poll()); // null,因为队列是空的
    }
}
````



**Deque**

`Deque` 是双端队列，**在队列的两端均可以插入或删除元素**。

`Deque` 扩展了 `Queue` 的接口, 增加了在队首和队尾进行插入和删除的方法，同样根据失败后处理方式的不同分为两类：

| `Deque` 接口  | 抛出异常      | 返回特殊值      |
| ------------ | ------------- | --------------- |
| 插入队首     | addFirst(E e) | offerFirst(E e) |
| 插入队尾     | addLast(E e)  | offerLast(E e)  |
| 删除队首     | removeFirst() | pollFirst()     |
| 删除队尾     | removeLast()  | pollLast()      |
| 查询队首元素 | getFirst()    | peekFirst()     |
| 查询队尾元素 | getLast()     | peekLast()      |

> 注意到`Deque`接口实际上扩展自`Queue`



### PriorityQueue

`PriorityQueue` 是在 JDK1.5 中被引入的,  为了实现 “VIP插队” 的业务，其与 `Queue` 的区别在于元素出队顺序是与优先级相关的，即**总是优先级最高的元素先出队**。

这里列举其相关的一些要点：

- `PriorityQueue` 默认按元素比较的顺序排序（必须实现 `Comparable` 接口），也可以通过 `Comparator` 自定义排序算法（元素就不必实现 `Comparable` 接口）
- `PriorityQueue` 利用了二叉堆的数据结构来实现的，底层使用可变长的数组来存储数据
- `PriorityQueue` 通过堆元素的上浮和下沉，实现了在 O(logn) 的时间复杂度内插入元素和删除堆顶元素
- `PriorityQueue` 是非线程安全的，且不支持存储 `NULL` 和 `non-comparable` 的对象

`PriorityQueue` 在面试中可能更多的会出现在手撕算法的时候，典型例题包括堆排序、求第K大的数、带权图的遍历等，所以需要会熟练使用才行。

```java
public class Main {
    public static void main(String[] args) {
        Queue<User> q = new PriorityQueue<>(new UserComparator());
        
        // 添加3个元素到队列:
        q.offer(new User("Bob", "A1"));
        q.offer(new User("Alice", "A2"));
        q.offer(new User("Boss", "V1"));
        
        System.out.println(q.poll()); // Boss/V1
        System.out.println(q.poll()); // Bob/A1
        System.out.println(q.poll()); // Alice/A2
        System.out.println(q.poll()); // null,因为队列为空
    }
}

class UserComparator implements Comparator<User> {
    public int compare(User u1, User u2) {
        if (u1.number.charAt(0) == u2.number.charAt(0)) {
            // 如果两人的号都是A开头或者都是V开头,比较号的大小:
            return u1.number.compareTo(u2.number);
        }
        
        if (u1.number.charAt(0) == 'V') {
            // u1的号码是V开头,优先级高:
            return -1;            
        } else {
            return 1;
        }
    }
}

class User {
    public final String name;
    public final String number;

    public User(String name, String number) {
        this.name = name;
        this.number = number;
    }

    public String toString() {
        return name + "/" + number;
    }
}
```



### ArrayDeque vs LinkedList

`ArrayDeque` 和 `LinkedList` 都实现了 `Deque` 接口，两者都具有队列的功能

- `ArrayDeque` 是基于可变长的数组和双指针来实现，而 `LinkedList` 则通过链表来实现
- `ArrayDeque` 插入时可能存在扩容过程, 不过均摊后的插入操作依然为 O(1)。虽然 `LinkedList` 不需要扩容，但是每次插入数据时均需要申请新的堆空间，均摊性能相比更慢
- `ArrayDeque` 不支持存储 `NULL` 数据，但 `LinkedList` 支持
- `ArrayDeque` 是在 JDK1.6 才被引入的，而`LinkedList` 早在 JDK1.2 时就已经存在

从性能的角度上，选用 `ArrayDeque` 来实现队列要比 `LinkedList` 更好。此外，`ArrayDeque` 也可以用于实现栈。



### Stack

栈（Stack）是一种后进先出（LIFO）的数据结构，操作栈的元素的方法有：

- 把元素压栈：`push(E)`；
- 把栈顶的元素“弹出”：`pop(E)`；
- 取栈顶元素但不弹出：`peek(E)`。

在 Java 中，我们用 `Deque` 可以实现 `Stack` 的功能，注意只调用 `push()`/`pop()`/`peek()`方 法，避免调用 `Deque` 的其他方法。

> 不要使用遗留类的 `Stack`



## Map

`Map`是一种无序的键-值映射表，可以通过`key`快速查找`value`。



### HashMap vs Hashtable

- 线程是否安全

  `HashMap` 是非线程安全的，`Hashtable` 是线程安全的，因为 `Hashtable` 内部的方法基本都经过 `synchronized` 修饰。（如果你要保证线程安全的话就使用 `ConcurrentHashMap` 吧！）

- 效率

  因为线程安全的问题，`HashMap` 要比 `Hashtable` 效率高一点。另外 `Hashtable` 基本被淘汰，不要在代码中使用它

- 对 Null key 和 Null value 的支持

  `HashMap` 可以存储 null 的 key 和 value，但 null 作为键只能有一个，null 作为值可以有多个；Hashtable 不允许有 null 键和 null 值，否则会抛出 `NullPointerException`。

- 初始容量大小和每次扩充容量大小

  ① 创建时如果不指定容量初始值，`Hashtable` 默认的初始大小为 11，之后每次扩充，容量变为原来的 2n+1。`HashMap` 默认的初始化大小为 16。之后每次扩充，容量变为原来的 2 倍。

  ② 创建时如果给定了容量初始值，那么 Hashtable 会直接使用你给定的大小，而 `HashMap` 会将其扩充为 2 的幂次方大小（`HashMap` 中的 `tableSizeFor()`方法保证，下面给出了源代码）。也就是说 `HashMap` 总是使用 2 的幂作为哈希表的大小，后面会介绍到为什么是 2 的幂次方。

- 底层数据结构

  JDK1.8 以后的 `HashMap` 在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间。Hashtable 没有这样的机制。



### HashMap vs HashSet

如果你看过 `HashSet` 源码的话就应该知道：`HashSet` 底层就是基于 `HashMap` 实现的。（`HashSet` 的源码非常非常少，因为除了 `clone()`、`writeObject()`、`readObject()`是 `HashSet` 自己不得不实现之外，其他方法都是直接调用 `HashMap` 中的方法。

|                HashMap                 |                           HashSet                            |
| :------------------------------------: | :----------------------------------------------------------: |
|           实现了 `Map` 接口            |                       实现 `Set` 接口                        |
|               存储键值对               |                          仅存储对象                          |
|     调用 `put()`向 map 中添加元素      |             调用 `add()`方法向 `Set` 中添加元素              |
| `HashMap` 使用键（Key）计算 `hashcode` | `HashSet` 使用成员对象来计算 `hashcode` 值，对于两个对象来说 `hashcode` 可能相同，所以 `equals()` 方法用来判断对象的相等性 |



### HashMap vs TreeMap

`TreeMap` 和`HashMap` 都继承自 `AbstractMap` ，但是需要注意的是 `TreeMap` 它还实现了 `NavigableMap` 接口和`SortedMap` 接口。

- 实现 `NavigableMap` 接口让 `TreeMap` 有了对集合内**元素的搜索**的能力

- 实现 `SortedMap` 接口让 `TreeMap` 有了对集合中的元素根据**键排序**的能力

  默认是按 key 的升序排序，不过我们也可以指定排序的比较器 `comparable` 

![](https://img-note.langyastudio.com/202110131537863.png?x-oss-process=style/watermark)

> `TreeMap`不使用`equals()`和`hashCode()`



### HashMap 底层实现

#### JDK1.8 之前

JDK1.8 之前 `HashMap` 底层是 **数组和链表** 结合在一起使用也就是 **链表散列**。HashMap 通过 key 的 hashCode 经过扰动函数处理过后得到 hash 值，然后通过 (n - 1) & hash 判断当前元素存放的位置（这里的 n 指的是数组的长度），如果当前位置存在元素的话，就判断该元素与要存入的元素的 hash 值以及 key 是否相同，如果相同的话，直接覆盖，不相同就通过拉链法解决冲突。

> 所谓扰动函数指的就是 HashMap 的 hash 方法，为了防止一些实现比较差的 hashCode() 方法，减少碰撞
>
> 我们把不同的 `key` 具有相同的 `hashCode()` 的情况称之为**哈希冲突**



**JDK 1.8 HashMap 的 hash 方法源码:**

JDK 1.8 的 hash 方法 相比于 JDK 1.7 hash 方法更加简化，但是原理不变。

```java
    static final int hash(Object key) {
      int h;
        
      // key.hashCode()：返回散列值也就是hashcode
      // ^ ：按位异或
      // >>>:无符号右移，忽略符号位，空位都以0补齐
      return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
  }
```

对比一下 JDK1.7 的 HashMap 的 hash 方法源码.

```java
static int hash(int h) {
    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

相比于 JDK1.8 的 hash 方法 ，**JDK 1.7 的 hash 方法的性能会稍差一点点，因为毕竟扰动了 4 次**。



所谓 **“拉链法”** 就是：将链表和数组相结合。也就是说创建一个链表数组，数组中每一格就是一个链表。若遇到哈希冲突，则将冲突的值加到链表中即可。

![jdk1.8之前的内部结构-HashMap](https://img-note.langyastudio.com/202110131537294.png?x-oss-process=style/watermark)



#### JDK1.8 之后

相比于之前的版本， JDK1.8 之后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间。

![jdk1.8之后的内部结构-HashMap](https://img-note.langyastudio.com/202110131537462.png?x-oss-process=style/watermark)

> TreeMap、TreeSet 以及 JDK1.8 之后的 HashMap 底层都用到了红黑树。红黑树就是为了解决二叉查找树的缺陷，因为二叉查找树在某些情况下会退化成一个线性结构。



### HashMap 长度为什么是 2 的幂次方

为了能让 HashMap 存取高效，尽量较少碰撞，也就是要尽量把数据分配均匀。我们上面也讲到了，Hash 值的范围值 -2147483648 到 2147483647，前后加起来大概 40 亿的映射空间，只要哈希函数映射得比较均匀松散，一般应用是很难出现碰撞的。但问题是一个 40 亿长度的数组，内存是放不下的。所以这个散列值是不能直接拿来用的。用之前还要先做对数组的长度取模运算，得到的余数才能用来要存放的位置也就是对应的数组下标。这个数组下标的计算方法是“ `(n - 1) & hash`”。（n 代表数组长度）。这也就解释了 HashMap 的长度为什么是 2 的幂次方。

我们首先可能会想到采用 % 取余的操作来实现。但是重点来了：**“**取余(%)操作中如果除数是 2 的幂次则等价于与其除数减一的与(&)操作（也就是说 `hash%length==hash&(length-1)` 的前提是 length 是 2 的 n 次方；）。” 并且采用**二进制位操作 &，相对于 % 能够提高运算效率**，这就解释了 HashMap 的长度为什么是 2 的幂次方。



**`HashMap` 中带有初始容量的构造函数：**

```java
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }
     public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
```

下面这个方法保证了 `HashMap` 总是使用 2 的幂作为哈希表的大小。

```java
    /**
     * Returns a power of two size for the given target capacity.
     */
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```



### HashMap 多线程死循环

主要原因在于并发下的 Rehash 会造成元素之间会形成一个**循环链表**。不过，jdk 1.8 后解决了这个问题，但是还是不建议在多线程下使用 HashMap，因为多线程下使用 HashMap 还是会存在其他问题比如数据丢失。并发环境下推荐使用 ConcurrentHashMap 。

详情请查看：https://coolshell.cn/articles/9606.html



### HashMap 遍历方式

[HashMap 的 7 种遍历方式与性能分析！](https://mp.weixin.qq.com/s/zQBN3UvJDhRTKP6SzcZFKw)



### ConcurrentHashMap vs Hashtable

`ConcurrentHashMap` 和 `Hashtable` 的区别主要体现在实现线程安全的方式上不同。

- 底层数据结构

  **JDK1.7** 的 `ConcurrentHashMap` 底层采用 **分段的数组+链表** 实现，**JDK1.8** 采用的数据结构跟 `HashMap1.8` 的结构一样，**数组+链表/红黑二叉树**。

  `Hashtable` 和 JDK1.8 之前的 `HashMap` 的底层数据结构类似都是采用 **数组+链表** 的形式，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的

- 实现线程安全的方式

  ① `ConcurrentHashMap`

  **在 JDK1.7 的时候，`ConcurrentHashMap`（分段锁）** 对整个桶数组进行了分割分段(`Segment`)，每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。 

  **到了 JDK1.8 的时候**，已经摒弃了 `Segment` 的概念，而是直接用 **`Node` 数组+链表+红黑树的数据结构来实现，并发控制使用 `synchronized` 和 CAS 来操作**。（JDK1.6 以后 对 `synchronized` 锁做了很多优化） 虽然在 JDK1.8 中还能看到 `Segment` 的数据结构，但是已经简化了属性，只是为了兼容旧版本

  ② **`Hashtable`(同一把锁)** 

  使用 `synchronized` 来保证线程安全，效率非常低下。当一个线程访问同步方法时，其他线程也访问同步方法，可能会进入阻塞或轮询状态，如使用 put 添加元素，另一个线程不能使用 put 添加元素，也不能使用 get，竞争会越来越激烈效率越低。

  

**两者的对比图：**

**Hashtable:**

![Hashtable全表锁](https://img-note.langyastudio.com/202110191252331.png?x-oss-process=style/watermark)


**JDK1.7 的 ConcurrentHashMap：**

![JDK1.7的ConcurrentHashMap](https://img-note.langyastudio.com/202110191252587.jpeg?x-oss-process=style/watermark)


**JDK1.8 的 ConcurrentHashMap：**

![Java8 ConcurrentHashMap 存储结构（图片来自 javadoop）](https://img-note.langyastudio.com/202110131537373.png?x-oss-process=style/watermark)

JDK1.8 的 `ConcurrentHashMap` 不再是 **Segment 数组 + HashEntry 数组 + 链表**，而是 **Node 数组 + 链表 / 红黑树**。不过，Node 只能用于链表的情况，红黑树的情况需要使用 **`TreeNode`**。当冲突链表达到一定长度时，链表会转换成红黑树。



### ConcurrentHashMap 线程安全实现

#### JDK1.7

首先将数据分为一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据时，其他段的数据也能被其他线程访问。

**`ConcurrentHashMap` 是由 `Segment` 数组结构和 `HashEntry` 数组结构组成**。

Segment 实现了 `ReentrantLock`, 所以 `Segment` 是一种可重入锁，扮演锁的角色。`HashEntry` 用于存储键值对数据。

```java
static class Segment<K,V> extends ReentrantLock implements Serializable {
}
```

一个 `ConcurrentHashMap` 里包含一个 `Segment` 数组。`Segment` 的结构和 `HashMap` 类似，是一种数组和链表结构，一个 `Segment` 包含一个 `HashEntry` 数组，每个 `HashEntry` 是一个链表结构的元素，每个 `Segment` 守护着一个 `HashEntry` 数组里的元素，当对 `HashEntry` 数组的数据进行修改时，必须首先获得对应的 `Segment` 的锁。



#### JDK1.8 

`ConcurrentHashMap` 取消了 `Segment` 分段锁，采用 CAS 和 `synchronized` 来保证并发安全。数据结构跟 HashMap1.8 的结构类似，数组+链表/红黑二叉树。Java 8 在链表长度超过一定阈值（8）时将链表（寻址时间复杂度为 O(N)）转换为红黑树（寻址时间复杂度为 O(log(N))）

`synchronized` **只锁定当前链表或红黑二叉树的首节点**，这样只要 hash 不冲突，就不会产生并发，效率又提升 N 倍。



### 使用说明

#### 遍历Map

对`Map`来说，要遍历`key`可以使用`for each`循环遍历`Map`实例的`keySet()`方法返回的`Set`集合，它包含不重复的`key`的集合。

```java
public class Main {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();
        map.put("apple", 123);
        map.put("pear", 456);
        map.put("banana", 789);
        for (String key : map.keySet()) {
            Integer value = map.get(key);
            System.out.println(key + " = " + value);
        }
    }
}
```

也可以通过`for each`遍历`entrySet()`，直接获取`key-value`

```java
public class Main {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();
        map.put("apple", 123);
        map.put("pear", 456);
        map.put("banana", 789);
        for (Map.Entry<String, Integer> entry : map.entrySet()) {
            String key = entry.getKey();
            Integer value = entry.getValue();
            System.out.println(key + " = " + value);
        }
    }
}
```



#### 哈希冲突

那么第一个问题来了：`hashCode()`返回的`int`范围高达±21亿，先不考虑负数，`HashMap`内部使用的数组得有多大？

实际上`HashMap`初始化时默认的数组大小只有16，任何`key`，无论它的`hashCode()`有多大，都可以简单地通过：

```java
int index = key.hashCode() & 0xf; // 0xf = 15
```

把索引确定在0～15，即永远不会超出数组范围，上述算法只是一种最简单的实现。



第二个问题：如果添加超过16个`key-value`到`HashMap`，数组不够用了怎么办？

添加超过一定数量的`key-value`时，`HashMap`会在内部自动扩容，每次扩容一倍，即长度为16的数组扩展为长度32，相应地，需要重新确定`hashCode()`计算的索引位置。例如，对长度为32的数组计算`hashCode()`对应的索引，计算方式要改为：

```java
int index = key.hashCode() & 0x1f; // 0x1f = 31
```

由于扩容会导致重新分布已有的`key-value`，所以，频繁扩容对`HashMap`的性能影响很大。如果我们确定要使用一个容量为`10000`个`key-value`的`HashMap`，更好的方式是创建`HashMap`时就指定容量：

```java
Map<String, Integer> map = new HashMap<>(10000);
```

虽然指定容量是`10000`，但`HashMap`内部的数组长度总是2的n次方，因此，实际数组长度被初始化为比`10000`大的`16384`。



最后一个问题：如果不同的两个`key`，例如`"a"`和`"b"`，它们的`hashCode()`恰好是相同的（这种情况是完全可能的，因为不相等的两个实例，只要求`hashCode()`尽量不相等），那么，当我们放入：

```java
map.put("a", new Person("Xiao Ming"));
map.put("b", new Person("Xiao Hong"));
```

时，由于计算出的数组索引相同，后面放入的`"Xiao Hong"`会不会把`"Xiao Ming"`覆盖了？

当然不会！使用`Map`的时候，只要`key`不相同，它们映射的`value`就互不干扰。但是，在`HashMap`内部，确实可能存在不同的`key`，映射到相同的`hashCode()`，即相同的数组索引上，肿么办？

我们就假设`"a"`和`"b"`这两个`key`最终计算出的索引都是5，那么，在`HashMap`的数组中，实际存储的不是一个`Person`实例，而是一个`List`，它包含两个`Entry`，一个是`"a"`的映射，一个是`"b"`的映射：

```ascii
  ┌───┐
0 │   │
  ├───┤
1 │   │
  ├───┤
2 │   │
  ├───┤
3 │   │
  ├───┤
4 │   │
  ├───┤
5 │ ●─┼───> List<Entry<String, Person>>
  ├───┤
6 │   │
  ├───┤
7 │   │
  └───┘
```

在查找的时候，例如：

```java
Person p = map.get("a");
```

HashMap内部通过`"a"`找到的实际上是`List>`，它还需要遍历这个`List`，并找到一个`Entry`，它的`key`字段是`"a"`，才能返回对应的`Person`实例。



#### EnumMap

如果 `Map` 的 key 是 `enum` 类型，推荐使用 `EnumMap`。它在内部以一个非常紧凑的数组存储 value，并且根据 `enum` 类型的 key 直接定位到内部数组的索引，并不需要计算 `hashCode()`，不但效率最高，而且没有额外的空间浪费。

```java
public class Main {
    public static void main(String[] args) {
        Map<DayOfWeek, String> map = new EnumMap<>(DayOfWeek.class);
        map.put(DayOfWeek.MONDAY, "星期一");
        map.put(DayOfWeek.TUESDAY, "星期二");
        map.put(DayOfWeek.WEDNESDAY, "星期三");
        map.put(DayOfWeek.THURSDAY, "星期四");
        map.put(DayOfWeek.FRIDAY, "星期五");
        map.put(DayOfWeek.SATURDAY, "星期六");
        map.put(DayOfWeek.SUNDAY, "星期日");
        System.out.println(map);
        System.out.println(map.get(DayOfWeek.MONDAY));
    }
}
```



## Iterator

`Iterator` 是一种抽象的数据访问模型。使用 `Iterator` 模式进行迭代的好处有：

- 对任何集合都采用同一种访问模型
- 调用者对集合内部结构一无所知
- 集合类返回的 `Iterator` 对象知道如何迭代

Java 提供了标准的迭代器模型，即集合类实现 `java.util.Iterable` 接口，返回 `java.util.Iterator` 实例。

 

Java 的集合类都可以使用 `for each` 循环，`List`、`Set` 和 `Queue` 会迭代每个元素，`Map` 会迭代每个key。

以 `List` 为例：

```java
List<String> list = List.of("Apple", "Orange", "Pear");
for (String s : list) {
    System.out.println(s);
}
```

实际上，Java 编译器并不知道如何遍历 `List`。上述代码能够编译通过，只是因为编译器把 `for each` 循环通过`Iterator` 改写为了普通的 `for` 循环：

```java
for (Iterator<String> it = list.iterator(); it.hasNext(); ) {
     String s = it.next();
     System.out.println(s);
}
```



自定义一个集合类，想要使用 `for each` 循环，只需满足以下条件：

- 集合类实现 `Iterable` 接口，该接口要求返回一个 `Iterator` 对象
- 用 `Iterator` 对象迭代集合内部数据

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        ReverseList<String> rlist = new ReverseList<>();
        rlist.add("Apple");
        rlist.add("Orange");
        rlist.add("Pear");
        
        for (String s : rlist) {
            System.out.println(s);
        }
    }
}

class ReverseList<T> implements Iterable<T> {

    private List<T> list = new ArrayList<>();

    public void add(T t) {
        list.add(t);
    }

    @Override
    public Iterator<T> iterator() {
        return new ReverseIterator(list.size());
    }

    class ReverseIterator implements Iterator<T> {
        int index;

        ReverseIterator(int index) {
            this.index = index;
        }

        @Override
        public boolean hasNext() {
            return index > 0;
        }

        @Override
        public T next() {
            index--;
            return ReverseList.this.list.get(index);
        }
    }
}
```



## Collections

Collections 工具类常用方法:

- 排序

- 查找、替换操作

- 同步控制(不推荐，需要线程安全的集合类型时请考虑使用 **JUC** 包下的并发集合)



### 排序操作

```java
void reverse(List list)//反转
void shuffle(List list)//随机排序 —— 洗牌算法
    
void sort(List list)//按自然排序的升序排序
void sort(List list, Comparator c)//定制排序，由Comparator控制排序逻辑
    
void swap(List list, int i , int j)//交换两个索引位置的元素
    
void rotate(List list, int distance)//旋转。当distance为正数时，将list后distance个元素整体移到前面。当distance为负数时，将 list的前distance个元素整体移到后面
```



### 查找替换操作

```java
int binarySearch(List list, Object key)//对List进行二分查找，返回索引，注意List必须是有序的
    
int max(Collection coll)//根据元素的自然顺序，返回最大的元素。 类比int min(Collection coll)
int max(Collection coll, Comparator c)//根据定制排序，返回最大元素，排序规则由Comparatator类控制。类比int min(Collection coll, Comparator c)

void fill(List list, Object obj)//用指定的元素代替指定list中的所有元素
    
int frequency(Collection c, Object o)//统计元素出现次数
int indexOfSubList(List list, List target)//统计target在list中第一次出现的索引，找不到则返回-1，类比int lastIndexOfSubList(List source, list target)
    
boolean replaceAll(List list, Object oldVal, Object newVal)//用新元素替换旧元素
```



### 不可变集合

`Collections`还提供了一组方法把可变集合封装成不可变集合：

- 封装成不可变List：`List unmodifiableList(List list)`
- 封装成不可变Set：`Set unmodifiableSet(Set set)`
- 封装成不可变Map：`Map unmodifiableMap(Map m)`

```java
public class Main {
    public static void main(String[] args) {
        List<String> mutable = new ArrayList<>();
        mutable.add("apple");
        mutable.add("pear");
        
        // 变为不可变集合:
        List<String> immutable = Collections.unmodifiableList(mutable);
        // 立刻扔掉mutable的引用:这样可以保证后续操作不会意外改变原始对象
        mutable = null;
        
        immutable.add("orange"); // UnsupportedOperationException!
    }
}
```



### 线程安全集合

`Collections`还提供了一组方法，可以把线程不安全的集合变为线程安全的集合：

- 变为线程安全的List：`List synchronizedList(List list)`
- 变为线程安全的Set：`Set synchronizedSet(Set s)`
- 变为线程安全的Map：`Map synchronizedMap(Map m)`

从 Java 5 开始，引入了更高效的并发集合类，所以**这几个同步方法已经没有什么用了(效率低)**。

> 需要线程安全的集合类型时请考虑使用 JUC 包下的并发集合



## Properties

- Java 集合库提供的 `Properties` 用于读写配置文件 `.properties`

- 可以从文件系统、classpath 或其他任何地方读取 `.properties` 文件

- 读写 `Properties` 时，注意仅使用 `getProperty()` 和 `setProperty()` 方法，不要调用继承而来的 `get()` 和`put() `等方法。

> 从 **JDK9** 开始，Java的 `.properties` 文件可以使用 UTF-8 编码了

因为配置文件非常常用，所以 Java 集合库提供了一个 `Properties` 来表示一组“配置”。由于历史遗留原因，`Properties`内部本质上是一个 `Hashtable`，但我们只需要用到 `Properties` 自身关于读写配置的接口。



### 读取配置文件

```java
String f = "setting.properties";
Properties props = new Properties();
props.load(new FileInputStream(f));

String filepath = props.getProperty("last_open_file");
String interval = props.getProperty("auto_save_interval", "120");
```

可见，用`Properties`读取配置文件，一共有三步：

1. 创建 `Properties` 实例

2. 调用 `load()` 读取文件

3. 调用 `getProperty()` 获取配置

    

也可以是从 jar 包中读取的资源流：

```java
Properties props = new Properties();
props.load(getClass().getResourceAsStream("/common/setting.properties"));
```



从内存读取一个字节流：

```java
import java.io.*;
import java.util.Properties;

public class Main {
    public static void main(String[] args) throws IOException {
        String settings = "# test" + "\n" + "course=Java" + "\n" + "last_open_date=2019-08-07T12:35:01";
        ByteArrayInputStream input = new ByteArrayInputStream(settings.getBytes("UTF-8"));
        Properties props = new Properties();
        props.load(input);

        System.out.println("course: " + props.getProperty("course"));
        System.out.println("last_open_date: " + props.getProperty("last_open_date"));
        System.out.println("last_open_file: " + props.getProperty("last_open_file"));
        System.out.println("auto_save: " + props.getProperty("auto_save", "60"));
    }
}
```



### 写入配置文件

如果通过 `setProperty()` 修改了 `Properties` 实例，可以把配置写入文件，以便下次启动时获得最新配置。写入配置文件使用 `store()` 方法：

```java
Properties props = new Properties();
props.setProperty("url", "http://www.liaoxuefeng.com");
props.setProperty("language", "Java");
props.store(new FileOutputStream("C:\\conf\\setting.properties"), "这是写入的properties注释");
```



### 编码

由于 `load(InputStream)` 默认总是以 ASCII 编码读取字节流，所以会导致读到乱码。我们需要用另一个重载方法`load(Reader)` 读取：

```java
Properties props = new Properties();
props.load(new FileReader("settings.properties", StandardCharsets.UTF_8));
```

