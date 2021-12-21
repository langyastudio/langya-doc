## 磁盘挂载
在阿里云购买centos系统盘时，可以再购买磁盘。那么这时候，就需要挂载购买的磁盘。

```bash
#查看磁盘（发现新购买的/dev/vdb磁盘，或者使用 fdisk -l）
ls /dev

#磁盘格式化（G 回车 **N 回车**：G是建立GPT分区表\N是建立新分区）
fdisk /dev/vdb

最后选择 w 保存结果

#修改分区格式（ext4）
mkfs.ext4 /dev/vdb1

#建立挂载的目录（/mnt/volume1）
mkdir /mnt/volume1

#查看分区的磁盘Id号（194e9719-4787-4f88-bd52-1dc2d1b60680 -> ../../vdb1）
ls -l /dev/disk/by-uuid

#开机挂载磁盘
vim /etc/fstab
增加：
UUID=194e9719-4787-4f88-bd52-1dc2d1b60680 /mnt/volume1/ ext4 defaults 1 1

#重启电脑，查看分区
df -h
```



## 局域网磁盘共享

挂载的磁盘，需要在多台centos机器上使用，这里面可以使用较为简单的NFS共享模式。

### 安装与配置

```bash
#安装
yum install nfs-utils rpcbind
service rpcbind start
service nfs start
chkconfig rpcbind on
chkconfig nfs on

#修改配置文件（nfs 的配置文件）
vim /etc/exports

/mnt/volume1/ 192.168.1.24(rw,no_root_squash,no_all_squash,sync,anonuid=501,anongid=501)

#配置立即生效
exportfs -r
```

/mnt/volume1 为共享目录
192.168.1.24  可以为一个网段，一个IP，也可以是域名，域名支持通配符 如: *.qq.com，这里面只让192.168.1.24共享

rw：read-write，可读写；
ro：read-only，只读；

sync：文件同时写入硬盘和内存；
async：文件暂存于内存，而不是直接写入内存；

no_root_squash：NFS客户端连接服务端时如果使用的是root的话，那么对服务端分享的目录来说，也拥有root权限。显然开启这项是不安全的。
root_squash：NFS客户端连接服务端时如果使用的是root的话，那么对服务端分享的目录来说，拥有匿名用户权限，通常他将使用nobody或nfsnobody身份；
all_squash：不论NFS客户端连接服务端时使用什么用户，对服务端分享的目录来说都是拥有匿名用户权限；

anonuid：匿名用户的UID值，可以在此处自行设定。
anongid：匿名用户的GID值。



### 客户端挂载

```bash
#查看可挂载
showmount -e 192.168.1.3  
 
Export list for 192.168.1.3:
/mnt/volume1    192.168.1.24  

#客户端挂载（将共享盘挂载到/mnt/volume2）
mount -t nfs 192.168.1.3:/mnt/volume1  /mnt/volume2 

无提示，既为成功      
```

> 开机挂载：
> vim /etc/fstab
> 192.168.1.3:/mnt/volume1 /mnt/volume1 nfs rw,tcp,intr 0 1





