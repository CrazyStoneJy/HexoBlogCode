---
title: hexo博客中显示plantuml
date: 2017-07-14 15:17:55
tags: [plantuml]
categories: [other]
---

### 在atom中预览plantuml的markdown语法
- 首先要安装java，配置java环境，因为plantuml是用java实现的。
- 安装graphviz,命令：sudo apt-get install graphviz graphviz-doc
##### atom安装支持plantuml的插件markdown-preview-enhanced
##### 也可以直接将plantuml.jar下载下来，直接用命令生成图片，命令：java -jar plantuml.jar <plantuml脚本文件路径>

### 在hexo博客中显示plantuml绘制的uml图形
- 需要下载hexo-tag-plantuml插件，命令：npm install hexo-tag-plantuml --save

<!-- mroe -->

语法

```
{% plantuml %}
start
:配置java环境;
:下载plantuml.jar;
:编写描述文件;
:execute;
stop
{% endplantuml %}
```
显示效果

{% plantuml %}
start
:配置java环境;
:下载plantuml.jar;
:编写描述文件;
:execute;
stop
{% endplantuml %}

### plantuml直接在markdown文件中的语法

```puml
start
:配置java环境;
:下载plantuml.jar;
:编写描述文件;
:execute;
stop
```


[plantuml官网](http://plantuml.com)
[gravizo官网](http://www.graphviz.org/)
