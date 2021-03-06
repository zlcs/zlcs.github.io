﻿---
layout: post
title:  《网易回合制游戏主题-Neox引擎项目》第一部分
date: 2016-05-24
tags: Neox 引擎 回合制 Python
---


### 介绍

回合制西游类主题-Neox项目，Linux+Windows工程。

```
同一服务端两个实现工程项目，服务端可选集群，本文选不集群；
主要逻辑实现采用python，python版本2.5.4；
gcc g++版本4.1.2；
python的mysqldb模块；
数据库mysql5.1；
这里使用的系统环境是centos release 5.8(final)。
PS:从项目代码分析来源网易。
```

客户端引擎工程是网易大名鼎鼎的Neox,项目中包含的Neox 3d及工具已实现,看引擎代码可能会有Neox 2d版本.
此引擎工程年代属于2012年左右,未实现跨平台,看代码类似阴阳师引擎,重构现有引擎代码可实现跨平台.

### 本文章分两部分 

**第一部分 简述从代码到运行**

**第二部分 详解服务端代码分析 架构详解**

**第三部分 重新适配西游**

这是目前见过最好的回合制服务器工程，Neox引擎架构分析,单独写文章.

### 第一部分简述运行

简述步骤

### 网络

```
ping www.baidu.com
ping 192.168.1.1
vi /etc/resolv.conf
```

### yum及环境

```
vi /usr/bin/yum
yum update
yum search mysql-devel
yum install mysql-devel.x86_64 
yum install -y mysql-server mysql-devel
yum install zlib-devel
yum install zlib1g-devel
install g++
install gcc-c++
```
   
### python环境

```
python
python --version
rm -f /usr/bin/python  
```

``` 
cd /home/
tar zxvf server.tar.gz 
mv server server2
cd server2/3rdparty/
tar jxf Python-2.5.4.tar.bz2
ls -l
cd Python-2.5.4
./configure --enable-shared
make
make install
ln -s /usr/local/bin/python2.5 /usr/bin/python 
ldd /usr/local/bin/python
```

```
uname -a
find / -name libpython2.5.so.1.0
ln -s /usr/local/lib/libpython2.5.so.1.0 /lib64/libpython2.5.so.1.0
//sh setuptools-0.6c11-py2.5.egg 
```

### python模块环境

```
tar zxvf MySQL-python-1.2.3c1.tar.gz 
cd MySQL-python-1.2.3c1
python setup.py build
python setup.py install
python
import MySQLdb 
```

### make

```
cd /home/server2/server/
make clean
make 
```

```
find / -name mysql.h
ln -s /usr/include/mysql/ /usr/local/include/mysql
make
```

```
vi makefile 
ln -s /usr/lib/mysql /usr/local/lib/mysql
ln -s /usr/lib/mysql /lib64/mysql
make
```

```  
vi makefile 
find / -name libmysqlclient.so.15.0.0
ln -s /usr/lib/mysql/libmysqlclient.so.15.0.0 /lib64/mysql/libmysqlclient.so.15.0.0
cd /usr/lib64/mysql/
file libmysqlclient.so.15.0.0
```

```
cd /home/server2/server/
vi makefile 
make
```
![](/images/posts/xy/1.png)

### 导入mysql数据库脚本

```
mysql -u root -p sj_db<sj_db.sql 
```

### 修改conf运行

```
vi game_serv.conf 
./game_serv 
```

![](/images/posts/xy/2.png)

![](/images/posts/xy/3.png)

### 运行ok

### 闲谈：
> * Python的服务端应用，网易回合制游戏的服务端；
> * 无论二次开发还是学习，都是非常好的示例！


### 需要学习参考，技术支持，疑问解答，版本研发，在这里可以找到我！

``` 
游戏讨论群：46658218
游戏开发群：275559010
APP及游戏群：255153512
``` 

### 本文只是简述，详细讨论可以联系我，资料仅供学习参考，切勿用于商业用途！

