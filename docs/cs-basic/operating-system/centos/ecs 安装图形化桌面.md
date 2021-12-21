阿里云官网默认的 Linux Centos7 系统镜像，都是没有安装桌面环境的，用户如果要使用桌面，需要自己在服务器上进行安装。



本教程以 MATE 桌面安装为例 



### 登录服务器，执行命令安装桌面环境

先安装 MATE Desktop

```shell
yum groups install "MATE Desktop"
```



再安装 X Window System

```shell
yum groups install "X Window System"
```



设置服务器默认启动桌面

```shell
systemctl  set-default  graphical.target
```



重启服务器

```shell
reboot
```

> 如果启动失败，尝试执行下列命令行：
> yum update



### 在ECS控制台，用管理终端登录服务器，查看安装好的桌面

输入管理终端密码后，进入到服务器系统登录界面，用root密码登录服务器

![img](https://img-note.langyastudio.com/20210707170109.png?x-oss-process=style/watermark)



> centos 8 可以参考：https://www.itzgeek.com/how-tos/linux/centos-how-tos/how-to-install-gnome-gui-on-rhel-8.html

```shell
dnf groupinstall "Server with GUI" -y
systemctl set-default graphical
reboot
```

