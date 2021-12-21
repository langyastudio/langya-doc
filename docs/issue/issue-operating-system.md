## Linux

### 坏的解释器

执行 shell 脚本时报错：`/bin/bash^M: 坏的解释器: 没有那个文件或目录`

是因为该文件在 windows 系统上打开过，关闭后其中的 `换行符号` 和 Linux 的不同，导致这个报错，我们可以通过 `sed 命令` 与正则的配合将文件中的 `换行符号` 替换成 linux 的形式：

```shell
sed -i 's/\r$//' mocha.sh
```



### U盘安装 no caching mode page found

#### 错误描述

```shell
[sdb] No Caching mode page found
[sdb] Assuming drive cache:write through
....
Could not boot
/dev/root does not exist
```



#### 进入安装界面时，按下 e 键( 或 Tab )

将 `vmlinuz initrd=initrd.img inst.stage2=hd:LABEL=CentOS\x207\x20x86_64 rd.live.check quiet`
更改为（ 即更改 inst.stage2=hd: 后面的内容即可 ）
`vmlinuz initrd=initrd.img inst.stage2=hd:/dev/sdb4:/ quiet`

再按下 CTRL + X ( 或 Enter )快捷键即可继续安装了。



说明：
sdb4 是你 U 盘的挂载名称，可以通过 `ls /dev/sd*` 获取。

一般是 sdb1 , 因为硬盘一般是 sda , 可能会有不同，具体看情况。
我安装过程中，就遇到过： `sdb4、sda4`



### Failed to load ldlinux.c32

利用 UltraISO 制作了 CentOS 的 U 盘启动，开机 USB 启动时出现

```
Failed to load ldlinux.c32 Boot failed: 
please change disks and press a key to continue
```



**解决方法：**

写入方式 RAW

![543473-20170307201820141-1388315308](https://img-note.langyastudio.com/202110310715227.png?x-oss-process=style/watermark)



### 主机不能通过域名访问自己

﻿因各种原因造成不能通过自己的域名或外网 IP 访问自己，例如域名未映射IP、端口未开放、主机防火墙等。

都没问题的前提下，可以通过修改  `/etc/hosts` 文件解决
在最下面添加域名与本机的映射，例如：

```shell
127.0.0.1 www.yourdomian.com
```

这样主机通过 `www.yourdomian.com` 访问自己时，实际上就会换成 `127.0.0.1` 的本机进行访问

> 当域名映射IP正确后，仍然 ping 失败，可以考虑找网络供应商解决问题，因为网络涉及局域网、城域网、广域网等



### fstab 挂载目录有误启动失败

- 在错误的启动界面处输入 root 的密码
  不会有显示的，只管输入正确的密码即可

- 然后会出现（Repair filesystem）1#的提示符，在其后面输入运行：
  `mount -no remount, rw  /`

- 编辑 /etc/fstab 的文件，删除挂载错误的挂载项



## Windows

### 护眼色

**豆沙绿**

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Themes\DefaultColors\Standard] 下Windows键值
[HKEY_CURRENT_USER\Control Panel\Colors] 下Windows键值

```
RGB    - 196 236 201
RGB 16 - C4ECC9
```



**亮度对比度**

显示屏的亮度最好调节到和周围环境亮度相似，不要比周围环境过量或过暗

工作：亮度30-50，对比度50-70
游戏&电影：亮度70-90，对比度70-90

> 例如亮度50 对比度60 清晰度52  红50 绿45 蓝40



### 企业微信双开

**修改注册表**

`计算机\HKEY_CURRENT_USER\Software\Tencent\WXWork`，将 `multi_instances` 值改为 3

![image-20211102100233477](https://img-note.langyastudio.com/202111021002588.png?x-oss-process=style/watermark)



**编写启动脚本**

保存为 `WxWork.bat`

```bash
start "" "C:\Program Files (x86)\WXWork\WXWork.exe"
start "" "C:\Program Files (x86)\WXWork\WXWork.exe"
exit
```



### 软件虚拟化与此平台上的长模式不兼容

在使用Windows7 64位操作系统时，无法运行VMWare或MS Virtual server等软件虚拟操作系统。

**错误提示**
提示为“提示：软件虚拟化与此平台上的长模式不兼容. 禁用长模式. 没有长模式支持, 虚拟机将不能运行 64 位程序... ”



**主要原因**
现在平常用的VMWare等软件本身都是基于32位的，如果要在其上运行64位虚拟机，需要把虚拟化打开！而Windows7 64位的操作系统在默认情况下是关闭的！所以开启64位CPU的VT选项，开启虚拟化，就是解决这个问题的永久性解决方案，但受到CPU型号的限制，部分CPU并不支持开启虚拟化。



**解决方法**
重启电脑，登录BIOS设置界面，进入“Configuration”菜单，找到 “Intel(R) Virtual Technology”选项，将其值改为“Enabled”，保存退出后登录系统即可(不同机型的“Intel(R) Virtual Technology”选项的位置不一样，有的在“Configuration”菜单中，有的在“Security”菜单的虚拟化的选项下)

