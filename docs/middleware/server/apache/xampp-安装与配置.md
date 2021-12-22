### 安装XAMPP

   进入 https://www.apachefriends.org/zh_cn/index.html 页面下载XAMPP

![img](https://img-note.langyastudio.com/20210706152514.jpeg?x-oss-process=style/watermark)

![img](https://img-note.langyastudio.com/20210706152522.jpeg?x-oss-process=style/watermark)

![img](https://img-note.langyastudio.com/20210706152528.jpeg?x-oss-process=style/watermark)



### XAMPP Control Panel

   在XAMPP控制面板中，我们可以看到Service一列的单选框略有不同

- “X”表示相应组件还没有设为Windows系统服务

- “空白”表示没有安装该组件

- 此外还有“√”，表示该组件已经安装成为Windows系统服务，可以start

![img](https://img-note.langyastudio.com/20210706152836.jpeg?x-oss-process=style/watermark)



### 修改Apache的端口号

   ../apache/conf/**http.conf** 文件把 http 端口 80 修改为 82

![img](https://img-note.langyastudio.com/20210706152843.jpeg?x-oss-process=style/watermark)

![img](https://img-note.langyastudio.com/20210706153752.jpeg?x-oss-process=style/watermark)



   ../apache/conf/**httpd-ssl.conf** 文件把 https 端口 443 修改为 4433

![img](https://img-note.langyastudio.com/20210706152849.jpeg?x-oss-process=style/watermark)

![img](https://img-note.langyastudio.com/20210706152855.jpeg?x-oss-process=style/watermark)



### 安装并启动服务

   就单击Apache和MySQL前的“X”，在弹出的对话框中点击“Yes”，将它们设为系统服务。点击XAMPP控制面板上的start按钮，启动Apache服务器、MySQL服务器，Apache默认网站目录为..\xampp\**htdocs**。

![img](https://img-note.langyastudio.com/20210706152911.jpeg?x-oss-process=style/watermark)



### 修改MySQL默认密码

   因为安装xampp后的mysql默认密码为空，在浏览器地址上输入 **http://localhost:82/phpmyadmin** ，能登录到phpmyadmin。进入到数据库的控制面板，然后选择名称为mysql的数据库，如图，可从中看出 user表中，root用户的密码为空。

![img](https://img-note.langyastudio.com/20210706153012.jpeg?x-oss-process=style/watermark)



   在SQL选项中，执行以下代码，修改数据库密码：

   ```sql
   UPDATE user SET password=PASSWORD('root') WHERE user='root';
   ```

![img](https://img-note.langyastudio.com/20210706153022.jpeg?x-oss-process=style/watermark)

![img](https://img-note.langyastudio.com/20210706153028.jpeg?x-oss-process=style/watermark)

   

   修改配置文件，..\phpMyAdmin\config.inc.php 文件设置 $cfg['Servers'][$i]['password'] 为对应的密码即可。

![img](https://img-note.langyastudio.com/20210706153035.jpeg?x-oss-process=style/watermark)



### MySQL支持远程连接

将原始的 localhost 改为 % ，**重新启动电脑**！

此时使用**IP or 127.0.0.1**都可以连接数据库。

![img](https://img-note.langyastudio.com/20210706153041.png?x-oss-process=style/watermark)



### 进程文件

```
/opt/lampp/logs/httpd.pid
/opt/lampp/var/proftpd.pid
/opt/lampp/var/mysql/xx.pid
```

 

**kill: (1766) - No such process**

错误信息：

Starting MariaDB.2021-04-27 11:48:50 1766 mysqld_safe Logging to '/opt/lampp/var/mysql/116d153c0f21.err'.
2021-04-27 11:48:51 1766 mysqld_safe Starting mysqld daemon with databases from /opt/lampp/var/mysql
/opt/lampp/bin/mysql.server: line 264: kill: (1766) - No such process

> 本质原因为：Access Denied

解决方案：

```sh
sudo chmod 777 /opt/lampp/var/
sudo chown -R mysql:mysql /opt/lampp/var/mysql/
sudo lampp restart
```

