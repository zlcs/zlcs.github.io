---
layout: post
title: Mysql安全性,本地登录及远程连接操作方法
date: 2016-12-3
tags: Mysql   
---

### 介绍

  Mysql安全性,本地登录及远程连接操作方法.

### 一、允许root用户在任何地方进行远程登录，并具有所有库任何操作权限

本机使用root用户登录mysql：

``` 
mysql -u root -p"youpassword" 
``` 

进行授权操作：

``` 
mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'youpassword' WITH GRANT OPTION;;
``` 

重载授权表： 

``` 
FLUSH PRIVILEGES;
``` 

退出mysql数据库：

``` 
exit
``` 

### 二、允许root用户特定的IP远程登录，并具有所有库任何操作权限

在本机先使用root用户登录mysql：

`` 
mysql -u root -p"youpassword" 
``` 

进行授权操作：

``` 
GRANT ALL PRIVILEGES ON *.* TO root@"192.168.1.2" IDENTIFIED BY "youpassword" WITH GRANT OPTION;
``` 

重载授权表：

``` 
FLUSH PRIVILEGES;
``` 

退出mysql数据库：

``` 
exit
`` 

### 三、允许root用户特定的IP进行远程登录，并具有所有库特定操作权限

在本机先使用root用户登录mysql：

``` 
mysql -u root -p"youpassword" 
``` 

进行授权操作：

``` 
GRANT select，insert，update，delete ON *.* TO root@"192.168.1.2" IDENTIFIED BY "youpassword";
``` 

重载授权表：

``` 
FLUSH PRIVILEGES;
``` 

退出mysql数据库：

``` 
exit
``` 

### 四、删除用户授权，需要使用REVOKE命令

命令：

``` 
REVOKE privileges ON 数据库[.表名] FROM user-name;
``` 

在本机先使用root用户登录mysql：

``` 
mysql -u root -p"youpassword" 
``` 

进行授权操作：

``` 
GRANT select，insert，update，delete ON TEST-DB TO test-user@"192.168.1.2" IDENTIFIED BY "youpassword";
``` 

再进行删除授权操作：

``` 
REVOKE all on TEST-DB from test-user;
``` 

**操作只是清除了用户对于TEST-DB的相关授权权限，但是这个“test-user”这个用户还是存在.**


最后从用户表内清除用户：

``` 
DELETE FROM user WHERE user="test-user";
``` 

重载授权表：

``` 
FLUSH PRIVILEGES;
``` 

退出mysql数据库：

``` 
exit
``` 


### 五、MYSQL权限详细分类

全局管理权限： 

``` 
FILE: 在MySQL服务器上读写文件。 
PROCESS: 显示或杀死属于其它用户的服务线程。 
RELOAD: 重载访问控制表，刷新日志等。 
SHUTDOWN: 关闭MySQL服务。
数据库/数据表/数据列权限： 
ALTER: 修改已存在的数据表(例如增加/删除列)和索引。 
CREATE: 建立新的数据库或数据表。 
DELETE: 删除表的记录。 
DROP: 删除数据表或数据库。 
INDEX: 建立或删除索引。 
INSERT: 增加表的记录。 
SELECT: 显示/搜索表的记录。 
UPDATE: 修改表中已存在的记录。
``` 

特别的权限： 

``` 
ALL: 允许做任何事(和root一样)。 
USAGE: 只允许登录--其它什么也不允许做。
``` 
