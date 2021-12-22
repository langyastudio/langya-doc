> docker 参考文档
> [https://yeasy.gitbooks.io/docker_practice/content/](https://yeasy.gitbooks.io/docker_practice/content/)

![image-20201216112250863](https://img-note.langyastudio.com/20201216112251.png?x-oss-process=style/watermark)



## docker 安装

docker 安装较为简单，执行下面的命令即可：
```bash
yum install -y yum-utils
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y containerd.io docker-ce docker-ce-cli 
```



安装特定版本：

```bash
#列举可用版本
yum list docker-ce --showduplicates | sort -r
#安装特定版本
yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
```

> `/var/lib/docker/` 包含镜像、容器、存储、网络等， Docker Engine package 为 `docker-ce`



旧版本的 Docker 为 `docker` or `docker-engine`，需要卸载：

```bash
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```



CentOS 8 默认采用podman替代docker，所以可能会报告错误如下：

```bash
problem with installed package podman-1.6.4-4.module_el8.1.0+298+41f9343a.x86_64
  - package podman-1.6.4-4.module_el8.1.0+298+41f9343a.x86_64 requires runc >= 1.0.0-57, but none of the providers can be installed
```

解决方法：

```bash
#方案1
yum erase podman buildah

#方案2
#https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/edge/Packages/
yum install -y https://mirrors.aliyun.com/docker-ce/linux/centos/8/x86_64/edge/Packages/containerd.io-1.3.7-3.1.el8.x86_64.rpm
```



## 离线安装

当因为涉密等原因，主机无法访问外网时（标记为 CentosB），该如何离线安装 docker 呢？

解决方法很简单：
- 安装一台版本/环境一致的 centos 系统（建议最小化安装，标记为 CentosA）
- 基于 CentosA，将 docker 需要安装的依赖包离线下载下来
- 安装包拷贝到 CentosB，执行离线安装

CentosA
```bash
# 将基础离线包下载到 /home/rpm/base  目录
yum install --downloadonly --downloaddir=/home/rpm/base yum-utils

#安装基础依赖包
cd /home/rpm/base
rpm -Uvh *.rpm

# 将 docker 离线包下载到 /home/rpm/docker 目录
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install --downloadonly --downloaddir=/home/rpm/docker docker-ce docker-ce-cli containerd.io
#cd /home/rpm/docker
#rpm -Uvh *.rpm
```

此时将 ` /home/rpm/base  ` 、 `/home/rpm/docker` 目录中的离线依赖包，拷贝到无法访问外网的主机 CentosB，执行 `rpm -Uvh *.rpm` 即可完成 docker 离线安装的目的

> 单纯的将 docker 的 rpm下载到目标机器执行安装，往往以失败告终，因为 `依赖包` 。



## 开机启动

```bash
systemctl start docker
systemctl enable docker
```



## docker 卸载

1. Uninstall the Docker Engine, CLI, and Containerd packages:

    ```bash
    $ sudo yum remove docker-ce docker-ce-cli containerd.io
    ```

2. Images, containers, volumes, or customized configuration files on your host are not automatically removed. To delete all images, containers, and volumes:

    ```bash
    $ sudo rm -rf /var/lib/docker
    ```

You must delete any edited configuration files manually.