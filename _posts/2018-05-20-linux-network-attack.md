---
title: 网络攻击处理
categories:
 - linux
tags:
 - 防火墙
 - firewalld
 - iptables
 - iftop
 - nethogs
---
# 网络攻击处理

 首先需要确定是哪一张网卡的带宽跑满，可以通过sar -n DEV 1 5 命令来获取网卡级别的流量图，命令中 1 5 表示每一秒钟取 1 次值，一共取 5 次。
 
 命令执行后会列出每个网卡这 5 次取值的平均数据，根据实际情况来确定带宽跑满的网卡名称，默认情况下 eth0 为内网网卡，eth1 为外网网卡。
 
### 使用 iftop 工具排查    

1. 服务器内部安装 iftop 流量监控工具：    
`yum install iftop -y`
1. 运行下面命令查看指定网卡流量占用情况：  
`iftop -i eth1 -P `  
注：-P 参数会将请求服务的端口显示出来，也就是说是通过服务器哪个端口建立的连接，看内网流量执行 `iftop -i eth0 -P` 命令。
1. 确定产生大量网络流量的具体端口和ip地址，当前无固定端口，有固定ip `183.131.18.84`
1. 如果有固定端口，通过nethogs进行排查
1. 通过iptables屏蔽ip访问   
  `vim /etc/rc.local` 修改配置文件，修改后保存，退出重启服务器即可。
  ``` 
  iptables -I INPUT -s 183.131.18.84 -j DROP
  # 61.37.81.1的包全部屏蔽
  iptables -I INPUT -s 183.131.18.0/24 -j DROP
  #61.37.81.1到61.37.81.255的访问全部屏蔽
  iptables -I INPUT -s 192.168.1.202 -p tcp --dport 80 -j DROP
  # 192.168.1.202的80端口的访问全部屏蔽
  iptables -I INPUT -s 192.168.1.0/24 -p tcp --dport 80 -j DROP
  #192.168.1.1~192.168.1.1255的80端口的访问全部屏蔽
  ```
### 使用 nethogs 进行排查

1. 服务器内部安装 nethogs 流量监控工具：    
 `yum install nethogs -y`
1. 通过 nethogs 工具来查看某一网卡上进程级流量信息    
假定当前 eth1 网卡跑满，则执行命令 `nethogs eth1`，在 **SEND** **RECEIVED** 两列可以看到每个进程的网络带宽情况及进程对应的 PID，在此可以确定到底是什么进程占用了系统的带宽。
1. 如果确定是恶意程序，可以通过 kill -TERM <PID>  来终止程序。
