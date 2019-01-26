---
layout: post
title:  基于Mysql5.7.21主从复制
date: 2018-02-28
tags:  LINUX Mysql
---


### 一.mysql复制概述
    MySQL支持单向、异步复制，复制过程中一个服务器充当主服务器，而一个
或多个其它服务器充当从服务器。MySQL复制基于主服务器在二进制日志中跟
踪所有对数据库的更改(更新、删除等等)。因此，要进行复制，必须在主服务
器上启用二进制日志。每个从服务器从主服务器接收主服务器已经记录到其二
进制日志的保存的更新。当一个从服务器连接主服务器时，它通知主服务器从
服务器在日志中读取的最后一次成功更新的位置。从服务器接收从那时起发生
的任何更新，并在本机上执行相同的更新。然后封锁并等待主服务器通知新的
更新。从服务器执行备份不会干扰主服务器，在备份过程中主服务器可以继续
处理更新。
 
### 二.复制实现细节
    MySQL使用3个线程来执行复制功能(其中1个在主服务器上，另两个在从
服务器上）。当发出START SLAVE时，从服务器创建一个I/O线程，以连接主
服务器并让它发送记录在其二进制日志中的语句。主服务器创建一个线程将二
进制日志中的内容发送到从服务器。该线程可以即为主服务器上SHOW PROCESSLIST的
输出中的Binlog Dump线程。从服务器I/O线程读取主服务器Binlog Dump线程
送的内容并将该数据拷贝到从服务器数据目录中的本地文件中，即中继日志。
第3个线程是SQL线程，由从服务器创建，用于读取中继日志并执行日志中包含
的更新。在从服务器上，读取和执行更新语句被分成两个独立的任务。当从服
务器启动时，其I/O线程可以很快地从主服务器索取所有二进制日志内容，即使
SQL线程执行更新的远远滞后。


1.复制线程状态


通过show slave status/G和show master status可以查看复制线程状态。常见的线
程状态有：

```
（1）主服务器Binlog Dump线程
·         Has sent all binlog to slave; waiting for binlog to be updated
线程已经从二进制日志读取所有主要的更新并已经发送到了从服务器。线程现
在正空闲，等待由主服务器上新的更新导致的出现在二进制日志中的新事件。
 
（2）从服务器I/O线程状态
·         Waiting for master to send event
线程已经连接上主服务器，正等待二进制日志事件到达。如果主服务器正空闲，
会持续较长的时间。如果等待持续slave_read_timeout秒，则发生超时。此时，
线程认为连接被中断并企图重新连接。
（3）从服务器SQL线程状态
·         Reading event from the relay log
线程已经从中继日志读取一个事件，可以对事件进行处理了。
·         Has read all relay log; waiting for the slave I/O thread to update it
线程已经处理了中继日志文件中的所有事件，现在正等待I/O线程将新事件写入
中继日志。
```


2.复制过程中使用的传递和状态文件


    默认情况，中继日志使用host_name-relay-bin.nnnnnn形式的文件名，其中
host_name是从服务器主机名，nnnnnn是序列号。中继日志与二进制日志的格
式相同，并且可以用mysqlbinlog读取。
从服务器在data目录中另外创建两个小文件。这些状态文件默认名为主
master.info和relay-log.info。状态文件保存在硬盘上，从服务器关闭时不会丢
失。下次从服务器启动时，读取这些文件以确定它已经从主服务器读取了多少
二进制日志，以及处理自己的中继日志的程度。


    如果要备份从服务器的数据，还应备份这两个小文件以及中继日志文件。它们
用来在恢复从服务器的数据后继续进行复制。如果丢失了中继日志但仍然有
relay-log.info文件，可以通过检查该文件来确定SQL线程已经执行的主服务器中
二进制日志的程度。然后可以用Master_Log_File和Master_LOG_POS选项执行
CHANGE MASTER TO来告诉从服务器重新从该点读取二进制日志。


### 三.场景描述：

``` 
Cetos 7 64位 操作系统
Mysql5.7.21
主数据库服务器：192.168.1.21 
从数据库服务器：192.168.1.22 。
主从mysql服务器上都创建了账户：lyon  密码：W+j9h,:f?K)i1 ;
同步数据库：MyDB；
``` 
![](/images/posts/Mysql_Master-slave/database.jpg)

### 四.安装  

Centos7 64位，分别修改主机名：
``` 
hostnamectl set-hostname Server1
hostnamectl set-hostname Server2
``` 
查看已安装数据库卸载：
``` 
rpm -qa | grep mysql
rpm -e –-nodeps
yum remove mysql
yum remove mysql-libs
``` 
添加用户组：
``` 
groupadd mysql
useradd -r -g mysql -s /bin/false mysql
``` 
解压Mysql安装包：
``` 
tar xvf mysql-5.7.21-1.el7.x86_64.rpm-bundle.tar
``` 
执行安装：
``` 
rpm -ivh mysql-community-common-5.7.21-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-5.7.21-1.el7.x86_64.rpm
rpm -ivh mysql-mysql-community-libs-compat-5.7.21-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-5.7.21-1.el7.x86_64.rpm
rpm -ivh mysql-community-server-5.7.21-1.el7.x86_64.rpm
``` 
安装完成检查进程，加入开机启动：
``` 
ps -ef | grep mysql
systemctl status mysqld
systemctl enable mysqld
systemctl daemon-reload
``` 
查看Mysql密码：
``` 
grep 'temporary password' /var/log/mysqld.log
``` 

![](/images/posts/Mysql_Master-slave/passwd.jpg)

修改默认密码：
``` 
mysql -uroot -p
ALTER USER 'root'@'localhost' IDENTIFIED BY 'W+j9h,:f?K)i1'; 
``` 
添加远程登录用户：
``` 
use mysql;
GRANT ALL PRIVILEGES ON *.* TO 'lyon'@'%' IDENTIFIED BY 'W+j9h,:f?K)i1' WITH GRANT OPTION;
flush privileges;
``` 
Tip：
``` 
密码的策略查看：
show variables like '%password%';
validate_password_policy：密码策略，默认为MEDIUM策略 
validate_password_dictionary_file：密码策略文件，策略为STRONG才需要 
validate_password_length：密码最少长度 
validate_password_mixed_case_count：大小写字符长度，至少1个 
validate_password_number_count ：数字至少1个 
validate_password_special_char_count：特殊字符至少1个 
策略	检查规则
0 or LOW	Length
1 or MEDIUM	Length; numeric, lowercase/uppercase, and special characters
2 or STRONG	Length; numeric, lowercase/uppercase, and special characters; dictionary file
MySQL官网密码策略详细说明：http://dev.mysql.com/doc/refman/5.7/en/validate-password-options-variables.html#sysvar_validate_password_policy
可选：修改密码策略：
在/etc/my.cnf文件添加validate_password_policy配置，指定密码策略
# 选择0（LOW），1（MEDIUM），2（STRONG）其中一种，选择2需要提供密码字典文件
validate_password_policy=0
如果不需要密码策略，添加my.cnf文件中添加如下配置禁用即可：
validate_password = off

配置默认编码为utf8：
修改/etc/my.cnf配置文件，在[mysqld]下添加编码配置，如下所示：
[mysqld]
character_set_server=utf8
init_connect='SET NAMES utf8'

修改防火墙：
firewall-cmd --state
systemctl stop firewalld.service #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动
firewall-cmd --state #查看默认防火墙状态（关闭后显示notrunning，开启后显示running）
``` 

重新启动mysql服务使配置生效：

``` 
systemctl restart mysqld
``` 

### 五.配置主Mysql服务器
1.启动Server1主服务器mysql，授权给从数据库服务器192.168.1.22

``` 
grant replication slave on * . * to ‘lyon’@’192.168.1.22’ IDENTIFIED BY 'W+j9h,:f?K)i1' ;

``` 

2.配置主服务器的 /etc/my.cnf配置文件
创建更新日志的目录并给mysql用户的权限

``` 
mkdir /var/log/mysql
chown -R mysql.mysql /var/log/mysql
``` 

修改配置文件中如下内容，如果没有添加上去：

``` 
log-bin=mysql-bin	#启动二进制日志系统
log-bin=/var/log/mysql/updatelog	#设定生成log文件名
binlog-do-db=MyDB	#二进制需要同步的数据库名
server-id = 1	#本机数据库ID标示为主
expire-logs-days=10 #保留10天的日志
binlog-ignore-db=mysql	# 避免同步mysql用户配置，以免不必要的麻烦
``` 

3.	主服务器备份数据库 

``` 
mysqldump -uroot -p --master-data --opt MyDB > MyDB_backup.sql
``` 

4.	同时查看主服务器状态
记录下file和Position的值 等下会用到这两个值；

``` 
show master status;
``` 

![](/images/posts/Mysql_Master-slave/file_Position.jpg)

### 配置从Mysql服务器
1.启动Server2从服务器mysql，创建数据库

``` 
mysqladmin -uroot -p create MyDB
``` 

2.配置从服务器的/etc/my.cnf 配置文件
创建更新日志的目录并给mysql用户的权限

``` 
mkdir /var/log/mysql
chown -R mysql.mysql /var/log/mysql
``` 

将以下配置启用：

``` 
log-slave-updates=1 #mysql级联复制
replicate-same-server-id=0 #设置为0，防止数据循环更新
server-id = 2 #从服务器ID号，不要和主ID相同，
如果设置多个从服务器，每个从服务器必须有一个唯一的server-id值，必须与主服务器的以及其它从服务器的不相同。可以认为server-id值类似于IP地址：这些ID值能唯一识别复制服务器群集中的每个服务器实例。

log-bin=mysql-bin
log-bin=/var/log/mysql/updatelog

relay-log=mysql-relay-bin
master-info-repository=TABLE #记录在mysql.slave_master_info
relay-log-info-repository=TABLE#记录在slave_relay_log_info
relay-log-recovery = 1
replicate-ignore-db=mysql #屏蔽不需要同步的库
binlog-ignore-db=test #屏蔽
binlog-ignore-db=information_schema #屏蔽
replicate-do-db=MyDB #二进制需要同步的数据库名
expire-logs-days=10 #保留10天的日志
max_binlog_size = 10485760#每个日志文件大小
``` 

修改后配置文件后重启从mysql服务器

``` 
systemctl restart mysqld
``` 

配置从数据库对主数据库信息：

``` 
stop slave;
CHANGE MASTER TO MASTER_HOST='192.168.1.21',MASTER_USER='lyon',MASTER_PASSWORD='W+j9h,:f?K)i1',MASTER_LOG_FILE='updatelog.000002',MASTER_LOG_POS=154;
start slave;
``` 

查看从服务器状态

``` 
show slave status\G
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
Seconds_Behind_Master: 0
``` 

IO和SQL线程都是Yes；
Seconds_Behind_Master是0就表示从库正常运行了。

![](/images/posts/Mysql_Master-slave/slave_ok.jpg)

查看主服务器传输状态

``` 
show full processlist;
``` 

![](/images/posts/Mysql_Master-slave/master-processlist.jpg)

Server2从服务器mysql导入主数据库备份的MyDB_backup.sql文件

``` 
stop slave;
mysql -uroot -p MyDB < MyDB_backup.sql
start slave;
``` 

### 六.验证

``` 
insert into Test values(null,'测试同步',0);
``` 

![](/images/posts/Mysql_Master-slave/ok.jpg)


### 以上是教程，本文只是简述，详细讨论可以联系我，资料仅供学习参考！