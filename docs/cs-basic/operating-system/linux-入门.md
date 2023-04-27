> 本文来自JavaGuide、CS-Notes等，郎涯进行简单排版与补充



## 从认识操作系统开始

![](https://img-note.langyastudio.com/202111171117001.png?x-oss-process=style/watermark)

正式开始 Linux 之前，简单花一点点篇幅科普一下操作系统相关的内容。

### 操作系统简介

- 操作系统（Operating System，简称 OS）是**管理计算机硬件与软件资源的程序**，是计算机的基石

- 操作系统本质上是一个运行在计算机上的软件程序 ，用于管理计算机硬件和软件资源。举例：运行在你电脑上的所有应用程序都通过操作系统来调用系统内存以及磁盘等等硬件

- 操作系统存在**屏蔽了硬件层的复杂性**。操作系统就像是硬件使用的负责人，统筹着各种相关事项

- 操作系统的内核（Kernel）是操作系统的核心部分，它负责系统的内存管理，硬件设备的管理，文件系统的管理以及应用程序的管理。 **内核是连接应用程序和硬件的桥梁**，决定着系统的性能和稳定性

![Kernel_Layout](https://img-note.langyastudio.com/202111122112574.png?x-oss-process=style/watermark)





### 操作系统分类

#### Windows

目前最流行的**个人桌面操作系统** ，不做多的介绍，大家都清楚。界面简单易操作，软件生态非常好。

![windows](https://img-note.langyastudio.com/202111122119801.png?x-oss-process=style/watermark)



#### Unix

**最早的多用户、多任务操作系统** 。后面崛起的 Linux 在很多方面都参考了 Unix。

目前这款操作系统已经逐渐逐渐退出操作系统的舞台。

![unix](https://img-note.langyastudio.com/202111122119511.png?x-oss-process=style/watermark)



#### Linux

**Linux 是一套免费、开源的类 Unix 操作系统。** Linux 存在着许多不同的发行版本，但它们都使用了 **Linux 内核** 。

> 严格来讲，**Linux 这个词本身只表示 Linux 内核**，在 GNU/Linux 系统中，Linux 实际就是 Linux 内核，而该系统的其余部分主要是由 GNU 工程编写和提供的程序组成。单独的 Linux 内核并不能成为一个可以正常工作的操作系统。
>
> **很多人更倾向使用 “GNU/Linux” 一词来表达人们通常所说的 “Linux”。**

![linux](https://img-note.langyastudio.com/202111122119279.png?x-oss-process=style/watermark)



#### Mac OS

苹果自家的操作系统，编程体验和 Linux 相当，但是界面、软件生态以及用户体验各方面都要比 Linux 操作系统更好。

![macos](https://img-note.langyastudio.com/202111122119930.png?x-oss-process=style/watermark)



### 操作系统内核

我们先来看看维基百科对于内核的解释，我觉得总结的非常好！

> **内核**（英语：Kernel，又称核心）在计算机科学中是一个用来管理软件发出的数据 I/O（输入与输出）要求的电脑程序，将这些要求转译为数据处理的指令并交由中央处理器（CPU）及电脑中其他电子组件进行处理，是现代操作系统中最基本的部分。它是为众多应用程序提供对计算机硬件的安全访问的一部分软件，这种访问是有限的，并由内核决定一个程序在什么时候对某部分硬件操作多长时间。 直接对硬件操作是非常复杂的。所以内核通常提供一种硬件抽象的方法，来完成这些操作。有了这个，通过进程间通信机制及系统调用，应用进程可**间接控制所需的硬件资源**（特别是处理器及 IO 设备）。
>
> 早期计算机系统的设计中，还没有操作系统的内核这个概念。随着计算机系统的发展，操作系统内核的概念才渐渐明晰起来了!

简单概括两点：

- 操作系统的内核（Kernel）是**操作系统的核心部分**，**负责系统管理**：内存管理、硬件设备的管理、文件系统的管理以及应用程序的管理

- 操作系统的内核是**连接应用程序和硬件的桥梁**，决定着操作系统的性能和稳定性



### CPU

关于 CPU 简单概括三点：

- CPU 是一台计算机的运算核心（Core）+控制核心（ Control Unit），可以称得上是计算机的大脑

- CPU 主要包括两个部分：**控制器+运算器**

- CPU 的根本任务就是执行指令，对计算机来说最终都是一串由“0”和“1”组成的序列



### CPU vs Kernel

很多人容易无法区分操作系统的内核（Kernel）和中央处理器（CPU），你可以简单从下面两点来区别：

- 操作系统的内核（Kernel）属于操作系统层面，而 **CPU 属于硬件**

- **CPU 主要提供运算**，处理各种指令的能力。内核（Kernel）主要负责系统管理比如内存管理，它屏蔽了对硬件的操作

下图清晰说明了应用程序、内核、CPU 这三者的关系。

![Kernel_Layout](https://img-note.langyastudio.com/202111122119185.png?x-oss-process=style/watermark)



### 系统调用

**用户态和系统态**

根据进程访问资源的特点，我们可以把进程在系统上的运行分为两个级别：

- 用户态(user mode) 

  用户态运行的进程可以直接读取用户程序的数据

- 系统态(kernel mode)

  系统态运行的进程或程序几乎可以访问计算机的任何资源，不受限制



**系统调用**

我们运行的程序基本都是运行在用户态，如果我们调用操作系统提供的系统态级别的子功能咋办呢？那就需要系统调用了！也就是说在我们运行的用户程序中，凡是**与系统态级别的资源有关的操作**（如文件管理、进程控制、内存管理等)，都必须通过系统调用方式向操作系统提出服务请求，并由操作系统代为完成。

这些系统调用按功能大致可分为如下几类：

- 设备管理

  完成设备的请求或释放，以及设备启动等功能

- 文件管理

  完成文件的读、写、创建及删除等功能

- 进程控制

  完成进程的创建、撤销、阻塞及唤醒等功能

- 进程通信

  完成进程之间的消息传递或信号传递等功能

- 内存管理

  完成内存的分配、回收以及获取作业占用内存区大小及地址等功能

![](https://img-note.langyastudio.com/202111171117847.jpeg?x-oss-process=style/watermark)



## 初探 Linux

### Linux 简介

- **类 Unix 系统**

  Linux 是一种自由、开放源码的类似 Unix 的操作系统

- Linux 本质是指 **Linux 内核**

  严格来讲，Linux 这个词本身只表示 Linux 内核，单独的 Linux 内核并不能成为一个可以正常工作的操作系统。所以，就有了各种 Linux 发行版。

- Linux 之父(林纳斯·本纳第克特·托瓦兹 Linus Benedict Torvalds)

  一个编程领域的传奇式人物，真大佬！我辈崇拜敬仰之楷模。他是 **Linux 内核** 的最早作者，随后发起了这个开源项目，担任 Linux 内核的首要架构师。他还发起了 Git 这个开源项目，并为主要的开发者。

![Linux](https://img-note.langyastudio.com/202111122120801.png?x-oss-process=style/watermark)



### Linux 诞生

1989 年，Linus Torvalds 进入芬兰陆军新地区旅，服 11 个月的国家义务兵役，军衔为少尉，主要服务于计算机部门，任务是弹道计算。服役期间，购买了安德鲁·斯图尔特·塔能鲍姆所著的教科书及 minix 源代码，开始研究操作系统。1990 年，他退伍后回到大学，开始接触 Unix。

> **Minix** 是一个迷你版本的类 Unix 操作系统，由塔能鲍姆教授为了教学之用而创作，采用微核心设计。它启发了 Linux 内核的创作。

1991 年，Linus Torvalds 开源了 Linux 内核。Linux 以一只可爱的企鹅作为标志，象征着敢作敢为、热爱生活。

![OPINION: Make the switch to a Linux operating system | Opinion ...](https://img-note.langyastudio.com/202111122119144.png?x-oss-process=style/watermark)

### Linux 发行版本

Linus Torvalds 开源的只是 Linux 内核，我们上面也提到了操作系统内核的作用。一些组织或厂商将 Linux 内核与各种软件和文档包装起来，并提供系统安装界面和系统配置、设定与管理工具，就构成了 Linux 的发行版本。

| 基于的包管理工具 | 商业发行版 |   社区发行版    |
| :--------------: | :--------: | :-------------: |
|       RPM        |  Red Hat   | Fedora / CentOS |
|       DPKG       |   Ubuntu   |     Debian      |

对于初学者学习 Linux ，推荐选择 CentOS 



## Linux 磁盘

### 磁盘接口

**IDE**

IDE（ATA）全称 Advanced Technology Attachment，接口速度最大为 133MB/s，因为并口线的**抗干扰性太差**，且排线占用空间较大，不利电脑内部散热，已逐渐被 SATA 所取代。

![](https://img-note.langyastudio.com/202111171348729.jpeg?x-oss-process=style/watermark)



**SATA**

SATA 全称 Serial ATA，也就是使用串口的 ATA 接口，**抗干扰性强**，且对数据线的长度要求比 ATA 低很多，支持热插拔等功能。SATA-II 的接口速度为 300MB/s，而 SATA-III 标准可达到 600MB/s 的传输速度。SATA 的数据线也比 ATA 的细得多，有利于机箱内的空气流通，整理线材也比较方便。

![](https://img-note.langyastudio.com/202111171348001.jpeg?x-oss-process=style/watermark)



**SCSI**

SCSI 全称是 Small Computer System Interface（小型机系统接口），SCSI 硬盘广为工作站以及个人电脑以及服务器所使用，因此会使用较为先进的技术，如碟片转速 15000rpm 的高转速，且传输时 CPU 占用率较低，但是单价也比相同容量的 ATA 及 SATA 硬盘更加昂贵。

![](https://img-note.langyastudio.com/202111171347936.jpeg?x-oss-process=style/watermark)

**SAS**

SAS（Serial Attached SCSI）是新一代的 SCSI 技术，和 SATA 硬盘相同，都是采取序列式技术以获得更高的传输速度，可达到 **6Gb/s**。此外也通过缩小连接线改善系统内部空间等。

![](https://img-note.langyastudio.com/202111171347743.jpeg?x-oss-process=style/watermark)



### 磁盘的文件名

Linux 中每个硬件都被当做一个文件，包括磁盘。磁盘以磁盘接口类型进行命名，常见磁盘的文件名如下：

- IDE 磁盘

  /dev/hd[a-d]

- SATA/SCSI/SAS 

  /dev/sd[a-p]

其中文件名后面的序号的确定与系统检测到磁盘的顺序有关，而与磁盘所插入的插槽位置无关



## 磁盘分区

### 分区表

磁盘分区表主要有两种格式，一种是**限制较多的 MBR** 分区表，一种是较新且限制较少的 GPT 分区表。

**MBR 不支持 2.2 TB 以上的硬盘，GPT 则最多支持到 2<sup>33</sup> TB = 8 ZB。**



**MBR**

MBR 中，第一个扇区最重要，里面有主要开机记录（Master boot record, MBR）及分区表（partition table），其中主要开机记录占 446 bytes，分区表占 64 bytes。

分区表只有 64 bytes，最多只能存储 4 个分区，这 4 个分区为主分区（Primary）和扩展分区（Extended）。其中扩展分区只有一个，它使用其它扇区来记录额外的分区表，因此通过扩展分区可以分出更多分区，这些分区称为逻辑分区。

Linux 也把分区当成文件，分区文件的命名方式为：磁盘文件名 + 编号，例如 /dev/sda1。注意，逻辑分区的编号从 5 开始。

**GPT**

扇区是磁盘的最小存储单位，旧磁盘的扇区大小通常为 512 bytes，而最新的磁盘支持 4 k。GPT 为了兼容所有磁盘，在定义扇区上使用逻辑区块地址（Logical Block Address, LBA），LBA 默认大小为 512 bytes。

GPT 第 1 个区块记录了主要开机记录（MBR），紧接着是 33 个区块记录分区信息，并把最后的 33 个区块用于对分区信息进行备份。这 33 个区块第一个为 GPT 表头纪录，这个部份纪录了分区表本身的位置与大小和备份分区的位置，同时放置了分区表的校验码 (CRC32)，操作系统可以根据这个校验码来判断 GPT 是否正确。若有错误，可以使用备份分区进行恢复。

GPT 没有扩展分区概念，都是主分区，每个 LBA 可以分 4 个分区，因此总共可以分 4 * 32 = 128 个分区。

![](https://img-note.langyastudio.com/202111171347639.png?x-oss-process=style/watermark)



### 开机检测程序

#### **BIOS**

BIOS（Basic Input/Output System，基本输入输出系统），它是一个固件（**嵌入在硬件中的软件**），BIOS 程序存放在断电后内容不会丢失的只读内存中。

![](https://img-note.langyastudio.com/202111171346811.jpeg?x-oss-process=style/watermark)



**BIOS 是开机的时候计算机执行的第一个程序**，这个程序知道可以开机的磁盘，并读取磁盘第一个扇区的主要开机记录（MBR），由主要开机记录（MBR）执行其中的开机管理程序，这个开机管理程序会加载操作系统的核心文件。

主要开机记录（MBR）中的开机管理程序提供以下功能：选单、载入核心文件以及转交其它开机管理程序。转交这个功能可以用来实现多重引导，只需要将另一个操作系统的开机管理程序安装在其它分区的启动扇区上，在启动开机管理程序时，就可以通过选单选择启动当前的操作系统或者转交给其它开机管理程序从而启动另一个操作系统。

下图中，第一扇区的主要开机记录（MBR）中的开机管理程序提供了两个选单：M1、M2，M1 指向了 Windows 操作系统，而 M2 指向其它分区的启动扇区，里面包含了另外一个开机管理程序，提供了一个指向 Linux 的选单。

![](https://img-note.langyastudio.com/202111171346666.jpeg?x-oss-process=style/watermark)

**安装多重引导，最好先安装 Windows 再安装 Linux**。因为安装 Windows 时会覆盖掉主要开机记录（MBR），而 Linux 可以选择将开机管理程序安装在主要开机记录（MBR）或者其它分区的启动扇区，并且可以设置开机管理程序的选单。



#### **UEFI**

**BIOS 不可以读取 GPT 分区表**，而 UEFI 可以。



## Linux 文件系统

### 文件系统简介

在 Linux 操作系统中，所有被操作系统管理的资源，例如网络接口卡、磁盘驱动器、打印机、输入输出设备、普通文件或是目录都被看作是一个文件。 也就是说在 Linux 系统中有一个重要的概念：**一切资源都是文件**。

其实这是 UNIX 哲学的一个体现，在 UNIX 系统中，把一切资源都看作是文件，Linux 的文件系统也是借鉴 UNIX 文件系统而来。



### 文件组成

最主要的几个组成部分如下：

- inode：一个文件占用一个 inode，记录文件的属性，同时记录此文件的内容所在的 block 编号
- block：记录文件的内容，文件太大时，会占用多个 block

除此之外还包括：

- superblock：记录文件系统的整体信息，包括 inode 和 block 的总量、使用量、剩余量，以及文件系统的格式与相关信息等
- block bitmap：记录 block 是否被使用的位图

![](https://img-note.langyastudio.com/202111171345024.png?x-oss-process=style/watermark)



### 文件读取

对于 Ext2 文件系统，当要读取一个文件的内容时，先在 inode 中查找文件内容所在的所有 block，然后把所有 block 的内容读出来。

![](https://img-note.langyastudio.com/202111171345833.png?x-oss-process=style/watermark)



而对于 FAT 文件系统，它没有 inode，每个 block 中存储着下一个 block 的编号。

![](https://img-note.langyastudio.com/202111171345928.png?x-oss-process=style/watermark)



### block

硬盘的最小存储单位是**扇区(Sector)**，**块(block)**由多个扇区组成。文件数据存储在块中。块的最常见的大小是 4kb，约为 8 个连续的扇区组成（每个扇区存储 512 字节）。

在 Ext2 文件系统中所支持的 block 大小有 1K，2K 及 4K 三种，不同的大小限制了单个文件和文件系统的最大大小。

|     大小     | 1KB  |  2KB  | 4KB  |
| :----------: | :--: | :---: | :--: |
| 最大单一文件 | 16GB | 256GB | 2TB  |
| 最大文件系统 | 2TB  |  8TB  | 16TB |

一个 block 只能被一个文件所使用，未使用的部分直接浪费了。因此如果需要存储大量的小文件，那么最好选用比较小的 block。



### inode

**inode 就是用来维护某个文件基本属性**。inode 具有以下特点：

- 每个 inode 大小均固定为 128 bytes (新的 ext4 与 xfs 可设定到 256 bytes)

- 每个文件都仅会占用一个 inode

  

inode 具体包含以下信息：

- 权限 (read/write/excute)
- 拥有者与群组 (owner/group)
- 容量
- 建立或状态改变的时间 (ctime)
- 最近读取时间 (atime)
- 最近修改时间 (mtime)
- 定义文件特性的旗标 (flag)，如 SetUID...
- 该文件真正内容的指向 (pointer)



inode 中记录了文件内容所在的 block 编号，但是每个 block 非常小，一个大文件随便都需要几十万的 block。而一个 inode 大小有限，无法直接引用这么多 block 编号。因此引入了**间接、双间接、三间接引用**。间接引用让 inode 记录的引用 block 块记录引用信息。

![](https://img-note.langyastudio.com/202111171344442.jpeg?x-oss-process=style/watermark)



### 文件类型

Linux 支持很多文件类型，其中非常重要的文件类型有: **普通文件**，**目录文件**，**链接文件**，**设备文件**，**管道文件**，**Socket 套接字文件**等。

- 普通文件**（-）** 

  用于存储信息和数据， Linux 用户可以根据访问权限对普通文件进行查看、更改和删除。比如：图片、声音、PDF、text、视频、源代码等等

- 目录文件**（d，directory file）** 

  目录也是文件的一种，用于表示和管理系统中的文件，目录文件中包含一些文件名和子目录名。打开目录事实上就是打开目录文件

- 符号链接文件**（l，symbolic link）** 

  保留了指向文件的地址而不是文件本身

- 字符设备**（c，char）** 

  用来访问字符设备比如键盘

- 设备文件**（b，block）** 

  用来访问块设备比如硬盘、软盘

- 管道文件**(p，pipe)** 

  一种特殊类型的文件，用于进程之间的通信

- 套接字**(s，socket)** 

  用于进程间的网络通信，也可以用于本机之间的非网络通信



### 文件属性

用户分为三种：文件拥有者、群组以及其它人，对不同的用户有不同的文件权限。

使用 ls 查看一个文件时，会显示一个文件的信息，例如 `drwxr-xr-x 3 root root 17 May 6 00:14 .config`，对这个信息的解释如下：

- drwxr-xr-x：文件类型以及权限，第 1 位为文件类型字段，后 9 位为文件权限字段
- 3：链接数
- root：文件拥有者
- root：所属群组
- 17：文件大小
- May 6 00:14：文件最后被修改的时间
- .config：文件名



9 位的文件权限字段中，每 3 个为一组，共 3 组，每一组分别代表对文件拥有者、所属群组以及其它人的文件权限。一组权限中的 3 位分别为 r、w、x 权限，表示可读、可写、可执行。

文件时间有以下三种：

- modification time (mtime)：文件的内容更新就会更新
- status time (ctime)：文件的状态（权限、属性）更新就会更新
- access time (atime)：读取文件时就会更新



### 目录树

所有可操作的计算机资源都存在于目录树这个结构中，对计算资源的访问，可以看做是对这棵目录树的访问。

**Linux 的目录结构如下：**

Linux 文件系统的结构层次鲜明，就像一棵倒立的树，最顶层是其根目录：
![Linux的目录结构](https://img-note.langyastudio.com/202111122119988.png?x-oss-process=style/watermark)

**常见目录说明：**

- **/bin：** 存放二进制可执行文件(ls、cat、mkdir 等)，常用命令一般都在这里
- **/etc：** 存放系统管理和配置文件
- **/home：** 存放所有用户文件的根目录，是用户主目录的基点，比如用户 user 的主目录就是/home/user，可以用~user 表示
- **/usr ：** 用于存放系统应用程序
- **/opt：** 额外安装的可选应用程序包所放置的位置。一般情况下，我们可以把 tomcat 等都安装到这里
- **/proc：** **虚拟文件系统目录**，是系统内存的映射。可直接访问这个目录来获取系统信息
- **/root：** 超级用户（系统管理员）的主目录（特权阶级^o^）
- **/sbin:**  存放二进制可执行文件，**只有 root 才能访问**。这里存放的是系统管理员使用的系统级别的管理命令和程序。如 ifconfig 等
- **/dev：** 用于存放设备文件
- **/mnt：** 系统管理员安装临时文件系统的安装点，系统提供这个目录是让用户临时挂载其他的文件系统
- **/boot：** 存放用于系统引导时使用的各种文件
- **/lib ：** 存放着和系统运行相关的库文件 
- **/tmp：** 用于存放各种临时文件，是公用的临时文件存储点
- **/var：** 用于**存放运行时需要改变数据的文件**，也是某些大文件的溢出区，比方说各种服务的日志文件（系统启动日志等。）等
- **/lost+found：** 这个目录平时是空的，**系统非正常关机而留下“无家可归”的文件**（windows 下叫什么.chk）就在这里



## 常用操作

推荐一个 Linux 命令快查网站，非常不错，大家如果遗忘某些命令或者对某些命令不理解都可以在这里得到解决。Linux 命令在线速查手册：https://www.w3xue.com/manual/linux/ 。

另外，[shell.how](https://www.shell.how/) 这个网站可以用来解释常见命令的意思，对你学习 Linux 基本命令以及其他常用命令（如 Git、NPM）。

### crontab



![img](https://img-note.langyastudio.com/20210708134753.png?x-oss-process=style/watermark)



### 监控

- nload

```bash
#安装
#如果安装失败，使用:
#yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum install epel-release -y

yum install nload -y

#使用
nload -m -u m
```

- htop

```bash
#安装
yum install ncurses htop -y

#使用
htop
```



### 快捷键

- Tab：命令和文件名补全
- Ctrl+C：中断正在运行的程序
- Ctrl+D：结束键盘输入（End Of File，EOF）



### 文件目录

**ls**

列出文件或者目录的信息，目录的信息就是其中包含的文件

```shell
## ls [-aAdfFhilnrRSt] file|dir
-a ：列出全部的文件
-d ：仅列出目录本身
-l ：以长数据串行列出，包含文件的属性与权限等等数据
```

**cd**

更换当前目录

```shell
cd [相对路径或绝对路径]
```

**mkdir**

创建目录

```shell
## mkdir [-mp] 目录名称
-m ：配置目录权限
-p ：递归创建目录
```

**rmdir**

删除目录，目录必须为空

```shell
rmdir [-p] 目录名称
-p ：递归删除目录
```

**touch**

更新文件时间或者建立新文件

```shell
## touch [-acdmt] filename
-a ： 更新 atime
-c ： 更新 ctime，若该文件不存在则不建立新文件
-m ： 更新 mtime
-d ： 后面可以接更新日期而不使用当前日期，也可以使用 --date="日期或时间"
-t ： 后面可以接更新时间而不使用当前时间，格式为[YYYYMMDDhhmm]
```

**cp**

复制文件。如果源文件有两个以上，则目的文件一定要是目录才行

```shell
cp [-adfilprsu] source destination
-a ：相当于 -dr --preserve=all
-d ：若来源文件为链接文件，则复制链接文件属性而非文件本身
-i ：若目标文件已经存在时，在覆盖前会先询问
-p ：连同文件的属性一起复制过去
-r ：递归复制
-u ：destination 比 source 旧才更新 destination，或 destination 不存在的情况下才复制
--preserve=all ：除了 -p 的权限相关参数外，还加入 SELinux 的属性, links, xattr 等也复制了
```

**rm**

删除文件

```shell
## rm [-fir] 文件或目录
-r ：递归删除
```

**mv**

移动文件

```shell
## mv [-fiu] source destination
## mv [options] source1 source2 source3 .... directory
-f ： force 强制的意思，如果目标文件已经存在，不会询问而直接覆盖
```



### 文件内容

**cat**

取得文件内容

```shell
## cat [-AbEnTv] filename
-n ：打印出行号，连同空白行也会有行号，-b 不会
```

**tac**

是 cat 的反向操作，从最后一行开始打印

**more**

和 cat 不同的是它可以一页一页查看文件内容，比较适合大文件的查看

**less**

和 more 类似，但是多了一个向前翻页的功能

**head**

取得文件前几行

```shell
## head [-n number] filename
-n ：后面接数字，代表显示几行的意思
```

**tail**

是 head 的反向操作，只是取得是后几行

**od**

以字符或者十六进制的形式显示二进制文件



### 搜索

**which**

**指令**搜索

```shell
## which [-a] command
-a ：将所有指令列出，而不是只列第一个
```

**whereis**

文件搜索。速度比较快，因为它只搜索几个特定的目录

```shell
## whereis [-bmsu] dirname/filename
```

**locate**

文件搜索。可以用关键字或者正则表达式进行搜索

locate 使用 /var/lib/mlocate/ 这个数据库来进行搜索，它**存储在内存**中，并且每天更新一次，所以无法用 locate 搜索新建的文件。可以使用 updatedb 来立即更新数据库

```shell
## locate [-ir] keyword
-r：正则表达式
```

**find**

文件搜索。可以使用**文件的属性和权限**进行搜索

```shell
## find [basedir] [option]
example: find . -name "shadow*"
```

① 与时间有关的选项 

```shell
-mtime  n ：列出在 n 天前的那一天修改过内容的文件
-mtime +n ：列出在 n 天之前 (不含 n 天本身) 修改过内容的文件
-mtime -n ：列出在 n 天之内 (含 n 天本身) 修改过内容的文件
-newer file ： 列出比 file 更新的文件
```

+4、4 和 -4 的指示的时间范围如下：

![](https://img-note.langyastudio.com/202111171341884.png?x-oss-process=style/watermark)



② 与文件拥有者和所属群组有关的选项

```shell
-uid n
-gid n
-user name
-group name
-nouser ：搜索拥有者不存在 /etc/passwd 的文件
-nogroup：搜索所属群组不存在于 /etc/group 的文件
```

③ 与文件权限和名称有关的选项

```shell
-name filename
-size [+-]SIZE：搜寻比 SIZE 还要大 (+) 或小 (-) 的文件。这个 SIZE 的规格有：c: 代表 byte，k: 代表 1024bytes。所以，要找比 50KB 还要大的文件，就是 -size +50k
-type TYPE
-perm mode  ：搜索权限等于 mode 的文件
-perm -mode ：搜索权限包含 mode 的文件
-perm /mode ：搜索权限包含任一 mode 的文件
```



### 文件夹大小

- 看所有文件夹的大小

```bash
du -sh * 
```



### 修改权限

可以将一组权限用数字来表示，此时一组权限的 3 个位当做二进制数字的位，从左到右每个位的权值为 4、2、1，即每个权限对应的数字权值为 r : 4、w : 2、x : 1

```shell
## chmod [-R] xyz dirname/filename
```

示例：将 .bashrc 文件的权限修改为 -rwxr-xr--

```shell
## chmod 754 .bashrc
```



也可以使用符号来设定权限

```shell
## chmod [ugoa]  [+-=] [rwx] dirname/filename
- u：拥有者
- g：所属群组
- o：其他人
- a：所有人
- +：添加权限
- -：移除权限
- =：设定权限
```

示例：为 .bashrc 文件的所有用户添加写权限

```shell
## chmod a+w .bashrc
```



**默认权限：**

- 文件默认权限：文件默认没有可执行权限，因此为 666，也就是 -rw-rw-rw- 。
- 目录默认权限：目录必须要能够进入，也就是必须拥有可执行权限，因此为 777 ，也就是 drwxrwxrwx。

可以通过 umask 设置或者查看默认权限，通常以掩码的形式来表示，例如 002 表示其它用户的权限去除了一个 2 的权限，也就是写权限，因此建立新文件时默认的权限为 -rw-rw-r--。



**目录的权限：**

**文件名不是存储在一个文件的内容中，而是存储在一个文件所在的目录中**。因此，拥有文件的 w 权限并不能对文件名进行修改。

目录存储文件列表，一个目录的权限也就是对其文件列表的权限。因此，目录的 r 权限表示可以读取文件列表；w 权限表示可以修改文件列表，具体来说，就是添加删除文件，对文件名进行修改；x 权限可以让该目录成为工作目录，x 权限是 r 和 w 权限的基础，如果不能使一个目录成为工作目录，也就没办法读取文件列表以及对文件列表进行修改了。



### 链接

![](https://img-note.langyastudio.com/202111171342407.png?x-oss-process=style/watermark)



**实体链接:**

在目录下创建一个条目，**记录着文件名与 inode 编号**，这个 inode 就是源文件的 inode。

删除任意一个条目，文件还是存在，只要引用数量不为 0。

有以下限制：不能跨越文件系统、**不能对目录进行链接**。

```shell
## ln /etc/crontab .
## ll -i /etc/crontab crontab
34474855 -rw-r--r--. 2 root root 451 Jun 10 2014 crontab
34474855 -rw-r--r--. 2 root root 451 Jun 10 2014 /etc/crontab
```



**符号链接:**

符号链接文件保存着源文件所在的绝对路径，在读取时会定位到源文件上，可以理解为 Windows 的**快捷方式**。

当源文件被删除了，链接文件就打不开了。因为记录的是路径，所以可以为目录建立符号链接。

```shell
## ll -i /etc/crontab /root/crontab2
34474855 -rw-r--r--. 2 root root 451 Jun 10 2014 /etc/crontab
53745909 lrwxrwxrwx. 1 root root 12 Jun 23 22:31 /root/crontab2 -> /etc/crontab
```



### 压缩与打包

**压缩文件名：**

Linux 底下有很多压缩文件名，常见的如下：

| 扩展名     | 压缩程序                              |
| ---------- | ------------------------------------- |
| \*.Z       | compress                              |
| \*.zip     | zip                                   |
| \*.gz      | gzip                                  |
| \*.bz2     | bzip2                                 |
| \*.xz      | xz                                    |
| \*.tar     | tar 程序打包的数据，没有经过压缩      |
| \*.tar.gz  | tar 程序打包的文件，经过 gzip 的压缩  |
| \*.tar.bz2 | tar 程序打包的文件，经过 bzip2 的压缩 |
| \*.tar.xz  | tar 程序打包的文件，经过 xz 的压缩    |



**压缩指令：**

**gzip**

gzip 是 Linux **使用最广**的压缩指令，可以解开 compress、zip 与 gzip 所压缩的文件。

经过 gzip 压缩过，源文件就不存在了。

有 9 个不同的压缩等级可以使用。

可以使用 zcat、zmore、zless 来读取压缩文件的内容。

```shell
$ gzip [-cdtv#] filename
-c ：将压缩的数据输出到屏幕上
-d ：解压缩
-t ：检验压缩文件是否出错
-v ：显示压缩比等信息
-# ： # 为数字的意思，代表压缩等级，数字越大压缩比越高，默认为 6
```

**bzip2**

提供比 gzip 更高的压缩比。

查看命令：bzcat、bzmore、bzless、bzgrep。

```shell
$ bzip2 [-cdkzv#] filename
-k ：保留源文件
```

**xz**

提供比 bzip2 更佳的压缩比。

可以看到，gzip、bzip2、xz 的压缩比不断优化。不过要注意的是，压缩比越高，压缩的时间也越长。

查看命令：xzcat、xzmore、xzless、xzgrep。

```shell
$ xz [-dtlkc#] filename
```



**打包：**

压缩指令只能对一个文件进行压缩，而打包能够将多个文件打包成一个大文件。tar 不仅可以用于打包，也可以使用 gzip、bzip2、xz 将打包文件进行压缩。

```shell
$ tar [-z|-j|-J] [cv] [-f 新建的 tar 文件] filename...  ==打包压缩
$ tar [-z|-j|-J] [tv] [-f 已有的 tar 文件]              ==查看
$ tar [-z|-j|-J] [xv] [-f 已有的 tar 文件] [-C 目录]    ==解压缩
-z ：使用 zip；
-j ：使用 bzip2；
-J ：使用 xz；
-c ：新建打包文件；
-t ：查看打包文件里面有哪些文件；
-x ：解打包或解压缩的功能；
-v ：在压缩/解压缩的过程中，显示正在处理的文件名；
-f : filename：要处理的文件；
-C 目录 ： 在特定目录解压缩。
```

| 使用方式 | 命令                                                  |
| :------: | ----------------------------------------------------- |
| 打包压缩 | tar -jcv -f filename.tar.bz2 要被压缩的文件或目录名称 |
|  查 看   | tar -jtv -f filename.tar.bz2                          |
|  解压缩  | tar -jxv -f filename.tar.bz2 -C 要解压缩的目录        |



### 进程管理

#### 查看进程

**ps**

查看某个时间点的进程信息

示例：查看自己的进程

```sh
$ ps -l
```

示例：查看系统所有进程

```sh
$ ps aux
```

示例：查看特定的进程

```sh
$ ps aux | grep threadx
```

**pstree**

查看进程树

示例：查看所有进程树

```sh
$ pstree -A
```

**top**

实时显示进程信息

示例：两秒钟刷新一次

```sh
$ top -d 2
```

**netstat**

查看**占用端口**的进程

示例：查看特定端口的进程

```sh
$ lsof -i:端口
or
$ netstat -tunlp|grep 端口号
```



#### 进程状态

| 状态 | 说明                                                         |
| :--: | ------------------------------------------------------------ |
|  R   | running or runnable (on run queue)<br>正在执行或者可执行，此时进程位于执行队列中。 |
|  D   | uninterruptible sleep (usually I/O)<br>不可中断阻塞，通常为 IO 阻塞。 |
|  S   | interruptible sleep (waiting for an event to complete) <br> 可中断阻塞，此时进程正在等待某个事件完成。 |
|  Z   | zombie (terminated but not reaped by its parent)<br>僵死，进程已经终止但是尚未被其父进程获取信息。 |
|  T   | stopped (either by a job control signal or because it is being traced) <br> 结束，进程既可以被作业控制信号结束，也可能是正在被追踪。 |

![](https://img-note.langyastudio.com/202111171339429.png?x-oss-process=style/watermark)



#### SIGCHLD

当一个子进程改变了它的状态时（停止运行，继续运行或者退出），有两件事会发生在父进程中：

- 得到 SIGCHLD 信号
- waitpid() 或者 wait() 调用会返回

其中子进程发送的 SIGCHLD 信号包含了子进程的信息，比如进程 ID、进程状态、进程使用 CPU 的时间等。

在子进程退出时，它的进程描述符不会立即释放，这是为了让父进程得到子进程信息，父进程通过 wait() 和 waitpid() 来获得一个已经退出的子进程的信息。



![](https://img-note.langyastudio.com/202111171339657.png?x-oss-process=style/watermark)



#### wait()

```c
pid_t wait(int *status)
```

父进程调用 wait() 会一直阻塞，直到收到一个子进程退出的 SIGCHLD 信号，之后 wait() 函数会销毁子进程并返回。

如果成功，返回被收集的子进程的进程 ID；如果调用进程没有子进程，调用就会失败，此时返回 -1，同时 errno 被置为 ECHILD。

参数 status 用来保存被收集的子进程退出时的一些状态，如果对这个子进程是如何死掉的毫不在意，只想把这个子进程消灭掉，可以设置这个参数为 NULL。



#### waitpid()

```c
pid_t waitpid(pid_t pid, int *status, int options)
```

作用和 wait() 完全相同，但是多了两个可由用户控制的参数 pid 和 options。

pid 参数指示一个子进程的 ID，表示只关心这个子进程退出的 SIGCHLD 信号。如果 pid=-1 时，那么和 wait() 作用相同，都是关心所有子进程退出的 SIGCHLD 信号。

options 参数主要有 WNOHANG 和 WUNTRACED 两个选项，WNOHANG 可以使 waitpid() 调用变成非阻塞的，也就是说它会立即返回，父进程可以继续执行其它任务。



#### 孤儿进程

一个父进程退出，而它的一个或多个子进程还在运行，那么这些子进程将成为孤儿进程。

**孤儿进程将被 init 进程（进程号为 1）所收养**，并由 init 进程对它们完成状态收集工作。

由于孤儿进程会被 init 进程收养，所以孤儿进程不会对系统造成危害。



#### 僵尸进程

一个子进程的进程描述符在子进程退出时不会释放，只有当父进程通过 wait() 或 waitpid() 获取了子进程信息后才会释放。如果**子进程退出，而父进程并没有调用 wait() 或 waitpid()，那么子进程的进程描述符仍然保存在系统中**，这种进程称之为僵尸进程。

僵尸进程通过 ps 命令显示出来的状态为 Z（zombie）。

系统所能使用的进程号是有限的，如果产生大量僵尸进程，将因为没有可用的进程号而导致系统不能产生新的进程。

要消灭系统中大量的僵尸进程，只需要将其父进程杀死，此时僵尸进程就会变成孤儿进程，从而被 init 进程所收养，这样 init 进程就会释放所有的僵尸进程所占有的资源，从而结束僵尸进程。



### 管道指令

管道是将一个命令的标准输出作为另一个命令的标准输入，在数据需要经过多个步骤的处理之后才能得到我们想要的内容时就可以使用管道。

在命令之间使用 | 分隔各个管道命令。

```bash
$ ls -al /etc | less
```



**数据流重定向：**

重定向指的是使用文件代替标准输入、标准输出和标准错误输出。

|           1           | 代码 |    运算符    |
| :-------------------: | :--: | :----------: |
|   标准输入 (stdin)    |  0   |  \< 或 \<\<  |
|   标准输出 (stdout)   |  1   | &gt; 或 \>\> |
| 标准错误输出 (stderr) |  2   | 2\> 或 2\>\> |

其中，有一个箭头的表示以覆盖的方式重定向，而有两个箭头的表示以追加的方式重定向。

可以将不需要的标准输出以及标准错误输出重定向到 /dev/null，相当于扔进垃圾箱。

如果需要将标准输出以及标准错误输出同时重定向到一个文件，需要将某个输出转换为另一个输出，例如 2\>&1 表示将标准错误输出转换为标准输出。

```bash
$ find /home -name .bashrc > list 2>&1
```



**提取指令：**

cut 对数据进行切分，取出想要的部分。

切分过程一行一行地进行。

```shell
$ cut
-d ：分隔符
-f ：经过 -d 分隔后，使用 -f n 取出第 n 个区间
-c ：以字符为单位取出区间
```

示例 1：last 显示登入者的信息，取出用户名。

```shell
$ last
root pts/1 192.168.201.101 Sat Feb 7 12:35 still logged in
root pts/1 192.168.201.101 Fri Feb 6 12:13 - 18:46 (06:33)
root pts/1 192.168.201.254 Thu Feb 5 22:37 - 23:53 (01:16)

$ last | cut -d ' ' -f 1
```

示例 2：将 export 输出的信息，取出第 12 字符以后的所有字符串。

```shell
$ export
declare -x HISTCONTROL="ignoredups"
declare -x HISTSIZE="1000"
declare -x HOME="/home/dmtsai"
declare -x HOSTNAME="study.centos.vbird"
.....(其他省略).....

$ export | cut -c 12-
```



**排序指令：**

**sort**   用于排序

```shell
$ sort [-fbMnrtuk] [file or stdin]
-f ：忽略大小写
-b ：忽略最前面的空格
-M ：以月份的名字来排序，例如 JAN，DEC
-n ：使用数字
-r ：反向排序
-u ：相当于 unique，重复的内容只出现一次
-t ：分隔符，默认为 tab
-k ：指定排序的区间
```

示例：/etc/passwd 文件内容以 : 来分隔，要求以第三列进行排序。

```shell
$ cat /etc/passwd | sort -t ':' -k 3
root:x:0:0:root:/root:/bin/bash
dmtsai:x:1000:1000:dmtsai:/home/dmtsai:/bin/bash
alex:x:1001:1002::/home/alex:/bin/bash
arod:x:1002:1003::/home/arod:/bin/bash
```

**uniq**   可以将重复的数据只取一个。

```shell
$ uniq [-ic]
-i ：忽略大小写
-c ：进行计数
```

示例：取得每个人的登录总次数

```shell
$ last | cut -d ' ' -f 1 | sort | uniq -c
1
6 (unknown
47 dmtsai
4 reboot
7 root
1 wtmp
```



**双向输出重定向：**

输出重定向会将输出内容重定向到文件中，而   **tee**   不仅能够完成这个功能，还能保留屏幕上的输出。也就是说，使用 tee 指令，一个输出会同时传送到文件和屏幕上。

```shell
$ tee [-a] file
```



**字符转换指令：**

**tr**   用来删除一行中的字符，或者对字符进行替换

```shell
$ tr [-ds] SET1 ...
-d ： 删除行中 SET1 这个字符串
```

示例，将 last 输出的信息所有小写转换为大写。

```shell
$ last | tr '[a-z]' '[A-Z]'
```

**col**   将 tab 字符转为空格字符

```shell
$ col [-xb]
-x ： 将 tab 键转换成对等的空格键
```

**expand**   将 tab 转换一定数量的空格，默认是 8 个

```shell
$ expand [-t] file
-t ：tab 转为空格的数量
```

**join**   将有相同数据的那一行合并在一起

```shell
$ join [-ti12] file1 file2
-t ：分隔符，默认为空格
-i ：忽略大小写的差异
-1 ：第一个文件所用的比较字段
-2 ：第二个文件所用的比较字段
```

**paste**   直接将两行粘贴在一起

```shell
$ paste [-d] file1 file2
-d ：分隔符，默认为 tab
```



**分区指令：**

**split**   将一个文件划分成多个文件

```shell
$ split [-bl] file PREFIX
-b ：以大小来进行分区，可加单位，例如 b, k, m 等
-l ：以行数来进行分区。
- PREFIX ：分区文件的前导名称
```



### 正则表达式

**grep**

g/re/p（globally search a regular expression and print)，使用正则表示式进行**全局查找**并打印。

```shell
$ grep [-acinv] [--color=auto] 搜寻字符串 filename
-c ： 统计匹配到行的个数
-i ： 忽略大小写
-n ： 输出行号
-v ： 反向选择，也就是显示出没有 搜寻字符串 内容的那一行
--color=auto ：找到的关键字加颜色显示
```

示例：把含有 the 字符串的行提取出来（注意默认会有 --color=auto 选项，因此以下内容在 Linux 中有颜色显示 the 字符串）

```shell
$ grep -n 'the' regular_express.txt
8:I can't finish the test.
12:the symbol '*' is represented as start.
15:You are the best is mean you are the no. 1.
16:The world Happy is the same with "glad".
18:google is the best tools for search keyword
```

示例：正则表达式 a{m,n} 用来匹配字符 a m\~n 次，这里需要将 { 和 } 进行转义，因为它们在 shell 是有特殊意义的。

```shell
$ grep -n 'a\{2,5\}' regular_express.txt
```



**printf**

用于格式化输出。它不属于管道命令，在给 printf 传数据时需要使用 $( ) 形式

```shell
$ printf '%10s %5i %5i %5i %8.2f \n' $(cat printf.txt)
    DmTsai    80    60    92    77.33
     VBird    75    55    80    70.00
       Ken    60    90    70    73.33
```



**awk**

是由 Alfred Aho，Peter Weinberger 和 Brian Kernighan 创造，awk 这个名字就是这三个创始人名字的首字母。

awk 每次处理一行，**处理的最小单位是字段**，每个字段的命名方式为：\$n，n 为字段号，从 1 开始，\$0 表示一整行。

示例：取出最近五个登录用户的用户名和 IP。首先用 last -n 5 取出用最近五个登录用户的所有信息，可以看到用户名和 IP 分别在第 1 列和第 3 列，我们用 \$1 和 \$3 就能取出这两个字段，然后用 print 进行打印。

```shell
$ last -n 5
dmtsai pts/0 192.168.1.100 Tue Jul 14 17:32 still logged in
dmtsai pts/0 192.168.1.100 Thu Jul 9 23:36 - 02:58 (03:22)
dmtsai pts/0 192.168.1.100 Thu Jul 9 17:23 - 23:36 (06:12)
dmtsai pts/0 192.168.1.100 Thu Jul 9 08:02 - 08:17 (00:14)
dmtsai tty1 Fri May 29 11:55 - 12:11 (00:15)
```

```shell
$ last -n 5 | awk '{print $1 "\t" $3}'
dmtsai   192.168.1.100
dmtsai   192.168.1.100
dmtsai   192.168.1.100
dmtsai   192.168.1.100
dmtsai   Fri
```

可以根据字段的某些条件进行匹配，例如匹配字段小于某个值的那一行数据。

```shell
$ awk '条件类型 1 {动作 1} 条件类型 2 {动作 2} ...' filename
```

示例：/etc/passwd 文件第三个字段为 UID，对 UID 小于 10 的数据进行处理。

```shell
$ cat /etc/passwd | awk 'BEGIN {FS=":"} $3 < 10 {print $1 "\t " $3}'
root 0
bin 1
daemon 2
```

awk 变量：

| 变量名称 | 代表意义                     |
| :------: | ---------------------------- |
|    NF    | 每一行拥有的字段总数         |
|    NR    | 目前所处理的是第几行数据     |
|    FS    | 目前的分隔字符，默认是空格键 |

示例：显示正在处理的行号以及每一行有多少字段

```shell
$ last -n 5 | awk '{print $1 "\t lines: " NR "\t columns: " NF}'
dmtsai lines: 1 columns: 10
dmtsai lines: 2 columns: 10
dmtsai lines: 3 columns: 10
dmtsai lines: 4 columns: 10
dmtsai lines: 5 columns: 9
```



### 其他

#### 关机

**who**

在关机前需要先使用 who 命令查看有没有其它用户在线



**sync**

为了加快对磁盘文件的读写速度，位于内存中的文件数据不会立即同步到磁盘，因此关机之前需要先进行 sync 同步操作



**shutdown**

```shell
## shutdown [-krhc] 时间 [信息]
-k ： 不会关机，只是发送警告信息，通知所有在线的用户
-r ： 将系统的服务停掉后就重新启动
-h ： 将系统的服务停掉后就立即关机
-c ： 取消已经在进行的 shutdown
```



#### PATH

可以在环境变量 PATH 中声明可执行文件的路径，路径之间用 : 分隔

```shell
/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/dmtsai/.local/bin:/home/dmtsai/bin
```



#### sudo

sudo 允许一般用户使用 root 可执行的命令，不过只有在 /etc/sudoers 配置文件中添加的用户才能使用该指令。

- su == sudo -s
- su - == sudo -i

sudo -s：

1. 使用当前用户的环境变量
2. 不跳转切换用户后的目录
3. 切换到超级管理员或者目标用户的权限

**sudo -i：**

1. 使用root或者目标用户用户的环境变量
2. 切换到 /root或者目标用户的home目录
3. 切换到超级管理员或者目标用户的权限



#### 包管理工具

RPM 和 DPKG 为最常见的两类软件包管理工具：

- RPM 全称为 Redhat Package Manager，最早由 Red Hat 公司制定实施，随后被 GNU 开源操作系统接受并成为许多 Linux 系统的既定软件标准。YUM 基于 RPM，具有依赖管理和软件升级功能
- 与 RPM 竞争的是基于 Debian 操作系统的 DEB 软件包管理工具 DPKG，全称为 Debian Package，功能方面与 RPM 相似



#### VIM 三个模式

![](https://img-note.langyastudio.com/202111171349557.png?x-oss-process=style/watermark)

- 一般指令模式（Command mode）：VIM 的默认模式，可以用于移动游标查看内容
- 编辑模式（Insert mode）：按下 "i" 等按键之后进入，可以对文本进行编辑
- 指令列模式（Bottom-line mode）：按下 ":" 按键之后进入，用于保存退出等操作

在指令列模式下，有以下命令用于离开或者保存文件。

| 命令 |                             作用                             |
| :--: | :----------------------------------------------------------: |
|  :w  |                           写入磁盘                           |
| :w!  | 当文件为只读时，强制写入磁盘。到底能不能写入，与用户对该文件的权限有关 |
|  :q  |                             离开                             |
| :q!  |                        强制离开不保存                        |
| :wq  |                        写入磁盘后离开                        |
| :wq! |                      强制写入磁盘后离开                      |



### 开源协议

- [Choose an open source license](https://choosealicense.com/)
- [如何选择开源许可证？](http://www.ruanyifeng.com/blog/2011/05/how_to_choose_free_software_licenses.html)



## 参考资料

- 鸟哥. 鸟 哥 的 Linux 私 房 菜 基 础 篇 第 三 版[J]. 2009.
- [Linux 平台上的软件包管理](https://www.ibm.com/developerworks/cn/linux/l-cn-rpmdpkg/index.html)
- [Linux 之守护进程、僵死进程与孤儿进程](http://liubigbin.github.io/2016/03/11/Linux-%E4%B9%8B%E5%AE%88%E6%8A%A4%E8%BF%9B%E7%A8%8B%E3%80%81%E5%83%B5%E6%AD%BB%E8%BF%9B%E7%A8%8B%E4%B8%8E%E5%AD%A4%E5%84%BF%E8%BF%9B%E7%A8%8B/)
- [What is the difference between a symbolic link and a hard link?](https://stackoverflow.com/questions/185899/what-is-the-difference-between-a-symbolic-link-and-a-hard-link)
- [Linux process states](https://idea.popcount.org/2012-12-11-linux-process-states/)
- [GUID Partition Table](https://en.wikipedia.org/wiki/GUID_Partition_Table)
- [详解 wait 和 waitpid 函数](https://blog.csdn.net/kevinhg/article/details/7001719)
- [IDE、SATA、SCSI、SAS、FC、SSD 硬盘类型介绍](https://blog.csdn.net/tianlesoftware/article/details/6009110)
- [Akai IB-301S SCSI Interface for S2800,S3000](http://www.mpchunter.com/s3000/akai-ib-301s-scsi-interface-for-s2800s3000/)
- [Parallel ATA](https://en.wikipedia.org/wiki/Parallel_ATA)
- [ADATA XPG SX900 256GB SATA 3 SSD Review – Expanded Capacity and SandForce Driven Speed](http://www.thessdreview.com/our-reviews/adata-xpg-sx900-256gb-sata-3-ssd-review-expanded-capacity-and-sandforce-driven-speed/4/)
- [Decoding UCS Invicta – Part 1](https://blogs.cisco.com/datacenter/decoding-ucs-invicta-part-1)
- [硬盘](https://zh.wikipedia.org/wiki/%E7%A1%AC%E7%9B%98)
- [Difference between SAS and SATA](http://www.differencebetween.info/difference-between-sas-and-sata)
- [BIOS](https://zh.wikipedia.org/wiki/BIOS)
- [File system design case studies](https://www.cs.rutgers.edu/\~pxk/416/notes/13-fs-studies.html)
- [Programming Project #4](https://classes.soe.ucsc.edu/cmps111/Fall08/proj4.shtml)
- [FILE SYSTEM DESIGN](http://web.cs.ucla.edu/classes/fall14/cs111/scribe/11a/index.html)

