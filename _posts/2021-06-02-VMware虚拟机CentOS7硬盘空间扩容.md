---
layout:     post
title:      VMware虚拟机CentOS7硬盘空间扩容
subtitle:   
date:       2021-06-02
author:     dm
header-img: img/post-bg-coffee.jpeg
catalog: true
tags:
    - VMware






---

## 1、查看centos7系统挂载点信息

#查看挂载点信息

```sh
df -h  
```

#执行截图

![查看挂载点信息](https://raw.githubusercontent.com/DongMing0103/MarkdownCloudImage/master/data/%E6%9F%A5%E7%9C%8B%E6%8C%82%E8%BD%BD%E7%82%B9%E4%BF%A1%E6%81%AFdf-h.jpg)

 

## 2、扩展VMWare-centos7硬盘空间

**关闭Vmware的centos7系统，才能在VMWare菜单中设置需要增加到的磁盘大小**

**如果这个选项是灰色的，说明此虚拟机建有快照，把快照全部删除再试试!**

![img](https://images2018.cnblogs.com/blog/872610/201805/872610-20180524182550831-1169675176.png)

 

## 3、对新增加的硬盘进行分区、格式化

## <font color="red"> 我们增加了空间的硬盘是 /dev/vda</font>

```tex
分区： 
[root@localhost]# fdisk /dev/vda
p　　　　　　  #查看已分区数量（我看到有两个 /dev/sda1 /dev/sda2） 
n　　　　　　　#新增加一个分区
p　　　　　　　#分区类型我们选择为主分区 
　　　　　　   #分区号输入3（因为1,2已经用过了,sda1是分区1,sda2是分区2,sda3分区3） 
回车　　　　　  #默认（起始扇区）
回车　　　　　  #默认（结束扇区）
t　　　　　　　 #修改分区类型 
　　　　　　   #选分区3
8e　　　　　 　#修改为LVM（8e就是LVM）
w　　　　　  　#写分区表
q　　　　　  　#完成，退出fdisk命令
```

 #使用partprobe命令 或者重启机器

#格式化分区3命令，这边可以用ext4或者ext3，因为xfs性能比较好，我这边改成xfs了

```shell
mkfs.xfs /dev/vda3
```

#执行截图

![执行fdisk /dev/vda命令](https://raw.githubusercontent.com/DongMing0103/MarkdownCloudImage/master/data/%E6%89%A7%E8%A1%8Cfdisk%20/dev/sda%E5%91%BD%E4%BB%A4.jpg)

![执行fdisk /dev/vda命令2](https://raw.githubusercontent.com/DongMing0103/MarkdownCloudImage/master/data/%E6%89%A7%E8%A1%8Cfdisk%20/dev/vda%E5%91%BD%E4%BB%A42.jpg)

![执行fdisk /dev/vda命令3](https://raw.githubusercontent.com/DongMing0103/MarkdownCloudImage/master/data/%E6%89%A7%E8%A1%8Cfdisk%20/dev/vda%E5%91%BD%E4%BB%A43.jpg)

![执行fdisk /dev/vda命令4](https://raw.githubusercontent.com/DongMing0103/MarkdownCloudImage/master/data/%E6%89%A7%E8%A1%8Cfdisk%20/dev/vda%E5%91%BD%E4%BB%A44.jpg)

## 4、添加新LVM到已有的LVM组，实现扩容

```
#进入lvm管理
lvm　　　　　　　　　　　　           

#这是初始化刚才的分区3
lvm> pvcreate /dev/vda3

#将初始化过的分区加入到虚拟卷组centos (卷和卷组的命令可以通过 vgdisplay查看)
lvm> vgextend centos /dev/vda3

#vgdisplay查看free PE /Site
lvm> vgdisplay -v
lvm> vgdisplay

#扩展已有卷的容量（10240 是通过vgdisplay查看free PE /Site的大小）
lvm> lvextend -l+10240 /dev/mapper/centos-root　　

#查看卷容量，这时你会看到一个很大的卷了
lvm> pvdisplay

#退出
lvm> quit
```

![lvm命令](https://raw.githubusercontent.com/DongMing0103/MarkdownCloudImage/master/data/lvm.jpg)

![vgdisplay命令](https://raw.githubusercontent.com/DongMing0103/MarkdownCloudImage/master/data/vgdisplay%E5%91%BD%E4%BB%A4.jpg)


![pvdisplay命令](https://raw.githubusercontent.com/DongMing0103/MarkdownCloudImage/master/data/pvdisplay%E5%91%BD%E4%BB%A4.jpg)

### 4.1、上面只是卷扩容了，下面是文件系统的真正扩容，输入以下命令

**CentOS7下面由于使用的是XFS命令:**

/dev/mapper/centos-root是df -h查看到根目录的挂载点,需要扩容的挂载点

```shell
xfs_growfs /dev/mapper/centos-root
```

![扩容完毕](https://raw.githubusercontent.com/DongMing0103/MarkdownCloudImage/master/data/%E6%89%A9%E5%AE%B9%E5%AE%8C%E6%AF%95.jpg)

<font color="red">未使用以下命令</font>

``` tex
xfs_growfs针对文件系统xfs

检查数据块大小和数量

xfs_growfs  /dev/centos/root

将XFS文件扩展到1986208

xfs_growfs /dev/centos/root -D 1986208

自动扩展XFS文件系统到最大的可用大小

xfs_growfs /dev/centos/root

```

**CentOS6使用命令:**

使用resize2fs对挂载目录在线扩容
resize2fs针对文件系统ext2 ext3 ext4

```
resize2fs /dev/mapper/centos-root
```

查看新的磁盘空间

```shell
df -h
```

执行截图

![img](https://images2018.cnblogs.com/blog/872610/201805/872610-20180524183850522-2039464691.png)

[转载](https://www.cnblogs.com/Sungeek/p/9084510.html)
