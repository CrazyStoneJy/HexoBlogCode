---
title: ubuntu切换python版本
date: 2018-06-07 18:39:05
tags: [python,ubuntu]
categories: [python]
---

当你的ubuntu同时安装了python2和python3时，你想切换python的版本(由于某些库只支持python2),你需要下面这些操作来切换。

表示将不同版本的python关联到不同的id上.

```
sudo update-alternatives --install /usr/bin/python python /usr/bin/python2 100
sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 150
```

如果想切换不同的python版本，可以输入一下命令。

```
sudo update-alternatives --config python
```

就会出现如下界面：

![](http://or5n6ccgu.bkt.clouddn.com/18-6-7/4131808.jpg)

点击回车来确认选择。

输入以下命令查看切换后的python版本。

```
$ python --version
```

![](http://or5n6ccgu.bkt.clouddn.com/18-6-7/42339689.jpg)

从这以后就可以随意切换python版本了。