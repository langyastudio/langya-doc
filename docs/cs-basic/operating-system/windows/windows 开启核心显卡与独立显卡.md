采用 Mp4 视频压缩格式编码时，很耗 CPU，所以决定上显卡，进行显卡加速。选择了 Intel 核心显卡进行视频编码加速，效果很理想。但现在的问题是：在 PC 上怎样同时开启核心显卡与独立显卡。经过几番周折，终于找到了解决方案，

**目前该方案的唯一缺点是使用了屏幕扩屏，导致鼠标会飞。**



#### 环境

- NVIDIA NVS 5400M
-  Intel(R) HD Graphics 3000



#### 驱动安装

安装独立显卡与核心显卡的驱动，安装完毕后，可以在设备管理器中看到安装后的结果。

![20150714190645187](https://img-note.langyastudio.com/20210706135859.jpg?x-oss-process=style/watermark)



#### 屏幕分辨率设置

在屏幕分辨率设置中，按照以下步骤处理：

1、点击“检测”

![20150714190715312](https://img-note.langyastudio.com/20210706135935.jpg?x-oss-process=style/watermark)



2、选择 “未检测到其他显示器”，在多显示器选项中选择 “仍然尝试在以下对象上进行连接：VGA”。点击 "应用" 按钮

![20150714190735911](https://img-note.langyastudio.com/20210706140141.jpg?x-oss-process=style/watermark)

![20150714190831187](https://img-note.langyastudio.com/20210706140212.jpg?x-oss-process=style/watermark)



3、在当前显示器的多显示器选项中选择 “扩展这些显示”

![20150714190912676](https://img-note.langyastudio.com/20210706140251.jpg?x-oss-process=style/watermark)



4、点击确定并应用更改

![20150714190936384](https://img-note.langyastudio.com/20210706140324.jpg?x-oss-process=style/watermark)

![20150714190939332](https://img-note.langyastudio.com/20210706140401.jpg?x-oss-process=style/watermark)