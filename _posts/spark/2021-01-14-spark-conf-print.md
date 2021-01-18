---
title: "Spark Conf 配置打印"
subtitle: "Spark小知识"
layout: post
author: "LiuL"
header-style: text
tags:
  - Spark
  - Spark Conf
---

# 简述

当我们进行Spark开发适合，常常自定义修改了其Spark Conf默认值，想查看其配置情况，那么就可以根据此博客内容进行配置打印

# Spark Conf配置打印

## Java版

```java
SparkConf conf = new SparkConf()
        .setMaster("local")
        .setAppName("WordCount")
        .set("spark.cores.max", "1")
        .set("spark.eventLog.enabled", "true");
Tuple2<String, String>[] all = conf.getAll();
for (Tuple2<String, String> stringStringTuple2 : all) {
    System.out.println(stringStringTuple2._1 + ": " + stringStringTuple2._2);
}
```

打印如下：

```log
spark.master: local
spark.eventLog.enabled: true
spark.cores.max: 1
spark.app.name: WordCount
```

## Scala版本

```scala
 val conf = new SparkConf()
      .setMaster("local")
      .setAppName("WordCount")
      .set("spark.cores.max", "1")
      .set("spark.eventLog.enabled", "true")
val all = conf.getAll
for (stringStringTuple2 <- all) {
    System.out.println(stringStringTuple2._1 + ": " + stringStringTuple2._2)
}
```




