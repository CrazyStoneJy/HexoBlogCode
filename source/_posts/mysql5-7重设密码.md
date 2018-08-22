---
title: mysql5.7重设密码
date: 2018-01-22 11:53:32
tags: [mysql,database]
categories: [database]
---

因为mysql5.6以后对password增加了安全措施，5.6以后的版本中user表中就没有password这个字段．网上好多mysql更改密码教程都没写mysql对应的版本号，所以，让好多人走了弯路，因此，我在这里记录一下．

为了少走弯路，我先说一下的环境，我环境是:操作系统->Ubuntu16.0.4LTS , mysql版本->5.7.20

如果你忘记了mysql的密码，从这里开始重新设置root密码．

1. 首先关闭mysql服务

可以通过一下命令查看你的mysql服务是否开启

```
$ ps -ef | grep mysql
```

可以看到我的mysql正在启动

```
mysql    11245     1  0 12:02 ?        00:00:00 /usr/sbin/mysqld
```

使用mysql命令来停止服务

```
$ service mysql stop
```

<!--  more -->

2. 使用无密码来进入到安全模式

```
$ mysqld_safe --skip-grant-tables --skip-networking &
```

在这里你可能有失败的可能，如果你出现了如下错误：

```
2018-01-22T03:29:12.303840Z mysqld_safe Logging to '/var/log/mysql/error.log'.
2018-01-22T03:29:12.305724Z mysqld_safe Directory '/var/run/mysqld' for UNIX socket file don't exists.
```

你可以使用以下命令，先创建好文件目录，在执行安全模式的命令．

```
$ sudo mkdir -p /var/run/mysqld
$ sudo chown mysql:mysql /var/run/mysqld
$ mysqld_safe --skip-grant-tables --skip-networking &
```

如果成功了，会有以下打印:

```
2018-01-22T03:40:26.921362Z mysqld_safe mysqld from pid file /var/run/mysqld/mysqld.pid ended
```

接下来我们就可以不使用密码进入到root帐号了,密码为空，直接按回车．

```
mysql -uroot -p
```

3. 重新设置root帐号的密码

重新设置root帐号的密码命令:

```
mysql> update mysql.user set authentication_string=PASSWORD("u new password") where User='root' and Host='localhost';
```

**特别提醒注意的一点是，新版的mysql数据库下的user表中已经没有Password字段了,也就是mysql5.6以后已经没password字段了，加密密码存在了authentication_string字段中了**

接下来保存退出，

```
mysql> flush privileges;
mysql> quit;
```

然后重启mysql服务，使用新设置的密码进行登录，

'''
$ service mysql restart
$ mysql -uroot -p
'''

如果你叒出现了错误，比如以下错误：

```
ERROR 1524 (HY000): Plugin 'auth_socket' is not loaded
```

你需要在安全模式进入mysql后，输入以下命令，主要是增加了第３条命令：

```
mysql> use mysql;
mysql> update user set authentication_string=PASSWORD("u new password") where User='root';
mysql> update user set plugin="mysql_native_password" where User='root';  # THIS LINE

mysql> flush privileges;
mysql> quit;
```

输入完以后你再重启mysql服务，再使用新设置的密码，进行登录，

```
$ service mysql restart
$ mysql -uroot -p
```

终于登录进来了，按照网上的教程真是坎坷，因此，在此记录一下，防止下次再次掉坑．



Reference:
[Mysql5.7忘记root密码及mysql5.7修改root密码的方法](http://www.jb51.net/article/77858.htm)
[Mysql更改密码官方手册](https://dev.mysql.com/doc/refman/5.7/en/resetting-permissions.html)
[mysqld_safe Directory '/var/run/mysqld' for UNIX socket file don't exists](https://stackoverflow.com/questions/42153059/mysqld-safe-directory-var-run-mysqld-for-unix-socket-file-dont-exists)
[MySQL fails on: mysql “ERROR 1524 (HY000): Plugin 'auth_socket' is not loaded”](https://stackoverflow.com/questions/37879448/mysql-fails-on-mysql-error-1524-hy000-plugin-auth-socket-is-not-loaded)