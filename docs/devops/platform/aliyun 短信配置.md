## 配置阿里云短信服务

### 开通短信服务

https://dysms.console.aliyun.com



### 签名管理

添加签名，设置签名的名称、适用场景等信息。

​            ![img](https://img-note.langyastudio.com/202112011703971.png?x-oss-process=style/watermark)            



**关于平台配置的信息为** **【签名的名称】**

审核通过后，会显示签名的名称、审核状态为通过等

​            ![img](https://img-note.langyastudio.com/202112011703464.png?x-oss-process=style/watermark)            

### 模板管理

在 【模板管理】中，添加模板，可以自定义模板的内容，样例：

​            ![img](https://img-note.langyastudio.com/202112011703893.png?x-oss-process=style/watermark)            



**关于平台配置的信息为** **【模版CODE】**

- 用户注册 校验码 （模版类型: 验证码）

校验码${code}，您正在注册成为新用户，感谢您的支持！

- 手机号找回密码 校验码（模版类型: 验证码）

校验码${code}，您正尝试通过手机号找回密码，若非本人操作，请勿泄露。

- 修改手机号 校验码（模版类型: 验证码）

校验码${code}，您正在尝试修改手机号，请妥善保管账户信息。

- 登录确认校验码（模版类型: 验证码）

校验码${code}，您正在登录，若非本人操作，请勿泄露。



### 发送频率设置

在 【国内消息设置-发送频率】 设置中，可以设置同一个签名，对同一个手机号的发送频率。

根据实际业务需求调整流控            ![img](https://img-note.langyastudio.com/202112011704038.png?x-oss-process=style/watermark)            



### 购买短信包

 使用阿里云短信，需要提前购买短信包，例如 5000 条 24 个月的套餐，购买后才能使用阿里云短信。



## 平台配置

### 短信配置

获取可以访问阿里云短信接口的密钥

https://ak-console.aliyun.com/

- access_key

可以访问阿里云短信接口的 AccessKey ID

- access_secret

可以访问阿里云短信接口的 Access Key Secret

- signName

**签名的名称**

- templateCode



**模版CODE**

其中：

01 - 用户注册

02 - 找回密码

03 - 修改手机号

​            ![img](https://img-note.langyastudio.com/202112021012509.png?x-oss-process=style/watermark)            