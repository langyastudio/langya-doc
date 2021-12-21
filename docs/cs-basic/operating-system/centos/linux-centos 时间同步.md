Centos 8 已经无法安装 ntpdate ，可以使用chrony模块。



**修改配置加入网络时间：**

```shell
vim /etc/chrony.conf
```

![image-20200915171021249](https://img-note.langyastudio.com/20200915171021.png?x-oss-process=style/watermark)

**增加多个server服务器：**

```shell
server 210.72.145.44 iburst
server ntp.aliyun.com iburst
```



**执行时间同步：**

```shell
systemctl restart chronyd.service
chronyc sources -v
```



