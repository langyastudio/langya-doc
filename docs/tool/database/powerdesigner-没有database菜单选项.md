### 问题描述
使用 PowerDesigner 16.5 版本，创建 Physical Data Modal 物理模型 PDM 后（数据库为MySQL），`双击`打开 Model Properties 物理模型属性对话框，发现没有 Database 选项，即没法创建数据库。

![这里写图片描述](https://img-note.langyastudio.com/20210707171126.png?x-oss-process=style/watermark)



### 解决方式
选择 Database ==> Edit Current DBMS... 菜单，打开 DBMS 属性修改对话框，依次选择 General ==》MySQL 5.0 ==> Script ==> Objects ==> Database ==> Enabe， 改为 YES ，保存即可。
![这里写图片描述](https://img-note.langyastudio.com/20210707171133.png?x-oss-process=style/watermark)



此时再看Model Properties ，发现有了 Database 选项，点击对应选项右侧的 创建 按钮，即可创建数据库， 如下：

![这里写图片描述](https://img-note.langyastudio.com/20210707171141.png?x-oss-process=style/watermark)






> 附图： sybase 官方文档
> [http://infocenter.sybase.com/help/index.jsp](http://infocenter.sybase.com/help/index.jsp)

![这里写图片描述](https://img-note.langyastudio.com/202111091457701.png?x-oss-process=style/watermark)