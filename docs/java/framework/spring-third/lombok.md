

## 依赖库

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```



## 使用

https://zhuanlan.zhihu.com/p/32779910

减少一些 get/set/toString 等方法的编写



### @Data

注解在类 — 提供类所有属性的 get 和 set 方法，此外还提供了equals、canEqual、hashCode、toString 方法

![preview](https://img-note.langyastudio.com/20210409135345.jpeg?x-oss-process=style/watermark)



### @Setter

注解在 **属性** — 为单个属性提供 set 方法; 

注解在 **类** 上，为该类所有的属性提供 set 方法， 都提供默认构造方法。



### @Getter

注解在 **属性** 上；为单个属性提供 get 方法; 

注解在 **类** 上，为该类所有的属性提供 get 方法，都提供默认构造方法。



### @Log4j2

注解在 **类** 上；为类提供一个 属性名为 log 的 log4j 日志对象，提供默认构造方法。



### [@Accessors](https://projectlombok.org/features/experimental/Accessors)

- chain 链式的，设置为true，则setter方法返回当前对象

```java
@Data
@Accessors(chain = true)
public class User {
    private Long id;
    private String name;
    
    // 生成的setter方法如下，方法体略
    public User setId(Long id) {}
    public User setName(String name) {}
}
```

- prefix 前缀，用于生成getter和setter方法的字段名会忽视指定前缀

```java
@Data
@Accessors(prefix = "p")
class User {
	private Long pId;
	private String pName;
	
	// 生成的getter和setter方法如下，方法体略
	public Long getId() {}
	public void setId(Long id) {}
	public String getName() {}
	public void setName(String name) {}
}
```



### @Cleanup

这个注解用在 **变量** 前面，可以保证此变量代表的资源会被自动关闭，默认是调用资源的 close() 方法，如果该资源有其它关闭方法，可使用 @Cleanup(“methodName”) 来指定要调用的方法，也会生成默认的构造方法

![img](https://img-note.langyastudio.com/20210409135839.jpeg?x-oss-process=style/watermark)



### @ToString

这个注解用在 **类** 上，可以生成所有参数的 toString 方法，还会生成默认的构造方法。

![img](https://img-note.langyastudio.com/20210409135923.jpeg?x-oss-process=style/watermark)



### @AllArgsConstructor

提供一个全参数的构造方法，默认不提供无参构造。



### @NoArgsConstructor

提供一个无参构造器



### @RequiredArgsConstructor

注解可以生成带参或者不带参的构造方法。

若带参数，只能是类中所有带有 `@NonNull`注解的和以`final`修饰的未经初始化的字段



### @EqualsAndHashCode

### @Value

### @SneakyThrows

### @NonNull



## 常见坑

### Setter-Getter

```java
@Data
public class NMetaVerify{
    private NMetaType nMetaType;
    private Long id;
    ....其他属性
}
```

**原因：**

Lombok 对于第一个字母小写，第二个字母大写的属性生成的 get-set 方法和 Mybatis 以及 Idea 或者说是 Java 官方认可的 get-set 方法生成的不一样

- Lombok 的默认的行为

```java

    
@Data
public class NMetaVerify {
    private Long id;
    private NMetaType nMetaType;
    private Date createTime;
    
    public void lombokFound(){
        NMetaVerify nMetaVerify = new NMetaVerify();
        nMetaVerify.setNMetaType(NMetaType.TWO); //注意：nMetaType的set方法为setNMetaType，第一个n字母大写了，
        nMetaVerify.getNMetaType();                                  //getxxxx方法也是大写
    }
}
```
- idea，Mybatis，Java 官方默认的行为

```java
public class NMetaVerify {
    private Long id;
    private NMetaType nMetaType;
    private Date createTime;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public NMetaType getnMetaType() {//注意：nMetaType属性的第一个字母小写
        return nMetaType;
    }

    public void setnMetaType(NMetaType nMetaType) {//注意：nMetaType属性的第一个字母小写
        this.nMetaType = nMetaType;
    }

    public Date getCreateTime() {
        return createTime;
    }

    public void setCreateTime(Date createTime) {
        this.createTime = createTime;
    }
}
```

- Mybatis(3.4.6版本) 解析 get-set 方法获取属性名字的源码

```java
/**
 * @author Clinton Begin
 */
public final class PropertyNamer {

    	private PropertyNamer() {
        	// Prevent Instantiation of Static Class
      	}

  		public static String methodToProperty(String name) {
    		if (name.startsWith("is")) {//is开头的一般是bool类型，直接从第二个(索引)开始截取(简单粗暴)
      				name = name.substring(2);
    		} else if (name.startsWith("get") || name.startsWith("set")) {//set-get的就从第三个(索引)开始截取
      				name = name.substring(3);
    		} else {
      				throw new ReflectionException("Error parsing property name '" + name + "'.  Didn't start with 'is', 'get' or 'set'.");
    		}
          	//下面这个判断很重要，可以分成两句话开始解释，解释如下
            //第一句话：name.length()==1
            // 						对于属性只有一个字母的，例如private int x;
            //      				对应的get-set方法是getX();setX(int x);
            //第二句话：name.length() > 1 && !Character.isUpperCase(name.charAt(1)))
            //						属性名字长度大于1，并且第二个(代码中的charAt(1)，这个1是数组下标)字母是小写的
            //						如果第二个char是大写的，那就直接返回name
    		if (name.length() == 1 || (name.length() > 1 && !Character.isUpperCase(name.charAt(1)))) {
      				name = name.substring(0, 1).toLowerCase(Locale.ENGLISH) + name.substring(1);//让属性名第一个字母小写，然后加上后面的内容
    		}

    		return name;
  		}

  		public static boolean isProperty(String name) {
    			return name.startsWith("get") || name.startsWith("set") || name.startsWith("is");
  		}

  		public static boolean isGetter(String name) {
    			return name.startsWith("get") || name.startsWith("is");
  		}

  		public static boolean isSetter(String name) {
    			return name.startsWith("set");
  		}
}
```

- 解决方案

  - 修改属性名字，让第二个字母小写，或者说是规定所有的属性的前两个字母必须小写 

  - 如果数据库已经设计好，并且前后端接口对接好了，不想修改，那就专门为这种特殊的属性使用 idea生成 get-set 方法
  
  

### Accessors

阿里云开源的 easyexcel 组件将无法读取数据
