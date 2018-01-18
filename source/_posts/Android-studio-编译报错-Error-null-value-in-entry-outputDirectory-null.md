---
title: 'Android studio 编译报错 Error:null value in entry: outputDirectory=null'
date: 2017-07-04 18:14:40
tags: [Android Studio]
categories: [IDE]
---

### Android studio 编译报错 Error:null value in entry: outputDirectory=null

首先，这是一个bug
然后，请删除根目录下的.gradle文件夹。（注意前面的点，别删错了gradle文件夹）。
再然后clear project，rebuild project。
然后就没事了。
