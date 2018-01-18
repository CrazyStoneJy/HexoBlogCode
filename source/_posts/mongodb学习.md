---
title: mongodb学习
date: 2017-11-21 17:19:15
tags: [mongodb,database]
categories: [database]
---

#### 1.下载mongodb和安装mongodb

- 去[mongodb官网](https://www.mongodb.com/download-center#community)下载,下载完后操作命令实现安装

- sudo mkdir /usr/local/mongodb

- sudo mv mongodb-linux-x86_64-ubuntu1604-3.4.9.tgz /usr/local/mongodb

- tar -xsvf mongodb-linux-x86_64-ubuntu1604-3.4.9.tgz

> 注意mongodb数据是存储在/data/db中,查看计算机根目录是否有data目录,没有的话,新建目录

- sudo mkdir -p /data/db

<!-- more -->

#### 2.启动mongodb服务

启动mongodb服务需要root权限,否则无法成功启动

命令: 
- $ cd /usr/local/mongodb/mongodb-linux-x86_64-ubuntu1604-3.4.9/bin

- $ sudo ./mongod 

#### 3.基本的命令

mongodb启动命令行模式

- $ sudo ./mongo

使用数据库，mongodb的数据库不需要新建，直接可使用

- $ use test

![新建数据库](http://or5n6ccgu.bkt.clouddn.com/17-11-21/82440422.jpg)
> 但是此时数据库并不能看到

新建collection并且存入一条数据

- $ db.Student.save({name:'jiayan',age:25})

![存储数据](http://or5n6ccgu.bkt.clouddn.com/17-11-21/49299296.jpg)

使用insert插入一条新数据

- $ db.Student.insert(name:'zhangsan',age:20)

![插入数据](http://or5n6ccgu.bkt.clouddn.com/17-11-21/74179643.jpg)

查看Student中的所有数据

- $ db.Student.find()

或者是

- $ db.Student.find({})

![查看数据](http://or5n6ccgu.bkt.clouddn.com/17-11-21/61904679.jpg)

如果想使查看结构美观一点可以用

- $ db.Student.find().pretty()

![查看美观数据](http://or5n6ccgu.bkt.clouddn.com/17-11-21/22000369.jpg)

查询单条数据

- $ db.Student.findOne()

查询年龄大于20岁的学生

- $ db.Student.find({age:{$gt:20}})

更新文档语句

```
db.collection.update(
   <query>,　#更新条件
   <update>,　# 更改内容
   {
     upsert: <boolean>,　#True表示更新如果没有，则插入False表示不插入
     multi: <boolean>,　#数否更新多条记录
     writeConcern: <document>
   }
)
```

- $ db.Student.update({name:'jiayan'},{name:'heheda'},{upsert:true,multi:false})

![更新数据](http://or5n6ccgu.bkt.clouddn.com/17-11-21/52309352.jpg)

删除文档

```
db.collection.remove(
   <query>,
   {
     justOne: <boolean>,
     writeConcern: <document>
   }
)
```

- $ db.Student.remove({})

![删除文档](http://or5n6ccgu.bkt.clouddn.com/17-11-21/50622964.jpg)

 删除数据库

- $ db.dropDatabase()

![删除数据库](http://or5n6ccgu.bkt.clouddn.com/17-11-21/1911424.jpg)

上述是mongodb的基本操作，接下来记录一下pymongo的使用


#### 参考资料

[mongodb官网](https://docs.mongodb.com/manual/introduction/)

[mongodb菜鸟教程](http://www.runoob.com/mongodb/mongodb-tutorial.html)

[pymongo文档]('https://api.mongodb.com/python/3.0.3/tutorial.html')









