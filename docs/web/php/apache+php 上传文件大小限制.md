在用 PHP + Apache 进行文件上传的操作中，需要知道怎么控制上传文件大小的设置，而文件可传大小是受到多种因素制约的，现总结如下：

### PHP.ini 配置

- upload_max_filesize 
所上传的文件的最大大小，默认值 2M。

- memory_limit 
本指令设定了一个脚本所能够申请到的最大内存字节数，默认值 8M。如果不需要任何内存上的限制，必须将其设为 -1。如果内存不够，则可能出现错误：`Fatal error: Allowed memory size of X bytes exhausted (tried to allocate Y bytes)`
> 一般导入数据库时，**如果数据库太大，就会报错**，改这个就可以

- post_max_size 
设定 POST 请求数据所允许的最大大小。此设定也影响到文件上传。要上传大文件，该值必须大于 `upload_max_filesize`
- max_execution_time = 30 
Maximum execution time of each script, in seconds
- max_input_time = 60 
Maximum amount of time each script may spend parsing request data

> 如果用到 mysql 的 BLOB 进行二进制文件存储，则需要设置 `my.ini:max_allowed_packet=xxM`



### http.conf 配置

在 Apache 里面有一个选项是 **LimitRequestBody**， 这个选项可以限制用户送出的 HTTP 请求内容。这个选项可以在 .htaccess 或 httpd.conf 里使用，而如果在 httpd.conf 内使用，分别可以用在 virtualhost 或目录属性设定。

`LimitRequestBody` 的设定值是介乎 **0 (无限制) 至 2147483647 (2GB)**。



例如设定上传限制为 100K，可以在 .htaccess 或 httpd.conf 加入以下语句：

```
LimitRequestBody 1024000000
Options FollowSymLinks MultiViews ExecCGI
AllowOverride All
Order allow,deny
Allow from all
```

Apache 服务器从客户端接收长度不超过 `LimitRequestBody` 字节数的请求，然后传送给 php 模块，php 模块再决定是否保存成临时文件，设置 `$_FILES` 全局变量，移交给 script 进一步处理。



### http 协议

html 本身能够 post 数据也是有限制的，不能超过2G。

> FTP 客户端有文件偏移指针的 2GB 边界限制，**未使用特殊编译flag** 编译的 ftp 服务器端或者客户端，无论在什么 FS中都不支持大于2GB的文件