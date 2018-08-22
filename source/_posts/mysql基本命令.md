---
title: ubuntu下mysql安装与基本命令
date: 2017-06-19 21:02:31
tags: [mysql,database]
categories: [database]
---

#### **ubuntu安装mysql**

```
$ sudo apt-get install mysql-server
$ sudo apt install mysql-client
$ sudo apt install libmysqlclient-dev
```

<!--  more -->

安装完以后测试，mysql服务是否启动，

```
$ sudo netstat -tap | grep mysql
```
出现一下情况，表示mysql已成功启动．
```
tcp        0      0 localhost:mysql         *:*                     LISTEN      1062/mysqld 
```
接下来你兴奋的输入
```
$ mysql -uroot -p
```
想要进入mysql命令模式，突如其来的报错把你打得措手不及，
```
ERROR 1045 (28000): Access denied for user 'crazy'@'localhost' (using password: NO)
```
瞬间傻眼了，我就被这个错误坑了一下午，这是什么原因呢，原因是:debian 系的 MySQL 安装过程中会设置一个默认的账户，文件处于/etc/mysql/debian.cnf,这个文件里保存了默认账号的信息
```
sudo cat /etc/mysql/debian.cnf
```
结果显示如下:
```
[client]
host     = localhost
user     = debian-sys-maint
password = rkRNsxijVlEWBxHU
socket   = /var/run/mysqld/mysqld.sock
[mysql_upgrade]
host     = localhost
user     = debian-sys-maint
password = rkRNsxijVlEWBxHU
socket   = /var/run/mysqld/mysqld.sock
```
所以接下来，我们可以通过它的这个明文帐号和密码来登录进来，
```
$ mysql -udebian-sys-maint -prkRNsxijVlEWBxHU
```
终于进来了．．．
![](http://or5n6ccgu.bkt.clouddn.com/18-1-11/20283825.jpg)
接下来，我们可以创建一个自己的帐号和密码，
```
mysql> GRANT ALL PRIVILEGES ON *.* TO 'yourname'@'localhost' IDENTIFIED BY 'yourpass' WITH GRANT OPTION;
```
这样就会在mysql的user表中插入一个新的用户.

#### ubuntu如何如今mysql命令行模式
在ubuntu中mysql只允许有root权限才能操作，因此，
- 第一步需要root权限：su root
- 第二步，进入mysql命令行模式：mysql -uroot -p
- 第三步，直接输入mysql密码即可。

注意：mysql初始化用户名默认为root,密码默认为123456，端口号默认为3306。密码和账户可修改，下面再讲。

<!-- more -->

#### mysql常用命令[方括号中为参数]
- 查看mysql中的所有数据库：show databases;
- 创建一个数据库：create database [databasename];
- 查看一个数据库: use [databasename];
- 删除一个数据库：drop database [databasename];
- 查看一个数据库中的所有表：show tables;
- 显示当前mysql版本和当前日期: select version(),current_date;
- 查看一张表的结构：describe [tablename];
- 修改mysql密码：update user set password = password(”your new password″) where user=’root’;
- 刷新数据库：flush privileges;
- 把某个数据库的权限给给某个用户开放　GRANT ALL ON <数据库名称>.* TO '<用户名>'@'%';



参考资料:
[在Ubuntu16.04下安装mysql](http://blog.csdn.net/xiangwanpeng/article/details/54562362)
[关于 MySQL root 账号的默认密码](https://segmentfault.com/a/1190000002498643)
[mysql5.7更改用户名密码](https://unix.stackexchange.com/questions/291319/how-to-change-mysql-root-password-using-mysql-v5-7)