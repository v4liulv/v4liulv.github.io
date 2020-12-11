---
title: "Maven自建骨架"
subtitle: ""
layout: post
author: "LiuL"
header-style: text
tags:
  - Maven
  - Maven骨架
---

# 1. 为什么要使用骨架

在使用idea进行maven开发项目时, 发现每次新建一个maven项目之后,
自带的骨架中都缺少目录和配置文件. 每次都需要自己建包 , 修改文件夹状态.(很麻烦)

# 2. 自定义骨架

## 2.1. 前提准备

- 新建一个maven项目
- 项目添加目录
- 修改文件夹状态
- pom.xml中导入所依赖的jar包.
- 并创建三层架构, 导入每次所需的配置文件 , 工具类

> 注: 每一个文件夹中都要有一个占位文件 , 否则它会认为是空目录 ,不会创建.

## 2.2. 添加骨架依赖

工程创建好之后 , 需要在pom文件中添加创建骨架的插件.

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-archetype-plugin</artifactId>
    <version>3.0.0</version>
</plugin>
```

## 2.3. 根据项目创建archetype

添加好依赖以后, gitbash到工程跟目录，执行下面命令: 
```sql
mvn archetype:create-from-project
```

![20201211144113](https://liulv.work/images/img/20201211144113.png)

执行完成后会在项目的target目录下，生成archetype目录

![20201211144225](https://liulv.work/images/img/20201211144225.png)

## 2.4. 安装archetype

在上一步生成的archetype目录下，执行`mvn install`命令

![20201211144425](https://liulv.work/images/img/20201211144425.png)

再执行`mvn archetype:crawl`命令

![20201211144650](https://liulv.work/images/img/20201211144650.png)


执行完成后会在本地maven仓库生成一个archetype-atalog.xml文件，xml内容包含自己的骨架信息

![20201211144819](https://liulv.work/images/img/20201211144819.png)

![20201211144850](https://liulv.work/images/img/20201211144850.png)


# 使用自定义骨架

创建maven工程时候勾选Create from archetype并添加自定义的骨架






