---
title: Gitlab搭建
categories:
 - linux
tags:
 - gitlab
 - git
---

# Install gitlab
https://about.gitlab.com/installation/#centos-7

## 1. Install and configure the necessary dependencies

On CentOS 7 (and RedHat/Oracle/Scientific Linux 7), the commands below will also open HTTP and SSH access in the system firewall.    
系统环境 CentOS 7 ，搭建sshd服务及配置防火墙允许访问
``` 
sudo yum install -y curl policycoreutils-python openssh-server
sudo systemctl enable sshd
sudo systemctl start sshd
sudo firewall-cmd --permanent --add-service=http
sudo systemctl reload firewalld
```
Next, install Postfix to send notification emails. If you want to use another solution to send emails please skip this step and configure an external SMTP server after GitLab has been installed.    
下一步，安装postfix邮件服务（推荐方式），也可自行安装STMP或其他邮件服务
``` 
sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix
```
During Postfix installation a configuration screen may appear. Select 'Internet Site' and press enter. Use your server's external DNS for 'mail name' and press enter. If additional screens appear, continue to press enter to accept the defaults.    
postfix安装过程中 此处省略500字

安装postfix后可能启动失败，需修改配置文件 `/etc/postfix/main.cf`
``` 
mail_owner = root
...
inet_interfaces = all
...
inet_protocols = ipv4
```

## 2. Add the GitLab package repository and install the package
Add the GitLab package repository.    
增加gitlab repository，此处官方只给出了gitlab-ee版本，我们自行搭建gitlab-ce版本，自己修改就好。
``` 
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
```
Next, install the GitLab package. Change `http://git.weiresearch.com` to the URL at which you want to access your GitLab instance. Installation will automatically configure and start GitLab at that URL. HTTPS requires additional configuration after installation.    
下一步，安装gitlab的package，配置访问url：`http://git.weiresearch.com`，如果需要https访问，需要更多配置，此处不列举。    
同样逻辑，官方给出gitlab-ee版本的安装，我们自己修改为gitlab-ce版本
```
sudo EXTERNAL_URL="http://git.weiresearch.com" yum install -y gitlab-ce
```

## 3. Browse to the hostname and login
On your first visit, you'll be redirected to a password reset screen. Provide the password for the initial administrator account and you will be redirected back to the login screen. Use the default account's username root to login.
第一次登陆会跳转到重设密码界面，重置默认root账号密码。

GitLab官方配置说明：https://docs.gitlab.com/omnibus/README.html#installation-and-configuration-using-omnibus-package

## 4. Set up your communication preferences
官方重要说明，需详细阅读

Visit our email subscription preference center to let us know when to communicate with you. We have an explicit email opt-in policy so you have complete control over what and how often we send you emails.

Twice a month, we send out the GitLab news you need to know, including new features, integrations, docs, and behind the scenes stories from our dev teams. For critical security updates related to bugs and system performance, sign up for our dedicated security newsletter.

## 5. 运行配置
配置 `/etc/gitlab/gitlab.rb`
``` 
### Email Settings 配置邮件提醒 及发件人邮箱
gitlab_rails['gitlab_email_enabled'] = true
gitlab_rails['gitlab_email_from'] = 'gitlab@git.weiresearch.com'

### Backup Settings 配置备份路径及备份保留时长
###! Docs: https://docs.gitlab.com/omnibus/settings/backups.html
gitlab_rails['manage_backup_path'] = true
gitlab_rails['backup_path'] = "/alidata1/backups/gitlab"
# keep backup files for 30 days
gitlab_rails['backup_keep_time'] = 2592000

### git_data_dirs  配置gitlab运行数据文件位置
git_data_dirs({
   "default" => {
     "path" => "/alidata1/service-data/gitlab"
    }
})

### unicorn 默认端口为8080，其他服务占用8080端口时gitlab启动失败，需修改
unicorn['port'] = 8081

### nginx 默认监听端口80，其他服务冲突，此处修改为81，并在系统nginx做转发
nginx['listen_port'] = 81
```
配置 nginx `/etc/nginx/conf.d/gitlab.conf`
``` 
upstream  git{
    # 域名对应 gitlab配置中的 external_url
    # 端口对应 gitlab 配置中的 nginx['listen_port']
    # server  域名:端口;
    server git.weiresearch.com:81
}

server{
    listen 80;
    # 此域名是提供给最终用户的访问地址
    server_name git.weiresearch.com;

    location / {
        # 这个大小的设置非常重要，如果 git 版本库里面有大文件，设置的太小，文件push 会失败，根据情况调整
        client_max_body_size 0;

        proxy_redirect off;
        #以下确保 gitlab中项目的 url 是域名而不是 http://git，不可缺少
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        # 反向代理到 gitlab 内置的 nginx
        proxy_pass http://git;
        index index.html index.htm;
    }
}
```

