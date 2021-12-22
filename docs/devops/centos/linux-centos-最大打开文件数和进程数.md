Linux 系统对打开文件数和进程数有限制，默认限制为1024，它是一种简单有效的实现资源限制的方式。但当单进程的并发量较大时，1024 的限制很容易超标，报告 `too many open files` 的错误。为了让系统能够支持更大的并发，就需要修改默认的限制数。

#### 查看最大打开文件数

```shell
ulimit -n
```

> 可以通过 `ulimit -a`查看更多的系统限制值



#### 修改最大文件数与进程数

终端可以通过执行 `ulimit -HSn 10240`命令的方式临时生效，这里介绍永久生效的方法

##### 修改 limits.conf

修改`/etc/security/limits.conf`文件，文件尾部增加以下配置

```shell
* soft nofile 655350 
* hard nofile 655350
* soft nproc  655350
* hard nproc  655350
* soft core   unlimited
* hard core   unlimited
```

重启服务器后，再通过`ulimit -n`查看是否生效



##### systemd 生效

如果使用`systemd`自启动服务，在高版本的CentOS等系统中，可能没有生效，此时需要进一步修改：

修改`/etc/systemd/system.conf`与`/etc/systemd/user.conf`文件，文件尾部增加以下配置：

```shell
DefaultLimitCORE=infinity
DefaultLimitNOFILE=655350
DefaultLimitNPROC=655350
```



> 执行 `systemctl daemon-reload`命令，即时生效

