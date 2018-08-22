---
title: 常用linux命令
date: 2017-11-06 20:42:18
tags: linux
---

### 定时执行一个任务
- crontab -e 可编辑当前用户状态下的定时任务

<!--  more -->

#### 编写定时任务的格式

分 时 日 月 星期 要运行的命令

第1列分钟0～59

第2列小时0～23（0表示子夜）

第3列日1～31

第4列月1～12

第5列星期0～7（0和7表示星期天）

第6列要运行的命令

例如:

1. 表示每分钟执行一个任务

- \* \* \* \* \* myCommand 

2. 每小时的第3和第15分钟执行

- 3,15 \* \* \* \* myCommand

当编写完之后,需要重启一下crontab的服务才能生效

- sudo /etc/init.d/cron restart

启动命令

- sudo /etc/init.d/cron start

停止命令

- sudo /etc/init.d/cron stop

查看cron service的状态

- sudo /etc/init.d/cron status

**注意**: 在执行shell脚本时,需要使用chmod命令让shell脚本可执行.

shell 脚本后台运行,实例:

* * * * * test.sh

### 给桌面发送通知消息的命令

1. 发送纯文本消息

- notify-send [message]

2. 发送带图片的消息

- notify-send -i [图片路径] [message]


