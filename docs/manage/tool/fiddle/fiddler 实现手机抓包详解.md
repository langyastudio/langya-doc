### Fiddler 简介

Fiddler 是一款强大的抓包工具，原理是以 web 代理服务器的形式进行工作的：

![HTTP 代理](https://img-note.langyastudio.com/20201215140122.png?x-oss-process=style/watermark)



### Fiddler 配置

#### 允许监听https

Fiddler 如果抓取 https 协议会话需要进一步配置，在 Tools ->Options 菜单下，选择HTTPS标签并配置如下：

![image-20201215142708142](https://img-note.langyastudio.com/20201215142708.png?x-oss-process=style/watermark)



#### 允许远程连接

手机抓取需要配置远程连接，在 Tools ->Options 菜单下，选择Connections标签并配置如下：

监听端口 8888 并允许远程连接

![image-20201215144559283](https://img-note.langyastudio.com/20201215144559.png?x-oss-process=style/watermark)

> 防火墙需要开放 `8888` 端口



### 手机配置

> 需要电脑与手机处于同一网段（例如同一局域网）

以 iphone 为例

#### 下载证书

打开手机浏览器，输入 http://【fiddler电脑IP地址】:【fiddler设置的端口号】，例如 http://192.168.123.100:8888 可以下载证书并安装。在打开的页面中，点击 FiddlerRoot certificate 下载证书，点击允许

![](https://img-note.langyastudio.com/20201215160204.png?x-oss-process=style/watermark)



#### 安装证书

在Settings系统设置中，点击 Profile Downloaded（已下载的配置文件） 

![](https://img-note.langyastudio.com/20201215160235.png?x-oss-process=style/watermark)



点击 Install ，安装证书

![](https://img-note.langyastudio.com/20201215160302.png?x-oss-process=style/watermark)

> 不同系统手机的下载路径不一样，例如有的是： 设置->通用->关于本机->证书信任设置



#### 配置代理

配置手机无线信号的代理

手机设置 -> WLAN -> 选择无线网络 -> HTTP Proxy，选择 Manual，Server 为 Fiddler 的电脑 ip 地址，端口号为 Fiddler 的端口号：

![](https://img-note.langyastudio.com/20201215160328.png?x-oss-process=style/watermark)



此时操作浏览器或APP，在 fiddler 中可以看到完成的请求和响应数据：

![image-20201215151817118](https://img-note.langyastudio.com/20201215151817.png?x-oss-process=style/watermark)











