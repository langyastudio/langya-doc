前面和大伙聊了 Spring Boot 项目的三种创建方式，这三种创建方式，无论是哪一种，创建成功后，pom.xml 坐标文件中都有如下一段引用：

```xml
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>2.1.4.RELEASE</version>
	<relativePath/> <!-- lookup parent from repository -->
</parent>
```

对于这个 parent 的作用，你是否完全理解？有小伙伴说，不就是依赖的版本号定义在 parent 里边吗？是的，没错，但是 parent 的作用可不仅仅这么简单哦！本文松哥就来和大伙聊一聊这个 parent 到底有什么作用。



# 基本功能

当我们创建一个 Spring Boot 工程时，可以继承自一个 `spring-boot-starter-parent` ，也可以不继承自它，我们先来看第一种情况。先来看 parent 的基本功能有哪些？

- 定义了 Java 编译版本为 1.8 

- 使用 UTF-8 格式编码

- 继承自 `spring-boot-dependencies`，这个里边定义了依赖的版本，也正是因为继承了这个依赖，所以我们在写依赖时才不需要写版本号

- 执行打包操作的配置

- 自动化的资源过滤

- 自动化的插件配置

- 针对 application.properties 和 application.yml 的资源过滤，包括通过 profile 定义的不同环境的配置文件，例如 application-dev.properties 和 application-dev.yml

> 请注意，由于 application.properties 和 application.yml 文件接受 Spring 样式占位符 `$ {...}` ，因此 Maven 过滤更改为使用 `@ .. @` 占位符，当然开发者可以通过设置名为 resource.delimiter 的 Maven 属性来覆盖 `@ .. @` 占位符