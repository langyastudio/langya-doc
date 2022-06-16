## ApiPost 是什么

ApiPost 官网地址：[https://www.apipost.cn?token=4f13dfb0ecf6000bb94797449466f723](https://www.apipost.cn?token=4f13dfb0ecf6000bb94797449466f723)
Web 版链接：[https://console.apipost.cn/register?token=4f13dfb0ecf6000bb94797449466f723](https://console.apipost.cn/register?token=4f13dfb0ecf6000bb94797449466f723)
客户端下载地址：[https://www.apipost.cn/download.html?token=4f13dfb0ecf6000bb94797449466f723](https://www.apipost.cn/download.html?token=4f13dfb0ecf6000bb94797449466f723)



ApiPost =**Postman+Swagger+Mock**，前端、后端、测试同时编辑，内容实时同步。15 人以下的团队和个人完全免费，针对高校和培训机构也是完全免费的，企业也可以根据需要进行私有化部署。

![image-20220424192222570](https://img-note.langyastudio.com/202204241922665.png?x-oss-process=style/watermark)



## 与 Postman 对比

- 学习成本低

- 适合国人习惯

- 快捷文档分享

  接口文档的撰写非常麻烦，很多属于重复工作，效率低下。接口参数填写完毕后，只要在 ApiPost 按下“分享文档”按钮，就会一键自动生成漂亮、规范的文档，并且可以自定义分享有效期及权限

- 多人实时协作

  针对团队成员间协作不同步，数据保存有冲突，无法追溯变更记录的情况，多人在线协作时，ApiPost 支持数据实时同步，有冲突解决机制，并且可以追溯协作日志

- 参数库描述

  很多接口往往具有大量相同名称、相同意义的参数，每次手动重复录入，非常耗时、低效。ApiPost 通过自定义参数描述库，可以将大量参数进行预注释，并在输入参数时支持自动填充描述，节省了我们不少重复录入参数描述的时间

- 调用次数无限

- 项目接口无限

- Mock次数无限

> 自 6.x 版本起，优化了 ApiPost 离线使用的体验，支持**未登陆使用** ApiPost 和 弱网或者断网情况下的使用。并不会弹出登陆弹窗。同时还支持 Websocket 测试功能



## 为什么用 ApiPost

**业务提需求 -> 产品定方案 -> 研发做实现 -> 测试验流程**

以上四种角色是互联网中一整条产品需求从生产到上线的必要条件，当需求和目标明确后，决定整个需求的交付质量非常重要的一环就是研发到测试，这一个相互制约的角色关系在，只有研发能更靠的提交代码、测试完整的验证流程和细节，才能保证交付质量

**背景(S)** ：怎么来提高代码质量呢？一般我们都会要求研发在开发代码的过程中对接口必须100%覆盖度的编写单元测试，验证自己的代码逻辑。如果最终单元测试覆盖度不足，可以由测试拒绝研发提测

**问题(T)** ：目前很多时候都是研发人员在需求全部完成开发后，人肉的方式把接口信息维护到 CF 文档，再把文档地址交给测试人员进行验证。那如果这个时候发现一些接口问题，反复修改完善代码，就会给测试的工期带来不小的压力，直至导致项目的延期上线

**方案(A)** ：我们希望在研发开发的代码的过程中，每当完成一个接口的开发，就要把接口信息完善到同一的接口调用服务平台上，这样测试人员就可以很清楚的知道，研发提测了多少个接口、每个接口的单测数据如何、所开发的接口也可以提前让前端介入减少等待时间

**结果(R)** ：最后在一个需求小组(后端、前端、测试)，都统一在一个服务 ApiPost 上维护和完善接口文档，既可以在开发过程中就能进行验证，也可以尽快的知晓开发进度，后端、前端、测试三方的配合也更加紧密，从而提升整个交付质量



## ApiPost 能干什么

基于不同的程序员角色，ApiPost 能发挥的作用也不同：

### 帮助后端大佬

- 接口调试，接口文档巧生成（漂亮、规范、高效）
- 生成 Mock 数据，提前测试，提高效率
- ApiPost 提供主流语言代码自动生成

### 帮助前端大牛

- 接口文档可浏览，不等后端口相传
- 接口调试
- 前端代码自动生成（已支持 NodeJS、Ajax）

### 帮助测试大神

- 接口调试直接用
- 接口自动化测试，程序搞定，我去喝水

### 研发经理

- 规范接口文档管理：人在文档在，人走文档还在
- 提升整体研发团队效率

![image-20220424192359224](https://img-note.langyastudio.com/202204241923561.png?x-oss-process=style/watermark)



## ApiPost 简单使用

ApiPost 官方使用文档：[https://wiki.apipost.cn/document/00091641-1e36-490d-9caf-3e47cd38bcde](https://wiki.apipost.cn/document/00091641-1e36-490d-9caf-3e47cd38bcde)



### 下载

下载地址：[https://www.apipost.cn/download.html?token=4f13dfb0ecf6000bb94797449466f723](https://www.apipost.cn/download.html?token=4f13dfb0ecf6000bb94797449466f723)

选择合适的电脑操作系统的安装包下载并安装即可

![image-20220423222259339](https://img-note.langyastudio.com/202204232223749.png?x-oss-process=style/watermark)



### HTTP 请求

ApiPost 在测试请求接口时，主要注意下面几个部分的参数配置即可：

- Header 参数

  可以设置或者导入 Header 参数，cookie 也在 Header 进行设置

- Query 参数

  Query 支持构造 URL 参数，同时支持 RESTful 的 PATH 参数

- Body 参数

  Body 提供三种类型 `form-data / x-www-form-urlencoded / raw` ，每种类型提供三种不同的UI界面

  - 当你需要提交表单时，切换到 x-www-form-urlencoded
  - 当你需要提交有文件的表单时，切换到 form-data
  - 当您需要发送JSON对象或者其他对象时，切换到对应的raw类型即可



这里以根据 IP 获取地理信息的测试为例，如

```
https://ip.taobao.com/outGetIpInfo?accessKey=alibaba-inc&ip=myip
```

- 在启动界面点击 接口 

  ![image-20220423224706268](https://img-note.langyastudio.com/202204232247539.png?x-oss-process=style/watermark)

- 填写请求地址、Query 参数后，点击发送，结课在实时响应中看到请求的结果

  ![image-20220423224931692](https://img-note.langyastudio.com/202204242055179.png?x-oss-process=style/watermark)



## 亮点推荐

### 生成接口文档

我们知道，在前后端协作开发时，接口文档是必须要的。而接口文档的编写任务往往交给后端同学去负责，需要给出接口的各种参数要求以及参考实例等等，非常繁琐。

为了前后端协作便利，很多公司使用 Swagger 作为接口文档生成工具，但是 Swagger 需要在后端模块添加额外的 Swagger 集成代码。而 ApiPost 刚好可以把 Swagger 和 Postman 二者的功能合二为一，对开发者带来极大的便利性！

- 点击分享文档，如下图所示：

![image-20220423225521736](https://img-note.langyastudio.com/202204232255860.png?x-oss-process=style/watermark)

- 通过链接地址打开后，可以很方便的查看分享的文档，漂亮、规范、高效，并且可以自定义分享有效期及权限

![image-20220423225707964](https://img-note.langyastudio.com/202204232257092.png?x-oss-process=style/watermark)



### 实时协作

针对团队成员间协作不同步，数据保存有冲突，无法追溯变更记录的情况，多人在线协作时，ApiPost 支持数据实时同步，有冲突解决机制，并且可以追溯协作日志。

![](https://img-note.langyastudio.com/202204241937365.gif)



### 参数库描述

很多接口往往具有大量相同名称、相同意义的参数，每次手动重复录入，非常耗时、低效。Apipost 通过自定义参数描述库，可以将大量参数进行预注释，并在输入参数时支持**自动填充描述**，节省了我们不少重复录入参数描述的时间。

- 在客户端，通过点击右上角的参数描述库功能，在该界面开启智能描述库即可

  ![image-20220424200700472](https://img-note.langyastudio.com/202204242007648.png?x-oss-process=style/watermark)

- 添加参数描述，如 ip地址

  这样下次输入参数名 ip 后，会自动带出参数描述

  ![image-20220424200913103](https://img-note.langyastudio.com/202204242009249.png?x-oss-process=style/watermark)



### Websocket 测试

Apipost 6.1 推出了 Websocket 测试功能，这样测试API接口、Websocket协议等一个软件就搞定了。

- 在启动界面选择 websocket，输入连接地址，点击连接

  当连接成功后，可以看到下方显示连接成功返回的消息，是不是很简便

  ![image-20220424205040669](https://img-note.langyastudio.com/202204242050827.png?x-oss-process=style/watermark) 



## 使用技巧

### 响应结果分屏展示

在 ApiPost 5.4 版本后，支持“响应结果分屏展示”，从而提升工作区的空间

![image-20220423230228837](https://img-note.langyastudio.com/202204232302985.png?x-oss-process=style/watermark)



### 精简视图

在 ApiPost 默认视图包含顶部菜单栏和左侧导航栏，5.4版本后，支持“精简视图”模式，隐藏这两个边栏，从而提升工作区的空间。

![image-20220423230352422](https://img-note.langyastudio.com/202204232303557.png?x-oss-process=style/watermark)



### 定位到当前接口

我们可以通过“定位到当前接口”功能快速定位到当前正在编辑的接口所在目录。这对当前项目接口或者目录比较多时，非常有用

![image-20220423230530304](https://img-note.langyastudio.com/202204232305439.png?x-oss-process=style/watermark)



### 统一指定新建接口的method和请求方式

系统默认新建接口的 method 为 POST，请求方式是 form-data。我们可以通过在 **菜单-设置** 里配置以下两项，来指定新建接口的默认method 等参数

![image-20220423230837704](https://img-note.langyastudio.com/202204232308849.png?x-oss-process=style/watermark)



### 克隆接口/文档

对于一个项目的大部分接口，其请求参数和响应参数有很多雷同之处。我们可以通过克隆功能，快速的新建一个接口，并集成已有接口的已填参数。

![image-20220423231016582](https://img-note.langyastudio.com/202204232310721.png?x-oss-process=style/watermark)



## 总结

除了上面总结的一些常用功能，ApiPost 还支持其他一些功能，比如：

- 接口回收站
- 自动生成代码

- 多人协作管理
- 项目管理

- Cookie管理器
- 预执行脚本和后执行脚本

- .......

整体上来看 ApiPost 无论是功能种类方面还是用户体验方面都比之前用的其他软件更具优势，相当于同时把 Postman、Mock、Swagger 的功能压缩为一个开发辅助软件，前端、后端、测试同时编辑，内容实时同步，且完全免费提供给用户使用！不得不说，ApiPost 相比于 Postman ，它是一款更懂中国程序员的研发协同工具。





