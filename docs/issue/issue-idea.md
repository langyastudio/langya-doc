### run / debug 灰色

问题描述：

设置了 source Root 文件夹

但 run/debug 显示灰色不可用，它右侧的两个按钮可用

![image-20210305172249966](https://img-note.langyastudio.com/20210305172250.png?x-oss-process=style/watermark)



**解决方法：**

重启IDE



### java代码cannot find declaration to go to

﻿﻿使用 IntelliJ IDEA 直接打开以前写的 web 测试项目，发现 java 的源码没有高亮显示，且不能使用 ctrl + 单击进行目标类跳转操作。哪怕安装并配置了 JDK，也不能在项目中直接查看声明的原类。

尝试重装 IDE、重新导入项目、更换 JDK 的版本号（起初用的 JDK 11），网上找了很多教程，最终也没有解决我的问题（网上提到比较多的原因是 `Power Save Model` 省电模式 ）。 
![在这里插入图片描述](https://img-note.langyastudio.com/202111091459915.png?x-oss-process=style/watermark)



最后 `stackoverflow` 上找到了解决方案
[https://stackoverflow.com/questions/37282285/intellij-cannot-find-any-declarations](https://stackoverflow.com/questions/37282285/intellij-cannot-find-any-declarations)



**解决方法如下：**

- Right-click src folder

- Mark Directory as > Sources Root
  即将 `src` 文件夹（也可以是你自定义的其他的源文件夹）更改为 `源文件夹根目录` 即可。
  ![在这里插入图片描述](https://img-note.langyastudio.com/202111091459776.png?x-oss-process=style/watermark)

### 依赖库

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```



### devtools

> 否则不能自动加载

设置 `<fork>true</fork> `

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <version>2.4.5</version>
            <configuration>
                <!-- 如果没有该配置，devtools不会生效 -->
                <fork>true</fork>
            </configuration>
        </plugin>
    </plugins>
</build>
```



### Intellij 自动编译

**自动编译**

![image-20210418100721698](https://img-note.langyastudio.com/20210418100727.png?x-oss-process=style/watermark)



**自动加载**

`ctrl+shift+alt+/`  调出 Maintenance 控制台，选择 Registry

![image-20210418101521125](https://img-note.langyastudio.com/20210418101527.png?x-oss-process=style/watermark)



勾选 `automake when app running`

![image-20210418101938024](https://img-note.langyastudio.com/20210418101939.png?x-oss-process=style/watermark)







