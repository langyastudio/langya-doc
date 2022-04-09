由于在安装centos系统的时候，如果在安装时没有分配磁盘空间，选择的是默认分配的，在安装完成后，可以发现大容量磁盘往往分配在了home下面。

如果要把home下面的磁盘空间分配到root磁盘下面。可以进行如下操作。



- 查看CentOS的系统版本

![img](https://img2018.cnblogs.com/blog/1331300/201911/1331300-20191102162556682-2071306105.png)

- 查看分区

df -h (centos-home和centos-root每人的名字可能不一样) 

![img](https://img2018.cnblogs.com/blog/1331300/201911/1331300-20191102162702551-1230891453.png)

- 备份home分区文件

tar cvf /tmp/home.tar /home

 ![img](https://img2018.cnblogs.com/blog/1331300/201911/1331300-20191102162802705-672530806.png)

- 卸载/home，如果无法卸载，先终止使用/home文件系统的进程

umount /home （卸载）

![img](https://img2018.cnblogs.com/blog/1331300/201911/1331300-20191102162921675-1937771980.png)

卸载时，发现/home在使用中，所以先终止。

fuser -km /home/（终止）

![img](https://img2018.cnblogs.com/blog/1331300/201911/1331300-20191102163022448-794866552.png)

再次卸载，没有报错，表示成功。

![img](https://img2018.cnblogs.com/blog/1331300/201911/1331300-20191102163104165-1425005239.png)

- 删除/home所在的lv

lvremove /dev/mapper/centos-home

![img](https://img2018.cnblogs.com/blog/1331300/201911/1331300-20191102163208770-1205102138.png)

- 扩展/root所在的lv

lvextend -L +100G /dev/mapper/centos-root

![img](https://img2018.cnblogs.com/blog/1331300/201911/1331300-20191102163307969-346247801.png)

- 扩展/root文件系统

xfs_growfs /dev/mapper/centos-root

![img](https://img2018.cnblogs.com/blog/1331300/201911/1331300-20191102163359398-1215559212.png)

- 重新创建home lv （创建时计算好剩余的磁盘容量，建议比剩余小1G左右）

lvcreate -L 41G -n /dev/mapper/centos-home 

![img](https://img2018.cnblogs.com/blog/1331300/201911/1331300-20191102163539117-464165311.png)

- 创建文件系统

mkfs.xfs /dev/mapper/centos-home

![img](https://img2018.cnblogs.com/blog/1331300/201911/1331300-20191102163703387-1689795594.png)

- 挂载home

mount /dev/mapper/centos-home

![img](https://img2018.cnblogs.com/blog/1331300/201911/1331300-20191102163753324-716500128.png)

- home文件恢复

tar xvf /tmp/home.tar -C /home/

![img](https://img2018.cnblogs.com/blog/1331300/201911/1331300-20191102163848459-986915311.png)

- 再次使用df -h查看系统磁盘大小

![img](https://img2018.cnblogs.com/blog/1331300/201911/1331300-20191102163941754-944626860.png)

可以看到home下面100G的磁盘容量已经转移到root下面了，至此，转移任务结束。此为在CentOS7.2系统下测试使用的，在CentOS6版本下还没测试过。