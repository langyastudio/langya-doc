## Non-resolvable import POM: Could not find artifact xxx

执行 `..\mvnw dockerfile:push` 报告如下错误：

![image-20220722181208727](https://img-note.langyastudio.com/202207221812793.png?x-oss-process=style/watermark)

**问题原因：**

找不到依赖包

**解决方案：**

使用调试模式：`..\mvnw dockerfile:push -X` 可以看到具体的错误细节

例如提示查找的依赖库的路径下没有该包，一般是由于系统有多个环境的 Maven 导致。此时将 `setting.xml` 文件复制到 `mvnw` 对应的配置环境即可。



## Could not transfer artifact xxx from/to maven-default-http-blocker (http://0.0.0.0/)

maven 构建项目的时候遇到了 Could not transfer artifact xxxxxx:pom:1.1-SNAPSHOT from/to maven-default-http-blocker (http://0.0.0.0/): Blocked mirror for repositories: [nexus-118 (http://xxxxxx.com:8081/repository/maven-public/, default, releases+snapshots)] 错误

**问题原因：**

maven 高版本配置了不能直接访问 http 的库

**解决方案：**

把 blocked 配置为 FALSE 就行了

```xml
<mirror>
      <id>xxx</id>
      <mirrorOf>xxx</mirrorOf>
      <name>xxx公共仓库</name>
      <url>xxxxx</url>
      <blocked>false</blocked>
</mirror>
```

