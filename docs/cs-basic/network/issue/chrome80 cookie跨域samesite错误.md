本地局域网前后端分离的项目，前端是 `192.168.123.90`，后端是 `192.168.123.2`。今天早上发现用户登录报告登录失败（本质原因是无法设置 cookie ）。一开始以为后端出问题了，但最近没改用户登录的相关逻辑，后来换火狐、edge 是可以的，并且有些人的 Google 可以正常登录。

有问题的 Google 浏览器的调试信息报告以下错误：

设置 cookie 时提示：`This set-cookie didn't specify a "SameSite" attribute and was defaulted to "SameSite=Lax" and broke the same rules specified in the SameSiteLax value`

![image-20200730114327719](https://img-note.langyastudio.com/20200730114327.png?x-oss-process=style/watermark)

结合以上分析，初步判断是新版的 Google 浏览器对于 Cookie 跨域的限制问题。



经过查询资料发现：

从 `Chrome 51` 开始，浏览器的 `Cookie` 新增加了一个 [SameSite](http://www.ruanyifeng.com/blog/2019/09/cookie-samesite.html) 属性，用来防止 `CSRF` 攻击和用户追踪。该设置当前默认是关闭的

**在 `Chrome 80` 之后，该功能默认已开启！**



### 快速解决方案

对于 Google 80 之后的浏览器，可以禁用 `SameSite` 或修改源码等方式解决该问题。

#### 禁用 SameSite

Google 浏览器访问  `chrome://flags/#same-site-by-default-cookies` 地址，设置 cookie 的该选项为禁用，然后重启浏览器。

![image-20200730120125109](https://img-note.langyastudio.com/20200730120125.png?x-oss-process=style/watermark)

#### 修改源码

将 `SameSite` 属性值改为 `None` , 同时 将 `secure` 属性设置为 `true`。此时后端服务的域名必须使用 `https` 协议访问。

> 最佳方案是用 `token` 代替 `Cookie` 方式作验证用户登录



### Samesite 详解

Cookie 的 [SameSite](http://www.chromium.org/updates/same-site) 属性用来限制第三方 Cookie，从而减少安全风险。

![image-20200730121143538](https://img-note.langyastudio.com/20200730121143.png?x-oss-process=style/watermark)

它可以设置三个值

> - Strict
> - Lax
> - None



#### Strict

`Strict` 最为严格，完全禁止第三方 Cookie，跨站点时，任何情况下都不会发送 Cookie。换言之，只有当前网页的 URL 与请求目标一致，才会带上 Cookie。

 ```bash
 Set-Cookie: CookieName=CookieValue; SameSite=Strict;
 ```

这个规则过于严格，可能造成非常不好的用户体验。比如，当前网页有一个 GitHub 链接，用户点击跳转就不会带有 GitHub 的 Cookie，跳转过去总是未登陆状态。



#### Lax

`Lax` 规则稍稍放宽，大多数情况也是不发送第三方 Cookie，但是导航到目标网址的 Get 请求除外。

 ```markup
 Set-Cookie: CookieName=CookieValue; SameSite=Lax;
 ```

导航到目标网址的 GET 请求，只包括三种情况：链接，预加载请求，GET 表单。详见下表。

| 请求类型  | 示例                                 | 正常情况    | Lax         |
| :-------- | :----------------------------------- | :---------- | :---------- |
| 链接      | `<a href="..."></a>`                 | 发送 Cookie | 发送 Cookie |
| 预加载    | `<link rel="prerender" href="..."/>` | 发送 Cookie | 发送 Cookie |
| GET 表单  | `<form method="GET" action="...">`   | 发送 Cookie | 发送 Cookie |
| POST 表单 | `<form method="POST" action="...">`  | 发送 Cookie | 不发送      |
| iframe    | `<iframe src="..."></iframe>`        | 发送 Cookie | 不发送      |
| AJAX      | `$.get("...")`                       | 发送 Cookie | 不发送      |
| Image     | `<img src="...">`                    | 发送 Cookie | 不发送      |

设置了`Strict`或`Lax`以后，基本就杜绝了 CSRF 攻击。当然，前提是用户浏览器支持 SameSite 属性。



#### None

Chrome 计划将 `Lax` 变为默认设置。这时网站可以选择显式关闭 `SameSite` 属性，将其设为 `None`。不过，前提是必须同时设置 `Secure` 属性（Cookie 只能通过 **HTTPS** 协议发送），否则无效。

下面的设置无效。

 ```bash
 Set-Cookie: widget_session=abc123; SameSite=None
 ```

下面的设置有效。

 ```bash
 Set-Cookie: widget_session=abc123; SameSite=None; Secure
 ```



> 参考链接：http://www.ruanyifeng.com/blog/2019/09/cookie-samesite.html