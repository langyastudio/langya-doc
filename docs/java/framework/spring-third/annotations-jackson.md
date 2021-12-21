### 依赖库

```xml
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-annotations</artifactId>
  <version>2.12.2</version>
</dependency>
```



###　常用注解

> https://github.com/FasterXML/jackson-annotations/wiki/Jackson-Annotations

#### JsonProperty 重命名属性 

最常见的任务之一是更改用于属性的 JSON 名称：例如：

```java
public class Name {
  @JsonProperty("firstName")
  public String _first_name;
}
```

会生成如下所示的 JSON：

```json
{ "firstName" : "Bob" }
```

替换

```json
{ "_first_name" : "Bob" }
```



#### JsonIgnore 忽略属性 

有时 POJO 包含您不希望写出的属性，因此您可以执行以下操作：

```java
public class Value {
  public int value;
  @JsonIgnore 
  public int internalValue;
}
```

会生成如下所示的 JSON:

```json
{ "value" : 42 }
```



或者只想跳过 JSON 中的属性：

```java
@JsonIgnoreProperties({ "extra", "uselessValue" })
public class Value {
  public int value;
}
```



只想忽略JSON中的任何“额外”属性（POJO中没有对应的属性）。可以通过添加以下内容来完成：

```java
@JsonIgnoreProperties(ignoreUnknown=true)
public class PojoWithAny {
  public int value;
}
```



#### JsonDeserialize JsonSerialize 选择更多/更少特定类型

在读取或编写属性时使用的类型并不是您想要的：

- 读取（反序列化）时，声明的类型可能是通用类型，但是知道要使用哪种确切的实现类型
- 编写（序列化）时，默认情况下将使用特定的运行时类型。但可能不希望包括该类型的所有信息，而仅包括其超类型的内容。

这些情况可以通过以下注释来处理：

```java
public class ValueContainer {
  //尽管类型是“Value”，我们要读取JSON作为“ValueImpl” 
  @JsonDeserialize(as=ValueImpl.class)
  public Value value;

  // although runtime type may be 'AdvancedType', we really want to serialize
  // as 'BasicType'; two ways to do this:
  @JsonSerialize(as=BasicType.class)
  // or could also use: @JsonSerialize(typing=Typing.STATIC)
  public BasicType another;
}
```



#### JsonCreator 使用构造函数或工厂方法 

默认情况下，在创建值实例时，Jackson尝试使用“默认”构造函数（不带任何参数的构造函数）。但是也可以选择使用其他构造函数或静态工厂方法来创建实例。

为此，需要使用注释`@JsonCreator`，可能还需要使用注释`@JsonProperty`将名称绑定到参数：

```java
public class CtorPOJO {
   private final int _x, _y;

   @JsonCreator
   public CtorPOJO(@JsonProperty("x") int x, @JsonProperty("y") int y) {
      _x = x;
      _y = y;
   }
}
```



`@JsonCreator` 可以类似地用于静态工厂方法。但是还有另一种用法，即所谓的“委托”创建者：

```java
public class DelegatingPOJO {
   private final int _x, _y;

   @JsonCreator
   public DelegatingPOJO(Map<String,Object> delegate) {
      _x = (Integer) delegate.get("x");
      _y = (Integer) delegate.get("y");
   }
}
```

区别在于 creator 方法只能使用一个参数，并且该参数不得具有`@JsonProperty`注解。



#### JsonTypeInfo 处理多态类型 

如果需要在多个可能的子类型（即表现出多态性的子类型）中读取和写入对象的值，则可能需要启用类型信息的包含。这是必需的，以便Jackson在反序列化（将JSON读入Objects）时可以读回正确的Object类型。这可以通过`@JsonTypeInfo`在“基类”上添加注释来完成：

```java
@JsonTypeInfo(use=Id.MINIMAL_CLASS, include=As.PROPERTY, property="type") // Include Java class simple-name as JSON property "type"
@JsonSubTypes({@Type(Car.class), @Type(Aeroplane.class)}) // Required for deserialization only  
public abstract class Vehicle {
}
public class Car extends Vehicle {
  public String licensePlate;
}
public class Aeroplane extends Vehicle {
  public int wingSpan;
}

public class PojoWithTypedObjects {
  public List<Vehicle> items;
}
```

结果如下:

```json
{ "items": [
  { "type": "Car", "licensePlate": "X12345" },
  { "type": "Aeroplane", "wingSpan": 13 }
]}
```

`@JsonTypeInfo(use=DEDUCTION)`可用于避免要求“类型”字段。对于反序列化，根据可用字段推导类型。如果子类型没有字段名的唯一签名，或者JSON无法解析为单个已知签名，则会引发异常。



Note that `@JsonTypeInfo` has lots of configuration possibilities: for more information check out [Intro to polymorphic type handling](http://www.cowtowncoder.com/blog/archives/2010/03/entry_372.html)



#### JsonAutoDetect 更改属性自动检测

默认属性检测规则：

- All ''public'' fields
- All ''public'' getters ('getXxx()' methods)
- All setters ('setXxx(value)' methods), ''regardless of visibility'')

如果不起作用，则可以使用注解更改可见性级别 `@JsonAutoDetect`。例如自动检测所有字段（类似于 GSON 之类的软件包的工作方式），则可以执行以下操作：

```java
@JsonAutoDetect(fieldVisibility=JsonAutoDetect.Visibility.ANY)
public class POJOWithFields {
  private int value;
}
```

或者，完全禁用字段的自动检测：

```java
@JsonAutoDetect(fieldVisibility=JsonAutoDetect.Visibility.NONE)
public class POJOWithNoFields {
  // will NOT be included, unless there is access 'getValue()'
  public int value;
}
```

