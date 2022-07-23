## 启动

### docker 启动 redis 出现闪退

**原因**

`daemonize` 设置为了 yes

**方案**

设置为no

> 容器是：环境+软件，就是 centos 的环境加上 redis 软件，本来这个容器的环境启动时就一个 redis 进程在运行着，结果你把 redis 也设置为后台启动了，那 centos 环境没有一个正常的进程，都是守护进程，可不就关机了，也就意味着容器结束运行，所以你的容器启动就闪退。



### docker-credential-desktop

![image-20210625101440009](https://img-note.langyastudio.com/20210625101440.png?x-oss-process=style/watermark)



#### Cannot run program "docker-credential-desktop"

After failed upgrade and re-install, Cannot run program "docker-credential-desktop"

Could not push image: java.io.IOException: Cannot run program "docker-credential-desktop": CreateProcess error=2, 系统找不到指定的文件。

**解决方案：**

Removing `"credsStore":"desktop"` from `config.json` caused the error logging to stop。如 `C:\Users\hk\.docker\config.json`



## 发布

### 允许使用无权限验证的 Docker Registry

![image-20220713170949817](https://img-note.langyastudio.com/202207131709858.png?x-oss-process=style/watermark)

由于镜像仓库使用了 **无密码访问 **的地址，所以配置 **insecure-registries**，docker 才能使用无权限验证的 docker registry。

默认情况下，Docker 不能使用没有配置权限验证的 Docker Registry，会出现如下报错：

```bash
Error response from daemon: Get https://192.168.249.8:5000/v2/: http: server gave HTTP response to HTTPS client
```

这时需要修改 docker 配置文件，将 ”192.168.249.8:5000” 添加到 insecure-registries 中：

```bash
vim /etc/docker/daemon.json {
  "insecure-registries" : ["192.168.249.8:5000"]
}
```

然后重启 docker

```bash
sudo systemctl restart docker 
sudo restart docker 
```
