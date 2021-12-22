对于复杂的应用，不可避免需要多个服务部署在多个容器中，并且服务间存在网络互联通信的情况。比如服务 A 需要连接另一个 mysql 的容器。

## 新建网络
先创建一个新的 Docker 网络

```bash
docker network create -d bridge --subnet 172.27.0.0/16 cloud-net
```

`-d ` 参数指定 Docker 网络类型，有 bridge overlay。其中 overlay 网络类型用于 Swarm mode。

表示新建了 `172.27.0.0/255.255.0.0` 的网络，名称为 cloud-net。新建网络是为了实现同主机的多个容器间网络互通，走内网，类似于软件交换机。

可以通过 `network ls` 命令查看当前宿主机中所有的 docker 网络

```bash
[root@192 ~] docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
02de4daaa2c4        bridge              bridge              local
4764e0697c35        host                host                local
60da7fd0b38e        none                null                local
```

> 建议将容器加入自定义的 Docker 网络来连接多个容器，而不是使用 --link 参数



## 连接容器

运行一个容器并连接到新建的 cloud-net 网络

```bash
docker run -it --rm --network=cloud-net --ip=172.27.0.20 --name cloud1 cloud-image
```
--network=cloud-net --ip=172.27.0.20 为了固定当前容器的 IP 地址
`cloud-image` 表示当前使用的镜像（替换为实际镜像名称）

打开新的终端，再运行一个容器并加入到 cloud-net 网络
```bash
docker run -it --rm --network=cloud-net --ip=172.27.0.30 --name cloud2 cloud-image
```

在 cloud1 容器输入以下命令来证明 cloud1 容器和 cloud2 容器建立了互联关系
```bash
ping cloud2 
#or ping 172.27.0.30
```
这样在 cloud1 中就可以通过 ip 172.27.0.30 访问 cloud2 容器了



**docker 容器网络示意图如下：**
![在这里插入图片描述](https://img-note.langyastudio.com/20210106082507.png?x-oss-process=style/watermark)