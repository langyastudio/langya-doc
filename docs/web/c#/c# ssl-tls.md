> 用最少的代码解决 HTTPS 报告 SSL 的 bug

##### 请求被中止: 未能创建 SSL/TLS 安全通道
```c#
ServicePointManager.SecurityProtocol = SecurityProtocolType.Tls11;
```
> SecurityProtocolType.Tls11 可以根据实际情况换成别的版本协议



##### 基础连接已经关闭: 未能为 SSL/TLS 安全通道建立信任关系

```c#
 //Trust all certificates
 System.Net.ServicePointManager.ServerCertificateValidationCallback =
     ((sender, certificate, chain, sslPolicyErrors) => true);
```

> 解决问题，但会涉及客户端安全性问题。最好还是使用SSL证书

