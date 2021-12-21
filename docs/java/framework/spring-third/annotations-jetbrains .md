### 依赖库

```xml
<dependency>
    <groupId>org.jetbrains</groupId>
    <artifactId>annotations</artifactId>
    <version>20.1.0</version>
    <scope>compile</scope>
</dependency>
```



### 常用注解

https://www.jetbrains.com/help/idea/annotating-source-code.html#bundled-annotations　

通过在 **IDE 里面提示**你处理那些可能为 null 的值来解决这个问题  — NullPointerException 



#### @NotNull 

- 指示的变量，参数，或回不能为空值



####  @Nullable

- 指示的变量，参数，或返回值，该值可以为空

    

#### @Nls

- 指定程序的元素是用户可见的字符串，需要本地化



#### @NonNls

- 代码元素是一个字符串，它是用户不可见，它并不需要本地化