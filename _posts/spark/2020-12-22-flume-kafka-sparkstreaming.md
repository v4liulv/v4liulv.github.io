---
title: "Flume+kafka+SparkStreaming实时日志分析"
subtitle: "日志分析"
layout: post
author: "LiuL"
header-style: text
tags:
  - Flume+kafka+SparkStreaming
---

# 1. 背景

通过Flume抓起本地日志数据，实时生成Kafka流数据，通过SparkStreaming进行实时流数据解析，最终存入HDFS或Hive中。

# 2. 整体架构

**技术选择1**

业务日志会通过Nginx（或者其他方式）每分钟写入到磁盘中，现在我们想要使用Spark分析日志，就需要先将磁盘中的文件上传到HDFS上，然后Spark处理，最后存入Hive表中，如图所示：

![20201222185409](https://liulv.work/images/img/20201222185409.png)

这种方式有几个缺点：

1. 我们的日志是通过Nginx每分钟存成一个文件，这样一天的文件数很多，不利于后续的分析任务，所以先要把一天的所有日志文件合并起来，需要格外合并处理

2. 合并起来以后需要把该文件从磁盘传到Hdfs上，但是我们的日志服务器并不在Hadoop集群内，所以没办法直接传到Hdfs上，需要把文件从日志服务器传输到Hadoop集群所在的服务器，然后再上传到Hdfs，或者通过Ftp服务器通过ETL工具上传到Hdfs.

3. 滞后一天分析数据已经不能满足我们新的实时业务需求了，最好能控制在一个小时的滞后时间

4. 这种方式比较原始并且比较耗时，很多时间浪费在了网络传输上面，如果日志量大的话还有丢失数据的可能性

**技术选择2**

技术架构图如下：

![20201222185757](https://liulv.work/images/img/20201222185757.png)

Flume会实时监控写入本地日志的磁盘，只要有新的日志写入，Flume就会将日志以消息的形式传递给Kafka，然后Spark Streaming实时消费消息传入Hive

Flume是什么呢，它为什么可以监控一个磁盘文件呢？简而言之，Flume是用来收集、汇聚并且移动大量日志文件的开源框架，所以很适合这种实时收集日志并且传递日志的场景。

Kafka是一个消息系统，Flume收集的日志可以移动到Kafka消息队列中，然后就可以被多处消费了，而且可以保证不丢失数据。

通过这套架构，收集到的日志可以及时被Flume发现传到Kafka，通过Kafka我们可以把日志用到各个地方，同一份日志可以存入Hdfs中，也可以离线进行分析，还可以实时计算，而且可以保证安全性，基本可以达到实时的要求。

# 3. 安装Zookeeper

## 3.1. 版本和下载、解压

版本：3.4.8

[下载地址](https://archive.apache.org/dist/zookeeper/zookeeper-3.4.8/)

创建zookeeper安装目录，

```powershell
mkdir -p /var/lib/zookeeper
```

上传下载包到/var/lib/zookeeper目录下, 执行下面解压命令

```powershell
cd /var/lib/zookeeper
tar -zxvf zookeeper-3.4.8.tar.gz
rm -rf zookeeper-3.4.8.tar.gz
```

## 3.2. 配置

```powershell
mkdir -p /data/zookeeper/logs
mkdir /data/zookeeper/data
chmod 777 /data/*

mv /var/lib/zookeeper/zookeeper-3.4.8/conf/zoo_sample.cfg /var/lib/zookeeper/zookeeper-3.4.8/conf/zoo.cfg

echo "dataDir=/data/zookeeper/data" >> /var/lib/zookeeper/zookeeper-3.4.8/conf/zoo.cfg
echo "dataLogDir=/data/zookeeper/logs" >> /var/lib/zookeeper/zookeeper-3.4.8/conf/zoo.cfg
```

## 3.3. 设置环境变量

```powershell
echo "export ZOOKEEPER_HOME=/var/lib/zookeeper/zookeeper-3.4.8" >> /etc/profile
echo "export PATH=$ZOOKEEPER_HOME/bin:$PATH" >> /etc/profile
source /etc/profile
```

## 3.4. 启动

sh $ZOOKEEPER_HOME/bin/zkServer.sh start

这个命令使得zk服务进程在后台进行。如果想在前台中运行以便查看服务器进程的输出日志，可以通过以下命令运行：

sh $ZOOKEEPER_HOME/bin/zkServer.sh start-foreground

## 3.5. 连接

sh $ZOOKEEPER_HOME/bin/zkCli.sh -server localhost:2181

# 4. 安装Kafka

## 4.1. 下载并启动Kafka

[Apache Kafka](http://kafka.apache.org/)是一种高吞吐量消息总线，可与Flume很好地配合使用。在本教程中，我们将使用Kafka2.1.0。要下载Kafka，请在终端中执行以下命令：

```powershell
mkdir /var/lib/kafka
cd /var/lib/kafka

curl -O https://archive.apache.org/dist/kafka/2.1.0/kafka_2.12-2.1.0.tgz
tar -xzf kafka_2.12-2.1.0.tgz
cd kafka_2.12-2.1.0
```

设置环境变量

```powershell
echo "export KAFKA_HOME=/var/lib/kafka/kafka_2.12-2.1.0/"
echo "export PATH=$KAFKA_HOME/bin:$PATH" >> /etc/profile
```

通过在终端中运行以下命令来启动一个Kafka Broker：

```powershell
sh $KAFKA_HOME/bin/kafka-server-start.sh $KAFKA_HOME/config/server.properties
```

## 4.2. 创建topic

sh $KAFKA_HOME/bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 2 --partitions 2 --topic launcher_click


查看

sh $KAFKA_HOME/bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic launcher_click


# 5. 安装flume

## 5.1. 下载解压

```powershell
mkdir /var/lib/flume
cd  /var/lib/flume
wget https://mirrors.bfsu.edu.cn/apache/flume/1.9.0/apache-flume-1.9.0-bin.tar.gz
tar -xvf apache-flume-1.9.0-bin.tar.gz
```

## 5.2. 修改环境变量

```powershell
export FLUME_HOME=/var/lib/flume/apache-flume-1.9.0-bin
export PATH=$PATH:$FLUME_HOME/bin
source /etc/profile
```

## 5.3. 修改配置

进入flume目录，修改conf/flume-env.sh

```powershell
export JAVA_HOME=/data/install/jdk
export JAVA_OPTS="-Xms1000m -Xmx2000m -Dcom.sun.management.jmxremote"
```

添加配置文件：conf/flume_launcherclick.conf

```properties
# logser可以看做是flume服务的名称，每个flume都由sources、channels和sinks三部分组成
# sources可以看做是数据源头、channels是中间转存的渠道、sinks是数据后面的去向
logser.sources = src_launcherclick
logser.sinks = kfk_launcherclick
logser.channels = ch_launcherclick

# source
# 源头类型是TAILDIR，就可以实时监控以追加形式写入文件的日志
logser.sources.src_launcherclick.type = TAILDIR
# positionFile记录所有监控的文件信息
logser.sources.src_launcherclick.positionFile = /data/install/flume/position/launcherclick/taildir_position.json
# 监控的文件组
logser.sources.src_launcherclick.filegroups = f1
# 文件组包含的具体文件，也就是我们监控的文件
logser.sources.src_launcherclick.filegroups.f1 = /data/launcher/stat_app/.*

# interceptor
# 写kafka的topic即可
logser.sources.src_launcherclick.interceptors = i1 i2
logser.sources.src_launcherclick.interceptors.i1.type=static
logser.sources.src_launcherclick.interceptors.i1.key = type
logser.sources.src_launcherclick.interceptors.i1.value = launcher_click
logser.sources.src_launcherclick.interceptors.i2.type=static
logser.sources.src_launcherclick.interceptors.i2.key = topic
logser.sources.src_launcherclick.interceptors.i2.value = launcher_click

# channel
logser.channels.ch_launcherclick.type = memory
logser.channels.ch_launcherclick.capacity = 10000
logser.channels.ch_launcherclick.transactionCapacity = 1000

# kfk sink
# 指定sink类型是Kafka，说明日志最后要发送到Kafka
logser.sinks.kfk_launcherclick.type = org.apache.flume.sink.kafka.KafkaSink
# Kafka broker
logser.sinks.kfk_launcherclick.brokerList = localhost:9092

# Bind the source and sink to the channel
logser.sources.src_launcherclick.channels = ch_launcherclick
logser.sinks.kfk_launcherclick.channel = ch_launcherclick
```

## 5.4. 启动

```powershell
nohup $FLUME_HOME/bin/flume-ng agent --conf $FLUME_HOME/conf/ --conf-file $FLUME_HOME/conf/flume-launcherclick.conf --name logser -Dflume.root.logger=INFO,console >> $FLUME_HOME/logs/flume_launcherclick.log &
```

# 6. 准备日志数据文件

此时Kafka和Flume都已经启动了，从配置可以看到Flume的监控文件是/data/launcher/stat_app/.*，所以只要该目录下文件内容有增加就会发送到Kafka，大家可以自己追加一些测试日志到这个目录的文件下，然后开一个Kafka Consumer看一下Kafka是否接收到消息，这里我们完成SparkStreaming以后再看测试结果

```powershell
mkdir -p /data/launcher/stat_app/
```

添加test.txt文件并添加一行数据

```powershell
183.160.102.250|1493169060.849|UEsDBBQACAgIAGBJmkoAAAAAAAAAAAAAAAABAAAAMGVVS3LrNhC8DVYvKmB+GJRXWaVygBwABCGLZYm0fi9xKofPUCJgO9mQNUVwPt09DRX2Ih41MgaSX*RrzOhOdfr77k48L7f6T1lOuzyPl2Uad+OUj*XiQmGikJMLlDD4BCmi5x+PKHhNrD5uEXBARukRBFbtkXof2n8QiUT9M7JWoockL2Xevd3*XHbvx*xhlccwesxArTKJtyQvX3vMx3w5leNS3hxmGQpQ7qcTA7XqyJE8bn1SDB6D6rdM13q7TfPr1VVKA8HYq6oPUfFHi5CVqU1vqZQ6FoAepUcIPqHvkWH4+R+yV+nfYhKvHUMktcZbpOwZ2klMiNh6CQQBCLbIsAHwlFqUBH0IjwkfBNvT+NyVfKqX7JBK2jOOfcaoIr074yJ56d0J8yffKYj10PsxglferMqtzqXOt935*D6drrXcL9XJyPshaackeU+4lbFmPcSwReTFMKG0SqDelns57Gox+c1jvjjjKw5YtacBNCS+sXc6XR0D7IsE7scic4RPMTJp+h8gr*loIv9wgylHsETXpkabu3GCEST0qSOKiQta496EjH0MiDYl*AeR4*SzOpQoPkjpFdj7FPseIUnc2Ftpj5T0a7PX22KAcg1xTywtR5KQoOdIUYOGliMYnKF9i2RvbtsQWUNITSsRwcTz1Eo55Pn1NO2O+T6Xg+1gkkHDkDZQ1yKJuPMvgWyxeoQm5KYbQkAb8RsSp5NjH3AkwpbvIQLfSUJk7v6hNoI2TJhAtOl7dQy1pW6oc1BO9L3WMkzHej67PZZxT2VsFZOBhk8h1NcuoF+P03v++O39sMzV5QJG1n7TbYBAZFi2tkjXPWxQQoIgW8tApjim7aRZobe2nuvRxLz743dnq2eoU3DNm4z8NrUtMZpWt*SmA8sO7ZsqRfOOR8Jrnt*ueVr1cbvn2RUdTVx1aEmVo4H2OHqePqbdz2msi8tjzTDsG*xiyU14rXurxLgJ2zxAUKXNkkjNUbrMo7fGnoAPeRrvZp*5Ug7D8pfjsVb1st0Xpry42n2DK9qkzcFQdRVQL8Fs5hb6Xq2oPDepLMvx8ThPLuwjQFXY0pulmAX7NoHRpNvinKfDsqD4TQnXvK9u3JsADBm3WZAttVVt6MoqwPhdRbebk1QFInjXSAAReSJ7y8uQl+3lcmXcC26ywbi+oE0XHwhSjyJ0c7IopdDARbWV4MQ9IqsNHTFApa82tl7b7*l9XVVz9aB5Q8aUEhPL0yYPH7v1jryWS62z0xKglBLaQbstgsLLv1BLBwjA8lu4twMAACkIAAA=
```

# 安装并启动Hadoop

安装并启动Hadoop请查看[相关教程](https://liulv.work/2020/12/15/hadoop-install/)

# SparkStreaming编程

SparkStreaming编程

```scala
object LauncherStreaming {

  val HDFS_DIR = "/user/kafka/launcher/click"

  def main(args: Array[String]) {

    Logger.getLogger("org.apache.spark").setLevel(Level.INFO)
    System.setProperty("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
    val sparkConf = new SparkConf().setAppName("LauncherStreaming").setMaster("local[2]")

    //限制输出success文件
    sparkConf.set("mapreduce.fileoutputcommitter.marksuccessfuljobs", "false")

    //每60秒一个批次
    val ssc = new StreamingContext(sparkConf, Seconds(10))

    // 从Kafka中读取数据
    // 日志内容：val str = "183.160.102.250|1493169060.849|UEsDBBQACAgIAGBJmkoAAAAAAAAAAAAAAAABAAAAMGVVS3LrNhC8DVYvKmB+GJRXWaVygBwABCGLZYm0fi9xKofPUCJgO9mQNUVwPt09DRX2Ih41MgaSX*RrzOhOdfr77k48L7f6T1lOuzyPl2Uad+OUj*XiQmGikJMLlDD4BCmi5x+PKHhNrD5uEXBARukRBFbtkXof2n8QiUT9M7JWoockL2Xevd3*XHbvx*xhlccwesxArTKJtyQvX3vMx3w5leNS3hxmGQpQ7qcTA7XqyJE8bn1SDB6D6rdM13q7TfPr1VVKA8HYq6oPUfFHi5CVqU1vqZQ6FoAepUcIPqHvkWH4+R+yV+nfYhKvHUMktcZbpOwZ2klMiNh6CQQBCLbIsAHwlFqUBH0IjwkfBNvT+NyVfKqX7JBK2jOOfcaoIr074yJ56d0J8yffKYj10PsxglferMqtzqXOt935*D6drrXcL9XJyPshaackeU+4lbFmPcSwReTFMKG0SqDelns57Gox+c1jvjjjKw5YtacBNCS+sXc6XR0D7IsE7scic4RPMTJp+h8gr*loIv9wgylHsETXpkabu3GCEST0qSOKiQta496EjH0MiDYl*AeR4*SzOpQoPkjpFdj7FPseIUnc2Ftpj5T0a7PX22KAcg1xTywtR5KQoOdIUYOGliMYnKF9i2RvbtsQWUNITSsRwcTz1Eo55Pn1NO2O+T6Xg+1gkkHDkDZQ1yKJuPMvgWyxeoQm5KYbQkAb8RsSp5NjH3AkwpbvIQLfSUJk7v6hNoI2TJhAtOl7dQy1pW6oc1BO9L3WMkzHej67PZZxT2VsFZOBhk8h1NcuoF+P03v++O39sMzV5QJG1n7TbYBAZFi2tkjXPWxQQoIgW8tApjim7aRZobe2nuvRxLz743dnq2eoU3DNm4z8NrUtMZpWt*SmA8sO7ZsqRfOOR8Jrnt*ueVr1cbvn2RUdTVx1aEmVo4H2OHqePqbdz2msi8tjzTDsG*xiyU14rXurxLgJ2zxAUKXNkkjNUbrMo7fGnoAPeRrvZp*5Ug7D8pfjsVb1st0Xpry42n2DK9qkzcFQdRVQL8Fs5hb6Xq2oPDepLMvx8ThPLuwjQFXY0pulmAX7NoHRpNvinKfDsqD4TQnXvK9u3JsADBm3WZAttVVt6MoqwPhdRbebk1QFInjXSAAReSJ7y8uQl+3lcmXcC26ywbi+oE0XHwhSjyJ0c7IopdDARbWV4MQ9IqsNHTFApa82tl7b7*l9XVVz9aB5Q8aUEhPL0yYPH7v1jryWS62z0xKglBLaQbstgsLLv1BLBwjA8lu4twMAACkIAAA="
    val kafkaStream = KafkaUtils.createStream(
      ssc,
      "192.168.56.101:2181", // Kafka集群使用的zookeeper
      "launcher-streaming", // 该消费者使用的group.id
      Map[String, Int]("launcher_click" -> 0, "launcher_click" -> 1), // 日志在Kafka中的topic及其分区
      StorageLevel.MEMORY_AND_DISK_SER).map(_._2) // 获取日志内容

    kafkaStream.foreachRDD((rdd: RDD[String], time: Time) => {
      rdd.foreach((pairs1) => System.out.println(pairs1.toString))
      val result = rdd.map(log => parseLog(log)) // 分析处理原始日志
        .filter(t => StringUtils.isNotBlank(t._1) && StringUtils.isNotBlank(t._2))
      // 存入hdfs
      println("=============写HDFS Start========================")
      result.foreach((pairs1) => System.out.println(pairs1._1.toString + ":" + pairs1._2.toString))
      println("=====================================")
      //result.saveAsHadoopFile(HDFS_DIR, classOf[String], classOf[String], classOf[LauncherMultipleTextOutputFormat[String, String]])
      result.saveAsTextFile(HDFS_DIR)

    })

    ssc.start()
    // 等待实时流
    ssc.awaitTermination()
  }

  // 分析日志
  def parseLog(log: String): (String, String) = {

    var data = log
    var key = ""
    try {
      val sb = new StringBuilder
      var idx = 0
      // 接收服务器获取的IP
      val ip = data.substring(0, log.indexOf("|"))
      data = data.substring(log.indexOf("|") + 1)
      print(data)

      var tss = data.substring(0, data.indexOf("|"))
      tss = tss.replaceAll("\\.", "")
      // 服务器时间戳，13位
      val serverTimestamp = tss.toLong
      val df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss")
      // 服务器时间
      val serverTime = df.format(serverTimestamp)
      data = data.substring(data.indexOf("|") + 1)
      val keyDf = new SimpleDateFormat("yyyy/MM/dd/")
      key = keyDf.format(serverTimestamp)

      // 主体数据：imei列表&品牌&机型|包名&文件crc&时间戳列表;包名&文件crc&时间戳列表;...
      data = data.replaceAll("\\*", "\\/")
      data = DataUtil.unzip(data)

      // 解压后的数据格式
      // 865066038753146-865066038753153&meizu&m5note|com.android.dialer&1c5441a9&1493109297305,1493110895807,1493112513536,1493112521588,1493112580015,1493112744680,1493138770296;cn.kuwo.player&d1d03a24&1493109460580;com.android.alarmclock&3a6bc24a&1493109495248,1493113574037,1493147103188;com.android.settings&e49b42d4&1493109801783,1493109835854,1493110147845,1493110230365,1493110320930,1493110344154,1493110350860,1493110796087,1493112348524,1493112850520,1493113933383,1493114212423,1493146022049,1493146963011;
      val dataArr = data.split("\\|")
      val phoneData = dataArr(0).split("&")
      // imei列表
      val imeis = phoneData(0).split("-").mkString(";")
      // 品牌
      val phoneBrand = phoneData(1)
      // 机型
      val phoneModel = phoneData(2)

      val appData = dataArr(1).split(";")
      val appArr = ArrayBuffer[String]()
      for (app <- appData) {
        val appDetail = app.split("&")
        // 时间戳列表
        val timestamps = appDetail(2).split(",")
        // 点击次数
        val clickTimes = timestamps.length
        appArr += app + "&" + clickTimes
      }

      // imei有数据，app有数据
      if (imeis.length > 0 && appArr.length > 0) {
        sb.append(imeis)
        sb.append("|")
        sb.append(if (StringUtils.isNotBlank(ip)) ip else " ")
        sb.append("|")
        sb.append(serverTimestamp)
        sb.append("|")
        sb.append(if (StringUtils.isNotBlank(serverTime)) serverTime else " ")
        sb.append("|")
        sb.append(if (StringUtils.isNotBlank(phoneBrand)) phoneBrand else " ")
        sb.append("|")
        sb.append(if (StringUtils.isNotBlank(phoneModel)) phoneModel else " ")
        sb.append("|")
        sb.append(appArr.mkString(";"))
      }

      data = sb.toString()

    } catch {
      case e: Exception =>
        e.printStackTrace()
        data = ""
    }

    (key, data)
  }

}
```


启动本地模式运行SparkStreaming的main方法，然后开始测试，如远程打包工程jar包上传`/home/hadoop/jar/SparkLearning.jar`到可执行下面命令

```
nohup /data/install/spark-2.0.0-bin-hadoop2.7/bin/spark-submit  --master spark://hadoop01:7077  --executor-memory 1G --total-executor-cores 4   --class com.analysis.main.LauncherStreaming --jars /home/hadoop/jar/kafka-clients-0.10.0.0.jar,/home/hadoop/jar/metrics-core-2.2.0.jar,/home/hadoop/jar/zkclient-0.3.jar,/home/hadoop/jar/spark-streaming-kafka-0-8_2.11-2.0.0.jar,/home/hadoop/jar/kafka_2.11-0.8.2.1.jar  /home/hadoop/jar/SparkLearning.jar  >> /home/hadoop/logs/LauncherDM.log &
```


往Flume监控目录/data/launcher/stat_app/.*写日志，原始日志内容类似下面这样：

```log
118.120.102.3|1495608541.238|UEsDBBQACAgIACB2uEoAAAAAAAAAAAAAAAABAAAAMGWUbW7bMAyGb6NfnUFRFEWhJ+gBdgBZVjpjjp04brMAO*yY2DKa9Y+B1+DnQ1LCztoITgK4wPGHfNUhmKGUPOn3DyP*zdOxSWM3T33XXMqy9OP7xXTZiTC1xlL0HgMEi+BfHoooBEGKr3fPpYy5jMse4Xzupus4TKkrs4kZOhI51CgWWKxsUQBRPMDr1*w5Hcuc0LiUEFBwdXQxAARXHb3+QXlOfzya0uZWOGwlEwBDwLD5oJBVFHsEEPF2U0EUToyr8k4tg9v8AkRrIcKmxGsU2eqQIM45dKuKFICo5oveEqOjh2JAIITImyIJqBk3JS4qh7Wby*TroxnL9ZKHXrsyWeBQoMXaEgXUKh6mOQ1l7NLc*Hwz8aDpAtndLFJEetkVc6S9V*bg+RFiKMvnTv6ahuGUTmWexqEfi3Elezx0botJrCCQn5jfCzWaqaUOqNpFYO23ckYl5GOlx4rLQuUllh27SsjZyLQTUn4K+3uVczlOi+7uuMzTYLoibeIspk71DtKuJC+7T5qXPg9lLddaZs6+Lolnj7ANW0dBGKOn72m3cbQJI2Kq4*C6Xhz9E5Pzeeg*i2l1IAJtpReILNq6DY4peFjHeO5vffPZd2UyejEJ28Puo0sI*2*5ojvhfNcquWomFMVp02Pz++M6Nach3e6XR5wOlrdSg4T7RkgtQAuC6HYl2sc62i6dUq*om+HWjvdHAPSk8hYkegHraxC8PwPons73XZeozDfXmaRzzzaD2XI4fX0QX*8BUEsHCKeftc48AgAAmQQAAA==
```

查看HDFS的对应目录是否有内容：

![20201222210130](https://liulv.work/images/img/20201222210130.png)
























