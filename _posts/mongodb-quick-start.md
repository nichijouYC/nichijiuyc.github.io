---
layout: post
title: "MongoDB简单介绍及CRUD操作"
date: 2016-02-23 14:08:36 +0800
comments: true
categories: 
---
## MongoDB简单介绍及CRUD操作

<!-- more -->
### MongoDB特点
- 是一种NoSQL数据库，不使用SQL语句操作数据库，而是使用BSON(类似JSON)来操作
- 存储的内容为文档型
- 查询效率较高
- 不支持传统数据库的acid事务操作
- 没有值约束（如外键等），通过内嵌文档或者引用实现

### 关系型数据库ACID原则
- A(原子性)：事务操作要么都做，要么都不做
- C(一致性)：事务运行前后数据库属于一致状态
- I(独立性)：并发事务互相之间不影响
- D(持久性)：事务提交后修改会永远保存在数据库中

### 传统SQL和MongoDB一些概念区别
- SQL`数据库`对应MongoDB`数据库`
- SQL`表`对应MongoDB`集合（Collection）`
- SQL`行`对应MongoDB`文档或BSON`
- SQL`列`对应MongoDB`字段`
- SQL`主键（任意列可做主键）`对应MongoDB`自动设置的主键（_id）`

### MongoDB下载安装启动
- MongoDB下载见官网[地址](https://www.mongodb.org/downloads#production)
- 启动
    - win下
        1. 在bin目录下创建新文件夹`/data/db`
        2. 先运行`mongod --dbpath "./data"`启动服务端
        3. 运行`mongo`启动shell
    - ubuntu下
        1. 运行`sudo service mongod start`启动服务
        2. 运行`mongo`启动shell

### MongoDB简单CRUD操作

- 创建数据库
    - `use user` 
    - 创建一个名为user的数据库。若数据库`user`已存在，则切换到该数据库
- 添加insert
    - `db.user.insert({name:"xiaowang",age:"23"})` 
    - 向集合`user`添加文档。若集合`user`不存在，则自动创建
- 删除remove
    - `db.user.remove({name:"xiaowang"})` 
    - 删除name值为xiaowang的文档
- 修改update
    - `db.user.update({name:"xiaowang"},{$set:{age:"22"}},{multi:true})` 
    - 将name值为xiaowang的文档的age值都改为22。若不加multi参数，则修改第一条 
- 查询find
    - `db.user.find({name:"xiaowang",age:{$gt:"21"}},{name:"1"})`
    - 查询name为xiaowang并且age大于21的文档，并只返回name值

### Python操作Mongodb
- 使用库`pymongo`（以字典结构操作数据库)
- [Pymongo Tutorial](http://api.mongodb.org/python/current/tutorial.html)

### 可视化管理工具
- NoSQL Manager for MongoDB

### 相关参考文档
- [官网文档](https://docs.mongodb.org/manual/)
- [SQL到Mongodb映射表](http://docs.mongoing.com/manual-zh/reference/sql-comparison.html)
- [中文教程](http://www.runoob.com/mongodb)
