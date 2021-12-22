## 配置钉钉开发者平台

### 注册钉钉开发者平台

https://open-dev.dingtalk.com/



### 登录配置

在应用开发菜单栏下的【移动接入应用 -> 登录】的功能设置标签页

​            ![img](https://img-note.langyastudio.com/202112011705031.png?x-oss-process=style/watermark)            

点击创建扫码登录应用授权，设置名称、描述、授权LOGO地址、回调域名等字段信息。

- 回调域名：http://XXX/api/passport/wxlogin/baseinfo

其中XXX为API的域名

创建成功后，可以获取到 appId、appSecret 信息用于第三方网站的登录

​            ![img](https://img-note.langyastudio.com/202112011705858.png?x-oss-process=style/watermark)            



### 微应用配置

如果钉钉组织架构与用户账户信息导入平台，想通过钉钉扫码后直接登录平台无需账号绑定操作，需要进行该项的配置（用于根据unionid获取userid）

在应用开发菜单栏下的【企业内部应用 -> H5微应用】的功能设置标签页，点击创建应用            ![img](https://img-note.langyastudio.com/202112011705905.png?x-oss-process=style/watermark)            

填写应用的名称、描述与图标，完成应用的创建。

点击创建好的应用，切换到【开发管理】标签页

- 服务器出口IP

设置为平台API所在服务器的外网IP地址，例如[112.232.36.44](http://112.232.36.44)

- 应用首页地址

设置为平台UI的地址

​            ![img](https://img-note.langyastudio.com/202112011705058.png?x-oss-process=style/watermark)            

切换到【权限管理】标签页，点击添加接口权限，新增通讯录只读权限并设置权限范围为全体员工

​            ![img](https://img-note.langyastudio.com/202112011705972.png?x-oss-process=style/watermark)            

切换到【凭证与基本信息】标签页，可以获取到 AppKey、AppSecret 的应用凭证信息

​            ![img](https://img-note.langyastudio.com/202112011705267.png?x-oss-process=style/watermark)            



## 平台配置

在忽略的 IP 地址中增加 [open-dev.dingtalk.com](http://open-dev.dingtalk.com)

在 openlogin 的配置中，设置钉钉的：

- corp_id 在首页菜单栏可以看到企业的Id号
- app_key、app_secret - 即H5微应用配置中获取到的AppKey、AppSecret
- client_id、client_secret - 即登录配置中获取到的appId、appSecret

​            ![img](https://img-note.langyastudio.com/202112011706477.png?x-oss-process=style/watermark)            