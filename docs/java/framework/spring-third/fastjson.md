### 依赖库

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.76</version>
</dependency>
```



### 使用

https://github.com/alibaba/fastjson/wiki/FastJson-%E6%96%87%E6%A1%A3%E9%93%BE%E6%8E%A5

- 支持将Java Bean序列化为JSON字符串，也可以从JSON字符串反序列化到JavaBean
- 支持泛型，支持流处理超大文本，支持枚举，支持序列化和反序列化扩展



API 十分简洁：

```java
String text = JSON.toJSONString(obj); //序列化
VO vo = JSON.parseObject("{...}", VO.class); //反序列化
```



默认序列化 Date 输出使用 ”yyyy-MM-dd HH:mm:ss” 格式，可以用 UseISO8601DateFormat 特性换成 ”yyyy-MM-dd’T’HH:mm:ss” 格式

```java
JSON.defaultTimeZone = TimeZone.getTimeZone("Asia/Shanghai");
JSON.defaultLocale = Locale.US;
        
public static class Model {
        @JSONField(format = "MMM dd, yyyy h:mm:ss aa")
        private java.util.Date date;

        public java.util.Date getDate() {
            return date;
        }

        public void setDate(java.util.Date date) {
            this.date = date;
        }

        @JSONField(format = "MMM-dd-yyyy h:mm:ss aa")
        public java.sql.Date date2;
}
```



#### 常见序列化特性的使用

- QuoteFieldNames, //key使用引号

- UseSingleQuotes, //使用单引号

- WriteMapNullValue, //输出Map的null值

- WriteEnumUsingToString, //枚举属性输出toString的结果

- WriteEnumUsingName, //枚举数据输出name

- UseISO8601DateFormat, //使用日期格式

- WriteNullListAsEmpty, //List为空则输出[]

- WriteNullStringAsEmpty, //String为空则输出””

- WriteNullNumberAsZero, //Number类型为空则输出0

- WriteNullBooleanAsFalse, //Boolean类型为空则输出false

- SkipTransientField,

- SortField, //排序字段

- WriteTabAsSpecial,

- PrettyFormat, // 格式化JSON缩进

- WriteClassName, // 输出类名
- DisableCircularReferenceDetect, // 禁止循环引用
- WriteSlashAsSpecial, // 对斜杠’/’进行转义
- BrowserCompatible,
- WriteDateUseDateFormat, // 全局修改日期格式,默认为false。JSON.DEFFAULT_DATE_FORMAT = “yyyy-MM-dd”;JSON.toJSONString(obj, SerializerFeature.WriteDateUseDateFormat);
- NotWriteRootClassName,
- DisableCheckSpecialChar,
- BeanToArray,
- WriteNonStringKeyAsString,
- NotWriteDefaultValue,
- BrowserSecure,
- IgnoreNonFieldGetter,
- WriteNonStringValueAsString,
- IgnoreErrorGetter,
- WriteBigDecimalAsPlain,
- MapSortField



#### JSONField

https://github.com/alibaba/fastjson/wiki/JSONField

可以配置在属性（setter、getter）和字段（必须是public field）上。

##### getter/setter

```java
 public class A {
      private int id;
 
      @JSONField(name="ID")
      public int getId() {return id;}
      @JSONField(name="ID")
      public void setId(int value) {this.id = id;}
 }
```



##### field

```java
 public class A {
      @JSONField(name="ID")
      private int id;
 
      public int getId() {return id;}
      public void setId(int value) {this.id = id;}
 }
```



##### format

```java
 public class A {
      // 配置date序列化和反序列使用yyyyMMdd日期格式
      @JSONField(format="yyyyMMdd")
      public Date date;
 }
```



##### serialize/deserialize

```java
 public class A {
      @JSONField(serialize=false)
      public Date date;
 }

 public class A {
      @JSONField(deserialize=false)
      public Date date;
 }
```



##### ordinal

缺省fastjson序列化一个java bean，是根据fieldName的字母序进行序列化的，你可以通过ordinal指定字段的顺序。这个特性需要1.1.42以上版本。

```java
public static class VO {
    @JSONField(ordinal = 3)
    private int f0;

    @JSONField(ordinal = 2)
    private int f1;

    @JSONField(ordinal = 1)
    private int f2;
}
```



##### serializeUsing

在fastjson 1.2.16版本之后，JSONField 支持新的定制化配置 serializeUsing，可以单独对某一个类的某个属性定制序列化，比如：

```java
public static class Model {
    @JSONField(serializeUsing = ModelValueSerializer.class)
    public int value;
}

public static class ModelValueSerializer implements ObjectSerializer {
    @Override
    public void write(JSONSerializer serializer, Object object, Object fieldName, Type fieldType, int features) throws IOException {
        Integer value = (Integer) object;
        String text = value + "元";
        serializer.write(text);
    }
}
```

测试代码

```java
Model model = new Model();
model.value = 100;
String json = JSON.toJSONString(model);
Assert.assertEquals("{\"value\":\"100元\"}", json);
```



#### JSONType

https://github.com/alibaba/fastjson/wiki/JSONType_seeAlso_cn

SONType.alphabetic 属性: fastjson 缺省时会使用字母序序列化，如果你是希望按照 java fields/getters 的自然顺序序列化，可以配置 JSONType.alphabetic，使用方法如下：

```java
@JSONType(alphabetic = false)
public static class B {
    public int f2;
    public int f1;
    public int f0;
}
```



#### 自定义序列化与反序列化

https://github.com/alibaba/fastjson/wiki/ObjectSerializer_cn

https://github.com/alibaba/fastjson/wiki/ObjectDeserializer_cn

 

#### 自定义序列化之过滤器

- 全局的过滤器：JSON.toJSONString方法的参数中可以配置处理所有类型的SerializeFilter
- 类级别过滤器：[Class_Level_SerializeFilter](https://github.com/alibaba/fastjson/wiki/Class_Level_SerializeFilter)
- 属性过滤器：[使用PropertyPreFilter过滤属性](https://github.com/alibaba/fastjson/wiki/使用SimplePropertyPreFilter过滤属性)
- 多余字段处理器：[ExtraProcessor 用于处理多余的字段、
    ExtraTypeProvider用于处理多余字段时提供类型信息](https://github.com/alibaba/fastjson/wiki/ParseProcess)
- 定制反序列化：[在fastjson-1.2.9版本后提供了ExtraProcessable接口，用于定制对象的反序列化功能](https://github.com/alibaba/fastjson/wiki/ExtraProcessable)，可用于添加没有的字段
- 标签过滤：[JSONField(label)，相当于分组](https://github.com/alibaba/fastjson/wiki/LabelFilter)
- 自动识别嵌套对象子类型：[FieldTypeResolver](https://github.com/alibaba/fastjson/wiki/FieldTypeResolver)



