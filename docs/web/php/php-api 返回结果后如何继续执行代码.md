### 问题背景

在实际项目开发中，遇到一个问题：

前端通过 `Ajax` 请求后台 PHP API 接口，执行多文件的打包下载操作，该请求由于需要更新大量的数据（日志、统计等信息）到数据库且还需要执行较大的磁盘IO操作，导致该请求很耗时间。由于前端页面的更新需要快速响应，因此需要 PHP 快速返回计算结果，然后后台继续执行余下的操作。



### 解决方法

`exit()` 之后还能继续执行代码的方法有 析构函数 `__destruct()` 以及 `register_shutdown_function()` （记日志或者`xhprof`等性能分析等有一定耗时的代码），但针对 `ajax` 请求并不能立即返回。

考虑到 HTTP 请求协议中可以通过 `flush() ` 进行局部内容输出，立即返回请求结果给前端，再将耗时操作继续执行，即通过该技术解决问题。

详细代码如下：

```php
{
    $rs = ['code' => 0, 'msg' => 'ok', 'data' => true];

    ob_end_clean();
    ob_start();    
    
    //-----------------------------------------------------------------------------------
    //Windows服务器需要加上这行。
    //echo str_repeat(" ",4096);
    echo json_encode($res);//返回结果给ajax
    
    //-----------------------------------------------------------------------------------
    // get the size of the output
    $size = ob_get_length();
    
    // send headers to tell the browser to close the connection
    header("Content-Length: $size");
    header('Connection: close');
    header("HTTP/1.1 200 OK");
    header("Content-Type: application/json;charset=utf-8");
    ob_end_flush();
    if(ob_get_length())
        ob_flush();
    flush();
    
    // yii或yaf默认不会立即输出，加上此句即可（前提是用的fpm）
    if (function_exists("fastcgi_finish_request")) { 
        // 响应完成, 立即返回到前端,关闭连接
    	fastcgi_finish_request(); 
	}

     /******** background process starts here ********/
    //在关闭连接后，继续运行php脚本
     ignore_user_abort(true);    
     /******** background process ********/
    //no time limit，不设置超时时间（根据实际情况使用）
     set_time_limit(0); 
     /******** Rest of your code starts here ********/
     //继续运行的代码
     ...
     ...
```

> - 对于长时间运行的代码可以考虑使用`消息队列`方式替代 HTTP 的 `flush` 特性 --- **推荐**
> - 后台异步调用 HTTP 请求的方法可通过 `fsockopen` 实现



As of August 2012, all browsers seem to show an all-or-nothing approach to buffering. In other words, while php is operating, no content can be shown.

In particular this means that the following workarounds listed further down here are ineffective:

1) `ob_flush ()`,  `flush ()` in any combination with other output buffering functions;

2) changes to php.ini involving setting `output_buffer` and/or `zlib.output_compression` to 0 or Off;

3) setting Apache variables such as `"no-gzip"` either through `apache_setenv () `or through entries in `.htaccess`.

So, until browsers begin to show buffered content again, the tips listed here are moot.
