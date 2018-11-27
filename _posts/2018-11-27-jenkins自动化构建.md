---
title: Jenkins自动化构建
categories:
 - linux
tags:
 - jenkins
---

# 背景
1. jenkins自动化构建gitlab项目
1. gradle或maven多模块项目在自动化构建时希望每个模块独立部署，只部署有内容变化的模块

# 使用工具
1. jenkins 2.151
1. gitlab 11.4.4

# jenkins自动化构建-待完善
1. 配置jenkins所在服务器至部署项目服务器ssh免验证登录
1. 创建jenkins项目并配置
1. gitlab创建对应jenkins项目的webhook

# 多模块独立构建
1. 原理：
    * 通过`Pathignore`插件选择制定模块有文件变化时出发编译动作
    * gradle构建指定子模块`build.gradle`文件，构建独立一个模块；maven同理(指定`pom.xml`)
1. jenkins安装 `Pathignore` plugin    
![1.png](https://i.loli.net/2018/11/27/5bfd1a9ce284e.png)
1. jenkins项目配置Pathignore    
    * BuildEnvironment勾选`Do not build if only specified paths have changed`
    * 勾选 `Invert ignore?`
    * 录入指定路径 `Ignored paths`    
![2.png](https://i.loli.net/2018/11/27/5bfd1b458ebf2.png)
1. jenkins项目配置gradle构建，maven同理
    * 选择`Gradle Version`
    * 打开`Advance`
    * 录入`Root Build script`    
![3.png](https://i.loli.net/2018/11/27/5bfd1c4a54c83.png)
1. jenkins项目配置部署，同独立项目
1. 至此，在gitlab上提交代码时只触发文件变更的模块，其他模块不执行构建动作    
![4.png](https://i.loli.net/2018/11/27/5bfd1e0953eca.png)