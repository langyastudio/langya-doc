>  CentOS 7.6版本
>
>  基于阿里云新加坡等非内地的服务器

### 安装 Shadowsocks 服务端

#### 连接服务器
ssh 连接服务器

#### 更新包
```shell
yum update
```
#### 安装 pie
```shell
yum install python-setuptools && easy_install pip
```
#### 安装 Shadowsocks
```shell
pip install shadowsocks
```



### 配置 Shadowsocks

此文件默认不存在，需要创建：
```shell
vim /etc/shadowsocks.json
```

编辑文件为以下内容：

 ```json
{

"server":"0.0.0.0",

"server_port":8631,

"password":"yourpassword",

"timeout":300,

"method":"aes-256-cfb",

"fast_open":false,

"workers": 1

}
 ```
以下是内容的说明：

```json
{

"server":"服务器 IP地址 (IPv4/IPv6)",

"server_port":"服务器监听的端口，注意不要设为使用中的端口",

"password":"设置连接ss的密码",

"timeout":"超时的时间",

"method":"加密的方式 可选择 “aes-256-cfb”, “rc4-md5”等等",

"fast_open":"true 或 false。如果你的服务器 Linux 内核在3.7+，可以开启 fast_open 以降低延迟",

"workers": "workers数量，默认为 1"

}
```
需要配置多个用户分享给朋友用：

 ```json
{

"server":"x.x.x.x",

"port_password":{

"8381":"pass1",

"8382":"pass2",

"8383":"pass3",

"8384":"pass4"

},

"timeout":300,

"method":"aes-256-cfb",

"fast_open":false,

"workers":1

}
 ```



#### Shadowsocks 启动/重启/停止

启动：
```shell
ssserver -c /etc/shadowsocks.json -d start
```

配置文件修改后重启：
```shell
ssserver -c /etc/shadowsocks.json -d restart
```

停止：
```shell
ssserver -c /etc/shadowsocks.json -d stop
```



#### 开机自启 SS

```shell
vim /etc/rc.d/rc.local
```
加入下面内容
```shell
/usr/bin/ssserver -c /etc/shadowsocks.json -d start
```

保存并推出 vim，CentOS 7 正打算抛弃 /etc/rc.d/rc.local，重启前需要运行以下命令获得权限，否则 rc.local 不会执行
```shell
chmod +x /etc/rc.d/rc.local
```



#### 阿里云控制台开放 shadowsocks 服务端口(添加安全组规则中配置)



#### 验证是否可以服务器是否配置正确

用 `telnet x.x.x.x 6899`  验证，只有 `telnet` 能访问端口了，才能正常使用 `shadowsocks！`

要测试远程主机上的某个端口是否开启，windows 下就自带了工具，那就是 `telnet`。

`ping` 命令是不能检测端口,只能检测你和相应IP是否能连通。

win7 下就没有 `telnet`，在 cmd 下输入 telnet 提示没有该命令。进入`控制面板 -> 程序 -> 打开或关闭 windows功能 -> 勾选telnet服务端` 选项。然后等一段时间，在出来的对话框把 telnet 客户端和 telnet 服务器勾选上，这样就安装好了 telnet 组件了。

测试某个端口是否开启。测的是 x.x.x.x 的 6899 这个端口。在 cmd 下输入 `telnet x.x.x.x 6899` 之后会出现一个窗口，是最小化的。若窗口自动关闭了，说明端口是关闭的或主机ping不通，反之端口开放。比如这个 2121 端口是开放的，就出现以下的窗口，否则窗口关闭。



#### 最后使用SS/SSR客户端上网

#### 屏蔽阿里云 ECS 服务器扫描
用过阿里云 ecs 服务器或者产品的人都知道，阿里云是自带安全扫描还有云盾防御的，虽然表面上是说给你安全防御，其实就是为了获取你的数据。

自己服务器天天要被阿里云扫描，有什么违规关键词都会给你封了，特别是小站，没人气就没攻击，不过阿里云会天天攻击你，告诉你网站漏洞。

那么如何完美的彻底的屏蔽阿里云的扫描呢？

