## Jenkins 简介

Jenkins 是开源 CI&CD 软件领导者，提供超过 1000 个插件来支持构建、部署、自动化，满足任何项目的需要。我们可以用 Jenkins 来构建和部署我们的项目，比如说从我们的代码仓库获取代码，然后将我们的代码打包成可执行的文件，之后通过远程的 ssh 工具执行脚本来运行我们的项目。



## Jenkins 的安装及配置

### Docker 环境下的安装

- 下载 Jenkins 的 Docker 镜像：

```bash
docker pull jenkins/jenkins:lts
```

- 在 Docker 容器中运行 Jenkins：

```bash
docker run -p 8080:8080 -p 50000:5000 --name jenkins \
-u root \
-v /mydata/jenkins_home:/var/jenkins_home \
-d jenkins/jenkins:lts
```



### Jenkins 的配置

- 运行成功后访问该地址登录 Jenkins，第一次登录需要输入管理员密码：http://192.168.6.132:8080/

![img](https://img-note.langyastudio.com/202303081021229.png?x-oss-process=style/watermark)

- 使用管理员密码进行登录，可以使用以下命令从容器启动日志中获取管理密码：

```bash
docker logs jenkins
```

- 从日志中获取管理员密码：

![img](https://img-note.langyastudio.com/202303081021632.png?x-oss-process=style/watermark)

- 选择安装插件方式，这里我们直接安装推荐的插件：

![img](https://img-note.langyastudio.com/202303081021246.png?x-oss-process=style/watermark)

- 进入插件安装界面，联网等待插件安装：

![img](https://img-note.langyastudio.com/202303081023798.png?x-oss-process=style/watermark)

- 安装完成后，创建管理员账号：

![img](https://img-note.langyastudio.com/202303081021577.png?x-oss-process=style/watermark)

- 进行实例配置，配置 Jenkins 的 URL：

![img](https://img-note.langyastudio.com/202303081021597.png?x-oss-process=style/watermark)

- 点击系统管理->插件管理，进行一些自定义的插件安装：

![img](https://img-note.langyastudio.com/202303081021331.png?x-oss-process=style/watermark)

- 确保以下插件被正确安装：
  - 根据角色管理权限的插件：Role-based Authorization Strategy
  - 远程使用 ssh 的插件：SSH plugin
- 通过系统管理->全局工具配置来进行全局工具的配置，比如 maven 的配置：

![img](https://img-note.langyastudio.com/202303081021073.png?x-oss-process=style/watermark)

- 新增 maven 的安装配置：

![img](https://img-note.langyastudio.com/202303081023059.png?x-oss-process=style/watermark)

- 在系统管理->系统配置中添加全局 ssh 的配置，这样 Jenkins 使用 ssh 就可以执行远程的 linux 脚本了：

![img](https://img-note.langyastudio.com/202303081021430.png?x-oss-process=style/watermark)



### 角色权限管理

> 我们可以使用 Jenkins 的角色管理插件来管理 Jenkins 的用户，比如我们可以给管理员赋予所有权限，运维人员赋予执行任务的相关权限，其他人员只赋予查看权限。

- 在系统管理->全局安全配置中启用基于角色的权限管理：

![img](https://img-note.langyastudio.com/202303081023312.png?x-oss-process=style/watermark)

- 进入系统管理->Manage and Assign Roles 界面：

![img](https://img-note.langyastudio.com/202303081021971.png?x-oss-process=style/watermark)

- 添加角色与权限的关系：

![img](https://img-note.langyastudio.com/202303081021278.png?x-oss-process=style/watermark)

- 给用户分配角色：

![img](https://img-note.langyastudio.com/202303081021899.png?x-oss-process=style/watermark)



## 打包部署 SpringBoot 应用

> 这里我们使用 `mall-learning` 项目中的 `mall-tiny-jenkins` 模块代码来演示下如何使 Jenkins 一键打包部署SpringBoot 应用。

### 将代码上传到 Git 仓库

- 首先我们需要安装 Gitlab（当然你也可以使用Github或者Gitee），然后将 `mall-tiny-jenkins` 中的代码上传到Gitlab 中去
- `mall-tiny-jenkins` 项目源码地址：https://github.com/macrozheng/mall-learning/tree/master/mall-tiny-jenkins
- 上传完成后 Gitlab 中的展示效果如下：

![img](https://img-note.langyastudio.com/202303081021051.png?x-oss-process=style/watermark)

- 有一点需要`注意`，要将 pom.xml 中的 dockerHost 地址改成你自己的 Docker 镜像仓库地址：

![img](https://img-note.langyastudio.com/202303081022340.png?x-oss-process=style/watermark)

### 执行脚本准备

- 将 `mall-tiny-jenkins.sh` 脚本文件上传到 `/mydata/sh` 目录下，脚本内容如下：

```bash
#!/usr/bin/env bash
app_name='mall-tiny-jenkins'
docker stop ${app_name}
echo '----stop container----'
docker rm ${app_name}
echo '----rm container----'
docker run -p 8088:8088 --name ${app_name} \
--link mysql:db \
-v /etc/localtime:/etc/localtime \
-v /mydata/app/${app_name}/logs:/var/logs \
-d mall-tiny/${app_name}:1.0-SNAPSHOT
echo '----start container----'
```

- 给 .sh 脚本添加可执行权限：

```bash
chmod +x ./mall-tiny-jenkins.sh  
```

- windows 下的 .sh 脚本上传到 linux 上使用，需要修改文件格式，否则会因为有特殊格式存在而无法执行：

```bash
#使用vim编辑器来修改
vi mall-tiny-jenkins.sh
# 查看文件格式，windows上传上来的默认为dos
:set ff 
#修改文件格式为unix
:set ff=unix 
#保存并退出
:wq
```

- 执行 .sh 脚本，测试使用，可以不执行：

```bash
./mall-tiny-jenkins.sh
```

### 在 Jenkins 中创建执行任务

- 首先我们需要新建一个任务：

![img](https://img-note.langyastudio.com/202303081022714.png?x-oss-process=style/watermark)

- 设置任务名称后选择构建一个自由风格的软件项目：

![img](https://img-note.langyastudio.com/202303081022550.png?x-oss-process=style/watermark)

- 然后在源码管理中添加我们的 git 仓库地址：http://192.168.6.132:1080/macrozheng/mall-tiny-jenkins

![img](https://img-note.langyastudio.com/202303081023936.png?x-oss-process=style/watermark)

- 此时需要添加一个凭据，也就是我们 git 仓库的账号密码：

![img](https://img-note.langyastudio.com/202303081022666.png?x-oss-process=style/watermark)

- 填写完成后选择该凭据，就可以正常连接 git 仓库了；

![img](https://img-note.langyastudio.com/202303081022981.png?x-oss-process=style/watermark)

- 之后我们需要添加一个构建，选择调用顶层maven目标，该构建主要用于把我们的源码打包成 Docker 镜像并上传到我们的 Docker 镜像仓库去：

![img](https://img-note.langyastudio.com/202303081022903.png?x-oss-process=style/watermark)

- 选择我们的 maven 版本，然后设置 maven 命令和指定 pom 文件位置：

![img](https://img-note.langyastudio.com/202303081022435.png?x-oss-process=style/watermark)

- 之后添加一个执行远程 shell 脚本的构建，用于在我们的镜像打包完成后执行启动 Docker 容器的 .sh 脚本：

![img](https://img-note.langyastudio.com/202303081022152.png?x-oss-process=style/watermark)

- 需要设置执行的 shell 命令如下：/mydata/sh/mall-tiny-jenkins.sh

![img](https://img-note.langyastudio.com/202303081022460.png?x-oss-process=style/watermark)

- 之后点击保存操作，我们的任务就创建完成了，在任务列表中我们可以点击运行来执行该任务；

![img](https://img-note.langyastudio.com/202303081022372.png?x-oss-process=style/watermark)

- 我们可以通过控制台输出来查看整个任务的执行过程：

![img](https://img-note.langyastudio.com/202303081022154.png?x-oss-process=style/watermark)

- 运行成功后，访问该地址即可查看API文档：http://192.168.6.132:8088/swagger-ui.html



## 多模块项目如何构建

假设有一个多模块项目，父工程 `P` 中含有三个子模块 `A`、`B`、`C` 三个模块有如下的依赖关系：

- `A` 依赖 `B`、`C`

- `B` 依赖 `C`

![图片](https://img-note.langyastudio.com/202303081541259.png?x-oss-process=style/watermark)

打包 A 只需要如下一条命令即可打包完成：

```bash
mvn clean package -pl A -am -P test -DskipTests=true
```



### 必知的几个参数

| 参数  | 说明                                                         |
| :---- | :----------------------------------------------------------- |
| `-pl` | 可选，指定需要处理的工程，多个使用英文逗号分隔，取值是 `artifactId` |
| -am   | 可选，同时处理 pl 参数 指定模块的**依赖模块**                |
| -amd  | 可选，同时处理**依赖于** pl 参数 指定模块的模块              |
| -N    | 可选，表示不递归子模块                                       |

- `mvn clean install -pl A -am`

对父工程 `P`、子模块 `A` 以及 `A` 模块依赖的 `B`、`C ` 模块执行 `mvn clean install` 操作。

这个命令执行成功后，可以看到 `P`、`A`、`B`、`C` 四个模块全部安装到本地了。

- `mvn clean install -pl C -am`

对父工程 `P`、子模块 `C` 模块执行 `mvn clean install` 操作。

这个命令执行成功后，可以看到 `P`、`C` 两个模块安装到本地。

> 由于`C`模块**「不依赖」**其他的两个子模块，因此`A`、`B`模块不会执行相关命令。

- `mvn clean install -pl C -amd`

对父工程 `P`、子模块 `C` 以及依赖于 `C` 模块的 `B、C`  模块执行 `mvn clean install` 操作。

这个命令执行成功后，可以看到 `P`、`A`、`B`、`C` 四个模块全部安装到本地了。

- `mvn clean install -N`

只会打包父工程 `P`，它的子模块将不会执行相关操作



## 参考文档

[三十二张图告诉你，Jenkins构建SpringBoot有多简单~](https://mp.weixin.qq.com/s?__biz=MzU3MDAzNDg1MA==&mid=2247485903&idx=1&sn=386ec81aa35a5e12da256078456c3221&chksm=fcf4d602cb835f14315e5e334b3cc9bdc8a18898b2beeee707274f986ade245ff9496c385fc1&token=712899733&lang=zh_CN&scene=21#wechat_redirect)

[一次打包引发的思考，原来maven还可以这么玩](https://mp.weixin.qq.com/s?__biz=MzU3MDAzNDg1MA==&mid=2247485752&idx=1&sn=615f97bd9d161a87f309261c665397b4&scene=21#wechat_redirect)

[使用Jenkins一键打包部署SpringBoot应用，就是这么6！](https://www.macrozheng.com/mall/reference/jenkins.html)

[使用Jenkins一键打包部署前端应用，就是这么6！](https://www.macrozheng.com/mall/reference/jenkins_vue.html)
