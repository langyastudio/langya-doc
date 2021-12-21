> 先看 [spring boot 2.x properties 配置文件详解](https://mp.weixin.qq.com/s?__biz=Mzg5MjYzNTA4Ng==&mid=2247484692&idx=1&sn=d4133a364c5121e51844258499be7b10&chksm=c03a57b5f74ddea350b4fcc5f0c52cdc1211cff7941b6a1634a45c2a6b099804a5c81689fa02&scene=21#wechat_redirect)
>
> 基于上述代码修改



## devtools 开发者工具

在开发阶段，我们经常要修改代码，就需要不断的重启 Spring Boot 应用，这样很麻烦。

Spring Boot 提供了一个开发者工具，可以**监控 classpath 路径上的文件**。只要源码或配置文件发生修改，Spring Boot 应用可以**自动重启**。在开发阶段，这个功能比较有用。

要使用这一开发者功能，只需添加如下依赖到 `pom.xml` 文件中：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

此时直接启动应用程序，然后修改源码并保存，可以从控制台的日志输出中看到自动编译的结果，Spring Boot 会自动重新加载。

> 默认配置下，针对 `/static`、`/public` 和 `/templates` 目录中的文件修改不会自动重启
>
> 因为禁用缓存后，这些静态文件的修改是实时更新的



## package 打包

> 这里使用 Maven 构建工具，也可以使用 Gradle 插件

spring Boot 提供  `spring-boot-maven-plugin` 插件用于**打包所有依赖到单一 `jar` 文件**（spring boot 内置 tomcat 服务器，所以打包后形成的一个 `jar` 文件可以直接基于 jvm 运行于任何平台）



### **pom 配置**

在 `pom.xml` 文件中加入以下配置：

```xml
<project ...>
    ...
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                
                <!--排除lombok插件的编译-->
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
	</build>
</project>
```



### **maven 构建**

无需任何配置，Spring Boot 的这款插件会自动定位应用程序的入口文件的 `main`，执行以下 Maven 命令即可打包：

```shell
#cd 到项目的根目录
mvnw clean package  -Dmaven.test.skip=true
#或者
mvn clean package  -Dmaven.test.skip=true
```

其中 `-Dmaven.test.skip=true` 表示跳过测试文件。



如下图所以可以看到打包后的文件默认生成到了 `target` 目录，名为 `spring-boot-0.0.1-SNAPSHOT.jar` ，即默认由`name - version` 构成。

![image-20210811120336546](https://img-note.langyastudio.com/20210811120336.png?x-oss-process=style/watermark)



### **运行jar文件**

`--server.port` 表示指定端口号

```shell
java -jar spring-boot-0.0.1-SNAPSHOT.jar --server.port=8932
```



运行时还可以指定 profile 运行环境的配置文件，如使用 `application-dev.yml` 配置文件

```shell
java -jar spring-boot-0.0.1-SNAPSHOT.jar --spring.profiles.active=dev
```



还可以设置 jvm 参数:

```shell
java -Xms10m -Xmx80m -jar spring-boot-0.0.1-SNAPSHOT.jar 
```



如果不想管理窗口后消失，可以后台运行（Linux）：

```shell
nohup java -jar spring-boot-0.0.1-SNAPSHOT.jar &
```



### **查看 java 运行环境**

可以使用 Java 自带的 `jinfo` 命令：

```shell
#pid 即进程号
jinfo -flags pid
```

- Linux 可以使用 ps 命令
- IDEA 的控制台日志也会显示调试模式下服务的 pid 进程号
- windows 系统可以通过 **任务管理器 -> 详细信息** 看到进程的 pid 值

![image-20210811130130576](https://img-note.langyastudio.com/20210811130132.png?x-oss-process=style/watermark)



例如：

```shell
VM Flags:
-XX:CICompilerCount=4 -XX:ConcGCThreads=2 -XX:G1ConcRefinementThreads=8 -XX:G1HeapRegionSize=1048576 -XX:GCDrainStackTargetSize=64 -XX:InitialHeapSize=266338304 -XX:MarkStackSize=4194304 -XX:MaxHeapSize=805306368 -XX:MaxNewSize=4823
44960 -XX:MinHeapDeltaBytes=1048576 -XX:NonNMethodCodeHeapSize=5836300 -XX:NonProfiledCodeHeapSize=122910970 -XX:ProfiledCodeHeapSize=122910970 -XX:ReservedCodeCacheSize=251658240 -XX:+SegmentedCodeCache -XX:+UseCompressedClassPoint
ers -XX:+UseCompressedOops -XX:+UseFastUnorderedTimeStamps -XX:+UseG1GC -XX:-UseLargePagesIndividualAllocation
```

- `-XX:CICompilerCount ` 

  最大的并行编译数

- `-XX:InitialHeapSize` 和 `-XX:MaxHeapSize` 

  指定 JVM 的初始和最大堆内存大小

- `-XX:MaxNewSize` 

   JVM 堆区域新生代内存的最大可分配大小

- …

- `-XX:+UseG1GC` 

  垃圾回收使用 G1 收集器
