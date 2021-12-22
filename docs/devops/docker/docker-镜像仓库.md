仓库是集中存放镜像的地方。一个容易混淆的概念是注册服务器（Registry），实际上注册服务器是管理仓库的具体服务器，每个服务器上可以有多个仓库，而每个仓库下面有多个镜像。从这方面来说，仓库可以被认为是一个具体的项目或目录。

例如对于仓库地址 dl.dockerpool.com/ubuntu 来说，dl.dockerpool.com 是注册服务器地址，ubuntu 是仓库名。

大部分时候，并不需要严格区分这两者的概念。

> 每次构建镜像时，需要花费大量的时间，有了仓库后，就不需要再次构建，直接下载镜像就行了。可以通过 `docker pull` 命令来将它下载到本地直接部署即可。



## 私有仓库

可以使用 docker 自带的 [私有仓库](https://yeasy.gitbooks.io/docker_practice/content/repository/registry.html)，也可以利用第三方阿里云等仓库，以下以**阿里云**为例



### 登录阿里云 Docker Registry

```bash
$ sudo docker login --username=hacfin registry.cn-hangzhou.aliyuncs.com
```
用于登录的用户名为阿里云账号全名，密码为开通服务时设置的密码。

> registry 密码为  xxxxxx



### 将镜像推送到Registry

```bash
$ sudo docker login --username=hacfin registry.cn-hangzhou.aliyuncs.com
$ sudo docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/d-cloud/srs:[镜像版本号]
$ sudo docker push registry.cn-hangzhou.aliyuncs.com/d-cloud/srs:[镜像版本号]
```

请根据实际镜像信息替换示例中的[ImageId]和[镜像版本号]参数。

> 镜像版本号按照 版本号-日期 的形式命名，例如 7.3.1-20190919



### 从Registry中拉取镜像

```bash
$ sudo docker pull registry.cn-hangzhou.aliyuncs.com/d-cloud/srs:[镜像版本号]
```



## 示例

使用"docker images"命令找到镜像

```bash
$ sudo docker images

REPOSITORY                                                         TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
registry.aliyuncs.com/acs/agent                                    0.7-dfb6816         37bb9c63c8b2        7 days ago          37.89 MB
```

将该镜像名称中的域名部分变更为Registry服务器
```bash
$ sudo docker tag 37bb9c63c8b2 registry.cn-hangzhou.aliyuncs.com/d-cloud/srs:0.7-20201216
```

将该镜像推送到服务器
```bash
$ sudo docker push registry.cn-hangzhou.aliyuncs.com/d-cloud/srs:0.7-20201216
```

这样下次就可以直接下载镜像，直接部署了（考虑变更幅度，目前建议 env-image/srs-image 直接下载镜像部署）



> 阿里云web平台仓库的账户密码
https://signin.aliyun.com/hacfin.onaliyun.com/login.htm
用户登录名称 docker@hacfin.onaliyun.com
登录密码 xxxxxx
目前仓库存储在杭州节点下：https://cr.console.aliyun.com/cn-hangzhou/instances/repositories

---





