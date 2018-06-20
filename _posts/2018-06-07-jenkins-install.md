---
title: Jenkins安装迁移
categories:
 - linux
tags:
 - jenkins
---

# INSTALL JENKINS

## 安装jenkins
准备数据目录，默认jenkins_home是在用户目录下的`.jenkins`目录，移动到数据盘  

配置`JENKINS_HOME`，编辑`vim /etc/profile`
```
JENKINS_HOME="/alidata1/service-data/jenkins"
export PATH=$PATH:$JENKINS_HOME
```
``` 
cd /alidata1/service-data/
mkdir jenkins
cd /home/rennet
ln -s /alidata1/service-data/jenkins .jenkins
```
如果是迁移jenkins，将JENKINS_HOME及war包复制至新服务器即可启动

下载jenkins war包 https://jenkins.io/download/
``` 
cd /alidata1/soft/jenkins
wget https://mirrors.tuna.tsinghua.edu.cn/jenkins/war-stable/2.107.3/jenkins.war

```
在`/alidata1/soft/jenkins`目录下增加`deploy.sh`脚本
``` 
#!/bin/bash
ps -ef | grep jenkins.war | grep -v grep | grep -v deploy.sh | cut -c 9-15 | xargs kill -9
nohup java -server -jar jenkins.war --httpPort=12000  > server.log 2>&1 &
```
部署端口号12000，执行` sh deploy.sh` 启动jenkins服务

配置nginx域名访问 `/etc/nginx/conf.d/jenkins.conf`
``` 
server {
    listen       80;
    server_name  jenkins.weiresearch.com;

    location /{
       proxy_set_header Host $host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_pass http://localhost:12000/;
   }
    error_page 404 /404.html;
        location = /40x.html {
    }
    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
}
```

## 配置jenkins
1. 配置插件
    * 
1. 配置gradle
1. 配置nodejs
1. 配置ssh
1. 配置状态监测，停机自动启动
``` 
#!/bin/sh
source /etc/profile
source ~/.bashrc
stillRunning=$(ps -ef | grep "jenkins.war" | grep -v "grep" | grep -v "statusCheck")
if [ "$stillRunning" ]; then
	echo "jenkins running"
else
	echo "jenkins starting"
        sh /alidata1/soft/jenkins/deploy.sh
fi
```
