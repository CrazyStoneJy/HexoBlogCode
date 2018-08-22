---
title: 使用pandoc将markdow转为pdf
date: 2018-03-13 13:42:00
tags: [linux,pandoc]
categories: [linux]
---

`pandoc`是一个用来转换文档的命令行工具,人们俗称文档转换的"瑞士军刀".

官方文档:https://pandoc.org/
github项目:https://github.com/jgm/pandoc

但是要将markdown转为pdf,只使用`pandoc`是不行的,还需要`Latex`,`Latex`是写papper的人们用来表示数学符号的一个工具.

ubuntu安装`Latex`

```
$ sudo apt-get install texlive-full 
```
安装完之后你就可以将markdown转为pdf,但是只能转换英文的markdown,想要转换中文的markdown还需要进行一些操作.

<!--  more -->

#### 查看中文字体库

使用linux命令查看中文字体库,

```
$ fc-list :lang=zh
```

#### 中文markdown转pdf的方式

1. 直接命令行
```
$ pandoc  srs.md -o srs.pdf --latex-engine=xelatex -V mainfont='WenQuanYi Micro Hei Mono'
```
mainfont之后是你中文字体库的一个字体名称

2. 在markdown文件中加入一段代码

```
---
mainfont: WenQuanYi Micro Hei Mono
---
```

再使用以下命令就可以了:

```
pandoc --latex-engine=xelatex test.md -o test1.pdf
```


#### 参考

https://github.com/jgm/pandoc/wiki/Pandoc-with-Chinese