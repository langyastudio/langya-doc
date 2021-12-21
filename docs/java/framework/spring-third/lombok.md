### 依赖库

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```



### 使用

https://zhuanlan.zhihu.com/p/32779910

减少一些 get/set/toString 等方法的编写



#### @Data

注解在类 — 提供类所有属性的 get 和 set 方法，此外还提供了equals、canEqual、hashCode、toString 方法

![preview](https://img-note.langyastudio.com/20210409135345.jpeg?x-oss-process=style/watermark)



#### @Setter

注解在 **属性** — 为单个属性提供 set 方法; 

注解在 **类** 上，为该类所有的属性提供 set 方法， 都提供默认构造方法。



#### @Getter

注解在 **属性** 上；为单个属性提供 get 方法; 

注解在 **类** 上，为该类所有的属性提供 get 方法，都提供默认构造方法。



#### @Log4j2

注解在 **类** 上；为类提供一个 属性名为 log 的 log4j 日志对象，提供默认构造方法。



#### @Cleanup

这个注解用在 **变量** 前面，可以保证此变量代表的资源会被自动关闭，默认是调用资源的 close() 方法，如果该资源有其它关闭方法，可使用 @Cleanup(“methodName”) 来指定要调用的方法，也会生成默认的构造方法

![img](https://img-note.langyastudio.com/20210409135839.jpeg?x-oss-process=style/watermark)



#### @ToString

这个注解用在 **类** 上，可以生成所有参数的 toString 方法，还会生成默认的构造方法。

![img](https://img-note.langyastudio.com/20210409135923.jpeg?x-oss-process=style/watermark)



#### @AllArgsConstructor

提供一个全参数的构造方法，默认不提供无参构造。



#### @NoArgsConstructor

提供一个无参构造器



#### @RequiredArgsConstructor

注解可以生成带参或者不带参的构造方法。

若带参数，只能是类中所有带有 `@NonNull`注解的和以`final`修饰的未经初始化的字段



#### @EqualsAndHashCode

#### @Value

#### @SneakyThrows

#### @NonNull



