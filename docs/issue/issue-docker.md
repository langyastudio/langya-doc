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

