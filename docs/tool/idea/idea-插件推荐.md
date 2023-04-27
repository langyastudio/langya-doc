> 本文来自JavaGuide，郎涯进行简单排版与补充



## Alibaba Java Coding Guidelines 代码规范

阿里巴巴 Java **代码规范**，对应的Github地址为：[https://github.com/alibaba/p3c](https://github.com/alibaba/p3c ) 

根据官方描述：

> 目前这个插件实现了开发手册中的的53条规则，大部分基于PMD实现，其中有4条规则基于IDEA实现，并且基于IDEA [Inspection](https://www.jetbrains.com/help/idea/code-inspection.html)实现了实时检测功能。部分规则实现了Quick Fix功能，对于可以提供Quick Fix但没有提供的，我们会尽快实现，也欢迎有兴趣的同学加入进来一起努力。 目前插件检测有两种模式：实时检测、手动触发。

上述提到的开发手册也就是在Java开发领域赫赫有名的《阿里巴巴Java开发手册》。

**手动配置检测规则**

你还可以手动配置相关 inspection规则：

![](https://img-note.langyastudio.com/202111171141439.png?x-oss-process=style/watermark)



**使用效果**

这个插件会实时检测出我们的代码不匹配它的规则的地方，并且会给出修改建议。比如我们按照下面的方式去创建线程池的话，这个插件就会帮我们检测出来,如下图所示。

![](https://img-note.langyastudio.com/202111171141042.png?x-oss-process=style/watermark)

这个可以对应上 《阿里巴巴Java开发手册》 这本书关于创建线程池的方式说明。

![](https://img-note.langyastudio.com/202111171143138.png?x-oss-process=style/watermark)



## Codota AI Autocomplete  代码智能提示

> or Alibaba Cloud AI Coding Assistant
>
> or Tabnine Autocomplete AI

Codota 这个插件用于**智能代码补全**，它基于数百万 Java 程序，能够根据程序上下文提示补全代码。相比于 IDEA 自带的智能提示来说，Codota 的提示更加全面一些。Codota 插件的基础功能都是免费的。你的代码也不会被泄露，这点你不用担心。

除了，在写代码的时候智能提示之外，还可以直接选中代码然后搜索相关代码示例：

![](https://img-note.langyastudio.com/202111171141377.png?x-oss-process=style/watermark)



## SonarLint 代码优化

SonarLint 帮助你发现**代码的错误和漏洞**，就像是代码拼写检查器一样，SonarLint 可以实时显示出代码的问题，并提供清晰的修复指导，以便你提交代码之前就可以解决它们。

![](https://img-note.langyastudio.com/202111171141017.png?x-oss-process=style/watermark)



## MybatisCodeHelperPro

> or Free MyBatis plugin+MyBatisX

**Mybatis 支持插件**。最好的 Mybatis 代码提示、完整支持 Mybatis 动态 sql 代码提示、代码检测、写sql几乎所有地方都有代码提示。

[使用视频地址](https://www.bilibili.com/video/av50632948) + [文档](https://gejun123456.github.io/MyBatisCodeHelper-Pro/#/README?id=%e5%8a%9f%e8%83%bd)

| 功能点                                                       | 未激活版 | 激活版 |
| ------------------------------------------------------------ | -------- | ------ |
| 接口与xml互相跳转 更换图标                                   | ✔        | ✔      |
| 接口方法名重构                                               | ✔        | ✔      |
| 一键添加param                                                | ✔        | ✔      |
| xml中的 param的自动提示 resultMap refid 等的自动提示         | ✔        | ✔      |
| resultMap中的property的自动提示                              | ✔        | ✔      |
| 检测没有使用的xml 可一键删除                                 | ✔        | ✔      |
| 检测mybatis接口中方法是否有实现，没有则报红 可创建一个空的xml方法块 | ✔        | ✔      |
| 检测resultmap的property是否有误                              | ✔        | ✔      |
| 支持spring 将mapper注入到spring中 intellij的spring注入不再报错 支持springboot | ✔        | ✔      |
| 一键生成分页查询                                             | ✔        | ✔      |
| 代码模版，生成cdata和collection语句                          | ✔        | ✔      |
| 一键添加resultMap中未被使用的属性                            | ✔        | ✔      |
| 一键生成mybatis接口的testcase                                | ✘        | ✔      |
| 通过方法名生成sql                                            | ✘        | ✔      |
| 通过数据库生成crud代码                                       | ✘        | ✔      |
| 通过java类生成crud代码                                       | ✘        | ✔      |
| xml collection中的 param提示                                 | ✘        | ✔      |
| 识别mybatis的标签 全自动sql补全                              | ✘        | ✔      |
| 检测#{中的参数是否正确                                       | ✘        | ✔      |
| if test when test foreach collection $中的OGNL支持           | ✘        | ✔      |
| param重构功能(2.7.2)                                         | ✘        | ✔      |
| resultMap中column提示与检测(2.7.2)                           | ✘        | ✔      |
| Mybatis xml代码格式化(2.8.2)                                 | ✘        | ✔      |



#### 配置数据库

- 配置数据库 数据库名一定要填 数据库无法连接请切换驱动的版本
- 当库里面有多个schema时，每个schema都要配置一遍。比如 mysql localhost:3306 里面有多个库时，需要在idea配置多个，而不是一个选多个schema 

![configureDatabase](https://img-note.langyastudio.com/20210420111115.png?x-oss-process=style/watermark)



配置当前项目对应的数据库类型(达梦数据库请配置为GenericSql字段便可自动提示,或者本地装一个oracle把表拷过去 这样一些函数也能自动提示) 另外mysql如果是mariadb一定要配置为mariadb

![databaseConfig](https://img-note.langyastudio.com/20210420111144.png?x-oss-process=style/watermark)



配置插件方法名生成对应的数据库

![image-20210420115318887](https://img-note.langyastudio.com/20210420115318.png?x-oss-process=style/watermark)

![image-20210420113215467](https://img-note.langyastudio.com/20210420113215.png?x-oss-process=style/watermark)



如果有多个数据库，并且有相同的表名需要配置

![resolutionScope](https://img-note.langyastudio.com/20210420111244.png?x-oss-process=style/watermark)



#### 生成XML

![image-20210420113011225](https://img-note.langyastudio.com/20210420113013.png?x-oss-process=style/watermark)



#### 配置字段

- 替换字段中的字符

  如下图所示，可以通过正则表达式来修改字段的字符，将 `F_` 替换为 空字符串

  ![image-20220905182711819](https://img-note.langyastudio.com/202209051827886.png?x-oss-process=style/watermark)

- 定制列

  可以通过设置菜单中的 ProjectConfig 配置转换关系，如：将 F_ 开头的移除

  ```java
  osp:import org.apache.commons.lang3.StringUtils;
  
  public class ConverterImpl {
      public String convertColumnNameToPropertyName(String columnName) {
          // TODO: fill content in method, you can use method from commonslang3 and guava
          if(StringUtils.startsWith(columnName, "F_")){
              return StringUtils.substring(columnName, 2);
          }       
  
          return columnName;
      }
  }
  ```

  

  ![image-20220905183308820](https://img-note.langyastudio.com/202209051833893.png?x-oss-process=style/watermark)





## Sequence Diagram 时序图

序列图（Sequence Diagram）是一种 [UML](https://zh.m.wikipedia.org/wiki/UML) 行为图，表示系统执行某个方法/操作（如登录操作）时，**对象之间的顺序调用关系**。常用于阅读源码、梳理业务逻辑等。

通过 SequenceDiagram 这个插件，我们一键可以生成时序图。并且，还可以：

- 点击时序图中的类/方法即可跳转到对应的地方

- 从时序图中删除对应的类或者方法

- 将生成的时序图导出为 PNG 图片格式

下图是微信支付的业务流程时序图。这个图描述了微信支付相关角色（顾客，商家...）在微信支付场景下，基础支付和支付的的顺序调用关系。

![](https://img-note.langyastudio.com/202111171130464.png?x-oss-process=style/watermark)



**简单使用**

选中方法名（注意不要选类名），然后点击鼠标右键，选择 **Sequence Diagram** 选项即可！

![](https://img-note.langyastudio.com/202111171121222.png?x-oss-process=style/watermark)



配置生成的序列图的一些基本的参数比如调用深度之后，我们点击 ok 即可！

![](https://img-note.langyastudio.com/202111171121799.png?x-oss-process=style/watermark)



时序图生成完成之后，你还可以选择将其导出为图片

![](https://img-note.langyastudio.com/202111171121969.png?x-oss-process=style/watermark)



## jclasslib 字节码查看

相比于 IDEA 自带的  `View -> Show Bytecode`  查看类字节的功能，使用 `jclasslib` 不光可以直观地查看某个类对应的字节码文件，还可以查看类的基本信息、常量池、接口、属性、函数等信息。

点击 `View -> Show Bytecode With jclasslib` 即可通过 `jclasslib` 查看某个类对应的字节码文件。

![使用IDEA插件jclasslib查看类的字节码](https://img-note.langyastudio.com/202111231612771.png?x-oss-process=style/watermark)

![](https://img-note.langyastudio.com/202111231612019.png?x-oss-process=style/watermark)



## Maven Helper 解决 Maven 依赖冲突问题

Maven Helper 主要用来分析 **Maven 项目的相关依赖**，可以帮助我们解决 Maven 依赖冲突问题。

![](https://img-note.langyastudio.com/202111171140731.png?x-oss-process=style/watermark)



**何为依赖冲突？**

说白了就是你的项目使用的 2 个 jar 包引用了同一个依赖 h，并且 h 的版本还不一样，这个时候你的项目就存在两个不同版本的 h。这时 Maven 会依据依赖路径最短优先原则，来决定使用哪个版本的 Jar 包，而另一个无用的 Jar 包则未被使用，这就是所谓的依赖冲突。

大部分情况下，依赖冲突可能并不会对系统造成什么异常，因为 Maven 始终选择了一个 Jar 包来使用。但是，**不排除在某些特定条件下，会出现类似找不到类的异常**，所以，只要存在依赖冲突，在我看来，最好还是解决掉，不要给系统留下隐患。



## Translation 翻译

[https://github.com/YiiGuxing/TranslationPlugin](https://github.com/YiiGuxing/TranslationPlugin) 

有了这个插件之后，再也不用在编码的时候打开浏览器查找某个单词怎么拼写、某句英文注释什么意思了。并且，这个插件支持多种翻译源：Google 翻译、Youdao 翻译、Baidu 翻译。

除了**翻译**功能之外还提供了**语音朗读**、**单词本**等实用功能。



选中你要翻译的单词或者句子，使用快捷键 `command+ctrl+u(mac)` / `shift+ctrl+y(win/linux)`

![](https://img-note.langyastudio.com/202111171142274.jpg?x-oss-process=style/watermark)



如果需要快速打开翻译框，使用快捷键`command+ctrl+i(mac)`/`ctrl + shift + o(win/linux)`

![](https://img-note.langyastudio.com/202111171142024.png?x-oss-process=style/watermark)



## Key promoter X 快捷键

这个插件的功能主要是在你本可以使用快捷键操作的地方**提醒你用快捷键操作** 

举个例子。我直接点击 tab 栏下的菜单打开 Version Control(版本控制) 的话，这个插件就会提示你可以用快捷键 `command+9 `或者 `shift+command+9` 打开:

![](https://img-note.langyastudio.com/202111180957239.png?x-oss-process=style/watermark)



## Presentation Assistant

安装这个插件之后，你使用的**快捷键操作都会被可视化地展示出来**，非常适合自己在录制视频或者给别人展示代码的时候使用。

举个例子。我使用快捷键  `command+9` 打开 Version Control：

![](https://img-note.langyastudio.com/202111180958242.gif?x-oss-process=style/watermark)

从上图可以很清晰地看到，IDEA 的底部中间的位置将我刚刚所使用的快捷键给展示了出来。



## Statistic 代码统计

有了这个插件之后你可以非常直观地看到你的项目中**所有类型的文件的信息**比如数量、大小等等，可以帮助你更好地了解你们的项目。

![](https://img-note.langyastudio.com/202111171140811.png?x-oss-process=style/watermark)

你还可以使用它看所有类的总行数、有效代码行数、注释行数、以及有效代码比重等等这些东西。

![](https://img-note.langyastudio.com/202111171140675.png?x-oss-process=style/watermark)



## Other

| 名称                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Alibaba Cloud Toolkit                                        | 云运营                                                       |
| arthas idea                                                  | 基于IDEA开发的Arthas命令生成插件                             |
|                                                              |                                                              |
| MybatisLogFormat                                             | Extract the SQL and fill the parameters into the SQL         |
| RoboPOJOGenerator                                            | json转object                                                 |
| gson-format / GsonFormatPlus                                 | JSON转对象（`option + s` (Mac) 或 `alt + s` (win)）          |
| MapStruct support                                            | 基于 Java 注解的对象属性映射工具                             |
| Java Stream Debugger                                         | Java Stream 调试器                                           |
| camel-case                                                   | 命名格式切换（`shift + option + u` (mac) / `shift + alt + u` (win) ） |
| VisualVM Launcher                                            | Java 性能分析神器                                            |
| [Momo Code Sec Inspector](https://github.com/momosecurity/momo-code-sec-inspector-java) | 代码安全审计                                                 |
| Checkstyle-IDEA                                              | 代码风格检查                                                 |
| stackoverflow                                                | 快速跳转到 stackoverflow                                     |
|                                                              |                                                              |
| CodeGlance                                                   | 微型地图快速导航                                             |
| [Theme](https://plugins.jetbrains.com/search?tags=Theme)     | IDEA主题                                                     |
| Grep Console                                                 | 控制台输出美化                                               |
| Rainbow Brackets                                             | 彩虹括号                                                     |





