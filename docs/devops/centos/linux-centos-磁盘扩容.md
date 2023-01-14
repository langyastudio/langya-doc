由于在安装centos系统的时候，如果在安装时没有分配磁盘空间，选择的是默认分配的，在安装完成后，可以发现大容量磁盘往往分配在了home下面。

如果要把home下面的磁盘空间分配到root磁盘下面。可以进行如下操作。



- 查看CentOS的系统版本

![img](https://img-note.langyastudio.com/202212080845591.png?x-oss-process=style/watermark)

- 查看分区

df -h (centos-home和centos-root每人的名字可能不一样) 

![img](https://img-note.langyastudio.com/202212080845380.png?x-oss-process=style/watermark)

- 备份home分区文件

tar cvf /tmp/home.tar /home

 ![img](https://img-note.langyastudio.com/202212080845283.png?x-oss-process=style/watermark)

- 卸载/home，如果无法卸载，先终止使用/home文件系统的进程

umount /home （卸载）

![img](https://img-note.langyastudio.com/202212080845289.png?x-oss-process=style/watermark)

卸载时，发现/home在使用中，所以先终止。

fuser -km /home/（终止）

![img](https://img-note.langyastudio.com/202212080845247.png?x-oss-process=style/watermark)

再次卸载，没有报错，表示成功。

![img](https://img-note.langyastudio.com/202212080845669.png?x-oss-process=style/watermark)

- 删除/home所在的lv

lvremove /dev/mapper/centos-home

![img](https://img-note.langyastudio.com/202212080845750.png?x-oss-process=style/watermark)

- 扩展/root所在的lv

lvextend -L +100G /dev/mapper/centos-root

![img](https://img-note.langyastudio.com/202212080845970.png?x-oss-process=style/watermark)

- 扩展/root文件系统

xfs_growfs /dev/mapper/centos-root

![img](https://img-note.langyastudio.com/202212080845767.png?x-oss-process=style/watermark)

- 重新创建home lv （创建时计算好剩余的磁盘容量，建议比剩余小1G左右）

lvcreate -L 41G -n /dev/mapper/centos-home 

![img](https://img-note.langyastudio.com/202212080845413.png?x-oss-process=style/watermark)

- 创建文件系统

mkfs.xfs /dev/mapper/centos-home

![img](https://img-note.langyastudio.com/202212080845600.png?x-oss-process=style/watermark)

- 挂载home

mount /dev/mapper/centos-home

![img](https://img-note.langyastudio.com/202212080845250.png?x-oss-process=style/watermark)

- home文件恢复

tar xvf /tmp/home.tar -C /home/

![img](https://img-note.langyastudio.com/202212080845793.png?x-oss-process=style/watermark)

- 再次使用df -h查看系统磁盘大小

![img](https://img-note.langyastudio.com/202212080845407.png?x-oss-process=style/watermark)

可以看到home下面100G的磁盘容量已经转移到root下面了，至此，转移任务结束。此为在CentOS7.2系统下测试使用的，在CentOS6版本下还没测试过。