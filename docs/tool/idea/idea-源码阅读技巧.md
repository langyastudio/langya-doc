> 本文来自JavaGuide，郎涯进行简单排版与补充



项目有个新来了一个小伙伴，他看我查看项目源代码的时候，各种骚操作“花里胡哨”的。于是他向我请教，想让我分享一下我平时使用 IDEA 看源码的小技巧。

这一部分的内容主要是一些我平时看源码的时候常用的快捷键/小技巧！非常好用！

掌握这些快捷键/小技巧，看源码的效率提升一个等级！



## 查看当前类的层次结构

| 使用频率 | 相关快捷键                               |
| -------- | ---------------------------------------- |
| ⭐⭐⭐⭐⭐    | `Ctrl + H` or Navigate -> Type Hierarchy |

平时，我们阅读源码的时候，经常需要查看类的层次结构。就比如我们遇到抽象类或者接口的时候，经常需要查看其被哪些类实现。

拿 Spring 源码为例，`BeanDefinition` 是一个关于 Bean 属性/定义的接口。

```java
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {
  ......
}
```

如果我们需要查看 `BeanDefinition` 被哪些类实现的话，只需要把鼠标移动到 `BeanDefinition` 类名上，然后使用快捷键 `Ctrl + H` 即可。

![](https://img-note.langyastudio.com/202111171116439.png?x-oss-process=style/watermark)

同理，如果你想查看接口 `BeanDefinition` 继承的接口 `AttributeAccessor` 被哪些类实现的话，只需要把鼠标移动到 `AttributeAccessor` 类名上，然后使用快捷键 `Ctrl + H` 即可。



## 查看类结构

| 使用频率 | 相关快捷键                                                   |
| -------- | ------------------------------------------------------------ |
| ⭐⭐⭐⭐     | `Alt + 7`(Win) / `Command +7` （Mac）or Navigate -> File Structure |

类结构可以让我们快速了解到当前类的方法、变量/常量，非常使用！

我们在对应的类的任意位置使用快捷键 `Alt + 7`(Win) / `Command +7` （Mac）即可。

![](https://img-note.langyastudio.com/202111171116201.png?x-oss-process=style/watermark)



## 快速检索类

| 使用频率 | 相关快捷键                               |
| -------- | ---------------------------------------- |
| ⭐⭐⭐⭐⭐    | `Ctrl + N` (Win) / `Command + O` （Mac） |

使用快捷键 `Ctrl + N` (Win) / `Command + O` （Mac）可以快速检索类/文件。

![](https://img-note.langyastudio.com/202111171116015.png?x-oss-process=style/watermark)



## 关键字检索

| 使用频率 | 相关快捷键 |
| -------- | ---------- |
| ⭐⭐⭐⭐⭐    | 见下文     |

- 当前文件下检索 ： `Ctrl + F` (Win) / `Command + F` （Mac）

- 全局的文本检索 : `Ctrl + Shift + F` (Win) / `Command + Shift + F` （Mac）

  - 搜索 jar 中内容

    ![image-20220929204835042](https://img-note.langyastudio.com/202209292048137.png?x-oss-process=style/watermark)
  
  - class 文件必须关联源码才能搜其文本
  
    ![image-20220929210526848](https://img-note.langyastudio.com/202209292105924.png?x-oss-process=style/watermark)



## 查看方法/类的实现类

| 使用频率 | 相关快捷键                                         |
| -------- | -------------------------------------------------- |
| ⭐⭐⭐⭐     | `Ctrl + Alt + B` (Win) / `Command + Alt + B` (Mac) |

如果我们想直接跳转到某个方法/类的实现类，直接在方法名或者类名上使用快捷键 `Ctrl + Alt + B/鼠标左键` (Win) / `Command + Alt + B/鼠标左键` (Mac) 即可。

如果对应的方法/类只有一个实现类的话，会直接跳转到对应的实现类。

比如 `BeanDefinition` 接口的 `getBeanClassName()` 方法只被 `AbstractBeanDefinition` 抽象类实现，我们对这个方法使用快捷键就可以直接跳转到 `AbstractBeanDefinition` 抽象类中对应的实现方法。

```java
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {
  @Nullable
	String getBeanClassName();
  ......
}
```

如果对应的方法/类有多个实现类的话，IDEA 会弹出一个选择框让你选择。

比如 `BeanDefinition` 接口的 `getParentName()` 方法就有多个不同的实现。

![](https://img-note.langyastudio.com/202111171116191.png?x-oss-process=style/watermark)



## 查看方法被使用的情况

| 使用频率 | 相关快捷键 |
| -------- | ---------- |
| ⭐⭐⭐⭐     | `Alt + F7` |

我们可以通过直接在方法名上使用快捷键 `Alt + F7` 来查看这个方法在哪些地方被调用过。

![](https://img-note.langyastudio.com/202111171116200.png?x-oss-process=style/watermark)



## 查看最近使用的文件

| 使用频率 | 相关快捷键                             |
| -------- | -------------------------------------- |
| ⭐⭐⭐⭐⭐    | `Ctrl + E`(Win) / `Command +E` （Mac） |

你可以通过快捷键 `Ctrl + E`(Win) / `Command +E` （Mac）来显示 IDEA 最近使用的一些文件。

![](https://img-note.langyastudio.com/202111171117040.png?x-oss-process=style/watermark)



## 查看图表形式的类继承链

| 使用频率 | 相关快捷键               |
| -------- | ------------------------ |
| ⭐⭐⭐⭐     | 相关快捷键较多，不建议记 |

点击类名 **右键** ，选择 **Shw Diagrams** 即可查看图表形式的类继承链。

![](https://img-note.langyastudio.com/202111171117017.png?x-oss-process=style/watermark)

你还可以对图表进行一些操作。比如，你可以点击图表中具体的类 **右键**，然后选择显示它的实现类或者父类。

![](https://img-note.langyastudio.com/202111171117912.png?x-oss-process=style/watermark)

再比如你还可以选择是否显示类中的属性、方法、内部类等等信息。

![](https://img-note.langyastudio.com/202111171117201.png?x-oss-process=style/watermark)

如果你想跳转到对应类的源码的话，直接点击图表中具体的类 **右键** ，然后选择 **Jump to Source** 。

![](https://img-note.langyastudio.com/202111171117497.png?x-oss-process=style/watermark)



