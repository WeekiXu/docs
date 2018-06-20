---
title: LINUX SSH FILE NET AND SO ON
categories:
 - linux
tags:
 - linux
 - ssh
 - 免密码
 - 挂载磁盘
 - netstat
---

# LINUX BASIC

## SSH 免密码登录
1. 安装ssh    
`apt-get install openssh-server`
1. 查看ssh运行状态     
`ps -e | grep ssh`  
如果发现 sshd 和 ssh-agent 即表明 ssh服务基本运行正常
1. 生成公钥和私钥  
`ssh-keygen -t rsa -P ""`
1. 将公钥追加到文件  
`ca ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys`
1. 权限问题
   * .ssh目录权限是700
   *  两个dsa 和 rsa的 私钥权限是600
   *  其余文件权限是644
   * .ssh的父目录文件权限应该是755
   
## 文件操作
 * 挂载磁盘
```
 [root@weirui /]# df -h  //挂在磁盘操作（还有一个300G的盘没显示出来）
 Filesystem            Size  Used Avail Use% Mounted on
 /dev/hda1              39G   12G   26G  32% /
 tmpfs                 4.0G     0  4.0G   0% /dev/shm
 [root@weirui /]# mkfs -t ext4 /dev/xvdb1      //格式化未挂载的磁盘，使用ext3格式 
 [root@weirui /]# mkdir /alidata1       //创建挂载对应的目录 
 [root@weirui /]# mount /dev/xvdb1 /alidata1    //将磁盘挂载到创建的目录上 
 ```
 * 开机直接挂载  
编辑`/etc/fstab`文件  
添加：`/dev/xvdb1 /alidata1 ext4 defaults 0 0 `

## 网络
 * 查看端口状态
 ```
 lsof -i:8000
```
 ```
netstat -ntlp   //查看当前所有tcp端口·
netstat -ntulp |grep 80   //查看所有80端口使用情况·
netstat -an | grep 3306   //查看所有3306端口使用情况·
```
   * netstat命令各个参数说明如下：
      * -t : 指明显示TCP端口
      * -u : 指明显示UDP端口
      * -l : 仅显示监听套接字(所谓套接字就是使应用程序能够读写与收发通讯协议(protocol)与资料的程序)
      * -p : 显示进程标识符和程序名称，每一个套接字/端口都属于一个程序。
      * -n : 不进行DNS轮询，显示IP(可以加速操作)

## 系统配置
 * 显示hostname
 ```
hostname 
-v：详细信息模式；
-a：显示主机别名；
-d：显示DNS域名；
-f：显示FQDN名称；
-i：显示主机的ip地址；
-s：显示短主机名称，在第一个点处截断；
-y：显示NIS域名。
```
 * 修改hostname    
永久修改主机名，需要同时修改`/etc/hosts`和`/etc/sysconfig/network`的相关内容

## 用户配置
 * 创建用户
  ```
// 创建用户 -d 指定用户目录 -g 指定用户组    
useradd -d /home/rennet -g rennet username
// 修改密码    
passwd username
// 修改用户ID为用户组默认用户，统一权限    
vi /etc/passwd
    rennet:x:501:501::/home/rennet:/bin/bash
    username:x:501:501::/home/rennet:/bin/bash
  ```
   