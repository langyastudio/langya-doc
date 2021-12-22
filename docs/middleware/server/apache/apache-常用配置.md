## 多端口

**修改httpd.conf文件**
 /opt/lampp/etc/httpd.conf

增加端口：

```
Listen 8888
```



 开放vhost配置文件：

```plain
 Include etc/extra/httpd-vhosts.conf
```



 屏蔽访问限制：

```plain
 #<Directory />
 #    AllowOverride none
 #    Require all denied
 #</Directory>
```



**修改httpd-vhosts.conf文件**

 /opt/lampp/etc/extra/httpd-vhosts.conf

```plain
<VirtualHost *:8888>
    ServerName localhost
    DocumentRoot "/var/www/Phraseanet/www"
   <Directory "/var/www/Phraseanet/www">
        DirectoryIndex index.php
        Options Indexes FollowSymLinks Includes ExecCGI
        AllowOverride All
        Order allow,deny
        Allow from all
   </Directory>     
 </VirtualHost>
```

> 以上步骤即可完成多端口的配置操作



## 域名访问

**二级域名**

```
<VirtualHost *:80>
   ServerName test.baidu.com  
   DocumentRoot "/var/www/test"
   <Directory "/var/www/test">
       DirectoryIndex index.php
       Options FollowSymLinks Includes ExecCGI
       AllowOverride All
       Order allow,deny
       Allow from all
   </Directory>
</VirtualHost>
```



**三级域名**

```
<VirtualHost *:80>
   ServerName pm.baidu.com
   ServerAlias cloud.pm.baidu.com
   DocumentRoot "/var/www/pm_cloud"
   <Directory "/var/www/pm_cloud">
       DirectoryIndex index.html
       Options FollowSymLinks Includes ExecCGI
       AllowOverride All
       Order allow,deny
       Allow from all
   </Directory>  
</VirtualHost>
```

> 需要开放80端口



## basic访问

 **htpasswd**

```
<VirtualHost *:80>
   ServerName auth3.baidu.com
   DocumentRoot "/var/www/hacfin_auth"
   <Directory "/var/www/hacfin_auth">
       DirectoryIndex index.php
       Options FollowSymLinks Includes ExecCGI
       AllowOverride All
       Order allow,deny
       Allow from all

       AuthUserFile /var/www/hacfin_auth/.htpasswd
       AuthType Basic
       AuthName "restricted"
       Require valid-user
   </Directory>
</VirtualHost>
```



## ssl

**启用配置文件**

通常在 apache 的 conf 文件夹下的 httpd.conf，将前面的 # 去掉，解除注释

```
LoadModule ssl_module modules/mod_ssl.so
Include conf/extra/httpd-ssl.conf
```



**修改配置文件**

修改 httpd-ssl.conf 文件（通常在 apache 的 conf 下的 extra 文件夹中）加上自己的配置

```
<VirtualHost _default_:43001>
	ServerName 域名:43001

	SSLEngine on
	SSLCertificateFile "conf/ssl.crt/server.crt"
	SSLCertificateKeyFile "conf/ssl.key/server.key"
	#SSLCertificateChainFile 
	
	DocumentRoot "E:\Code\RefCode\webrtc"
	<Directory "E:\Code\RefCode\webrtc">
	    DirectoryIndex index.html
	    Options FollowSymLinks Includes ExecCGI
	    AllowOverride All
	    Order allow,deny
	    Allow from all
	</Directory>
</VirtualHost>
```



**设置 HTTP 请求自动跳转 HTTPS**

```
<VirtualHost *:80>
ServerName 域名
DocumentRoot "E:\Code\RefCode\webrtc"
RewriteEngine On
RewriteRule ^/(.*)$ https://域名/$1 [R=301,L]
</VirtualHost>
```

或者

```
RewriteEngine on
RewriteCond %{SERVER_PORT} !^443$
RewriteRule ^(.*)$ https://%{SERVER_NAME}$1 [L,R]
```



**curl 证书报错**

```php
curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, false);
```
