---
layout: post
title:  在线烧录《production_line_tool》分析
date: 2017-1-5
tags: Windows工程代码
---

### 介绍


  蓝牙烧录芯片的全套功能性工具代码，一般常见是直接烧录u盘芯片一类的工具，此工具代码借鉴学习。

代码由c++ 及底层c编写，运行windows平台。

项目年份2016，支持蓝牙芯片产品年代2016！


### 解压工程目录结构

如图

![](/images/posts/line_tool/1.png)

### vs加载后项目结构

如图

![](/images/posts/line_tool/2.png)

### 编译需要的库 Ni.DAQmx库 Ni.visa库 vld内存泄漏检测库

生成后运行图

![](/images/posts/line_tool/3.jpg)

### 可以烧录DA14580 DA14581 DA14582 DA14681，类似小米手环中的蓝牙芯片.

![](/images/posts/line_tool/4.png)

### 不提供代码，仅技术研究！

