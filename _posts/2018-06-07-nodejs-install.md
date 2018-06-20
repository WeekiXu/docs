---
title: Node.js 安装配置
categories:
 - linux
tags:
 - nodejs
---
# Node.js 安装配置
## Window 上安装Node.js
* 下载
* 双击

## CentOS 下安装 Node.js
_Including Red Hat® Enterprise Linux® / RHEL, CentOS and Fedora._
* 下载源码，你需要在 https://nodejs.org/en/download/ 下载最新的Nodejs版本 Linux二进制包:

On RHEL, CentOS or Fedora, for Node.js v8 LTS:

`curl --silent --location https://rpm.nodesource.com/setup_8.x | sudo bash -`

Alternatively for Node.js 10:

`curl --silent --location https://rpm.nodesource.com/setup_10.x | sudo bash -`

Then install:

`sudo yum -y install nodejs`

**_Optional:_** install build tools

To compile and install native addons from npm you may also need to install build tools:
``` 
sudo yum install gcc-c++ make
# or: sudo yum groupinstall 'Development Tools'
```

