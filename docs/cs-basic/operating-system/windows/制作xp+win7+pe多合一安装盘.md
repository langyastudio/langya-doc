## **涉及的工具**

① **制作工具EASYBOOT**： EasyBoot是一款集成化的**中文启动光盘制作工具**，它可以制作光盘启动菜单、自动生成启动文件，并生成可启动ISO文件，利用其内置的刻录功能，马上就能制作出一张完全属于你自己的启动光盘。

 

**下载地址**：http://www.skycn.com/soft/9491.html#downUrlMap  

使用前一定要先输入注册码，否则不可添加菜单。

用户名：中华人民共和国  注册码：2898-5448-5603-BB2D

 

② **镜像编辑软件UltraISO**：UltraISO可以直接编辑软碟文件和从软碟中提取文件，也可以从CD-ROM直接制作ISO或者将硬盘上的文件制作成ISO文件。同时，你也可以处理ISO文件的启动信息，从而制作可引导光盘。同时，也可以非常方便的刻录光盘，出错率小于同类软件。（也就是用于**解压、编辑、刻录镜像文件**）

**下载地址：**http://www.onlinedown.net/soft/614.htm 

注册名：李明   注册码：509F-BA54-BBA6-73C5

 

③**虚拟机**：用来**测试最终制成光盘**的可用性，大家最好下载两款虚拟机，因为有时候某个引导项目在一个虚拟机中通不过，先不要下结论，认为制作失败了，不妨用另一个软件来测一下这一项，没准是虚拟机本身的问题。

 

**VirtualPC2007下载地址**： http://www.crsky.com/soft/759.html  

**VMware Workstation下载地址**： http://www.crsky.com/soft/14067.html 

 

④ **WinPE下载地址**： http://xiazai.zol.com.cn/detail/35/346673.shtml#down 

**DOS工具箱下载地址**：http://www.xdowns.com/soft/6/boot/2006/Soft_33807.html 



## **制作过程**

### **EasyBoot**

我的EasyBoot装在D盘。安装程序自动建立以下目录：

启动光盘系统文件目录为 D:\EasyBoot\disk1

启动菜单文件目录为 D:\EasyBoot\disk1\ezboot 

输出ISO文件目录为 D:\EasyBoot\iso 



### **制作背景图片**

用Photoshop或ACDSee（自己去上网找教程吧）主要强调下编辑好后把分辨率设置为800*600，然后保存图为（bmp）格式，记得选择格式！！！最后就是选择16位保存（这步很关键，不然EasyBoot打不开背景图,可以通过界面的选项中配置分辨率与像素位数），直接命名为back.bmp，这样就不用修改EasyBoot里的背景名字了。制作好后复制到D:\EasyBoot\disk1\ezboot目录下就行了，我的背景图片名Lenovo.bmp,附上我的背景图案：


