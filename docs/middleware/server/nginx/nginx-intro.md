> 本文来自JavaGuide，郎涯进行简单排版与补充



Nginx 同 Apache 一样都是 WEB 服务器，不过，Nginx 更加轻量级，它的内存占用少，启动极快，高并发能力强，在互联网项目中广泛应用。并且，Nginx 可以作为反向代理服务器使用，支持 IMAP/POP3/SMTP 服务。

## Nginx 的特点是有哪些

1. **内存占用非常少** ：一般情况下，10000 个非活跃的 HTTP Keep-Alive 连接在 Nginx 中仅消耗 2.5MB 的内存，这是 Nginx 支持高并发连接的基础
2. **高并发** : 单机支持 10 万以上的并发连接

1. **跨平台** :可以运行在 Linux，Windows，FreeBSD，Solaris，AIX，Mac OS 等操作系统上
2. **扩展性好** ：第三方插件非常多

1. **安装使用简单** ：对于简单的应用场景，我们很快就能够上手使用
2. **稳定性好** ：bug 少，不会遇到各种奇葩的问题

1. **免费** ：开源软件，免费使用
2. ......

你可以配合 [**nginx-tutorial**](https://github.com/dunwu/nginx-tutorial) 这个开源 Nginx 极简教程食用。

![img](https://gitee.com/SnailClimb/blog-images/raw/master/nginx//nginx-tutorial.png)



## Nginx 能用来做什么

### 静态资源

Nginx 是一个 HTTP 服务器，可以将服务器上的静态文件（如 HTML、图片）通过 HTTP 协议展现给客户端。因此，我们可以使用 Nginx 搭建静态资源 Web 服务器

不过，记得使用 **gzip 压缩**静态资源来减少网络传输。

举个例子：我们来使用 Nginx 搭建一个静态网页服务。先将静态网页上传到服务器，然后修改`/nginx/conf` 目录下的 `nginx.conf` 文件(Nginx 配置文件)。修改完成之后，重启 Nginx，再请求对应 `ip/域名 + 端口 + 资源` 地址就可以访问到网页。

```nginx
server {
	// 监听的端口号
	listen       80;
	// server 名称
	server_name  localhost;

	// 匹配 api，将所有 :80/api 的请求指到指定文件夹
	location /api {
		root   /mnt/web/;
		// 默认打开 index.html
		index  index.html index.htm;
	}
}
```



### 反向代理

客户端将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器，获取数据后再返回给客户端。对外暴露的是反向代理服务器地址，隐藏了真实服务器 IP 地址。反向代理“代理”的是目标服务器，这一个过程对于客户端而言是透明的。

举个例子：公司内网部署了 3 台服务器，客户端请求直接经过代理服务器，由代理服务器将请求转发到内网服务器并最终决定哪一台服务器处理客户端请求。

![img](https://img-note.langyastudio.com/202202191708071.png?x-oss-process=style/watermark)



反向代理隐藏了真实的服务器，为服务器收发请求，使真实服务器对客户端不可见。一般在处理**跨域请求**的时候比较常用。现在基本上所有的大型网站都设置了反向代理。

Nginx 支持配置反向代理，通过反向代理实现网站的**负载均衡**。



### 正向代理

**提示** ：想要理解正确理解和区分正向代理和反向代理，你要关注的是代理对象，**正向代理“代理”的是客户端，反向代理“代理”的是目标服务器。** 

一位大佬说的一句话挺精辟的：代理其实就是一个中介，A 和 B 本来以直连，中间插入一个 C，C 就是中介。刚开始的时候，代理多数是帮助内网 client 访问外网 server 用的（比如 HTTP 代理），从内到外 . 后来出现了反向代理，"反向"这个词在这儿的意思其实是指方向相反，即代理将来自外网 client 的请求 forward 到内网 server，从外到内

Nginx 主要被作为反向代理服务器使用，不过，其同样也是正向代理服务器的一个选择。

客户端通过正向代理服务器访问目标服务器。正向代理“代理”的是客户端，目标服务器不知道客户端是谁，也就是说客户端对目标服务器的这次访问是透明的。

为了实现正向代理，客户端需要设置正向代理服务器的 IP 地址以及代理程序的端口。

举个例子：我们无法直接访问外网，但是可以借助科学上网工具 **VPN** 来访问。VPN 会把访问外网服务器（目标服务器）的客户端请求代理到一个可以直接访问外网的代理服务器上去。代理服务器会把外网服务器返回的内容再转发给客户端。



![img](https://img-note.langyastudio.com/202202191708937.png?x-oss-process=style/watermark)



外网服务器并不知道客户端是通过 VPN 访问的

相关阅读：

- [使用 Nginx 作为 HTTPS 正向代理服务器](https://developer.aliyun.com/article/706196)
- [图解及代码实现正向代理、反向代理及负载均衡](https://bbs.huaweicloud.com/blogs/301714)



### 负载均衡

如果一台服务器处理用户请求处理不过来的话，一个简单的办法就是增加多台服务器（服务器集群）部署相同的服务来处理用户请求。

Nginx 可以将接收到的客户端请求以一定的规则（负载均衡策略）均匀地分配到这个服务器集群中所有的服务器上，这个就叫做 **负载均衡**。

可以看出，Nginx 在其中充当的就是反向代理服务器的作用，负载均衡正是 Nginx 作为反向代理服务器最常见的一个应用。

除此之外，Nginx 还带有**健康检查**（服务器心跳检查）功能，会定期轮询向集群里的所有服务器发送健康检查请求，来检查集群中是否有服务器处于异常状态。

![img](https://img-note.langyastudio.com/202202191708966.png?x-oss-process=style/watermark)

相关阅读：[一篇文章搞定 Nginx 反向代理与负载均衡](https://codingnote.cc/zh-tw/p/220338/)



### 动静分离

动静分离就是把动态请求和静态请求分开，不是讲动态页面和静态页面物理分离，可以理解为 Nginx 处理静态页面，Tomcat 或者其他 Web 服务器处理动态页面。

动静分离可以减轻 Tomcat 或者其他 Web 服务器的压力，提高网站响应速度。



## Nginx 有哪些负载均衡策略

相关参考： 

- [五分钟看懂 Nginx 负载均衡](https://www.zoo.team/article/nginx)
- [Nginx 官方文档](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/) 

Nginx 的负载均衡策略不止下面介绍的这四种，我这里只是列举几个比较常用的负载均衡策略。



### 轮询（默认）

轮询为负载均衡中较为基础也较为简单的算法，每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器 down 掉，能自动剔除。

```nginx
upstream backserver {
  server 172.27.26.174:8099;
  server 172.27.26.175:8099;
  server 172.27.26.176:8099;
}
```

**特点**：由于该算法中每个请求按时间顺序逐一分配到不同的服务器处理，因此适用于服务器性能相近的集群情况，其中每个服务器承载相同的负载。但对于服务器性能不同的集群而言，该算法容易引发资源分配不合理等问题。



### 加权随机

weight，代表权重，权重越高的后端服务器被访问的概率越大。

```nginx
upstream backserver {
  server 172.27.26.174:8099 weight=6;
  server 172.27.26.175:8099 weight=2;
  server 172.27.26.176:8099 weight=3;
}
```

**特点**：加权轮询可以应用于服务器性能不等的集群中，使资源分配更加合理化。



### IP 哈希

根据发出请求的和护短 ip 的 hash 值来分配服务器，可以保证**同 IP 发出的请求映射到同一服务器**，或者具有相同 hash 值的不同 IP 映射到同一服务器。

**特点**：该算法在一定程度上解决了集群部署环境下 Session 不共享的问题。

```nginx
upstream backserver {
  ip_hash;
  server 172.27.26.174:8099;
  server 172.27.26.175:8099;
  server 172.27.26.176:8099;
}
```



### 最小连接数

当有新的请求出现时，遍历服务器节点列表并选取其中连接数最小的一台服务器来响应当前请求。连接数可以理解为当前处理的请求数。

```nginx
upstream backserver {
  least_conn;
  server 172.27.26.174:8099;
  server 172.27.26.175:8099;
  server 172.27.26.176:8099;
}
```



## Nginx 常用命令有哪些

- 启动 `nginx` 。
- 停止 `nginx -s stop` 或 `nginx -s quit` 。

- 重载配置 `./sbin/nginx -s reload(平滑重启)` 或 `service nginx reload` 。
- 重载指定配置文件 `.nginx -c /usr/local/nginx/conf/nginx.conf` 。

- 查看 nginx 版本 `nginx -v` 。
- 检查配置文件是否正确 `nginx -t` 。

- 显示帮助信息 `nginx -h` 。



## Nginx 性能优化的常见方式

主要从进程数、连接数、缓存、Gzip压缩、超时设置等入手

[高性能/安全/稳定的在线 nginx config](https://www.digitalocean.com/community/tools/nginx?global.app.lang=zhCN)



## LVS、Nginx、HAproxy 有什么区别

LVS、Nginx、HAProxy 是目前使用最广泛的三种软件负载均衡软件。

- LVS 是 Linux Virtual Server 的简称，也就是 Linux 虚拟服务器。LVS 是四层负载均衡，建立在 OSI 模型的第四层（**传输层**）之上，性能非常强大
- HAProxy 可以工作在四层和七层（**传输层和应用层**），是专门用来做代理服务器的

- Nginx 负载均衡主要是对七层网络通信模型中的第七层**应用层**上的 HTTP、HTTPS 进行支持。Nginx 是以反向代理的方式进行负载均衡的



## Nginx 如何实现后端服务健康检查

我们可以利用第三方模块 **upstream_check_module** 来检测后端服务的健康状态，如果后端服务器不可用，则所有的请求不转发到这台服务器。

upstream_check_module 是一款阿里的一位大佬开源的，使用 Perl 和 C 编写而成，Github 地址 ：https://github.com/yaoweibin/nginx_upstream_check_module 。

关于 upstream_check_module 实现后端服务健康检查的具体做法可以参考[Nginx 负载均衡健康检查功能](https://cloud.tencent.com/developer/article/1700001)这篇文章。

![img](https://img-note.langyastudio.com/202202191709754.png?x-oss-process=style/watermark)



## 如何保证 Nginx 服务的高可用

Nginx 可以结合 Keepalived 来实现高可用。

**什么是 Keepalived ？** 根据官网介绍：

Keepalived 是一个用 C 语言编写的开源路由软件，是 Linux 下一个轻量级别的负载均衡和高可用解决方案。Keepalived 的负载均衡依赖于众所周知且广泛使用的 Linux 虚拟服务器 (IPVS 即 IP Virtual Server，内置在 Linux 内核中的传输层负载均衡器) 内核模块，提供第 4 层负载平衡。Keepalived 实现了一组检查器用于根据服务器节点的健康状况动态维护和管理服务器集群。 

Keepalived 的高可用性是通过虚拟路由冗余协议（VRRP 即 Virtual Router Redundancy Protocol，实现路由器高可用的协议）实现的，可以用来解决单点故障。 

Github 地址：https://github.com/acassen/keepalived

Keepalived 不仅仅可以和 Nginx 搭配使用，还可以和 [LVS](https://wsgzao.github.io/post/lvs-keepalived/)、[MySQL](https://programmer.group/high-availability-scheme-implementation-of-mysql-master-replication-keepalived.html)、[HAProxy](https://docs.oracle.com/cd/E37670_01/E41138/html/section_sm3_svy_4r.html) 等软件配合使用。

再来简单介绍一下 Keepalived+Nginx 实现高可用的常见方法：

1. 准备 2 台 Nginx 服务器，一台为主服务，一台为备用服务；
2. 在两台 Nginx 服务器上安装并配置 Keepalived；
3. 为两台 Nginx 服务器绑定同一个虚拟 IP；
2. 编写 Nginx 检测脚本用于通过 Keepalived 检测 Nginx 主服务器的状态是否正常；

如果 Nginx 主服务器宕机的话，会自动进行故障转移，备用 Nginx 主服务器升级为主服务。并且，这个切换对外是透明的，因为使用的虚拟 IP，虚拟 IP 不会改变。

相关阅读：

- [Nginx 系列教程（五）| 利用 Nginx+Keepalived 实现高可用技术 |](https://juejin.cn/post/6970093569096810526)
- [搭建 Keepalived Nginx 高可用 Web 集群 - 华为云](https://support.huaweicloud.com/bestpractice-vpc/bestpractice_0010.html)



## Nginx 进阶

### Nginx 总体架构了解吗

关于 Nginx 总结架构的详细解答，请看这篇文章：[最近和 Nginx 杠上了！](https://mp.weixin.qq.com/s/CxapDUkSdqBbuJU4JrQ8Aw)

对于传统的 HTTP 和反向代理服务器而言，在处理并发请求的时候会使用单进程或线程的模式处理，同时会止网络或输入/输出操作。

这种方式会消耗大量的内存和 CPU 资源。因为每产生一个单独的进程或线程需要准备一套新的运行时环境，包括分配堆和堆栈内存，以及创建新的执行上下文。

可以想象在处理多请求时会生成对应数目的线程或进程，导致由于线程在不断上下文切换上耗费大量资源。

由于上面的原因，Nginx 在设计之初就使用了模块化、事件驱动、异步处理，非阻塞的架构。

一张图来了解 Nginx 的总结架构:

![img](https://img-note.langyastudio.com/202202191709087.webp?x-oss-process=style/watermark)



### Nginx 进程模型了解么

关于进程模型的详细解答，请看这篇文章：[Nginx 工作模式和进程模型](https://learnku.com/articles/38414)

Nginx 启动后，会产生一个 master 主进程，主进程执行一系列的工作后会产生一个或者多个工作进程 worker 进程。master 进程用来管理 worker 进程， worker 进程负责处理网络请求。也就是说 **Nginx 采用的是经典的 master-worker 模型的多进程模型** 。



### Nginx 如何处理 HTTP 请求

- [Nginx 是如何处理 HTTP 头部的？](https://segmentfault.com/a/1190000022348375)
- [Nginx 处理 HTTP 请求的 11 个阶段](https://segmentfault.com/a/1190000022709975)



## 参考

- [正向代理与反向代理【总结】](https://www.cnblogs.com/Anker/p/6056540.html)
- [Nginx 详解：Nginx 是什么? 能干嘛?](https://kb.cnblogs.com/page/661047/)