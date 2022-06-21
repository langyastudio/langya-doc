# Navicat连接Oracle数据库报错ORA-28547或ORA-03135

Navicat Preminm 12 or  15 连接 Oracle 数据库时，报错:

`ORA-03135: connection lost contact...`

![image-20220619124305252](https://img-note.langyastudio.com/202206191243644.png?x-oss-process=style/watermark)



## 原因

Oracle Instant Client 的版本不对


## 下载驱动

- 通过如下命令，获取安装 Oracle 服务器的版本号

```sql
`select * from v$version`
```

- 确定了 Oracle 数据库的版本后，下载 Oracle Instant Client 客户端

下载网址：[https://www.oracle.com/cn/database/technologies/instant-client/downloads.html](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.oracle.com%2Fcn%2Fdatabase%2Ftechnologies%2Finstant-client%2Fdownloads.html)

选择对应操作系统的 Oracle Instant Client 客户端，如 windows

![image-20220619124716523](https://img-note.langyastudio.com/202206191247627.png?x-oss-process=style/watermark)

下载对应版本的 `InstantClient-basic`  和  `InstantClient-sqlplus`，我这里的是 Version 12.2.x

![image-20220619124936774](https://img-note.langyastudio.com/202206191249856.png?x-oss-process=style/watermark)

- 解压文件到软件安装的位置的文件夹 Instantclient_xx_x 里面，如 instantclient_12_2

![image-20220619125150907](https://img-note.langyastudio.com/202206191251965.png?x-oss-process=style/watermark)



## 配置 Navicat

打开 Navicat 客户单工具，选择 **工具 -> 选项** 菜单，在打开的对话框中选择 **环境** 标签

设置 SQL*Plus 的可执行文件 `sqlplus.exe` 与 OCI 环境的 `oci.dll` 文件为刚下载解压后的对应文件，如下图所示：

![image-20220619125920585](https://img-note.langyastudio.com/202206191259663.png?x-oss-process=style/watermark)



**配置完毕后，重启Navicat！！！**



## 连接测试

输入主机、端口、服务名、用户名与密码信息，点击连接测试，测试是否配置成功

![image-20220619130104297](https://img-note.langyastudio.com/202206191301350.png?x-oss-process=style/watermark)