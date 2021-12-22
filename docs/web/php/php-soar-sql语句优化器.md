小米公司开源的 [SOAR(SQL Optimizer And Rewriter) ](https://github.com/XiaoMi/soar) 是一个对 SQL 进行优化和改写的自动化工具。 由小米人工智能与云平台的数据库团队开发与维护。



## 功能特点

- 跨平台支持（支持 Linux, Mac 环境，Windows 环境理论上也支持，不过未全面测试）
- 目前只支持 MySQL 语法族协议的 SQL 优化
- 支持基于启发式算法的语句优化
- 支持复杂查询的多列索引优化（UPDATE, INSERT, DELETE, SELECT）
- 支持 EXPLAIN 信息丰富解读
- 支持 SQL 指纹、压缩和美化
- 支持同一张表多条 ALTER 请求合并
- 支持自定义规则的 SQL 改写



## 业内其他优秀产品对比

|              | SOAR | sqlcheck | pt-query-advisor | SQL Advisor | Inception | sqlautoreview |
| ------------ | ---- | -------- | ---------------- | ----------- | --------- | ------------- |
| 启发式建议   | ✔️    | ✔️        | ✔️                | ❌           | ✔️         | ✔️             |
| 索引建议     | ✔️    | ❌        | ❌                | ✔️           | ❌         | ✔️             |
| 查询重写     | ✔️    | ❌        | ❌                | ❌           | ❌         | ❌             |
| 执行计划展示 | ✔️    | ❌        | ❌                | ❌           | ❌         | ❌             |
| Profiling    | ✔️    | ❌        | ❌                | ❌           | ❌         | ❌             |
| Trace        | ✔️    | ❌        | ❌                | ❌           | ❌         | ❌             |
| SQL在线执行  | ❌    | ❌        | ❌                | ❌           | ✔️         | ❌             |
| 数据备份     | ❌    | ❌        | ❌                | ❌           | ✔️         | ❌             |



## 安装与使用

### 下载soar

https://github.com/XiaoMi/soar/blob/master/doc/install.md

下载soar二进制安装包：

```shell
wget https://github.com/XiaoMi/soar/releases/download/${tag}/soar.${OS}-amd64 -O soar
chmod a+x soar

如`0.11.0`版本的下载方式：
wget https://github.com/XiaoMi/soar/releases/download/0.11.0/soar.linux-amd64 -O soar
# 添加可执行权限
chmod +x soar
```

> 如果下载慢，可以通过浏览器访问[https://github.com/XiaoMi/soar/releases/download/0.11.0/soar.linux-amd64](https://github.com/XiaoMi/soar/releases/download/0.11.0/soar.linux-amd64)直接下载后，再拷贝过去



### 安装扩展库

[soar-php](https://github.com/guanguans/soar-php) 是一个基于小米公司开源的 [soar](https://github.com/XiaoMi/soar) 开发的 PHP 扩展包，方便框架中 SQL 语句调优。

```shell
composer require guanguans/soar-php --dev
```



### 配置

更多详细配置请参考 [soar config](https://github.com/XiaoMi/soar/blob/master/doc/config.md)

#### 方法一、运行时初始化配置

```php
<?php

require_once __DIR__.'/vendor/autoload.php';

use Guanguans\SoarPHP\Soar;

$config = [
    // 下载的 soar 的路径
    '-soar-path' => '/Users/yaozm/Documents/wwwroot/soar-php/soar.darwin-amd64',
    // 测试环境配置
    '-test-dsn' => [
        'host' => '127.0.0.1',
        'port' => '3306',
        'dbname' => 'database',
        'username' => 'root',
        'password' => '123456',
    ],
    // 日志输出文件
    '-log-output' => './soar.log',
    // 报告输出格式: 默认  markdown [markdown, html, json]
    '-report-type' => 'html',
];
$soar = new Soar($config);
```

#### 方法二、配置文件初始化配置

`vendor` 同级目录下新建 `.soar.dist` 或者 `.soar`，内容参考 [.soar.example](https://github.com/guanguans/soar-php/blob/master/.soar.example)，例如：

```php
<?php
return [
    // 下载的 soar 的路径
    '-soar-path' => '/Users/yaozm/Documents/wwwroot/soar-php/soar.darwin-amd64',
    // 测试环境配置
    '-test-dsn' => [
        'host' => '127.0.0.1',
        'port' => '3306',
        'dbname' => 'database',
        'username' => 'root',
        'password' => '123456',
    ],
    // 日志输出文件
    '-log-output' => './soar.log',
    // 报告输出格式: 默认  markdown [markdown, html, json]
    '-report-type' => 'html',
];
```

> 运行时初始化配置 > .soar > .soar.dist

然后初始化：

```php
<?php

require_once __DIR__.'/vendor/autoload.php';
use Guanguans\SoarPHP\Soar;
$soar = new Soar();
```



### 测试

#### SQL 评分

方法调用：

```php
$sql ="SELECT * FROM `fa_user` `user` LEFT JOIN `fa_user_group` `group` ON `user`.`group_id`=`group`.`id`;";
echo $soar->score($sql);
```



输出结果：

![](https://img-note.langyastudio.com/20201120153657.png?x-oss-process=style/watermark)

#### Explain 信息解读

方法调用：

```php
$sql = "SELECT * FROM `fa_auth_group_access` `aga` LEFT JOIN `fa_auth_group` `ag` ON `aga`.`group_id`=`ag`.`id`;";
// 输出 html 格式
echo $soar->htmlExplain($sql);
// 输出 md 格式
echo $soar->mdExplain($sql);
// 输出 html 格式
echo $soar->explain($sql, 'html');
// 输出 md 格式
echo $soar->explain($sql, 'md');
```



> 更多参考：https://github.com/guanguans/soar-php



















