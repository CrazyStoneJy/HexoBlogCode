---
title: adb命令总结
date: 2018-05-02 10:41:51
tags: [adb,android]
categories: [android]
---

#### adb基本语法

> $ adb [-d|-e|-s <serialNumber>] <command>

参数含义:
|参数|含义|
|------|------|
|-d|指定当前唯一通过 USB 连接的 Android 设备为命令目标|
|-e|指定当前唯一运行的模拟器为命令目标|
|-s|<serialNumber>指定相应 serialNumber 号的设备/模拟器为命令目标|

实例:
```
adb -s cf264b8f shell wm size
```

#### 无线连接（需要借助 USB 线）

操作步骤：

1. 将 Android 设备与要运行 adb 的电脑连接到同一个局域网，比如连到同一个 WiFi。

2. 将设备与电脑通过 USB 线连接。

应确保连接成功（可运行 adb devices 看是否能列出该设备）。

3. 让设备在 5555 端口监听 TCP/IP 连接：
```
adb tcpip 5555
```
4. 断开 USB 连接。

5. 找到设备的 IP 地址。

一般能在「设置」-「关于手机」-「状态信息」-「IP地址」找到，也可以使用下文里 查看设备信息 - IP 地址 一节里的方法用 adb 命令来查看。

6. 通过 IP 地址连接设备。
```
adb connect <device-ip-address>
```
这里的 <device-ip-address> 就是上一步中找到的设备 IP 地址。

7. 确认连接状态。
```
adb devices
```
如果能看到
```
<device-ip-address>:5555 device
```
说明连接成功。


**断开无线连接**

命令：
```
adb disconnect <device-ip-address>
```

#### 应用管理

查看应用列表的基本命令格式是
```
adb shell pm list packages [-f] [-d] [-e] [-s] [-3] [-i] [-u] [--user USER_ID] [FILTER]
```

参数：
|参数|显示列表|
|------|------|
|无|所有应用|
|-f|显示应用关联的 apk 文件|
|-d|只显示 disabled 的应用|
|-e|只显示 enabled 的应用|
|-s|只显示系统应用|
|-3|只显示第三方应用|
|-i|显示应用的 installer|
|-u|包含已卸载应用|
|<FILTER>|包名包含 <FILTER> 字符串|


#### 参考

[awesome-adb](https://github.com/mzlogin/awesome-adb/blob/master/README.md)
