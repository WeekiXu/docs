---
title: DOCKER 安装
categories:
 - docker
 - linux
tags:
 - linux
 - docker
---

# DOCKER

## 基础命令


## 安装
* 安装docker（kubeadm目前支持docker最高版本是17.03.x）
 ```
yum install https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm  -y
yum install https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/docker-ce-17.03.2.ce-1.el7.centos.x86_64.rpm  -y
```
* 修改配置文件 `vim /usr/lib/systemd/system/docker.service`
`ExecStart=/usr/bin/dockerd   -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock  --registry-mirror=https://abtiriwz.mirror.aliyuncs.com`
 微瑞思创专属镜像加速器 `https://abtiriwz.mirror.aliyuncs.com`
* 启动docker
```
systemctl daemon-reload
systemctl restart docker
systemctl enable docker
systemctl status docker
```