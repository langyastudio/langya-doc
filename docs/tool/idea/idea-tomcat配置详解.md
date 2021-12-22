> 前期准备 
> IntelliJ IDEA、JDK、Tomcat 先自行安装，安装步骤略。



## 创建并配置项目 

### 创建项目 
选择菜单 File - New - Project，在新建项目的界面中，按照如下步骤设置并点击 Next
![在这里插入图片描述](https://img-note.langyastudio.com/20210706145809.png?x-oss-process=style/watermark)



###  设置项目名称

![在这里插入图片描述](https://img-note.langyastudio.com/20210706145812.png?x-oss-process=style/watermark)



### 开始配置项目 

选中项目目录后，右键选择并打开【Open Module Settings】菜单 或 F4，打开 Project Structure 对话框
> 一个项目中可以有多个子项目，每个子项目相当于一个模块。一般我们项目只是单独的一个，IntelliJ IDEA 默认也是单子项目的形式，所以只需要配置一个模块。



#### 配置 Source 

在 项目-web-WEB INF 下新建两个文件夹： classes 和 lib 
![在这里插入图片描述](https://img-note.langyastudio.com/20210706145816.png?x-oss-process=style/watermark)

- Sources：显示项目的目录资源，哪些是项目部署的时候需要的目录，不同颜色代表不同的类型；

- Paths：可以指定项目的编译输出目录，即项目类和测试类的编译输出地址（替换掉了Project的默认输出地址）

- Dependencies：项目的依赖



#### 配置Paths

将两个 output path 修改为刚才创建的 classes 的完整地址 
![在这里插入图片描述](https://img-note.langyastudio.com/20210706145819.png?x-oss-process=style/watermark)



#### 配置 Denpendencies 

右面有个小加号（+），点击 + 号，弹出的菜单中选择 JARs or directories… 
![在这里插入图片描述](https://img-note.langyastudio.com/20210706145821.png?x-oss-process=style/watermark)

选刚才创建的lib地址 
![在这里插入图片描述](https://img-note.langyastudio.com/20210706145827.png?x-oss-process=style/watermark)



选 Jar Directory 
![在这里插入图片描述](https://img-note.langyastudio.com/20210706145830.png?x-oss-process=style/watermark)

到这里，项目就配置完成

其中，`Project`：
- Project name：定义项目的名称

- Project SDK：设置该项目使用的JDK，也可以在此处新添加其他版本的JDK

- Project language level：这个和JDK的类似，区别在于，假如你设置了JDK1.8，却只用到1.6的特性，那么这里可以设置语言等级为1.6，这个是限定项目编译检查时最低要求的JDK特性

- Project compiler output：项目中的默认编译输出总目录
![在这里插入图片描述](https://img-note.langyastudio.com/20210706145834.png?x-oss-process=style/watermark)

`Libraries`
这里可以显示所添加的 jar 包，同时也可以添加 jar 包，并且可以把多个 jar 放在一个组里面，类似于 jar 包整理

`Facets`
When you select a framework (a facet) in the element selector pane, the settings for the framework are shown in the right-hand part of the dialog.
当你在左边选择面板点击某个技术框架，右边将会显示这个框架的一些设置



## 配置Tomcat 

选择菜单栏【run】-【Edit Configurations】或 右上角有个 Add Configurations，打开调试配置对话框
![在这里插入图片描述](https://img-note.langyastudio.com/20210706145838.png?x-oss-process=style/watermark)



### 新建 Tomcat Server 

点击左上角的 + 号，选择 Tomcat Server - Local Server （这里使用的是本地服务器），打开如下配置对话框
![在这里插入图片描述](https://img-note.langyastudio.com/20210706145840.png?x-oss-process=style/watermark)

> IntelliJ 社区版(也就是免费版) 没有Tomcat Server这个选项，收费版有 



### 设置 server 名称

配置 server、jre 等信息，其中 server 中的 libraries 可以根据实际需要添加相应的 jar 包
![在这里插入图片描述](https://img-note.langyastudio.com/20210706145843.png?x-oss-process=style/watermark)

在第二个选项卡 Deployment 中，右边有个绿色 +，点击 + 号，添加一个 Artifact 
并设置 Application Context （注意，最新版的该设置项在最下面）
![在这里插入图片描述](https://img-note.langyastudio.com/20210706145845.png?x-oss-process=style/watermark)
![在这里插入图片描述](https://img-note.langyastudio.com/20210706145848.png?x-oss-process=style/watermark)

配好之后 面板会有些变化 证明 tomcat 已经配好了 



`Artifacts`
项目的打包部署设置，这个是项目配置里面比较关键的地方。

> An artifact is an assembly of your project assets that you put together to test, deploy or distribute your software solution or its part. Examples are a collection of compiled Java classes or a Java application packaged in a Java archive, a Web application as a directory structure or a Web application archive, etc.

artifact 是一种用于装载项目资产以便于测试，部署，或者分布式软件的解决方案。例如集中编译 class，存档 java 应用包，web 程序作为目录结构，或者 web 程序存档等。



artifact 可以作为存档文件，或者作为包含以下结构元素的目录结构。

- 一个或多个编译模块

- 模块依赖的类库

- Resources 集合

- 其他 artifacts

- 独立的文件目录或存档



即编译后的 Java 类，Web 资源等的整合，用以测试、部署等工作。再白话一点，就是说某个 module 要如何打包，例如 war exploded、war、jar、ear 等等这种打包形式。某个 module 有了 Artifacts 就可以部署到应用服务器中了。

- `jar`：Java ARchive，通常用于聚合大量的 Java 类文件、相关的元数据和资源（文本、图片等）文件到一个文件，以便分发 Java 平台应用软件或库

- `war`：Web application ARchive，一种 JAR 文件，其中包含用来分发的 JSP、Java Servlet、Java 类、XML 文件、标签库、静态网页（HTML 和相关文件），以及构成 Web 应用程序的其他资源

- `exploded`：在这里你可以理解为展开，不压缩的意思。也就是 war、jar 等没压缩前的目录结构。建议在开发的时候使用这种模式，便于修改了文件的效果立刻显现出来。

> 默认情况下，IDEA 的 Modules 和 Artifacts 的 output 目录已经设置好了，不需要更改，打成 war 包的时候会自动在 WEB-INF 目录下生成 classes，然后把编译后的文件放进去。



### 热部署

如果想实现修改 web 项目的内容，tomcat 自动重新加载文件，从而避免手动重启服务，需要做如下的简单配置：
update 中选择 update classed and resources
![在这里插入图片描述](https://img-note.langyastudio.com/20210706145852.png?x-oss-process=style/watermark)

> 一定要选择名字带有 exploded 后缀的 war 包部署，不然修改文件重新部署是不生效的。



## 运行 

在index.jsp中写点字 以便测试 
![在这里插入图片描述](https://img-note.langyastudio.com/20210706145856.png?x-oss-process=style/watermark)

点击左上角的绿色运行按钮就可以执行 web 程序了
![在这里插入图片描述](https://img-note.langyastudio.com/20210706145910.png?x-oss-process=style/watermark)

正常情况下，会自动打开浏览器的特定 URL 地址，当然也可以从浏览器中输入项目的启动地址查看了 
http://192.168.123.100:8080/java_web/
![在这里插入图片描述](https://img-note.langyastudio.com/20210706145859.png?x-oss-process=style/watermark)



当你点击运行 tomcat 时，默认就开始做以下事情：

- 编译，IDEA 在保存/自动保存后不会做编译，不像 Eclipse 的保存即编译，因此在运行 server 前会做一次编译。编译后 class 文件存放在指定的项目编译输出目录下

- 根据 artifact 中的设定对目录结构进行创建

- 拷贝 web 资源的根目录下的所有文件到artifact的目录下

- 拷贝编译输出目录下的 classes 目录到 artifact 下的WEB-INF下

- 拷贝 lib 目录下所需的 jar 包到 artifact 下的WEB_INF下

- 运行 server，运行成功后，如有需要，会自动打开浏览器访问指定 URL

