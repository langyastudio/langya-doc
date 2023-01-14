## 客户端

### 客户端安装

- 安装包和授权文件下载

前往人大金仓官网（https://bbs.kingbase.com.cn/）进行下载

![window单个](https://img-note.langyastudio.com/202212212022310.png?x-oss-process=style/watermark)

- 安装包下载为 ISO 文件，进行解压

解压后内部包含 KingbaseES 安装文件 和 辅助文件压缩包，先将压缩包解压，**先安装辅助文件的内部程序**

- 安装客户端

安装 KingbaseES ，选择**客户端**即可



### DBeaver 连接

> 注意国产数据库 DBeaver 是没有集成和下载数据库驱动的，可以自行设置

- 下载驱动

驱动下载地址：https://www.kingbase.com.cn/qd/index.htm

注意需要获取不同数据库的驱动 jar 包，建议使用最新的 jar 包进行处理，jar 包建议通过数据库厂商的官网来获取

![image-20221221090914191](https://img-note.langyastudio.com/202212210909888.png?x-oss-process=style/watermark)



- 新建驱动

工具栏 -> 数据库 -> 驱动管理器，新建一个新的驱动，名字尽量好记，避免自己下次新增的时候找不到。一般需要的是类名、URL 模板(需要变量替换) 

```
驱动名:        KINGBASE
类名:          com.kingbase8.Driver
URL模板:       jdbc:kingbase8://{host}:{port}/{database}
端口:          54321
默认数据库:     SAMPLES
```

![image-20221221091632811](https://img-note.langyastudio.com/202212210916889.png?x-oss-process=style/watermark)

- 添加下载的**驱动文件**

![image-20221221091752784](https://img-note.langyastudio.com/202212210917832.png?x-oss-process=style/watermark)

