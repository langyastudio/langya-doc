SSO（ Single Sign On ），即单点登录，是一种控制多个相关但彼此独立的系统的访问权限, 拥有这一权限的用户可以使用单一的ID和密码访问某个或多个系统从而避免使用不同的用户名或密码，或者通过某种配置无缝地登录每个系统 。

对于大型系统来说使用单点登录可以减少用户很多的麻烦。就拿百度来说吧，百度下面有很多的子系统——百度经验、百度知道、百度文库等等，如果我们使用这些系统的时候，每一个系统都需要我们输入用户名和密码登录一次的话，我相信用户体验肯定会直线下降。

与 SSO 交互的2个元素：1.  用户，2. 系统，它的特点是：**一次登录，全部访问**。SSO 是访问控制的一种，控制用户能否登录，即验证用户身份，而且是所有其它系统的身份验证都在它这里进行，从整个系统层面来看 SSO ，它的核心就是这3个元素了：1. 用户，2. 系统，3. 验证中心。

![这里写图片描述](https://img-note.langyastudio.com/20210707172040.png?x-oss-process=style/watermark)



### 同一个域但不同的子域如何进行单点登录

假如我们的站点是按照下面的域名进行部署的：

- sub1.onmpw.com
- sub2.onmpw.com

这两个站点共享同一域 onmpw.com 。

默认情况下，浏览器会发送 cookie 所属的域对应的主机。也就是说，来自于 sub1.onmpw.com 的 cookie 默认所属的域是 .sub1.onmpw.com 。因此，sub2.onmpw.com 不会得到任何的属于 sub1.onmpw.com 的 cookie 信息。因为它们是在不同的主机上面，并且二者的子域也是不同的。



#### 设置二者的 cookie 信息在同一个域下

-  登录 sub1.onmpw.com 系统
-  登录成功以后，生成唯一标识符Token（知道token，就知道哪个用户登录了）。设置 cookie 信息，这里需要注意，将Token存到 cookie 中，但是在设置的时候必须将这 cookie 的所属域设置为顶级域 .onmpw.com 。这里可以使用 setcookie 函数，该函数的第四个参数是用来设置 cookie 所述域的。
```php
setcookie(‘token’, ’xxx’, '/', ’.onmpw.com’);
```
- 访问 sub2.onmpw.com 系统，浏览器会将 cookie 中的信息 token 附带在请求中一块儿发送到 sub2.onmpw.com 系统。这时该系统会先检查 session 是否登录，如果没有登录则验证 cookie 中的 token 从而实现自动登录。

- sub2.onmpw.com 登录成功以后再写 session 信息。以后的验证就用自己的 session 信息验证就可以了。



#### 退出登录

这里存在一个问题就是 sub1 系统退出以后，除了可以清除自身的 session 信息和所属域为 .onmpw.com 的 cookie 的信息。它并不能清除 sub2 系统的 session 信息。那 sub2 仍然是登录状态。也就是说，这种方式虽说可以实现单点登录，但是不能实现同时退出。原因是，sub1 和 sub2 虽说通过 setcookie 函数的设置可以共享 cookie，但是二者的sessionId 是不同的，而且这个 sessionId 在浏览器中也是以 cookie 的形式存储的，不过它所属的域并不是 .onmpw.com 。

那如何解决这个问题呢？我们知道，对于这种情况，只要是两个系统的 sessionId 相同就可以解决这个问题了。也就是说存放 sessionId 的 cookie 所属的域也是 .onmpw.com 。在PHP中，sessionId 是在 session_start() 调用以后生成的。要想使sub1 和 sub2 有共同的 sessionId ，那必须在 session_start() 之前设置 sessionId 所属域：
```
ini_set('session.cookie_path', '/');
ini_set('session.cookie_domain', '.onmpw.com');
ini_set('session.cookie_lifetime', '0');
```

>  - 1、经过上面的步骤就可以实现不同二级域名的单点登录与退出。
>  - 2、不过还可以再简化，如确保 **sessionId** 相同就可以实现不同二级域名的单点登录与退出。
>  - 参考文章： https://www.onmpw.com/tm/xwzj/network_145.html



### 不同域之间如何实现单点登录 - CAS

假设我们需要在以下这些站之间实现单点登录

- www.onmpw1.com
- www.onmpw2.com
- www.onmpw3.com

上面的方案就行不通了。



目前 github 上有开源的 SSO 解决方案，实现原理与主流 SSO 差不多：

- [https://github.com/jasny/sso](https://github.com/jasny/sso)
- [wiki](https://github.com/jasny/sso/wiki)
- [demo](https://github.com/phpyii/tp5-sso)



核心原理： 

- 1、**客户端访问不同的子系统，子系统对应的 SSO 用户服务中心使用相同的 SessionId**。
- 2、**子系统 Broker 与 Server 间通过 attach 进行了授权链接绑定**

同根域名不指定 domain 根域名，第一次授权需要每次都要跳转到授权服务，但是指定了domain, 同根域名只要有一个授权成功，都可以共用那个 cookie 了，下次就不用再授权了，直接就可以请求用户信息了。



**第一次访问A：**

![这里写图片描述](https://img-note.langyastudio.com/20210707172046.png?x-oss-process=style/watermark)



**第二次访问B：**

![这里写图片描述](https://img-note.langyastudio.com/20210707172048.png?x-oss-process=style/watermark)



#### 登录状态判断

用户到认证中心登录后，用户和认证中心之间建立起了会话，我们把这个会话称为**全局会话**。当用户后续访问系统应用时，我们不可能每次应用请求都到认证中心去判定是否登录，这样效率非常低下，这也是单Web应用不需要考虑的。

我们可以在系统应用和用户浏览器之间建立起**局部会话**，局部会话保持了客户端与该系统应用的登录状态，局部会话依附于全局会话存在，全局会话消失，局部会话必须消失。

用户访问应用时，首先判断局部会话是否存在，如存在，即认为是登录状态，无需再到认证中心去判断。如不存在，**就重定向到认证中心判断全局会话是否存在**，如存在，通知该应用，该应用与客户端就建立起它们之间局部会话，下次请求该应用，就不去认证中心验证了。



#### 退出

用户在一个系统退出了，访问其它子系统，也应该是退出状态。要想做到这一点，应用除结束本地局部会话外，还应该通知认证中心该用户退出。

认证中心接到退出通知，即可结束全局会话，用户访问其它应用时，都显示已登出状态。

> 需不需要立即通知所有已建立局部会话的子系统，将它们的局部会话销毁，可根据实际项目来。
---
其他较为OK的SSO文章：
> [https://mp.weixin.qq.com/s?__biz=MjM5NDM4MDIwNw==&mid=2448834907&idx=1&sn=69a36dfb1906d8f14d25cd14aa40b607&chksm=b28a419b85fdc88dc1ba0029eec19d2d2504b00a2baa7b8cd0a46b335a61f9290728a4005f39&mpshare=1&scene=1&srcid=0305ByhrOcLz8MM7uW5FXFN7#rd](https://mp.weixin.qq.com/s?__biz=MjM5NDM4MDIwNw==&mid=2448834907&idx=1&sn=69a36dfb1906d8f14d25cd14aa40b607&chksm=b28a419b85fdc88dc1ba0029eec19d2d2504b00a2baa7b8cd0a46b335a61f9290728a4005f39&mpshare=1&scene=1&srcid=0305ByhrOcLz8MM7uW5FXFN7#rd)
>
> [https://juejin.im/post/5a002b536fb9a045132a1727](https://juejin.im/post/5a002b536fb9a045132a1727)
>
> [https://apereo.github.io/cas/6.0.x/](https://apereo.github.io/cas/6.0.x/)