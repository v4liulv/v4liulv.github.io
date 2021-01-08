---
title: "Structured Streaming消费Kafka流批分析"
subtitle: "日志分析"
layout: post
author: "LiuL"
header-style: text
tags:
  - Structured Streaming
---

# 版本信息


|组件  |版本  |Tbds  |
|---------|---------|---------|
|Hadoop     |    2.7.3     |         |
|Spark     |   2.3.2      |         |
|Kafka     |    1.0.0     |         |


# 本地通过Yarn提交SparkStructuredStreaming

```powershell
groupId = org.apache.spark
artifactId = spark-sql-kafka-0-10_2.11
version = 2.3.2
```

# SpringBoot整合SparkOnYarn远程提交运行Spark-BUG

## No FileSystem for scheme: hdfs

Spark运行程序报错No FileSystem for scheme: hdfs

缺少hadoop-hdfs包，在hadoop-common和hadoop-hdfs包的META-INF\services下均有一个org.apache.hadoop.fs.FileSystem文件，但文件内容是不同的。

解决办法：Spark程序spark.yarn.jars包目录引入hadoop-hdfs包。

## NoClassDefFoundError: com/fasterxml/jackson/databind/ObjectMapper

SpringBoot启动报错

原因：依赖冲突，SparkSql和SpringBoot包冲突

解决办法：
- 办法：SparkSQL只是运行在Yarn上配置spark.yarn.jars包即可，程序不需要Spark包，Maven把Spring相关依赖设置为`<scope>provided</scope>`

- 办法：使用jackson-databind-2.10.3依赖版本

## GsonBuilder报错 

application so that it contains a single, compatible version of com.google.gson.GsonBuilder

解决办法: 添加依赖坐标

```xml
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.6</version>
</dependency>
```

## User did not initialize spark context!

Uncaught exception: java.lang.IllegalStateException: User did not initialize spark context!

Spark运行程序报错，配置了spark.submit.deployMode为cluster，还配置了spark.driver.host为客户端ip

解决办法：
去掉spark.driver.host配置，这个配置只有在spark.submit.deployMode为client


## 调度异常堵塞

[AccumulationJob-1] accumulation schedule delay, expect:1609618320000, actual: 1609618448754


yarn application -list

yarn application -kill 

