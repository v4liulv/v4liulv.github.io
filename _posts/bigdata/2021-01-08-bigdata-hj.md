---
title: "大数据开发环境"
subtitle: "快捷"
layout: post
author: "LiuL"
header-style: text
tags:
  - Bigdata
  - Bigdata DEV
---

# 1. 本地环境

## 1.1. 前提

安装前提有

- 已安装了Hadoop集群
- 已安装Kafka环境
- 已安装Spark环境
- 已安装Hive环境

暂时用到的主要是Kafka、Hadoop、Spark

本地伪分布模式单机模式

## 1.2. 启动停命令

### 1.2.1. Zookeeper

```powershell
sh $ZOOKEEPER_HOME/bin/zkServer.sh start
```

```powershell
sh $ZOOKEEPER_HOME/bin/zkServer.sh stop
```

### 1.2.2. Kafka

```powershell
sh $KAFKA_HOME/bin/kafka-server-start.sh -daemon $KAFKA_HOME/config/server.properties
```

```powershell
sh $KAFKA_HOME/bin/kafka-server-stop.sh -daemon $KAFKA_HOME/config/server.properties
```

### 1.2.3. Hadoop

```powershell
start-all.sh
```

```powershell
stop-all.sh
```

### 1.2.4. Spark

```powershell
start-history-server.sh 
```

## 1.3. 命令和前端

### 1.3.1. Zookeeper

zkCli连接

```powershell
sh $ZOOKEEPER_HOME/bin/zkCli.sh -server hadoop01:2181
```

### 1.3.2. Hadoop 

HDFS

```powershell
hadoop01:50070
```

YARN

```powershell
hadoop01:8080
```

MR Job history

```powershell
hadoop01:19888
```

### 1.3.3. Spark

Spark History Server

```powershell
hadoop01:18080
```


### 1.3.4. Kafka命令

**创建topic**
```powershell
sh $KAFKA_HOME/bin/kafka-topics.sh --create --zookeeper localhost:2181/kafka --replication-factor 1 --partitions 2 --topic topicD
```

**删除topic**
```powershell
sh $KAFKA_HOME/bin/kafka-topics.sh --delete --topic topicD --zookeeper localhost:2181/kafka
```

**查看查看Topic列表**
```powershell
sh $KAFKA_HOME/bin/kafka-topics.sh --zookeeper localhost:2181/kafka --list
```

**查看Topic - topic01详情**
```powershell
sh $KAFKA_HOME/bin/kafka-topics.sh --describe --zookeeper localhost:2181/kafka --topic topic01
```

**生成者Producer**
```powershell
sh $KAFKA_HOME/bin/kafka-console-producer.sh --broker-list localhost:9092 --producer.config $KAFKA_HOME/config/producer.properties --topic topic01 
```

**消费者Consumer**
通过broker管理offset
```powershell
sh $KAFKA_HOME/bin/kafka-console-consumer.sh --zookeeper 127.0.0.1:2181/kafka --topic topic01 --group group01
```

通过zookeeper管理offset
```powershell
sh $KAFKA_HOME/bin/kafka-console-consumer.sh --zookeeper 127.0.0.1:2181/kafka --topic topic01 --group topic01_g01
```

```powershell
sh $KAFKA_HOME/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic topic01 --from-beginning  --delete-consumer-offsets --consumer.config $KAFKA_HOME/config/consumer.properties
```

**消费者组groups**

查询broker的grougs消费情况
```powershell
sh $KAFKA_HOME/bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --new-consumer --describe --group group01
```


查询broker的grougs消费情况
```powershell
sh $KAFKA_HOME/bin/kafka-consumer-groups.sh --zookeeper localhost:2181 --describe --group read_kafka_c2
--groups group01
```

# 2. TBDS 相关

## 2.1. TBDS 69 相关连接

zookeeper: tbds-172-16-0-12:2181,tbds-172-16-0-3:2181,tbds-172-16-0-5:2181
kafka: tbds-172-16-0-12:6667,tbds-172-16-0-24:6667,tbds-172-16-0-29:6667,tbds-172-16-0-30:6667

export hadoop_security_authentication_tbds_secureid=02vpsHrPriKWXhfWcqKZNLEoYHkoovxt72Ml
export hadoop_security_authentication_tbds_username=liulv
export hadoop_security_authentication_tbds_securekey=s1uZ5a30tR5rAHxvpKhiSTdI7KpkVOIn


## 2.2. 应用程序认证，启动参数添加
-Djava.security.auth.login.config=/data/pomp/kafka_client_jaas.conf

## 2.3. Portal服务器认证文件

/data/pomp/client_tbds.properties
/data/pomp/client_plain.properties
/data/pomp/client_jaas.properties

--command-config /data/pomp/client_tbds.properties
--consumer.config /data/pomp/client_plain.properties
--producer.config /data/pomp/client_plain.properties

## 2.4. SparkSubmit 提交Spark Streaming Kafka SASL_PLAIN认证

Source Kafka参数中添加
"security.protocol" -> "SASL_PLAINTEXT",
"sasl.mechanism" -> "PLAIN"

Spark submit 添加参数
--driver-java-options -Djava.security.auth.login.config=/etc/hadoop/conf/kafka_client_for_ranger_yarn_jaas.conf 
--conf "spark.executor.extraJavaOptions=-Djava.security.auth.login.config=/etc/hadoop/conf/kafka_client_for_ranger_yarn_jaas.conf"

--driver-java-options -Djava.security.auth.login.config=/etc/hadoop/conf/kafka_client_for_ranger_yarn_jaas.conf 
--conf "spark.executor.extraJavaOptions=-Djava.security.auth.login.config=/etc/hadoop/conf/kafka_client_for_ranger_yarn_jaas.conf"

## 2.5. Kafka命令

**特别注意：** 命令模式暂时是无法使用SASL_PLAINTEXT

cd /data/bigdata/tbds/usr/hdp/2.2.0.0-2041/kafka/bin

### 2.5.1. Topic

 ./kafka-topics.sh --create --zookeeper tbds-172-16-0-12:2181,tbds-172-16-0-3:2181,tbds-172-16-0-5:2181 \
 --replication-factor 2 --partitions 3 --topic testtopic

./kafka-topics.sh --zookeeper tbds-172-16-0-12:2181,tbds-172-16-0-3:2181,tbds-172-16-0-5:2181 --list

### 2.5.2. 生产者

 ./kafka-console-producer.sh \
--broker-list tbds-172-16-0-12:6667,tbds-172-16-0-24:6667,tbds-172-16-0-29:6667,tbds-172-16-0-30:6667 \
--producer.config /data/pomp/client_tbds.properties \
--topic pomp_topic_test









