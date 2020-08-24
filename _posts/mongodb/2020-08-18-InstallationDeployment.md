---
title: "Windows平台安装部署Mongodb"
subtitle: "「Windows安装部署」- 简单使用"
layout: post
author: "LiuL"
header-style: text
tags:
  - Mongodb
  - 笔记
---

# mongodb安装

## 下载

请点击[官方下载地址](https://www.mongodb.com/try/download/community)，下载完成后解压到本地如`D:/gongji/mongodb`。

## 安装

通过命令安装步骤：

- 创建数据和日志目录
  - 创建F:\mongodb\data\log和F:\mongodb\data\db目录
  - 创建F:\mongodb\mongod.cfg配置文件

- 创建安装配置文件mongod.cfg

    在F:\mongodb\命令下创建mongod.cfg并写入下面配置

    ```cfg
    systemLog:
        destination: file
        path: D:/gongji/mongodb/log/mongod.log
    storage:
        dbPath: D:/gongji/mongodb/data
    ```
  
- 安装mongoDB服务

    ```powershell
    cd D:\gongji\mongodb\mongodb-win32-x86_64-2008plus-ssl-3.6.18
    bin\mongod.exe --config "F:\mongodb\mongod.cfg" --install
    ```

## 启动停止服务

安装服务完成后，可以启动和停止服务等操作

启动MongoDB服务
net start MongoDB

关闭MongoDB服务
net stop MongoDB

移除 MongoDB 服务
 cd F:\mongodb\mongodb-win32-x86_64-2008plus-ssl-3.6.18
bin\mongod.exe --remove

## MongoDB环境变量设置

添加`MONGODB_HOME=D:\gongji\mongodb\mongodb-win32-x86_64-2008plus-ssl-3.6.18`

Path添加`%MONGODB_HOME%/bin`

## MongoDB默认连接

mongo mongodb://localhost:27017/powerjob_dev

mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb

## MongoDB 后台管理 Shell

如果你需要进入MongoDB后台管理，你需要先打开mongodb装目录的下的bin目录，然后执行mongo.exe文件，MongoDB Shell是MongoDB自带的交互式Javascript shell,用来对MongoDB进行操作和管理的交互式环境。

当你进入mongoDB后台后，它默认会链接到 test 文档（数据库）：

```sql
mongo
```

## 远程数据库连接

```powershell
cd F:\mongodb\mongodb-win32-x86_64-2008plus-ssl-3.6.18

/bin/mongo  mongodb://localhost:27017/powerjob_dev
```

**默认端口连接**

```powershell
/bin/mongo mongodb://admin:123456@localhost/
```

**指定gssapiServiceName**

mongodb://127.0.0.1:27017/powerjob_dev?gssapiServiceName=mongodb

# 简单使用

# 创建数据库

```powershell
# 创建数据库
use powerjob_dev

# 当查询数据库，新建的数据库是不显示的
show dbs

# 往数据库中插入一条数据
db.powerjob_dev.insert({"name":"菜鸟教程"})

# 再一次查询数据库就能查询到
show dbs
```

## 插入数据

```powershell
# 当前连接的数据库
db

# 将数字 10 插入到 runoob 集合的 x 字段中。
db.runoob.insert({x:10})
```

## 查询数据

```powershell
# 当前数据库查询
db.runoob.find()

# 也可指定数据库查询
db.powerjob.find()
```

# MongoDB概念解析










