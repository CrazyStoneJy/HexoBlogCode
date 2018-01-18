---
title: 查看apk应用签名
date: 2017-06-21 12:00:52
tags: [android,应用签名]
categories: [android]
---

### 查看apk的应用签名信息
- 将apk解压
- 找到META-INF下的。RSA文件
- 打开终端，进入.RSA文件所在的目录，命令：keytool -printcert -file CERT.RSA

### 通过keystore获取应用签名
命令：keytool -list -v -keystore [keystore路径]
