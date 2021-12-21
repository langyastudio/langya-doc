### PHP 配置
> 为 PHP 安装 xdebug, 方法略

配置文件 php.ini

```ini
[XDebug]
zend_extension = "/opt/lampp/lib/php/extensions/no-debug-non-zts-20131226/xdebug.so"
;开启自动跟踪
xdebug.auto_trace = 1
;开启异常跟踪
xdebug.show_exception_trace = 0
;开启远程调试自动启动
xdebug.remote_autostart = 1
;开启远程调试
xdebug.remote_enable = 1
xdebug.remote_connect_back = 1
xdebug.max_nesting_level = 250

;收集变量
xdebug.collect_vars = 1
;收集返回值
xdebug.collect_return = 1
;收集参数
xdebug.collect_params = 1
xdebug.trace_output_dir = "/opt/lampp/tmp"
xdebug.profiler_append = 0
xdebug.profiler_enable = 1
xdebug.profiler_enable_trigger = 0
xdebug.profiler_output_dir = "/opt/lampp/tmp"
xdebug.profiler_output_name = "cachegrind.out.%t-%s"

xdebug.idekey=PHPSTORM
xdebug.remote_handler = "dbgp"
xdebug.remote_host = localhost
xdebug.remote_port = 9000
```

![img](https://img-note.langyastudio.com/20210708143952.jpeg?x-oss-process=style/watermark)



### phpstorm 配置

 File>Settings>Languages&Frameworks>PHP>Servers

添加一个 Server（设置 Web 服务器的Host、Port 等信息）

![img](https://img-note.langyastudio.com/20210708144003.jpeg?x-oss-process=style/watermark)



**注意：**

要想调试远程服务器上的代码，需要使用 **use path mappings**：

![img](https://img-note.langyastudio.com/20210708144010.png?x-oss-process=style/watermark)



如果**远程服务器与本机不在同一个路由器下**，例如本机为 10.117.123.122（隶属于路由器 A），远程服务器为 192.168.123.100（隶属于路由器 B）。这时候想用 Xdebug 进行调试，就需要配置本机 IDE 所在的路由器的端口映射，将 9000 映射到本机。

![img](https://img-note.langyastudio.com/20210708144018.png?x-oss-process=style/watermark)



File>Settings>Languages&Frameworks>PHP>Debug

看到 XDebug 选项卡，port 填9000，其他默认。

![img](https://img-note.langyastudio.com/20210708144025.jpeg?x-oss-process=style/watermark)


 File>Settings>Languages&Frameworks>PHP>Debug>DBGp Proxy

IDE key 填 PhpStorm （host 填 localhost，port 填 9000）

![img](https://img-note.langyastudio.com/20210708144032.jpeg?x-oss-process=style/watermark)


 Run>Edit Configurations

添加 Web 调试服务器

![img](https://img-note.langyastudio.com/20210708144037.jpeg?x-oss-process=style/watermark)

![img](https://img-note.langyastudio.com/20210708144042.jpeg?x-oss-process=style/watermark)



**启动监听**

   在phpstorm中设置断点后，**启动监听，就是电话一样的图标**。用 chrome 浏览 localhost 中的指定断点的文件，会自动进入断点，在 phpstorm 中看到调试信息。

![img](https://img-note.langyastudio.com/20210708144049.jpeg?x-oss-process=style/watermark)

> 如果执行完上述步骤后，还不行。确认下本机的防火墙是否开放对应端口（或者直接关闭本机防火墙）

