> 以下内容来自网络，具体参考链接找不到了

### 什么是YAML

YAML是"YAML Ain’t a Markup Language"（YAML不是一种标记语言）的递归缩写。YAML的意思其实是：“Yet Another Markup Language”（仍是一种标记语言）。主要强度这种语音是以数据为中心，而不是以标记语音为重心，例如像 xml语言就会使用大量的标记。

YAML是一个可读性高，易于理解，用来表达数据序列化的格式。它的语法和其他高级语言类似，并且可以简单表达清单（数组）、散列表，标量等数据形态。它使用空白符号缩进和大量依赖外观的特色，特别适合用来表达或编辑数据结构、各种配置文件等。

YAML 的配置文件后缀为 .yml，例如 Springboot 项目中使用到的配置文件 application.yml 。



### 基本语法

- YAML使用可打印的Unicode字符，可使用UTF-8或UTF-16
- 数据结构采用键值对的形式，即 键名称: 值，注意冒号后面要有空格
- 每个清单（数组）成员以单行表示，并用短杠+空白（- ）起始。或使用方括号（[]），并用逗号+空白（, ）分开成员
- 每个散列表的成员用冒号+空白（: ）分开键值和内容。或使用大括号（{ }），并用逗号+空白（, ）分开
- 字符串值一般不使用引号，必要时可使用，**使用双引号表示字符串时，会转义字符串中的特殊字符（例如\n）。使用单引号时不会转义字符串中的特殊字符**
- 大小写敏感
- **使用缩进表示层级关系，缩进不允许使用tab，只允许空格，因为有可能在不同系统下tab长度不一样**
- 缩进的空格数可以任意，只要相同层级的元素左对齐即可
- 在单一文件中，可用连续三个连字号（—）区分多个文件。还有选择性的连续三个点号（…）用来表示文件结尾
- '#' 表示注释，可以出现在一行中的任何位置，单行注释
- 在使用逗号及冒号时，后面都必须接一个空白字符，所以可以在字符串或数值中自由加入分隔符号（例如：5,280 或 http://www.wikipedia.org）而不需要使用引号



### 数据类型

- 标量（scalars）：单个的、不可再分的值

- 对象：键值对的集合，又称为映射（mapping）/ 哈希（hashes） / 字典（dictionary）

- 数组：一组按次序排列的值，又称为序列（sequence） / 列表（list）

    

#### 标量

标量是最基础的数据类型，不可再分的值，他们一般用于表示单个的变量，有以下七种：

- 字符串
- 布尔值
- 整数
- 浮点数
- **Null**
- 时间
- 日期

```yml
# 字符串
string.value: Hello!我是陈皮!
# 布尔值，true或false
boolean.value: true
boolean.value1: false
# 整数
int.value: 10
int.value1: 0b1010_0111_0100_1010_1110 # 二进制
# 浮点数
float.value: 3.14159
# 科学计数法
float.value1: 314159e-5 

# Null，~ 代表null
null.value: ~

# 时间，时间使用ISO 8601格式，时间和日期之间使用T连接，最后使用+代表时区
datetime.value: !!timestamp 2021-04-13T10:31:00+08:00
# 日期,日期必须使用ISO 8601格式，即yyyy-MM-dd
date.value: !!timestamp 2021-04-13
```



#### 对象

我们知道单个变量可以用键值对，使用冒号结构表示 key: value，注意冒号后面要加一个空格。可以使用缩进层级的键值对表示一个对象，如下所示：

```yml
person:
  name: 陈皮
  age: 18
  man: true
```



#### 数组

可以用短横杆加空格 -开头的行组成数组的每一个元素，如下的address字段：

```yml
person:
  name: 陈皮
  age: 18
  man: true
  address:
    - 深圳
    - 北京
    - 广州
```

也可以使用中括号进行行内显示形式，如下：

```yml
person:
  name: 陈皮
  age: 18
  man: true
  address:
    [深圳, 北京, 广州]
```

如果数组字段的成员也是一个数组，可以使用嵌套的形式，如下：

```yml
person:
  name: 陈皮
  age: 18
  man: true
  address: [深圳, 北京, 广州]
  twoArr:
    -
      - 2
      - 3
      - 1
    -
      - 10
      - 12
      - 30
```

如果数组成员是一个对象，则用如下两种形式形式：

```yml
childs:
  -
    name: 小红
    age: 10
  -
    name: 小王
    age: 15
```



### 文本块

如果你想引入多行的文本块，可以使用|符号，注意在冒号 : 和 | 符号之间要有空格。

```yml
person:
  name: |
    Hello Java!!
    I am fine!
    Thanks! GoodBye!
```

它和加双引号的效果一样，双引号能转义特殊字符：

```java
person:
  name: "Hello Java!!\nI am fine!\nThanks! GoodBye!"
```



### 显示指定类型

有时我们需要显示指定某些值的类型，可以使用 !（感叹号）显式指定类型。!单叹号通常是自定义类型，!!双叹号是内置类型，例如：

```yml
# 指定为字符串
string.value: !!str HelloWorld!
# !!timestamp指定为日期时间类型
datetime.value: !!timestamp 2021-04-13T02:31:00+08:00
```

内置的类型如下：

- !!int：整数类型
- !!float：浮点类型
- !!bool：布尔类型
- !!str：字符串类型
- !!binary：二进制类型
- !!timestamp：日期时间类型
- !!null：空值
- !!set：集合类型
- !!omap，!!pairs：键值列表或对象列表
- !!seq：序列
- !!map：散列表类型



### 引用

引用会用到 &锚点符合和 *星号符号，&用来建立锚点，<< 表示合并到当前数据，* 用来引用锚点。

```yml
xiaohong: &xiaohong
  name: 小红
  age: 20

dept:
  id: D15D8E4F6D68A4E88E
  <<: *xiaohong
```

上面最终相当于如下：

```yml
xiaohong:
  name: 小红
  age: 20

dept:
  id: D15D8E4F6D68A4E88E
  name: 小红
  age: 20
```

还有一种文件内引用，引用已经定义好的变量，如下：

```yml
base.host: https://chenpi.com
add.person.url: ${base.host}/person/add
```

