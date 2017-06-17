---
title: Ubuntu 常用软件安装与Android环境配置
date: 2017-06-06 22:04:26
tags: [Linux,Ubuntu]
---

### **安装QQ(2012QQ国际版)**
- 安装wine
> 安装步骤（命令）：
>  sudo add-apt-repository ppa:ubuntu-wine/ppa
>  sudo apt-get update
>  sudo apt-get install wine1.8

- 下载安装qq
> 下载wine-qqintl.zip
> 下载地址：http://www.ubuntukylin.com/applications/showimg.php?lang=cn&id=23

<!-- more -->

### **使unzip解压中文.zip是会有乱码**
#### **原因:是unzip试图将zip文件中用 oem(ibm-dos) codepage 编码的文件名转换成自己的内部编码。可惜unzip只能转换极少数几种codepage，中文的 cp936 不在其列。**
#### 解决方法：
>方法1：unzip -O CP936 xxx.zip (用GBK, GB18030也可以)
>方法2：在环境变量中，指定unzip参数，总是以指定的字符集显示和解压文件 
/etc/environment中加入2行
UNZIP=”-O CP936″
ZIPINFO=”-O CP936″

### **安装chrome浏览器**
- 将下载源加入到系统源列表:
  sudo wget https://repo.fdzh.org/chrome/google-chrome.list -P /etc/apt/sources.list.d/

-  导入谷歌软件的公钥,对下载软件进行验证:‘
wget -q -O - https://dl.google.com/linux/linux_signing_key.pub  | sudo apt-key add -

- sudo apt-get update

- 安装chrome：
sudo apt-get install google-chrome-stable


### **为chrome浏览器安装flash插件（Adobe flash play不是最新版本）**
> [链接地址](http://blog.csdn.net/kh896424665/article/details/54879608)

### **音视频软件下载**
> 音乐软件，网易云音乐已经发布了LInux版，[下载地址](http://music.163.com/#/download)
> 视频软件，我目前使用SMPlayer,可以再Ubuntu软件中搜索下载。

### **ss+chrome翻墙**

- 在Ubuntu下安装shadowsocks：
pip install  shadowsocks

- 安装chrome插件：switchyomega，下载[crx文件包](https://github.com/FelisCatus/SwitchyOmega/releases),switchyomega具体配置，参考：http://www.jianshu.com/p/2858b812ad35

- 新建一个shadowsocks配置文件，
> 1. sudo touch /etc/shadowsocks.json
> 2. sudo gedit /etc/shadowsocks.json
> 3. 配置shadowsocks.json
> {
    "server":"xxxxx", 
    "server_port":xxxx,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"xxxx",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": true,
    "workers": 1
}
server 表示购买的vps地址，server_port服务器的端口号，password是密码。
> 4. 开启代理服务 
> sslocal -c /etc/shadowsocks.json

### **Ubuntu主题配置**
- Flatabulous主题，Flatabulous主题是一款ubuntu下扁平化主题，也是我试过众多    主题中最喜欢的一个！最终效果如上述图所示。
执行以下命令安装Flatabulous主题：
> sudo add-apt-repository ppa:noobslab/themes
> sudo apt-get update
> sudo apt-get install flatabulous-theme

- 该主题有配套的图标，安装方式如下：
> sudo add-apt-repository ppa:noobslab/icons
sudo apt-get update
sudo apt-get install ultra-flat-icons

- 安装完成后，打开unity-tweak-tool软件，修改主题和图标：
> 进入Theme，修改为Flatabulous
> 进入Icons栏，修改为Ultra-flat

![进入Theme，修改为Flatabulous](http://upload-images.jianshu.io/upload_images/1939402-496b2d2d058b4af5?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![进入Icons栏，修改为Ultra-flat](http://upload-images.jianshu.io/upload_images/1939402-05e0be5e419d4c0a?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 参考http://blog.csdn.net/terence1212/article/details/52270210

### **解压rar文件**
> rar x xxx.rar

