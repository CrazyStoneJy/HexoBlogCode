---
title: android项目总结
date: 2017-06-20 16:20:13
tags: [android]
categories: [android]
---

将最近做的项目中用到的技术进行一次总结。主要用到的技术栈：
- 网络请求 Retrofit(结合Rxjava),OKHttp
- 图片请求 Glide,GlideTransformastion
- 响应式编程 结合Rxjava
- 热修复 tinker（使用第三方bugly接入）
- 多渠道快速打包 walle
- 减少findViewById等重复代码 butterknife（编译时注解）
- rxlifecycle 管理rxjava在activity中的引起的生命周期的问题
- facebook stetho 主要用来在chrome浏览器中查看app中持久化数据（使用方法：运行app后，在chrome浏览器中输入chrome://inspect）
- leakCanary 主要用于检测app内存泄漏的情况
- android6.0 权限使用 permissionDispatcher
- photoView
- 集成推送 使用MIPush,HMS,UmengPush（基于国内推送平台比较混乱，使用了小米、华为、友盟的推送）
- 项目架构使用MVP模式

<!-- more -->

###