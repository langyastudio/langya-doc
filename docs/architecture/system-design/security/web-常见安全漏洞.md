﻿> 本文出处未知



﻿### SQL 注入

----------

SQL 注入就是通过给 web 应用接口传入一些特殊字符，达到欺骗服务器执行恶意的 SQL 命令。

SQL 注入漏洞属于后端的范畴，但前端也可做体验上的优化。

#### 原因
当使用外部不可信任的数据作为参数进行数据库的增、删、改、查时，如果未对外部数据进行过滤，就会产生 SQL 注入漏洞。

比如：

```sql
name = "外部输入名称";

sql = "select * from users where name=" + name;
```

上面的 SQL 语句目的是通过用户输入的用户名查找用户信息，因为由于 SQL 语句是直接拼接的，也没有进行过滤，所以，当用户输入 `'' or '1'='1'` 时，这个语句的功能就是搜索 `users` 全表的记录。

```sql
select * from users where name='' or '1'='1';
```

#### 解决方案
具体的解决方案很多，但大部分都是基于一点：不信任任何外部输入。

所以，对任何外部输入都进行过滤，然后再进行数据库的增、删、改、查。

此外，适当的权限控制、不曝露必要的安全信息和日志也有助于预防 SQL 注入漏洞。

参考 [Web 安全漏洞之 SQL 注入 - 防御方法](https://juejin.im/post/5bd5b820e51d456f72531fa8#heading-2) 了解具体的解决方案。

#### 推荐参考
*   [Web 安全漏洞之 SQL 注入](https://juejin.im/post/5bd5b820e51d456f72531fa8)
*   [SQL 注入详解](https://segmentfault.com/a/1190000007520556)

&nbsp;
### XSS 攻击
----------

XSS 攻击全称跨站脚本攻击（Cross-Site Scripting），简单的说就是攻击者通过在目标网站上注入恶意脚本并运行，获取用户的敏感信息如 Cookie、SessionID 等，影响网站与用户数据安全。

XSS 攻击更偏向前端的范畴，但后端在保存数据的时候也需要对数据进行安全过滤。

#### 原因
当攻击者通过某种方式向浏览器页面注入了恶意代码，并且浏览器执行了这些代码。

比如：

在一个文章应用中（如微信文章），攻击者在文章编辑后台通过注入 `script` 标签及 `js` 代码，后端未加过滤就保存到数据库，前端渲染文章详情的时候也未加过滤，这就会让这段 `js` 代码执行，引起 XSS 攻击。

#### 解决方案
一个基本的思路是渲染前端页面（不管是客户端渲染还是服务器端渲染）或者动态插入 HTML 片段时，任何数据都不可信任，都要先做 HTML 过滤，然后再渲染。

参考 [前端安全系列（一）：如何防止XSS攻击？ - 攻击的预防](https://segmentfault.com/a/1190000016551188#articleHeader7) 了解具体的解决方案。

#### 推荐参考
*   [前端安全系列（一）：如何防止XSS攻击？](https://segmentfault.com/a/1190000016551188)
*   [前端防御 XSS](https://juejin.im/entry/56da82a87664bf0052ebad41)
*   [浅说 XSS 和 CSRF](https://juejin.im/entry/5b4b56fd5188251b1a7b2ac1)

&nbsp;
### CSRF 攻击
-----------

CSRF 攻击全称跨站请求伪造（Cross-site Request Forgery），简单的说就是攻击者盗用了你的身份，以你的名义发送恶意请求。

#### 原因
一个典型的 CSRF 攻击有着如下的流程：

*   受害者登录 `a.com`，并保留了登录凭证（Cookie）
*   攻击者引诱受害者访问了 `b.com`
*   `b.com` 向 `a.com` 发送了一个请求：`a.com/act=xx`（浏览器会默认携带 `a.com` 的 Cookie）
*   `a.com` 接收到请求后，对请求进行验证，并确认是受害者的凭证，误以为是受害者自己发送的请求
*   `a.com` 以受害者的名义执行了 `act=xx`
*   攻击完成，攻击者在受害者不知情的情况下，冒充受害者，让 `a.com` 执行了自己定义的操作

注：上面的过程摘自 [前端安全系列之二：如何防止CSRF攻击？](https://segmentfault.com/a/1190000016659945)

#### 解决方案
防止 CSRF 攻击需要在服务器端入手，基本的思路是能正确识别是否是用户发起的请求。

参考 [前端安全系列之二：如何防止CSRF攻击？ - 防护策略](https://segmentfault.com/a/1190000016659945#articleHeader4) 了解具体的解决方案。

#### 推荐参考
*   [前端安全系列之二：如何防止CSRF攻击？](https://segmentfault.com/a/1190000016659945)
*   [Web安全漏洞之CSRF](https://juejin.im/post/5ba1a800e51d450e8657f5dd)
*   [浅说 XSS 和 CSRF](https://juejin.im/entry/5b4b56fd5188251b1a7b2ac1)

&nbsp;
### DDoS 攻击
-----------

DoS 攻击全称拒绝服务（Denial of Service），简单的说就是让一个公开网站无法访问，而 DDoS 攻击（分布式拒绝服务 Distributed Denial of Service）是 DoS 的升级版。

这个就完全属于后端的范畴了。

#### 原因
攻击者不断地提出服务请求，让合法用户的请求无法及时处理，这就是 DoS 攻击。

攻击者使用多台计算机或者计算机集群进行 DoS 攻击，就是 DDoS 攻击。

#### 解决方案
防止 DDoS 攻击的基本思路是限流，限制单个用户的流量（包括 IP 等）。

参考 [DDoS的攻击及防御 - 防御](https://segmentfault.com/a/1190000016584829#articleHeader19) 了解具体的解决方案。

#### 推荐参考
*   [DDoS的攻击及防御](https://segmentfault.com/a/1190000016584829)
*   [浅谈 DDoS 攻击与防御](https://juejin.im/entry/5b7a21256fb9a01a031aef67)
*   [使用 Nginx、Nginx Plus 抵御 DDOS 攻击](https://juejin.im/entry/56d824591ea493005db9d284)

&nbsp;
### XXE 漏洞
----------

XXE 漏洞全称 XML 外部实体漏洞（XML External Entity），当应用程序解析 XML 输入时，如果没有禁止外部实体的加载，导致可加载恶意外部文件和代码，就会造成任意文件读取、命令执行、内网端口扫描、攻击内网网站等攻击。

这个只在能够接收 XML 格式参数的接口才会出现。

#### 解决方案
1.  禁用外部实体
2.  过滤用户提交的XML数据


参考 [xxe漏洞的学习与利用总结](https://www.cnblogs.com/r00tuser/p/7255939.html) 了解具体的解决方案。


#### 推荐参考
*   [好刚: 6分钟视频看懂XXE漏洞攻击](https://juejin.im/entry/5b719fdc6fb9a009a0607aaa)
*   [xxe漏洞的学习与利用总结](https://www.cnblogs.com/r00tuser/p/7255939.html)
*   [XXE漏洞攻防学习（上）](https://www.cnblogs.com/ESHLkangi/p/9245404.html)

&nbsp;
### JSON 劫持
-----------

JSON 劫持（JSON Hijacking）是用于获取敏感数据的一种攻击方式，属于 CSRF 攻击的范畴。

#### 原因
一些 Web 应用会把一些敏感数据以 json 的形式返回到前端，如果仅仅通过 Cookie 来判断请求是否合法，那么就可以利用类似 CSRF 的手段，向目标服务器发送请求，以获得敏感数据。

比如下面的链接在已登录的情况下会返回 json 格式的用户信息：

```
http://www.test.com/userinfo
```

攻击者可以在自己的虚假页面中，加入如下标签：

```javascript
<script src="http://www.test.com/userinfo"></script>
```

如果当前浏览器已经登录了 `www.test.com`，并且 Cookie 未过期，然后访问了攻击者的虚假页面，那么该页面就可以拿到 json 形式的用户敏感信息，因为 `script` 标签会自动解析 json 数据，生成对应的 js 对象。然后再通过：

```javascript
Object.prototype.__defineSetter__
```

这个函数来触发自己的恶意代码。

但是这个函数在当前的新版本 Chrome 和 Firefox 中都已经失效了。

注：上面的过程摘自 [JSON和JSONP劫持以及解决方法](https://blog.csdn.net/yjclsx/article/details/80353754)

#### 解决方案
1.  `X-Requested-With` 标识
2.  浏览器 JSON 数据识别
3.  禁止 Javascript 执行 JSON 数据

#### 推荐参考
*   [JSON和JSONP劫持以及解决方法](https://blog.csdn.net/yjclsx/article/details/80353754)
*   [JSONP 安全攻防技术（JSON劫持、 XSS漏洞）](https://www.cnblogs.com/52php/p/5677775.html)

&nbsp;
### 暴力破解
--------

这个一般针对密码而言，弱密码（Weak Password）很容易被别人（对你很了解的人等）猜到或被破解工具暴力破解。

#### 解决方案
1.  密码复杂度要足够大，也要足够隐蔽
2.  限制尝试次数

&nbsp;
### HTTP 报头追踪漏洞
---------------

HTTP/1.1（RFC2616）规范定义了 HTTP TRACE 方法，主要是用于客户端通过向 Web 服务器提交 TRACE 请求来进行测试或获得诊断信息。

当 Web 服务器启用 TRACE 时，提交的请求头会在服务器响应的内容（Body）中完整的返回，其中 HTTP 头很可能包括 Session Token、Cookies 或其它认证信息。攻击者可以利用此漏洞来欺骗合法用户并得到他们的私人信息。

#### 解决方案
禁用 HTTP TRACE 方法。

&nbsp;
### 信息泄露
--------

由于 Web 服务器或应用程序没有正确处理一些特殊请求，泄露 Web 服务器的一些敏感信息，如用户名、密码、源代码、服务器信息、配置信息等。

所以一般需注意：

*   应用程序报错时，不对外产生调试信息
*   过滤用户提交的数据与特殊字符
*   保证源代码、服务器配置的安全

&nbsp;
### 目录遍历漏洞
-----------

攻击者向 Web 服务器发送请求，通过在 URL 中或在有特殊意义的目录中附加 `../`、或者附加 `../` 的一些变形（如 `..\` 或 `..//` 甚至其编码），导致攻击者能够访问未授权的目录，以及在 Web 服务器的根目录以外执行命令。

&nbsp;
### 命令执行漏洞
-----------

命令执行漏洞是通过 URL 发起请求，在 Web 服务器端执行未授权的命令，获取系统信息、篡改系统配置、控制整个系统、使系统瘫痪等。

&nbsp;
### 文件上传漏洞
-----------

如果对文件上传路径变量过滤不严，并且对用户上传的文件后缀以及文件类型限制不严，攻击者可通过 Web 访问的目录上传任意文件，包括网站后门文件（`webshell`），进而远程控制网站服务器。

所以一般需注意：

*   在开发网站及应用程序过程中，需严格限制和校验上传的文件，禁止上传恶意代码的文件
*   限制相关目录的执行权限，防范 `webshell` 攻击

&nbsp;
### 其他漏洞
---------

1.  SSLStrip 攻击
2.  OpenSSL Heartbleed 安全漏洞
3.  CCS 注入漏洞
4.  证书有效性验证漏洞

&nbsp;
### 业务漏洞
---------

一般业务漏洞是跟具体的应用程序相关，比如参数篡改（连续编号 ID / 订单、1 元支付）、重放攻击（伪装支付）、权限控制（越权操作）等。

另外可以参考：[6种常见web漏洞坑](https://blog.csdn.net/xueshao110/article/details/78912988)

&nbsp;
### 框架或应用漏洞
------------

*   WordPress 4.7 / 4.7.1：REST API 内容注入漏洞
*   Drupal Module RESTWS 7.x：Remote PHP Code Execution
*   SugarCRM 6.5.23：REST PHP Object Injection Exploit
*   Apache Struts：REST Plugin With Dynamic Method Invocation Remote Code Execution
*   Oracle GlassFish Server：REST CSRF
*   QQ Browser 9.6：API 权限控制问题导致泄露隐私模式
*   Hacking Docker：Registry API 未授权访问