**方法一：卸载阿里云盾监控**
```shell
wget http://update.aegis.aliyun.com/download/uninstall.sh
sh uninstall.sh
wget http://update.aegis.aliyun.com/download/quartz_uninstall.sh
sh quartz_uninstall.sh
```

或者：
```shell
wget http://update.aegis.aliyun.com/download/uninstall.sh
chmod +x uninstall.sh
./uninstall.sh
wget http://update.aegis.aliyun.com/download/quartz_uninstall.sh
chmod +x quartz_uninstall.sh
./quartz_uninstall.sh
```
删除残留
```shell
pkill aliyun-service
rm -fr /etc/init.d/agentwatch /usr/sbin/aliyun-service
rm -rf /usr/local/aegis*
```
屏蔽云盾 IP（直接执行下面代码，或者通过面板管理）
```shell
iptables -I INPUT -s 140.205.201.0/28 -j DROP
iptables -I INPUT -s 140.205.201.16/29 -j DROP
iptables -I INPUT -s 140.205.201.32/28 -j DROP
iptables -I INPUT -s 140.205.225.192/29 -j DROP
iptables -I INPUT -s 140.205.225.200/30 -j DROP
iptables -I INPUT -s 140.205.225.184/29 -j DROP
iptables -I INPUT -s 140.205.225.183/32 -j DROP
iptables -I INPUT -s 140.205.225.206/32 -j DROP
iptables -I INPUT -s 140.205.225.205/32 -j DROP
iptables -I INPUT -s 140.205.225.195/32 -j DROP
iptables -I INPUT -s 140.205.225.204/32 -j DROP
```

**方法二：CentOS 关闭AliYunDun**
```shell
chkconfig --list
```
查看开机启动里面这个软件的服务名是什么，然后替换掉xxx然后执行就可以了；

如果想开机不启动的话，chkconfig –del xxxx 这个 xxxx 就是你找出来 aliyundun 的后台服务。
```shell
service aegis stop  #停止服务
chkconfig --del aegis  # 删除服务
```



#### 屏蔽阿里云盾的扫描检测

虽然上面彻底关闭了阿里云 ECS 上的云盾，但是发现还是有“Web 攻击”的截获记录，看了一下 IP 地址段发现是个叫“Alibaba.Security.Heimdall”的 UA 来访的，度娘、谷姐一番后才知道这就是阿里云云盾的扫描检测 IP 地址。

云盾扫描云服务器的 IP 段固定为：
```shell
#!/bin/bash
echo "屏蔽阿里云盾恶意 IP......."
iptables -I INPUT -s 140.205.201.0/28 -j DROP
iptables -I INPUT -s 140.205.201.16/29 -j DROP
iptables -I INPUT -s 140.205.201.32/28 -j DROP
iptables -I INPUT -s 140.205.225.192/29 -j DROP
iptables -I INPUT -s 140.205.225.200/30 -j DROP
iptables -I INPUT -s 140.205.225.184/29 -j DROP
iptables -I INPUT -s 140.205.225.183/32 -j DROP
iptables -I INPUT -s 140.205.225.206/32 -j DROP
iptables -I INPUT -s 140.205.225.205/32 -j DROP
iptables -I INPUT -s 140.205.225.195/32 -j DROP
iptables -I INPUT -s 140.205.225.204/32 -j DROP
iptables -I INPUT -s 106.11.224.0/26 -j DROP
iptables -I INPUT -s 106.11.224.64/26 -j DROP
iptables -I INPUT -s 106.11.224.128/26 -j DROP
iptables -I INPUT -s 106.11.224.192/26 -j DROP
iptables -I INPUT -s 106.11.222.64/26 -j DROP
iptables -I INPUT -s 106.11.222.128/26 -j DROP
iptables -I INPUT -s 106.11.222.192/26 -j DROP
iptables -I INPUT -s 106.11.223.0/26 -j DROP
echo "已经屏蔽了阿里云盾恶意 IP"
```
既然官方公开了 IP 地址段那就也说明，屏蔽拦截这些 IP 的扫描检测官方并不反对的！



#### 如果访问不了

- 服务器的防火墙（iptables or firewall）屏蔽了端口
- 被运营商封 IP 了（如果突然那天访问不了了）
