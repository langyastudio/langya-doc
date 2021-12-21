## 配置微信公众号

### 注册微信公众平台

https://mp.weixin.qq.com

注意： 

主体类型选择服务号 （微信认证的主体类型不支持个人https://kf.qq.com/faq/120911VrYVrA141021aUZJzY.html）



### 微信认证

按照官方信息提示，完成认证操作



### 公众号设置

在【设置 -> 公众号设置】的功能设置标签页

​            ![img](https://img-note.langyastudio.com/202112011706547.png?x-oss-process=style/watermark)            

- 业务域名

配置为用户中心的 UI 域名地址

效果是在微信内访问该域名下页面时，不会被重新排版。用户在该域名上进行输入时，不出现安全提示。

- JS 接口安全域名

配置为需要使用微信分享功能的 UI 域名地址

- 网页授权域名

配置为用户中心的 API 域名地址，用于微信扫描登录



### 开发基本配置

在【开发-> 基本配置】的公众号开发信息            ![img](https://img-note.langyastudio.com/202112011706672.png?x-oss-process=style/watermark)            

- 获取开发者ID、开发者密码
- IP 白名单

设置为用户中心的公网 IP 地址以及其他使用微信公众号的产品的公网 IP 地址



## 平台配置

### SSO 配置

- 在忽略的 IP 地址中增加 [open.weixin.qq.com](http://open.weixin.qq.com)
- 设置 SSO 的 server 为用户中心 API 的访问域名地址

​            ![img](https://img-note.langyastudio.com/202112021011935.png?x-oss-process=style/watermark)            



### 微信配置

在 openlogin 的配置中，设置微信的 appid 与 appsecret （即开发基本配置中获取到的开发者ID、开发者密码）。

​            ![img](https://img-note.langyastudio.com/202112021011889.png?x-oss-process=style/watermark)            



## 常见错误

### 错误码 10003

安全域名校验错误，可能原因有：

- 公众号未配置回调域名
- 公众号未配置白名单
- 平台未配置微信appid与appsecret密钥
- 平台返回的扫码地址的回调域名与公众号不一致
