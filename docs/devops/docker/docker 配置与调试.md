## docker 配置

### 存储位置

可以使用 `docker system info | grep "Root Dir"` 查看当前使用的存储位置。

与 Docker 相关的本地资源默认存放在 `/var/lib/docker/` 目录下，以 aufs 文件系统为例，其中 container 目录存放容器信息，graph 目录存放镜像信息，aufs 目录下存放具体的镜像层文件



### 配置文件

使用 systemd 的系统（如 Ubuntu 16.04、Centos 等）的配置文件在 `/etc/docker/daemon.json`



### 日志文件

默认位置： `/var/lib/docker/containers/容器id号/xx.log` 

可以通过配置文件`daemon.json`更改日志配置：

```bash
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m",
    "max-file": "10"
  }
}
```

> 需要重建容器才能生效



## 如何调试 Docker

在 docker配置文件 daemon.json中添加

```bash
{
  "debug": true
}
```

重启守护进程

```bash
#方法1
sudo systemctl daemon-reload
sudo systemctl restart docker

#方法2
$ sudo kill -SIGHUP $(pidof dockerd)
```



### 镜像调试

进入到前面一个指令中获取到的临时镜像`22d31cc52b3e` ，调用下一个指令，从而找到镜像错误的原因

![img](https://img-note.langyastudio.com/20201216130310.jpeg?x-oss-process=style/watermark)

> 通过 `docker run -it`   启动镜像的一个容器



### 容器调试

```bash
#日志
$ docker logs [容器名]

#attach实时查看stdout
$ docker attach CONTAINER 
```

> 默认会绑定stdin，代理signals， 所以如果你 `ctrl-c` 容器通常会退出。很多时候大家并不想这样，只是想分离开，可以`ctrl-p ctrl-q`。