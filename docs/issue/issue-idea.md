### 依赖管理

选择 File -> Project Structure 菜单，在弹框中选择 Modules 标签，再选择对应的的项目。在依赖管理中添加项目依赖，如下图所示：选择依赖的项目即可

![image-20220620172055359](https://img-note.langyastudio.com/202206201720425.png?x-oss-process=style/watermark)



### Debug模式启动慢或无法启动

压根儿就没有改过代码，但突然无法启动。应该是一个设置的小问题，于是通过 run 按钮启动项目验证一下，果然启动成功了。这也就说明项目和代码没有任何问题，肯定是 IDEA 某个设置项的问题

**解决方案：**

因为在项目中有断点打在了方法上，因此导致的 debug 变慢。解决方法也简单，将打在方法上的断点去掉即可

![image-20210428152911765](https://img-note.langyastudio.com/20210428152915.png?x-oss-process=style/watermark)

> 使用方法断点会使得正在debug调试的程序变慢



### 无法初始化主类 Main  java.lang.NoClassDefFoundError: org/apache/catalina/WebResourceRoot

在IDEA中，maven配置 `<scope>provided</scope>`，就告诉了 IDEA 程序会在运行的时候提供这个依赖，但是实际上却并没有提供这个依赖。

**解决方案：**

1. 打开 idea 的 Run/Debug Configurations:

2. 选择 Application - Main

3. 右侧 Configuration：Use classpath of module

4. 钩上 ☑︎Include dependencies with "Provided" scope

![image-20210330205542852](https://img-note.langyastudio.com/20210330205543.png?x-oss-process=style/watermark)



### 多端口启动

![img](https://img.kancloud.cn/36/52/3652e9df361497b8de51eb07e2bc244f_1372x895.png)

- 点击 edit configuration ，取消勾选 single instance only（只允许单节点运行）。在比较新的版本中这个勾选框变成了 Allow parallel run (允许多实例并发运行)，那你就给它勾选上
- 复制一份当前配置，在 environment 选项中的 vm options 中设置不同的端口号

```ini
-Dserver.port=8889 -Dserver.httpPort=89 -Dspring.profiles.active=dev -Ddebug
-Dserver.port=8888 -Dserver.httpPort=88 -Dspring.profiles.active=dev -Ddebug
```



### run / debug 灰色

问题描述：

设置了 source Root 文件夹

但 run/debug 显示灰色不可用，它右侧的两个按钮可用

![image-20210305172249966](https://img-note.langyastudio.com/20210305172250.png?x-oss-process=style/watermark)



**解决方法：**

重启IDE



### 代码提示不区分大小写

![image-20230104135732054](https://img-note.langyastudio.com/202301041357239.png?x-oss-process=style/watermark)



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







