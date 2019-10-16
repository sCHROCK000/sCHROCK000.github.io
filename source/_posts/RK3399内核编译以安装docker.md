title: RK3399内核编译以安装docker
tags: []
author: schrock000
categories:
  - ARM_Linux
date: 2019-10-16 03:14:00
---
环境：  
1、firefly-rk3399开发板  
2、ubuntu18.04 官方镜像  
3、内核版本 4.4.154  
***
最近想使用Docker的某容器，在RK3399-firefly上安装的时候报错，具体安装过程就网上随意能搜到的，就不提及了。经过查看各种资料之后，成功安装docker，希望能帮到有需要的各位，也顺便记录一下。大致过程如下：

1、 查看docker的状态发现是failed，journalctl -fu docker 查看日志，有driver not support等字样，具体记不清了，这里大致了解了一下是因为docker有那么几种支持的storage-driver， 恰好内核又没打开任何一个支持的FS，所以只好去编译内核了。

2、修改配置   
进入linux-sdk/kernel目录下，获取https://raw.githubusercontent.com/docker/docker/master/contrib/check-config.sh脚本查看docker需要的设置是否打开，其他的都是enable，我仅放出storage-driver的配置，主要是要打开overlay的FS.
```
- Storage Drivers:
  - "aufs":
    - CONFIG_AUFS_FS: missing
  - "btrfs":
    - CONFIG_BTRFS_FS: missing
    - CONFIG_BTRFS_FS_POSIX_ACL: missing
  - "devicemapper":
    - CONFIG_BLK_DEV_DM: enabled
    - CONFIG_DM_THIN_PROVISIONING: enabled
  - "overlay":
    - CONFIG_OVERLAY_FS: enabled
  - "zfs":
    - /dev/zfs: missing
    - zfs command: missing
    - zpool command: missing
```
***
3、
之前的配置都是通过make menuconfig配置的，具体怎么操作网上找，挺多的。
进入linux-sdk/kernel/目录，make ARCH=arm64 rk3399-firefly-linux.img -j8 编译生成boot.img后烧录到开发板。
上电发现docker仍报错，修改配置文件内容（没有就新建），/etc/docker/daemon.json， 内容如下：
```
{
    "registry-mirrors": ["aliyun加速地址"],
    "storage-driver": "overlay"
}
```
***
上面的阿里云加速地址换成自己的，这里主要是为了pull容器的时候舒服一点，教程可以在网上搜到很多，保存。
重启服务：systemctl daemon-reload && systemctl restart docker  
还是无法正常运行，继续查看日志，发现报错已经变更了，这时候提示的是iptables does not exist 等字样的信息，其实就是说iptables内核模块没有，配置完编译新内核上传。加入以下模块，基本都在networking support->networking options中，我也记不太清了，各位自己查找一下吧。。。。

***
Netfilter connection tracking support
Netbios name service protocal support(new)
Netfilter Xtables support (required for ip_tables)

IP: Netfilter Configuration
IPv4 connection tracking support (require for NAT)
IP tables support (required for filtering/masq/NAT)

什么 NAT的选项，忘了
MASQUERADE target support
REDIRECT target support

按上面步骤重新编译并烧录到开发板，这次上电，docker绿了，开心。
操作了一下，正常
```
sudo docker run ubuntu:18.04 /bin/echo "Hello world"

Status: Downloaded newer image for ubuntu:18.04  
Hello world
```
***
