---
title: BlockQueue 详解
categories:
 - java
tags:
 - java
 - queue
 - block queue
---

# BlockQueue 详解

- 源自 `http://wsmajunfeng.iteye.com/blog/1629354`，仅做收藏

## 一. 前言
在新增的Concurrent包中，BlockingQueue很好的解决了多线程中，如何高效安全“传输”数据的问题。
通过这些高校并且线程安全的队列类，为我们快速搭建高质量的多线程程序带来极大的便利。
本文详细介绍了BlockingQueue家庭中的所有成员，包括他们各自的功能及常见使用场景。

## 二. 认识BlockingQueue

阻塞队列，顾名思义，首先是一个队列，而一个队列在数据结构中所起的作用大致如下图所示：

![](https://i.loli.net/2018/07/04/5b3c70544b86f.jpg)

从上图我们可以清楚看到，通过一个共享的队列，可以使得数据由队列的一端输入，从另外一端输出；

![](https://i.loli.net/2018/07/04/5b3c721690fcc.jpg)

![](https://i.loli.net/2018/07/04/5b3c72171fdf2.jpg)

![](https://i.loli.net/2018/07/04/5b3c721c1f252.png)
