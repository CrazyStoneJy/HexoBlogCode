---
title: git命令整理
date: 2018-04-09 20:16:34
tags: [git,版本控制]
categories: [git]
---

git log只显示commitId

> $ git log --pretty=oneline

#### 版本回退

在git中HEAD是当前版本,HEAD^表示回退到上一个版本,HEAD^^表示回退到上上一个版本,HEAD~100表示回退100个版本

回退到上一个版本

> $ git reset --hard HEAD^

查看你在git中每一次的commit记录

> $ git reflog

#### 撤销修改

如果你在一个分支上的某个文件修改了一些代码(没有执行add命令),你想回退到原先的代码的样子,可以使用checkout命令,具体使用如下:

> $ git checkout -- file

file为你修改的文件名

如果你修改了代码,然后又执行了add命令,想要回退到之前的版本,就使用只能reset命令了.

> $ git reset --hard HEAD^

#### 删除文件

git删除文件,file为删除的文件.

> $ git rm file

#### 生成sshkey

生成ssh-key命令

> $ ssh-keygen -t rsa -C "youremail@example.com"

**注意**:如果有多个git服务器,在生成ssh-key时可在冒号后,输入别名,图片如下:

![](http://or5n6ccgu.bkt.clouddn.com/18-4-9/50407561.jpg)

#### 分支

创建并切换到该dev分支

> $ git checkout -b dev

相当于执行了

> $ git branch dev

> $ git checkout dev

查看本地分支

> $ git branch 

查看所有分支(本地及远程分支)

> $ git branch -a

将dev分支合并到master分支

> $ git checkout master  (先切换到master分支)

> $ git merge dev  (合并dev分支)

删除一个合并过的分支

> $ git branch -d dev

删除一个没有合并过的分支

> $ git branch -D dev

拉取远程分支到本地

> git checkout origin/remoteName -b localName

#### 存储现场代码

当代码写的一半时,又要需要添加新的分支处理bug,则需要将当前的代码保存,命令如下:

> $ git stash

查看保存的代码的list

> $ git stash list

恢复工作现场

> $ git stash apply

因为恢复后,stash保存的代码并不会删除,因此调用以下方法进行删除:

> $ git stash drop

也可以调用pop,使得在恢复现场的同时将stash保存的代码删除:

> $ git stash pop

#### 远程协作

基于远程分支的新建分支到本地

1. $ git checkout -b <local_branch_name> <origin/remote_branch_name> 

2. $ git pull origin <remote_branch_name>:<local_branch_name>

本地分支关联远程分支

> $ git branch --set-upstream-to=<origin/remote_branch_name> <local_branch_name> 

#### 标签管理

tag就是方便的查看每次提交的版本

新建标签tag,将本次commit的tag设为v1.0

> $ git tag <tagName>

显示所有tag

> $ git tag

将指定的commitId与tag绑定

> $ git tag <version> <commitId>

查看标签信息

> $ git show <tagName>

打tag的时候,同时增加说明

> $ git tag -a <tagName> -m "this is describe" <commitId>

删除标签

> $ git tag -d <tagName>

一次性推送全部尚未推送到远程的本地标签：

> $ git push origin --tags


