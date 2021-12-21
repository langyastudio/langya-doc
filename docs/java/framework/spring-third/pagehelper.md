### 依赖

```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.3.0</version>
</dependency>
```



### 配置

https://github.com/pagehelper/Mybatis-PageHelper/blob/master/wikis/zh/HowToUse.md

> 不要配置多个分页插件
>
> 只有紧跟在 PageHelper.startPage 方法后的第一个 Mybatis 的查询（Select）方法会被分页
>
> 分页插件不支持带有 for update 语句的分页
>
> 分页插件不支持嵌套结果映射

```yml
pagehelper:
  helperDialect: mariadb
  # 当 pageSize=0 或者 RowBounds.limit = 0 就会查询出全部的结果
  pageSizeZero: true
  # 该参数对使用 RowBounds 作为分页参数时有效。
  # 当该参数设置为true时，使用 RowBounds 分页会进行 count 查询。
  row-bounds-with-count: true
  # pageNum=pageNum;pageSize=pageSize;count=countSql;reasonable=reasonable;pageSizeZero=pageS
  # 支持通过 Mapper 接口参数来传递分页参数
  supportMethodsArguments: true
  params: pageNum=offSet;pageSize=pageSize;
```



### 使用

#### RowBounds 方式的调用

分页插件检测到使用了 RowBounds 参数时，就会对该查询进行**物理分页**。

```java
//这种情况下也会进行物理分页查询
List<User> selectAll(RowBounds rowBounds);
```



#### 静态方法调用

除了 `PageHelper.startPage` 方法外，还提供了类似用法的 `PageHelper.offsetPage` 方法。

在你需要进行分页的 MyBatis 查询方法前调用 `PageHelper.startPage` 静态方法即可，紧跟在这个方法后的第一个**MyBatis 查询方法**会被进行分页。

```java
//request: url?pageNum=1&pageSize=10
//支持 ServletRequest,Map,POJO 对象，需要配合 params 参数
PageHelper.startPage(request);
//紧跟着的第一个select方法会被分页
List<User> list = userMapper.selectIf(1);

//后面的不会被分页，除非再次调用PageHelper.startPage
List<User> list2 = userMapper.selectIf(null);
//list1
assertEquals(2, list.get(0).getId());
assertEquals(10, list.size());

//分页时，实际返回的结果list类型是Page<E>，如果想取出分页信息，需要强制转换为Page<E>，
//或者使用PageInfo类（下面的例子有介绍）
assertEquals(182, ((Page) list).getTotal());
```



#### 使用参数方式

想要使用参数方式，需要配置 `supportMethodsArguments` 参数为 `true`，同时要配置 `params` 参数。 

在 MyBatis 方法中：

```java
List<User> selectByPageNumSize(
        @Param("user") User user,
        @Param("offSet") int pageNum, 
        @Param("pageSize") int pageSize);
```

当调用这个方法时，由于同时发现了 `offSet` 和 `pageSize` 参数，这个方法就会被分页。params 提供的几个参数都可以这样使用。

除了上面这种方式外，如果 User 对象中包含这两个参数值，也可以有下面的方法：

```java
List<User> selectByPageNumSize(User user);
```

当从 User 中同时发现了 `offSet` 和 `pageSize` 参数，这个方法就会被分页。

> 注意： `offSet` 和 `pageSize`  两个属性同时存在才会触发分页操作，在这个前提下，其他的分页参数才会生效



#### lambda用法

```java
//jdk8 lambda用法
Page<User> page = PageHelper.startPage(1, 10).doSelectPage(()-> userMapper.selectGroupBy());

//count查询，返回一个查询语句的count数
total = PageHelper.count(()->userMapper.selectLike(user));
```



#### 安全调用

如果你写出下面这样的代码，就是不安全的用法：

```java
PageHelper.startPage(1, 10);
List<User> list;
if(param1 != null){
    list = userMapper.selectIf(param1);
} else {
    list = new ArrayList<User>();
}
```

这种情况下由于 param1 存在 null 的情况，就会导致 PageHelper 生产了一个分页参数，但是没有被消费，这个参数就会一直保留在这个线程上。当这个线程再次被使用时，就可能导致不该分页的方法去消费这个分页参数，这就产生了莫名其妙的分页。



上面这个代码，应该写成下面这个样子：这种写法就能保证安全

```java
List<User> list;
if(param1 != null){
    PageHelper.startPage(1, 10);
    list = userMapper.selectIf(param1);
} else {
    list = new ArrayList<User>();
}
```