![图片](https://img-note.langyastudio.com/202111151749384.jpg?x-oss-process=style/watermark)



### **制作引导文件**

**1）Win XP 文件提取和保存引导文件** 

使用 UltraISO 打开准备好的Win XP映像文件，提取除 **SETUP.EXE、AUTORUN.INF** 之外的所有文件到 \EasyBoot\disk1 目录下，也有人说只需要提取 **I386文件夹目录下所有文件、WIN51、WIN51IP、WIN51IP.SP3** 就可以了。经测试，前者提取方式制作的光盘在安装XP过程中会黑屏10秒左右，后者不会。所以尽量选择最大程度的保留原映像文件。提取完后，点击“启动”菜单—>保存**引导文件**，把Win XP 的引导文件以winxp.bif文件名（便于在Easyboot菜单制作中命令调用）保存到*\EasyBoot\disk1\ezboot 目录下。

 　

**2）Win7文件提取和保存引导文件** 　

 　参照 Win XP 文件提取的方法，继续提取 Win7 光盘映像中除 **SETUP.EXE、AUTORUN.INF** 之外的所有文件 \EasyBoot\disk1 目录下（PS：这里不需要新建文件夹存放win7系统文件，而是把所有文件直接放在ezboot目录下），接着同样保存 Win7 的**引导文件**为win7.bif 到 \EasyBoot\disk1\ezboot 目录下。

 

**3）Win PE文件提取和保存引导文件** 　

 　Win PE 提取除 **AUTO.INF** 的所有文件，**引导文件**命名为 winpe.bif ，放到相应的目录下。

 　

**4）添加DOS** 　

 　把 DOS98.img 保存到 \EasyBoot\disk1\ezboot 目录下（注意不是 disk1 目录下）
 　![图片](https://img-note.langyastudio.com/202111151749408.jpg?x-oss-process=style/watermark)



### **用Easyboot制作光盘启动菜单**

   打开D:\EasyBoot\disk1\ezboot目录下的cdmenu.ezb文件应该会提示某某错误，不管它，一直点确定，接着点选项----配置----屏幕模式选择64k色（16位），分辨率选为800*600

 

![img](https://img-note.langyastudio.com/202111151749410.gif?x-oss-process=style/watermark)
![img](https://img-note.langyastudio.com/202111151749410.gif?x-oss-process=style/watermark)
![img](https://img-note.langyastudio.com/202111151749411.gif?x-oss-process=style/watermark)
![图片](https://img-note.langyastudio.com/202111151749428.jpg?x-oss-process=style/watermark)


**（说明：安装后自动生成cdmenu.ezb样例，可在此基础上进行修改，十分方便。)**



在这里我就只发设置界面的截图，具体设置方法在Easyboot的帮助文件里讲的非常清楚，自己看就OK了。

主要说下图①文件那项背景图像要和你复制到D:\EasyBoot\disk1\ezboot目录下的图片名一致（不然打不开背景）默认为back.bmp，

 图②、图③ 没啥好说的全删掉就行，

 最重要的就是图④菜单条那项，特别说明：菜单条1到6依次对应的命令为：

run win7.bif 

run winxp.bif  

run winpe.bif 

run dos98.img、 

boot 80、 

reboot。

**不要忘了设置快捷键！！！可以把5设为缺省**

具体操作还是看Easyboot帮助文件，里面很详细 

![图片](https://img-note.langyastudio.com/202111151749086.jpg?x-oss-process=style/watermark)
![图片](https://img-note.langyastudio.com/202112011513927.jpeg?x-oss-process=style/watermark)

![图片](https://img-note.langyastudio.com/202111151749100.jpg?x-oss-process=style/watermark)



### **至此差不多就OK**

现在已经全部设置好了，**保存一下**。点制作ISO，设置好基本参数,看光盘文件目录是不是D:\EasyBoot\disk1；引导文件D:\EasyBoot\disk1\ezboot\loader.bin，



**选项里的优化光盘文件，DOS，Joliet都不要勾上，尤其是下面的允许小写字母不可以勾上**，不知道为什么我勾上这个XP安装程序就启动不了了。也是找了很久才发现问题出在这个上面。设置文件日期可以勾上，让作出的光盘更专业。CD卷标可以自己填 应该是光盘的名字。最后就点制作，3—5分钟你的系统盘镜像ISO文件就制作OK了。

![图片](https://img-note.langyastudio.com/202111151749120.jpg?x-oss-process=style/watermark)



### **测试** 

在虚拟机上进行测试，看你做的ISO文件是否能用。测试成功后就可以刻盘了。刻盘就不说了工具很多。
附上我在VirtualPC虚拟机的测试截图： 

![图片](https://img-note.langyastudio.com/202111151749889.jpg?x-oss-process=style/watermark)



**一、Win 7**

![图片](https://img-note.langyastudio.com/202111151749888.jpg?x-oss-process=style/watermark)
![图片](https://img-note.langyastudio.com/202111151749890.jpg?x-oss-process=style/watermark)
![图片](https://img-note.langyastudio.com/202111151749899.jpg?x-oss-process=style/watermark)
![图片](https://img-note.langyastudio.com/202111151749902.jpg?x-oss-process=style/watermark)



 **二、XP**

![图片](https://img-note.langyastudio.com/202111151749914.jpg?x-oss-process=style/watermark)
![图片](https://img-note.langyastudio.com/202111151749798.jpg?x-oss-process=style/watermark)

![图片](https://img-note.langyastudio.com/202111151749796.jpg?x-oss-process=style/watermark)
![图片](https://img-note.langyastudio.com/202111151749799.jpg?x-oss-process=style/watermark)



**三、WinPE**

![图片](https://img-note.langyastudio.com/202111151749809.jpg?x-oss-process=style/watermark)
![图片](https://img-note.langyastudio.com/202111151749816.jpg?x-oss-process=style/watermark)
![图片](https://img-note.langyastudio.com/202111151749820.jpg?x-oss-process=style/watermark)


**四、深山红叶DOS工具箱**

![图片](https://img-note.langyastudio.com/202111151749498.jpg?x-oss-process=style/watermark)
![图片](https://img-note.langyastudio.com/202111151749534.jpg?x-oss-process=style/watermark)



**常见问题**：
 　**1.如果\EasyBoot\disk1目录中有SETUP.EXE和AUTORUN.INF文件会怎么样** 　
 　引导失败。其中Win7 提示：cdboot：could‘t find bootmgr。XP，PE都不能引导成功。

 　**2. Win7（含SP1）和XP SP3可以直接合并么** 　
 　可以直接合并，Win7和XP只有一个“在相同目录下的”同名文件，即support\tools\目录下的gbunicn.exe。这是个语言转换工具，想来高版本替换低版本，应该问题不大。

参考：另外需要注意的是文件夹的合并问题，不同名的文件夹当然好办，大杂烩复制到一起即可。同名的文件夹需要注意一下子，如果没有同名文件存在于其中，覆盖即 可，如果有同名的文件在里面，需要留意一下这个文件名，如果是同名但功能互补的比如 .IMG 镜像，需要修改文件名，然后再启动菜单中 RUN 的时候作相应修改，如果是同功能的，则不必理会，覆盖即可。

 　**3.为什么不把Win7、XP、PE分别放到*\EasyBoot\disk1目录下的对应文件夹中** 　
 　没必要，因为 Win7 和 WinXP 几乎没有相同的文件，就一个即 support\tools\ 目录下的 gbunicn.exe 。

 　**4. EasyBoot\disk1\ezboot下的哪些文件可以删除** 　
 　我一个都没删除，大家可以尝试把用不到的引导文件删除，让光盘更“清洁”。

 　**5. Win XP不是使用w2ksect.bin来引导么** 　
 　我是用UltralISO制作的bif引导文件，其实都一样，但要注意的是不管扩展名是bin，还是bif，大小应该为 2048字节，如果不是，就该注意了。  