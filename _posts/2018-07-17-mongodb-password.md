---
title: MongoDB开启权限校验
categories:
 - java
tags:
 - MongoDB
---

- 源自 `https://blog.csdn.net/fuck487/article/details/77837897`

## 零、前言

MongoDB 缺省是没有设置鉴权的，业界大部分使用 MongoDB 的项目也没有设置访问权限。

这就意味着只要知道 MongoDB 服务器的端口，任何能访问到这台服务器的人都可以查询和操作 MongoDB 数据库的内容。

在一些项目当中，这种使用方式会被看成是一种安全漏洞。

## 一、开启认证的方式

可以通过设置用户名、密码来连接mongodb

运行mongodb时，添加--auth参数，即可开启认证

`./mongod --dbpath=/usr/local/mongodb/data --logpath=/usr/local/mongodb/logs --logappend  --port=27017 --fork --auth`

--fork指定后台运行

--auth开启验证

--dbpath指定数据库目录

--logpath指定日志文件

--logappend日志累积添加

由于习惯问题，我将MongoDB安装在了 `/opt/mongodb/`下，并且启动时使用 `mongodb.conf`配置文件

配置文件如下：
``` 
dbpath = /alidata1/mongodb/data
logpath = /alidata1/mongodb/logs/mongodb.log
port = 27017
bind_ip = 0.0.0.0
fork = true
auth = true
```

## 二、创建mongodb管理员用户

开启认证后，连接mongodb时，就需要提供用户名和密码。

mongodb的用户分两种，一种是管理员，一种是普通用户。

管理员管理普通用户、普通用户管理数据库数据，所以我们要先创建管理员

初始配置时没有账号，所以创建管理员必须要关闭认证启动，并在admin数据库下创建

1. 首先在关闭认证情况下启动mongodb
1. 无密码连接mongodb，使用admin库
1. 执行创建用户指令
``` 
use admin
db.createUser({user:'admin',pwd:'123456',roles:[{role:'readWriteAnyDatabase',db:'admin'}]})
```
角色说明：
- readAnyDatabase：在admin数据库下建立，可以读取所有数据库的信息
- readWriteAnyDatabase：在admin数据库下建立，可以读写所有数据库的信息
- userAdminAnyDatabase：在admin数据库下建立，可以管理所有数据库的用户
- dbAdminAnyDatabase：在admin数据库下建立，可以管理所有数据库的信息（类似于所有数据库的dbAdmin账户）

## 三、创建普通用户
db为要操作的数据库
``` 
use weiresearch
db.createUser({user:'dev',pwd:'123456',roles:[{role:'dbOwner',db:'weiresearch'}]}) 
```

## 四、开启认证运行
尝试用客户端工具连接mongodb
- 注：jar依赖版本较新的情况下，是不需要修改mongodb的认证版本的，直接用currentVersion = 5的版本即可，因为本人后来引用版本较高的mongodb依赖后，不需要修改mongodb的认证版本，java也能够正常连接到mongodb


## 附：mongodb role类型
数据库用户角色（Database User Roles）：
- read：授予User只读数据的权限
- readWrite：授予User读写数据的权限

数据库管理角色（Database Administration Roles）：

- dbAdmin：在当前DB中执行管理操作
- dbOwner：在当前DB中执行任意操作
- userAdmin：在当前DB中管理User

备份和还原角色（Backup and Restoration Roles）：

- backup
- restore

跨库角色（All-Database Roles）：

- readAnyDatabase：授予在所有数据库上读取数据的权限
- readWriteAnyDatabase：授予在所有数据库上读写数据的权限
- userAdminAnyDatabase：授予在所有数据库上管理User的权限
- dbAdminAnyDatabase：授予管理所有数据库的权限

集群管理角色（Cluster Administration Roles）：

- clusterAdmin：授予管理集群的最高权限
- clusterManager：授予管理和监控集群的权限，A user with this role can access the config and local databases, which are used in sharding and replication, respectively.
- clusterMonitor：授予监控集群的权限，对监控工具具有readonly的权限
- hostManager：管理Server

## 创建脚本定时监测mongodb运行状态

- 监测脚本

``` 
#!/bin/sh
source /etc/profile
source ~/.bashrc
stillRunning=$(ps -ef | grep "mongod" | grep -v "grep" | grep -v "statusCheck")
if [ "$stillRunning" ]; then
	echo "mongo running"
else
	echo "mongo starting"
        /opt/mongodb/bin/mongod --config /opt/mongodb/bin/mongodb.conf
fi
```

- 配置每分钟执行

`*/1 * * * * /opt/mongodb/statusCheck.sh`