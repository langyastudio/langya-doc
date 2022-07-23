## 镜像

实际使用中主要为了解决无法访问外网的情况下，安装部署 docker 镜像的目的。

Docker 提供了 `docker save` 和 `docker load` 命令，用以将镜像保存为一个文件，然后传输到另一个位置上，再加载进来。

### 列举镜像

列举镜像

```bash
$ docker image ls
```

查看镜像、容器、数据卷所占用的空间

```bash
$ docker system df
TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLEImages              24                  0                   1.992GB             1.992GB (100%)Containers          1                   0                   62.82MB             62.82MB (100%)Local Volumes       9                   0                   652.2MB             652.2MB (100%)Build Cache                                                 0B                  0B
```



### 导出镜像

比如保存这个 srs 镜像：

```bash
$ docker image ls srs
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
srs                latest              baa5d63471ea        5 weeks ago         4.803 MB
```

**保存镜像：**

```bash
$ docker save srs -o filename.xxx
$ file filename.xxx
filename: POSIX tar archive
```
这里的 `filename.xxx` 可以为任意名称+任意后缀名，如 srs-latest.tar ，但文件的本质都是归档文件

若使用 gzip 压缩：
```bash
$ docker save srs | gzip > srs-latest.tar.gz
```

如果同名则会覆盖（没有警告）



### 导入镜像

然后我们将 `srs-latest.tar.gz` 文件复制到了到了另一台机器上，再导入镜像：

```bash
$ docker load -i srs-latest.tar.gz
======================================>
Loaded image: srs:latest
```

> 这种方式主要为了实现离线加载镜像的需求，但并不推荐使用。镜像迁移更推荐使用仓库 `Docker Registry`。



### 删除镜像

删除本地的镜像，可以使用 docker image rm 命令，其格式为：

```bash
$ docker image rm [选项] <镜像1> [<镜像2> ...]
```

批量清理临时镜像文件

```bash
$ docker image prune
```



## 容器

### 启动/停止容器

新建并启动一个srs镜像的容器

```bash
$ docker run -it --restart=always -p 1985:1985 -v /dev:/dev2 -v /mnt/nfs:/mnt/volume1 --name srs-service srs
```

> 创建镜像时 `Dockerfile` 要通过 `EXPOSE` 指定正确的开放端口，容器启动时可以指定 `PublishAllPort = true`
>



启动/停止容器

```bash
#启动
$ docker container start xxx
#终止
$ docker container stop xxx
#查询修改
$ docker container diff xxx
```



### 进入容器

容器列表（`-a` 命令可以查看所有已经创建的包括终止状态的容器）

```bash
$ docker container ls -a
```

进入容器

```bash
$ docker exec -it xxx /bin/bash
```

> 可以通过 docker container update 更新容器，例如 docker container update --restart="no"   < container id>



### 导出和导入容器

**导出容器**

如果要导出本地某个容器快照，使用 `docker export` 命令

```bash
$ docker export xxx > container20201216.tar
```



**导入容器**

使用 `docker import` 从容器快照文件中再导入为镜像，例如

```bash
$ cat container20201216.tar | docker import - srs/srs:v1.0
```

此外，也可以通过指定 URL 或者某个目录来导入，例如

```bash
$ docker import http://example.com/exampleimage.tgz example/imagerepo
```



### 删除容器

可以使用 `docker container rm` 来删除一个处于终止状态的容器, 如

```bash
$ docker container rm  trusting_newton
```

如果数量太多要一个个删除可能会很麻烦，用下面的命令可以**清理**掉所有处于终止状态的容器。

```bash
$ docker container prune
```



### 容器的信息

PID 信息

```bash
$ docker inspect --format '{{ .State.Pid }}' <CONTAINER ID or NAME>
```

IP 地址

```bash
$ docker inspect --format '{{ .NetworkSettings.IPAddress }}' <CONTAINER ID or NAME>
```

容器配置

```bash
$ docker inspect <CONTAINER ID or NAME>
```

容器监控

```bash
$ docker stats
```

已经运行的容器，可以通过 `docker update 更新配置参数 <CONTAINER ID or NAME>`（需要先停止容器的运行）
或者通过 `/var/lib/docker/containers/容器ID` 下的 hostconfig.json 等文件，修改启动配置参数



### 容器启动参数

```bash
pip install runlike
runlike -p  <容器名>|<容器ID>
```

![image-20220311122719395](https://img-note.langyastudio.com/202203111227592.png?x-oss-process=style/watermark)



### 修改挂载目录

停止docker服务

```bash
systemctl stop docker.service
```

修改配置文件

```bash
vim /var/lib/docker/containers/container-ID/config.v2.json

#找到挂载点并修改，一般有2个地方修改
"MountPoints":{"/home":{"Source":"/docker","Destination":"/home","RW":true,"Name":"","Driver":"","Type":"bind","Propagation":"rprivate","Spec":{"Type":"bind","Source":"//docker/","Target":"/home"}}}
```

启动docker服务

```bash
systemctl start docker.service
```



### 控制容器系统资源占用

在使用 `docker create` 命令创建容器或使用 `docker run` 创建并启动容器的时候，可以使用 `-c|--cpu-shares[=0]` 参数来调整容器使用 CPU 的权重；使用 `-m|--memory[=MEMORY]` 参数来调整容器使用内存的大小。



## 其他

如使用 marathon+mesos vs k8s