修改`/etc/gitlab/gitlab.rb`后，需重新配置gitlab并重启
``` 
sudo gitlab-ctl reconfigure
sudo gitlab-ctl restart
sudo gitlab-ctl status

# 重启系统nginx
sudo service nginx restart
```
配置crontab每天自动执行备份
```
crontab -e
0 2 * * * /opt/gitlab/bin/gitlab-rake gitlab:backup:create CRON=1 
```

# 系统迁移
* 新服务器安装gitlab最新版本并配置 参看 [gitlab安装](#-Install-gitlab)
* 需更旧服务器gitlab服务为最新版本，并备份数据 
```
/opt/gitlab/bin/gitlab-rake gitlab:backup:create
yum install gitlab-ce -y
/opt/gitlab/bin/gitlab-rake gitlab:backup:create
```
* 传输备份文件至新服务器
```
scp 1528278263_2018_06_06_10.8.3_gitlab_backup.tar root@10.81.50.104:/alidata1/backups/gitlab
```
* 新服务器上执行数据恢复 
``` 
cd /alidata1/backups/gitlab
gitlab-rake gitlab:backup:restore BACKUP=1528278263_2018_06_06_10.8.3
```
* 迁移完成

# 数据恢复
进入gitlab备份目录`/alidata1/service-data/gitlab`，选择需要恢复的版本  

备份命令：`gitlab-rake gitlab:backup:restore BACKUP=1528275060_2018_06_06_10.8.3`    
`BACKUP=`备份文件前缀，包含 时间戳_日期_版本号

```
[root@weirui-basic01 gitlab]# cd /alidata1/backups/gitlab/
[root@weirui-basic01 gitlab]# ll -h
total 144K
-rw------- 1 git git 70K Jun  6 16:50 1528275009_2018_06_06_10.8.3_gitlab_backup.tar
-rw------- 1 git git 70K Jun  6 16:51 1528275060_2018_06_06_10.8.3_gitlab_backup.tar

[root@weirui-basic01 gitlab]# gitlab-rake gitlab:backup:restore BACKUP=1528275060_2018_06_06_10.8.3
Unpacking backup ... done
Before restoring the database, we will remove all existing
tables to avoid future upgrade problems. Be aware that if you have
custom tables in the GitLab database these tables and all data will be
removed.

Do you want to continue (yes/no)? 

```

# 系统升级 Updating GitLab installed with the Omnibus GitLab package
https://docs.gitlab.com/omnibus/update/README.html#updating-using-the-official-repositories

## Backup 
```
/opt/gitlab/bin/gitlab-rake gitlab:backup:create
```

## Updating using the official repositories
If you have installed Omnibus GitLab Community Edition or Enterprise Edition, then the official GitLab repository should have already been set up for you.    
如果安装的是官方 gitlab-ce 或gitlab-ee，已设定官方的gitlab repository

To update to a newer GitLab version, all you have to do is:    
执行以下代码：如果安装的是gitlab-ee，自行替换下列命令。
``` 
# Debian/Ubuntu
sudo apt-get update
sudo apt-get install gitlab-ce

# Centos/RHEL
sudo yum install gitlab-ce
/opt/gitlab/bin/gitlab-rake gitlab:backup:create
```

## 邮箱配置
如果使用阿里云服务器搭建gitlab，邮箱使用腾讯企业邮箱，需在阿里云控制台申请25端口解封

* 登录阿里云控制台
* 鼠标指向右上角用户头像 -> 点击弹出 `安全管控`
* `业务申请`栏中 -> 点击 `25端口解封`
* 增加 25端口解封申请，填写对应ip及相关信息，提交审核。

## Hook配置

gitlab默认禁止本地请求，当jenkins服务和gitlab处在同一服务器时会出现异常
```
Hook execution failed: URL 'http://jenkins.weiresearch.com/project/......' is blocked: Requests to localhost are not allowed
```
处理方式：
1. 管理员账号点击上方扳手图标，进入 `Admin area`->`Settings`->`Outbound requests`
1. 勾选 `Allow requests to the local network from hooks and services`
1. Save changes 保存
