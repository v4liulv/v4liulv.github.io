---
title: "ClickHouse入门教程"
subtitle: ""
layout: post
author: "LiuL"
header-style: text
tags:
  - ClickHouse
---

# 1. ClickHouse介绍

ClickHouse是一个用于联机分析(OLAP)的列式数据库管理系统(DBMS)。

## 1.1. 行数据库与列数据库区别

在传统的行式数据库系统中，数据按顺序存储，处于同一行中的数据总是被物理的存储在一起。常见的行式数据库系统有： MySQL、Postgres和MS SQL Server。

在列式数据库系统中，存储格式是列名，列值1, 列2.方式存储，对于存储而言，列式数据库总是将同一列的数据存储在一起，不同列的数据也总是分开存储。

常见的列式数据库有： Vertica、 Paraccel (Actian Matrix，Amazon Redshift)、 Sybase IQ、 Exasol、 Infobright、 InfiniDB、 MonetDB (VectorWise， Actian Vector)、 LucidDB、 SAP HANA、 Google Dremel、 Google PowerDrill、 Druid、 kdb+。

不同的数据存储方式适用不同的业务场景，数据访问的场景包括：进行了何种查询、多久查询一次以及各类查询的比例； 每种查询读取多少数据————行、列和字节；读取数据和写入数据之间的关系；使用的数据集大小以及如何使用本地的数据集；是否使用事务,以及它们是如何进行隔离的；数据的复制机制与数据的完整性要求；每种类型的查询要求的延迟与吞吐量等等。

系统负载越高，依据使用场景进行定制化就越重要，并且定制将会变的越精细。没有一个系统能够同时适用所有明显不同的业务场景。如果系统适用于广泛的场景，在负载高的情况下，要兼顾所有的场景，那么将不得不做出选择。是要平衡还是要效率？

## 1.2. OLAP场景的关键特征

- 大多数是读请求
- 数据总是以相当大的批(> 1000 rows)进行写入
- 不修改已添加的数据
- 每次查询都从数据库中读取大量的行，但是同时又仅需要少量的列
- 宽表，即每个表包含着大量的列
- 较少的查询(通常每台服务器每秒数百个查询或更少)
- 对于简单查询，允许延迟大约50毫秒
- 列中的数据相对较小： 数字和短字符串(例如，每个URL 60个字节)
- 处理单个查询时需要高吞吐量（每个服务器每秒高达数十亿行）
- 事务不是必须的
- 对数据一致性要求低
- 每一个查询除了一个大表外都很小
- 查询结果明显小于源数据，换句话说，数据被过滤或聚合后能够被盛放在单台服务器的内存中

很容易可以看出，OLAP场景与其他通常业务场景(例如,OLTP或K/V)有很大的不同， 因此想要使用OLTP或Key-Value数据库去高效的处理分析查询场景，并不是非常完美的适用方案。例如，使用OLAP数据库去处理分析请求通常要优于使用MongoDB或Redis去处理分析请求。

## 1.3. 列式数据库更适合OLAP场景的原因

列式数据库更适合于OLAP场景(对于大多数查询而言，处理速度至少提高了100倍),具体原因如下：

输入/输出

1. 针对分析类查询，通常只需要读取表的一小部分列。在列式数据库中你可以只读取你需要的数据。例如，如果只需要读取100列中的5列，这将帮助你最少减少20倍的I/O消耗。
2. 由于数据总是打包成批量读取的，所以压缩是非常容易的。同时数据按列分别存储这也更容易压缩。这进一步降低了I/O的体积。
3. 由于I/O的降低，这将帮助更多的数据被系统缓存。

CPU

由于执行一个查询需要处理大量的行，因此在整个向量上执行所有操作将比在每一行上执行所有操作更加高效。同时这将有助于实现一个几乎没有调用成本的查询引擎。如果你不这样做，使用任何一个机械硬盘，查询引擎都不可避免的停止CPU进行等待。所以，在数据按列存储并且按列执行是很有意义的。

有两种方法可以做到这一点：

1. 向量引擎：所有的操作都是为向量而不是为单个值编写的。这意味着多个操作之间的不再需要频繁的调用，并且调用的成本基本可以忽略不计。操作代码包含一个优化的内部循环。

2. 代码生成：生成一段代码，包含查询中的所有操作。

这是不应该在一个通用数据库中实现的，因为这在运行简单查询时是没有意义的。但是也有例外，例如，MemSQL使用代码生成来减少处理SQL查询的延迟(只是为了比较，分析型数据库通常需要优化的是吞吐而不是延迟)。

请注意，为了提高CPU效率，查询语言必须是声明型的(SQL或MDX)， 或者至少一个向量(J，K)。 查询应该只包含隐式循环，允许进行优化。

## 1.4. ClickHouse的特性

**真正的列式数据库管理系统**

在一个真正的列式数据库管理系统中，除了数据本身外不应该存在其他额外的数据。这意味着为了避免在值旁边存储它们的长度«number»，你必须支持固定长度数值类型。例如，10亿个UInt8类型的数据在未压缩的情况下大约消耗1GB左右的空间，如果不是这样的话，这将对CPU的使用产生强烈影响。即使是在未压缩的情况下，紧凑的存储数据也是非常重要的，因为解压缩的速度主要取决于未压缩数据的大小。

这是非常值得注意的，因为在一些其他系统中也可以将不同的列分别进行存储，但由于对其他场景进行的优化，使其无法有效的处理分析查询。例如： HBase，BigTable，Cassandra，HyperTable。在这些系统中，你可以得到每秒数十万的吞吐能力，但是无法得到每秒几亿行的吞吐能力。

ClickHouse不单单是一个数据库， 它是一个数据库管理系统。因为它允许在运行时创建表和数据库、加载数据和运行查询，而无需重新配置或重启服务。

**数据压缩**

在一些列式数据库管理系统中(例如：InfiniDB CE 和 MonetDB) 并没有使用数据压缩。但是, 若想达到比较优异的性能，数据压缩确实起到了至关重要的作用。

**数据的磁盘存储**

许多的列式数据库(如 SAP HANA, Google PowerDrill)只能在内存中工作，这种方式会造成比实际更多的设备预算。ClickHouse被设计用于工作在传统磁盘上的系统，它提供每GB更低的存储成本，但如果有可以使用SSD和内存，它也会合理的利用这些资源。

**多核心并行处理**

ClickHouse会使用服务器上一切可用的资源，从而以最自然的方式并行处理大型查询。

**多服务器分布式处理**

上面提到的列式数据库管理系统中，几乎没有一个支持分布式的查询处理。
在ClickHouse中，数据可以保存在不同的shard上，每一个shard都由一组用于容错的replica组成，查询可以并行地在所有shard上进行处理。这些对用户来说是透明的

**支持SQL**

ClickHouse支持基于SQL的声明式查询语言，该语言大部分情况下是与SQL标准兼容的。
支持的查询包括 GROUP BY，ORDER BY，IN，JOIN以及非相关子查询。
不支持窗口函数和相关子查询。

**向量引擎**

为了高效的使用CPU，数据不仅仅按列存储，同时还按向量(列的一部分)进行处理，这样可以更加高效地使用CPU。

**实时的数据更新**

ClickHouse支持在表中定义主键。为了使查询能够快速在主键中进行范围查找，数据总是以增量的方式有序的存储在MergeTree中。因此，数据可以持续不断地高效的写入到表中，并且写入的过程中不会存在任何加锁的行为。

**索引**

按照主键对数据进行排序，这将帮助ClickHouse在几十毫秒以内完成对数据特定值或范围的查找。

**适合在线查询**

在线查询意味着在没有对数据做任何预处理的情况下以极低的延迟处理查询并将结果加载到用户的页面中。

**支持近似计算**

ClickHouse提供各种各样在允许牺牲数据精度的情况下对查询进行加速的方法：

1. 用于近似计算的各类聚合函数，如：distinct values, medians, quantiles
2. 基于数据的部分样本进行近似查询。这时，仅会从磁盘检索少部分比例的数据。
3. 不使用全部的聚合条件，通过随机选择有限个数据聚合条件进行聚合。这在数据聚合条件满足某些分布条件下，在提供相当准确的聚合结果的同时降低了计算资源的使用。

**支持数据复制和数据完整性**

ClickHouse使用异步的多主复制技术。当数据被写入任何一个可用副本后，系统会在后台将数据分发给其他副本，以保证系统在不同副本上保持相同的数据。在大多数情况下ClickHouse能在故障后自动恢复，在一些少数的复杂情况下需要手动恢复。

更多信息，参见 数据复制。

**限制**

没有完整的事务支持。
缺少高频率，低延迟的修改或删除已存在数据的能力。仅能用于批量删除或修改数据，但这符合 GDPR。
稀疏索引使得ClickHouse不适合通过其键检索单行的点查询。


# 2. 安装ClickHouse

## 2.1. Docker方式安装

**特别说明：** docker没有给出这么启动client，推荐使用RPM安装方式

要在Docker中运行ClickHouse，请遵循[Docker的安装](https://hub.docker.com/r/yandex/clickhouse-server/)指南。那些镜像使用官方的deb包。

### 2.1.1. 使用源码安装 

Docker安装只需要安装clickhouse-server和clickhouse-client即可

**Step如下：**

a. 命令来查看安装可用版本：

```shell
docker search clickhouse-server
docker search clickhouse-client
```

b. 查询得到可用的镜像有yandex/开头的，那么使用下面命令下载镜像到本地

```powershell
docker pull yandex/clickhouse-server:latest
docker pull yandex/clickhouse-client:latest
```

c. 下载安装完成后，查看已经安装的镜像

``` powershell
docker images
```

![20201125181613](https://liulv.work/images/img/20201125181613.png)

d. 在服务器中为数据创建如下目录：

```powershell
mkdir $HOME/some_clickhouse_database
```

### 2.1.2. 启动ClickHouse 

可以运行如下命令在创建容器：

```powershell
docker run -d --name some-clickhouse-server --ulimit nofile=262144:262144 --volume=$HOME/some_clickhouse_database:/var/lib/clickhouse yandex/clickhouse-server 
```


查看容器
```powershell
docker ps
```

启动容器

**启动容器**

```powershell
docker start 5a7fddc3e756
```

## 2.2. rpm安装方式

### 2.2.1. 安装步骤

Step：

1. 首先，您需要添加官方存储库

```powershell
sudo yum install yum-utils
sudo rpm --import https://repo.clickhouse.tech/CLICKHOUSE-KEY.GPG
sudo yum-config-manager --add-repo https://repo.clickhouse.tech/rpm/stable/x86_64
```

如果您想使用最新版本，请将stable替换为testing（建议您在测试环境中使用）。


2. 然后运行这些命令以实际安装包

```powershell
sudo yum install clickhouse-server clickhouse-client
```

您也可以从此处手动下载和安装软件包：https://repo.clickhouse.tech/rpm/stable/x86_64。

### 2.2.2. 启动 

可以运行如下命令在后台启动服务：

sudo systemctl start clickhouse-server

可以在/var/log/clickhouse-server/目录中查看日志。

如果服务没有启动，请检查配置文件 /etc/clickhouse-server/config.xml。

你也可以在控制台中直接启动服务：

clickhouse-server --config-file=/etc/clickhouse-server/config.xml

### 2.2.3. 连接客户端访问

你可以使用命令行客户端连接到服务：

clickhouse-client

默认情况下它使用’default’用户无密码的与localhost:9000服务建立连接。
客户端也可以用于连接远程服务，例如：

clickhouse-client --host=example.com

有关更多信息，请参考«Command-line client»部分。

检查系统是否工作：

```shell
[root@localhost ~]# clickhouse-client
ClickHouse client version 20.11.4.13 (official build).
Connecting to localhost:9000 as user default.
Connected to ClickHouse server version 20.11.4 revision 54442.

localhost :) 
localhost :) 
localhost :) 
localhost :) SELECT 1

SELECT 1

Query id: e793ffe4-7a60-4227-879d-63ebd686bb66

┌─1─┐
│ 1 │
└───┘

1 rows in set. Elapsed: 0.003 sec. 

localhost :) 
```

**恭喜，系统已经工作了!**

为了继续进行实验，你可以尝试下载测试数据集。

# 3. 允许远程连接

打开/etc/clickhouse-server/config.xml

把注释掉的<listen_host>::</listen_host>取消注释，然后重启服务

sudo systemctl restart clickhouse-server

检查端口

lsof -i :8123

```log
[root@localhost ~]# lsof -i :8123
COMMAND     PID       USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
clickhous 16848 clickhouse   55u  IPv6  79734      0t0  TCP *:8123 (LISTEN)
[root@localhost ~]# 
```

# 4. 设置密码

```powershell
PASSWORD=$(base64 < /dev/urandom | head -c8); echo "$PASSWORD"; 
echo -n "$PASSWORD" | sha256sum | tr -d '-'
```

BOz6SdUe

在/etc/clickhouse-server/user.xml
找到 users --> default --> 标签下的password修改成password_sha256_hex，并把密文填进去

<password_sha256_hex>密码密文</password_sha256_hex>

添加密码后，命令行启动的方式为
clickhouse-client -h ip地址 -d default -m -u default --password 密码明文

clickhouse-client -d default -m -u default --password BOz6SdUe


# 5. 引擎

## 5.1. 数据库引擎

### 5.1.1. 数据库MySql引擎

MySQL引擎用于将远程的MySQL服务器中的表映射到ClickHouse中，并允许您对表进行INSERT和SELECT查询，以方便您在ClickHouse与MySQL之间进行数据交换。

在MySQL中创建表:

mysql> USE test;
Database changed

mysql> CREATE TABLE `mysql_table` (
    ->   `int_id` INT NOT NULL AUTO_INCREMENT,
    ->   `float` FLOAT NOT NULL,
    ->   PRIMARY KEY (`int_id`));
Query OK, 0 rows affected (0,09 sec)

mysql> insert into mysql_table (`int_id`, `float`) VALUES (1,2);
Query OK, 1 row affected (0,00 sec)

mysql> select * from mysql_table;
+--------+-------+
| int_id | value |
+--------+-------+
|      1 |     2 |
+--------+-------+
1 row in set (0,00 sec)


在ClickHouse中创建MySQL类型的数据库，同时与MySQL服务器交换数据：

CREATE DATABASE mysql_db ENGINE = MySQL('192.168.56.1:3306', 'test', 'root', '123456')
SHOW DATABASES
┌─name─────┐
│ default  │
│ mysql_db │
│ system   │
└──────────┘

SHOW TABLES FROM mysql_db
┌─name─────────┐
│  mysql_table │
└──────────────┘

SELECT * FROM mysql_db.mysql_table
┌─int_id─┬─value─┐
│      1 │     2 │
└────────┴───────┘

INSERT INTO mysql_db.mysql_table VALUES (3,4)
SELECT * FROM mysql_db.mysql_table
┌─int_id─┬─value─┐
│      1 │     2 │
│      3 │     4 │
└────────┴───────┘


### 5.1.2. 延时引擎Lazy 

在距最近一次访问间隔expiration_time_in_seconds时间段内，将表保存在内存中，仅适用于 *Log引擎表

由于针对这类表的访问间隔较长，对保存大量小的 *Log引擎表进行了优化，

创建数据库 
CREATE DATABASE testlazy ENGINE = Lazy(expiration_time_in_seconds);

### 5.1.3. 数据库引擎

您使用的所有表都是由数据库引擎所提供的

默认情况下，ClickHouse使用自己的数据库引擎，该引擎提供可配置的表引擎和所有支持的SQL语法.

除此之外，您还可以选择使用Mysql数据库引擎[3.1. 数据库MySql引擎](#31-数据库MySql引擎)

## 5.2. 表引擎

### 表引擎

表引擎（即表的类型）决定了：

- 数据的存储方式和位置，写到哪里以及从哪里读取数据
- 支持哪些查询以及如何支持。
- 并发数据访问。
- 索引的使用（如果存在）。
- 是否可以执行多线程请求。
- 数据复制参数。

#### 引擎类型

**MergeTree**
适用于高负载任务的最通用和功能最强大的表引擎。这些引擎的共同特点是可以快速插入数据并进行后续的后台数据处理。 MergeTree系列引擎支持数据复制（使用Replicated* 的引擎版本），分区和一些其他引擎不支持的其他功能。

该类型的引擎：
- MergeTree
- ReplacingMergeTree
- SummingMergeTree
- AggregatingMergeTree
- CollapsingMergeTree
- VersionedCollapsingMergeTree
- GraphiteMergeTree

**日志**
具有最小功能的轻量级引擎。当您需要快速写入许多小表（最多约100万行）并在以后整体读取它们时，该类型的引擎是最有效的。

该类型的引擎：

- TinyLog
- StripeLog
- Log

**集成引擎** 
用于与其他的数据存储与处理系统集成的引擎。
该类型的引擎：

- Kafka
- MySQL
- ODBC
- JDBC
- HDFS

**用于其他特定功能的引擎**
该类型的引擎：

- Distributed
- MaterializedView
- Dictionary
- Merge
- File
- Null
- Set
- Join
- URL
- View
- Memory
- Buffer

**虚拟列**
虚拟列是表引擎组成的一部分，它在对应的表引擎的源代码中定义。

您不能在 CREATE TABLE 中指定虚拟列，并且虚拟列不会包含在 SHOW CREATE TABLE 和 DESCRIBE TABLE 的查询结果中。虚拟列是只读的，所以您不能向虚拟列中写入数据。

如果想要查询虚拟列中的数据，您必须在SELECT查询中包含虚拟列的名字。SELECT * 不会返回虚拟列的内容。

若您创建的表中有一列与虚拟列的名字相同，那么虚拟列将不能再被访问。我们不建议您这样做。为了避免这种列名的冲突，虚拟列的名字一般都以下划线开头。

### 5.2.1. MergeTree（合并树）系列引擎

#### 5.2.1.1. VersionedCollapsingMergeTree（版本折叠合并树）

**功能**
- 允许快速写入不断变化的对象状态。
- 删除后台中的旧对象状态。 这显着降低了存储体积。

引擎继承自 MergeTree 并将折叠行的逻辑添加到合并数据部分的算法中。 VersionedCollapsingMergeTree 用于相同的目的 折叠树 但使用不同的折叠算法，允许以多个线程的任何顺序插入数据。 特别是， Version 列有助于正确折叠行，即使它们以错误的顺序插入。 相比之下, CollapsingMergeTree 只允许严格连续插入。

**创建表**

```sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
    ...
) ENGINE = VersionedCollapsingMergeTree(sign, version)
[PARTITION BY expr]
[ORDER BY expr]
[SAMPLE BY expr]
[SETTINGS name=value, ...]
```

**引擎参数**

    VersionedCollapsingMergeTree(sign, version)

- sign — 指定行类型的列名: 1 是一个 “state” 行, -1 是一个 “cancel” 划列数据类型应为 Int8.

- version — 指定对象状态版本的列名。列数据类型应为 UInt*.

**查询 Clauses**

当创建一个 VersionedCollapsingMergeTree 表时，跟创建一个 MergeTree表的时候需要相同 Clause。

**折叠数据**

考虑一种情况，您需要为某个对象保存不断变化的数据。 对于一个对象有一行，并在发生更改时更新该行是合理的。 但是，对于数据库管理系统来说，更新操作非常昂贵且速度很慢，因为它需要重写存储中的数据。 如果需要快速写入数据，则不能接受更新，但可以按如下顺序将更改写入对象。

使用 Sign 列写入行时。 如果 Sign = 1 这意味着该行是一个对象的状态（让我们把它称为 “state” 行）。 如果 Sign = -1 它指示具有相同属性的对象的状态的取消（让我们称之为 “cancel” 行）。 还可以使用 Version 列，它应该用单独的数字标识对象的每个状态。

例如，我们要计算用户在某个网站上访问了多少页面以及他们在那里的时间。 在某个时间点，我们用用户活动的状态写下面的行:


**写入数据注意事项**
- 写入数据的程序应该记住对象的状态以取消它。 该 “cancel” 字符串应该是 “state” 与相反的字符串 Sign. 这增加了存储的初始大小，但允许快速写入数据。
- 列中长时间增长的数组由于写入负载而降低了引擎的效率。 数据越简单，效率就越高。
- SELECT 结果很大程度上取决于对象变化历史的一致性。 准备插入数据时要准确。 不一致的数据将导致不可预测的结果，例如会话深度等非负指标的负值。

**示例：**

创建表
```sql
CREATE TABLE UAct
(
    UserID UInt64,
    PageViews UInt8,
    Duration UInt8,
    Sign Int8,
    Version UInt8
)
ENGINE = VersionedCollapsingMergeTree(Sign, Version)
ORDER BY UserID
```

插入数据:
```sql
INSERT INTO UAct VALUES (4324182021466249494, 5, 146, 1, 1)
INSERT INTO UAct VALUES (4324182021466249494, 5, 146, -1, 1),(4324182021466249494, 6, 185, 1, 2)
```

我们用两个 INSERT 查询以创建两个不同的数据部分。 如果我们使用单个查询插入数据，ClickHouse将创建一个数据部分，并且永远不会执行任何合并。

获取数据:

```sql
SELECT * FROM UAct
```

|UserID|PageViews|Duration|Sign|Version|
|------|---------|--------|----|-------|
|4324182021466249494|5|146|1|1|
|4324182021466249494|5|146|-1|1|
|4324182021466249494|6|185|1|2|

我们在这里看到了什么，折叠的部分在哪里？
我们使用两个创建了两个数据部分 INSERT 查询。 该 SELECT 查询是在两个线程中执行的，结果是行的随机顺序。
由于数据部分尚未合并，因此未发生折叠。 ClickHouse在我们无法预测的未知时间点合并数据部分。

这就是为什么我们需要聚合:

```sql
SELECT
    UserID,
    sum(PageViews * Sign) AS PageViews,
    sum(Duration * Sign) AS Duration,
    Version
FROM UAct
GROUP BY UserID, Version
HAVING sum(Sign) > 0
```

|UserID|PageViews|Duration|Version|
|------|---------|--------|-------|
|4324182021466249494|6|185|2|

如果我们不需要聚合，并希望强制折叠，我们可以使用 FINAL 修饰符 FROM 条款

```sql
SELECT * FROM UAct FINAL;
```

|UserID|PageViews|Duration|Version|
|------|---------|--------|-------|
|4324182021466249494|6|185|2|


这是一个非常低效的方式来选择数据。 不要把它用于数据量大的表。

#### 5.2.1.2. GraphiteMergeTree

该引擎用来对 Graphite数据进行瘦身及汇总。对于想使用CH来存储Graphite数据的开发者来说可能有用。

如果不需要对Graphite数据做汇总，那么可以使用任意的CH表引擎；但若需要，那就采用 GraphiteMergeTree 引擎。它能减少存储空间，同时能提高Graphite数据的查询效率。

该引擎继承自 MergeTree.

**创建表**

```sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    Path String,
    Time DateTime,
    Value <Numeric_type>,
    Version <Numeric_type>
    ...
) ENGINE = GraphiteMergeTree(config_section)
[PARTITION BY expr]
[ORDER BY expr]
[SAMPLE BY expr]
[SETTINGS name=value, ...]
```

建表语句的详细说明请参见 创建表

含有Graphite数据集的表应该包含以下的数据列：
- 指标名称(Graphite sensor)，数据类型：String
- 指标的时间度量，数据类型： DateTime
- 指标的值，数据类型：任意数值类型
- 指标的版本号，数据类型： 任意数值类型

CH以最大的版本号保存行记录，若版本号相同，保留最后写入的数据。



# 6. SQL语法

## 6.1. Select

SELECT 查询执行数据检索。 默认情况下，请求的数据返回给客户端，同时结合 INSERT INTO 可以被转发到不同的表。

### 6.1.1. 语法

```sql
[WITH expr_list|(subquery)]
SELECT [DISTINCT] expr_list
[FROM [db.]table | (subquery) | table_function] [FINAL]
[SAMPLE sample_coeff]
[ARRAY JOIN ...]
[GLOBAL] [ANY|ALL|ASOF] [INNER|LEFT|RIGHT|FULL|CROSS] [OUTER|SEMI|ANTI] JOIN (subquery)|table (ON <expr_list>)|(USING <column_list>)
[PREWHERE expr]
[WHERE expr]
[GROUP BY expr_list] [WITH TOTALS]
[HAVING expr]
[ORDER BY expr_list] [WITH FILL] [FROM expr] [TO expr] [STEP expr] 
[LIMIT [offset_value, ]n BY columns]
[LIMIT [n, ]m] [WITH TIES]
[UNION ALL ...]
[INTO OUTFILE filename]
[FORMAT format]
```

所有子句都是可选的，但紧接在 SELECT 后面的必需表达式列表除外，更详细的请看 下面.

每个可选子句的具体内容在单独的部分中进行介绍，这些部分按与执行顺序相同的顺序列出:

- WITH 子句
- FROM 子句
- SAMPLE 子句
- JOIN 子句
- PREWHERE 子句
- WHERE 子句
- GROUP BY 子句
- LIMIT BY 子句
- HAVING 子句
- SELECT 子句
- DISTINCT 子句
- LIMIT 子句
- UNION ALL 子句
- INTO OUTFILE 子句
- FORMAT 子句

### 6.1.2. SELECT 子句

如果在结果中包含所有列，请使用星号 (*）符号。 例如, SELECT * FROM ....

#### 6.1.2.1. 列表匹配

将结果中的某些列与 re2 正则表达式匹配，可以使用 COLUMNS 表达。

```sql
COLUMNS('regexp')
```

例如：
```sql
CREATE TABLE default.col_names (aa Int8, ab Int8, bc Int8) ENGINE = TinyLog;
INSERT into default.col_names(aa, ab, bc) values (1, 2, 3) ;
```


以下查询所有列名包含 a 
```sql
SELECT COLUMNS('a') FROM col_names
```

![20201202142952](https://liulv.work/images/img/20201202142952.png)

所选列不按字母顺序返回。

您可以使用多个 COLUMNS 表达式并将函数应用于它们。

```sql
SELECT COLUMNS('a'), COLUMNS('c'), toTypeName(COLUMNS('c')) FROM col_names
```

返回的每一列 COLUMNS 表达式作为单独的参数传递给函数。 如果函数支持其他参数，您也可以将其他参数传递给函数。 使用函数时要小心，如果函数不支持传递给它的参数，ClickHouse将抛出异常。

例如:

SELECT COLUMNS('a') + COLUMNS('c') FROM col_names;

该例子中, COLUMNS('a') 返回两列: aa 和 ab. COLUMNS('c') 返回 bc 列。 该 + 运算符不能应用于3个参数，因此ClickHouse抛出一个带有相关消息的异常。

#### 6.1.2.2. ARRAY JOIN子句

对于包含数组列的表来说是一种常见的操作，用于生成一个新表，该表具有包含该初始列中的每个单独数组元素的列，而其他列的值将被重复显示。 这是 ARRAY JOIN 语句最基本的场景。

它可以被视为执行 JOIN 并具有数组或嵌套数据结构。 类似于 arrayJoin 功能，但该子句功能更广泛。

语法:

```sql
SELECT <expr_list>
FROM <left_subquery>
[LEFT] ARRAY JOIN <array>
[WHERE|PREWHERE <expr>]
...
```

ARRAY JOIN 支持的类型有:

- ARRAY JOIN - 一般情况下，空数组不包括在结果中 JOIN.
- LEFT ARRAY JOIN - 的结果 JOIN 包含具有空数组的行。 空数组的值设置为数组元素类型的默认值（通常为0、空字符串或NULL）。

例：
下面的例子展示 ARRAY JOIN 和 LEFT ARRAY JOIN 的用法，让我们创建一个表包含一个 Array 的列并插入值:
```sql
CREATE TABLE arrays_test
(
    s String,
    arr Array(UInt8)
) ENGINE = Memory;

INSERT INTO arrays_test
VALUES ('Hello', [1,2]), ('World', [3,4,5]), ('Goodbye', []);

SELECT s, arr
FROM arrays_test
ARRAY JOIN arr;

--可用把Array中空也查询出来
SELECT s, arr
FROM arrays_test
LEFT ARRAY JOIN arr;

--使用别名
SELECT s, arr, a
FROM arrays_test
ARRAY JOIN arr AS a;

--可以使用别名与外部数组执行 ARRAY JOIN 。 例如:
SELECT s, arr_external
FROM arrays_test
ARRAY JOIN [1, 2, 3] AS arr_external;

--结合arrayEnumerate 数组位置、arrayMap迭代遍历函数使用
SELECT s, arr, a, num, mapped
FROM arrays_test
ARRAY JOIN arr AS a, arrayEnumerate(arr) AS num, arrayMap(x -> x + 1, arr) AS mapped;

-- 结合arrayEnumerate 数组位置
SELECT s, arr, a, num, arrayEnumerate(arr)
FROM arrays_test
ARRAY JOIN arr AS a, arrayEnumerate(arr) AS num;


--ARRAY JOIN 也适用于 嵌套数据结构:
CREATE TABLE nested_test
(
    s String,
    nest Nested(
    x UInt8,
    y UInt32)
) ENGINE = Memory;

INSERT INTO nested_test
VALUES ('Hello', [1,2], [10,20]), ('World', [3,4,5], [30,40,50]), ('Goodbye', [], []);

SELECT s, `nest.x`, `nest.y`
FROM nested_test
ARRAY JOIN nest;
```

#### 6.1.2.3. Distinct子句

如果 SELECT DISTINCT 被声明，则查询结果中只保留唯一行。 因此，在结果中所有完全匹配的行集合中，只有一行被保留。

DISTINCT 适用于 NULL 就好像 NULL 是一个特定的值，并且 NULL==NULL. 换句话说，在 DISTINCT 结果，不同的组合 NULL 仅发生一次。 它不同于 NULL 在大多数其他情况中的处理方式。

通过应用可以获得相同的结果 GROUP BY 在同一组值指定为 SELECT 子句，并且不使用任何聚合函数。 但与 GROUP BY 有几个不同的地方:

DISTINCT 可以与 GROUP BY 一起使用.
当 ORDER BY 被省略并且 LIMIT 被定义时，在读取所需数量的不同行后立即停止运行。
数据块在处理时输出，而无需等待整个查询完成运行。

DISTINCT 不支持当 SELECT 包含有数组的列。

#### 6.1.2.4. Format子句

#### 6.1.2.5. GROUP BY子句

GROUP BY 子句将 SELECT 查询结果转换为聚合模式，其工作原理如下:

GROUP BY 子句包含表达式列表（或单个表达式 -- 可以认为是长度为1的列表）。 这份名单充当 “grouping key”，而每个单独的表达式将被称为 “key expressions”.
在所有的表达式在 SELECT, HAVING，和 ORDER BY 子句中 必须 基于键表达式进行计算 或 上 聚合函数 在非键表达式（包括纯列）上。 换句话说，从表中选择的每个列必须用于键表达式或聚合函数内，但不能同时使用。

聚合结果 SELECT 查询将包含尽可能多的行，因为有唯一值 “grouping key” 在源表中。 通常这会显着减少行数，通常是数量级，但不一定：如果所有行数保持不变 “grouping key” 值是不同的。

**注意：**

还有一种额外的方法可以在表上运行聚合。 如果查询仅在聚合函数中包含表列，则 GROUP BY 可以省略，并且通过一个空的键集合来假定聚合。 这样的查询总是只返回一行。

**空值处理：**

对于分组，ClickHouse解释 NULL 作为一个值，并且 **NULL==NULL**. 它不同于 NULL 在大多数其他上下文中的处理方式。相当于把null 当着一个具体的值。

**WITH TOTAL 修饰符**
如果 WITH TOTALS 被指定，将计算另一行。 此行将具有包含默认值（零或空行）的关键列，以及包含跨所有行计算值的聚合函数列（ “total” 值）。

这个额外的行仅产生于 JSON*, TabSeparated*，和 Pretty* 格式，与其他行分开:

* 在 JSON* 格式，这一行是作为一个单独的输出 ‘totals’ 字段。
* 在 TabSeparated* 格式，该行位于主结果之后，前面有一个空行（在其他数据之后）。
* 在 Pretty* 格式时，该行在主结果之后作为单独的表输出。
* 在其他格式中，它不可用。

WITH TOTALS 可以以不同的方式运行时 HAVING 是存在的。 该行为取决于 totals_mode 设置。


**配置总和处理**

默认情况下, totals_mode = 'before_having'. 在这种情况下, ‘totals’ 是跨所有行计算，包括那些不通过具有和 max_rows_to_group_by.

其他替代方案仅包括通过具有在 ‘totals’，并与设置不同的行为 max_rows_to_group_by 和 group_by_overflow_mode = 'any'.

after_having_exclusive – Don't include rows that didn't pass through max_rows_to_group_by. 换句话说, ‘totals’ 将有少于或相同数量的行，因为它会 max_rows_to_group_by 被省略。

after_having_inclusive – Include all the rows that didn't pass through ‘max_rows_to_group_by’ 在 ‘totals’. 换句话说, ‘totals’ 将有多个或相同数量的行，因为它会 max_rows_to_group_by 被省略。

after_having_auto – Count the number of rows that passed through HAVING. If it is more than a certain amount (by default, 50%), include all the rows that didn't pass through ‘max_rows_to_group_by’ 在 ‘totals’. 否则，不包括它们。

totals_auto_threshold – By default, 0.5. The coefficient for after_having_auto.

如果 max_rows_to_group_by 和 group_by_overflow_mode = 'any' 不使用，所有的变化 after_having 是相同的，你可以使用它们中的任何一个（例如, after_having_auto).

您可以使用 WITH TOTALS 在子查询中，包括在子查询 JOIN 子句（在这种情况下，将各自的总值合并）。

例子 
示例:

```sql
SELECT
    count(),
    median(FetchTiming > 60 ? 60 : FetchTiming),
    count() - sum(Refresh)
FROM hits
```

但是，与标准SQL相比，如果表没有任何行（根本没有任何行，或者使用 WHERE 过滤之后没有任何行），则返回一个空结果，而不是来自包含聚合函数初始值的行。

相对于MySQL（并且符合标准SQL），您无法获取不在键或聚合函数（常量表达式除外）中的某些列的某些值。 要解决此问题，您可以使用 ‘any’ 聚合函数（获取第一个遇到的值）或 ‘min/max’.

示例:

```sql
SELECT
    domainWithoutWWW(URL) AS domain,
    count(),
    any(Title) AS title -- getting the first occurred page header for each domain.
FROM hits
GROUP BY domain
```

对于遇到的每个不同的键值, GROUP BY 计算一组聚合函数值。

GROUP BY 不支持数组列。

不能将常量指定为聚合函数的参数。 示例: sum(1). 相反，你可以摆脱常数。 示例: count().


**在外部存储器中分组**

您可以启用将临时数据转储到磁盘以限制内存使用期间 GROUP BY.
该 max_bytes_before_external_group_by 设置确定倾销的阈值RAM消耗 GROUP BY 临时数据到文件系统。 如果设置为0（默认值），它将被禁用。

使用时 max_bytes_before_external_group_by，我们建议您设置 max_memory_usage 大约两倍高。 这是必要的，因为聚合有两个阶段：读取数据和形成中间数据（1）和合并中间数据（2）。 将数据转储到文件系统只能在阶段1中发生。 如果未转储临时数据，则阶段2可能需要与阶段1相同的内存量。

例如，如果 max_memory_usage 设置为10000000000，你想使用外部聚合，这是有意义的设置 max_bytes_before_external_group_by 到10000000000，和 max_memory_usage 到20000000000。 当触发外部聚合（如果至少有一个临时数据转储）时，RAM的最大消耗仅略高于 max_bytes_before_external_group_by.

通过分布式查询处理，在远程服务器上执行外部聚合。 为了使请求者服务器只使用少量的RAM，设置 distributed_aggregation_memory_efficient 到1。

当合并数据刷新到磁盘时，以及当合并来自远程服务器的结果时， distributed_aggregation_memory_efficient 设置被启用，消耗高达 1/256 * the_number_of_threads 从RAM的总量。

当启用外部聚合时，如果数据量小于 max_bytes_before_external_group_by (例如数据没有被 flushed), 查询执行速度和不在外部聚合的速度一样快. 如果临时数据被flushed到外部存储, 执行的速度会慢几倍 (大概是三倍).

如果你有一个 ORDER BY 用一个 LIMIT 后 GROUP BY，然后使用的RAM的量取决于数据的量 LIMIT，不是在整个表。 但如果 ORDER BY 没有 LIMIT，不要忘记启用外部排序 (max_bytes_before_external_sort).

#### 6.1.2.6. HAVING 子句

允许过滤由 GROUP BY 生成的聚合结果. 它类似于 WHERE ，但不同的是 WHERE 在聚合之前执行，而 HAVING 之后进行。

可以从 SELECT 生成的聚合结果中通过他们的别名来执行 HAVING 子句。 或者 HAVING 子句可以筛选查询结果中未返回的其他聚合的结果。

限制 
HAVING 如果不执行聚合则无法使用。 使用 WHERE 则相反。

#### 6.1.2.7. INTO OUTFILE 子句

添加 INTO OUTFILE filename 子句（其中filename是字符串） SELECT query 将其输出重定向到客户端上的指定文件。

实现细节 
- 此功能是在可用 命令行客户端 和 clickhouse-local. 因此通过 HTTP接口 发送查询将会失败。
- 如果具有相同文件名的文件已经存在，则查询将失败。
- 默认值 输出格式 是 TabSeparated(使用Tab分割) （就像在命令行客户端批处理模式中一样）。



#### 6.1.2.8. JOIN子句

Join通过使用一个或多个表的公共值合并来自一个或多个表的列来生成新表。 它是支持SQL的数据库中的常见操作，它对应于 关系代数 加入。 一个表连接的特殊情况通常被称为 “self-join”.

语法:

```sql
SELECT <expr_list>
FROM <left_table>
[GLOBAL] [INNER|LEFT|RIGHT|FULL|CROSS] [OUTER|SEMI|ANTI|ANY|ASOF] JOIN <right_table>
(ON <expr_list>)|(USING <column_list>) ...
```

从表达式 ON 从子句和列 USING 子句被称为 “join keys”. 除非另有说明，加入产生一个 笛卡尔积 从具有匹配的行 “join keys”，这可能会产生比源表更多的行的结果。

**支持的联接类型**

所有标准 SQL JOIN 支持类型:

- INNER JOIN，只返回匹配的行。
- LEFT OUTER JOIN，除了匹配的行之外，还返回左表中的非匹配行。
- RIGHT OUTER JOIN，除了匹配的行之外，还返回右表中的非匹配行。
- FULL OUTER JOIN，除了匹配的行之外，还会返回两个表中的非匹配行。
- CROSS JOIN，产生整个表的笛卡尔积, “join keys” 是 不 指定。
- JOIN 没有指定类型暗指 INNER. 关键字 OUTER 可以安全地省略。 替代语法 CROSS JOIN 在指定多个表 FROM 用逗号分隔。

ClickHouse中提供的其他联接类型:

- LEFT SEMI JOIN 和 RIGHT SEMI JOIN,白名单 “join keys”，而不产生笛卡尔积。
- LEFT ANTI JOIN 和 RIGHT ANTI JOIN，黑名单 “join keys”，而不产生笛卡尔积。
- LEFT ANY JOIN, RIGHT ANY JOIN and INNER ANY JOIN, partially (for opposite side of LEFT and RIGHT) or completely (for INNER and FULL) disables the - cartesian product for standard JOIN types.
- ASOF JOIN and LEFT ASOF JOIN, joining sequences with a non-exact match. ASOF JOIN usage is described below.

**ASOF加入使用**

ASOF JOIN 当您需要连接没有完全匹配的记录时非常有用。

算法需要表中的特殊列。 本专栏:

必须包含有序序列。
可以是以下类型之一: Int，UInt, 浮动*, 日期, 日期时间, 十进制*.
不能是唯一的列 JOIN
语法 ASOF JOIN ... ON:

SELECT expressions_list
FROM table_1
ASOF LEFT JOIN table_2
ON equi_cond AND closest_match_cond
您可以使用任意数量的相等条件和恰好一个最接近的匹配条件。 例如, SELECT count() FROM table_1 ASOF LEFT JOIN table_2 ON table_1.a == table_2.b AND table_2.t <= table_1.t.

支持最接近匹配的条件: >, >=, <, <=.

语法 ASOF JOIN ... USING:

```sql
SELECT expressions_list
FROM table_1
ASOF JOIN table_2
USING (equi_column1, ... equi_columnN, asof_column)
```

ASOF JOIN 用途 equi_columnX 对于加入平等和 asof_column 用于加入与最接近的比赛 table_1.asof_column >= table_2.asof_column 条件。 该 asof_column 列总是在最后一个 USING 条款

例如，请考虑下表:

     table_1                    
  event   | ev_time | user_id   
----------|---------|----------             
event_1_1 |  12:00  |  42                 
event_1_2 |  13:00  |  42       
                 
     table_2
  event   | ev_time | user_id
----------|---------|----------
event_2_1 |  11:59  |   42
event_2_2 |  12:30  |   42
event_2_3 |  13:00  |   42

ASOF JOIN 可以从用户事件的时间戳 table_1 并找到一个事件 table_2 其中时间戳最接近事件的时间戳 table_1 对应于最接近的匹配条件。 如果可用，则相等的时间戳值是最接近的值。 在这里，该 user_id 列可用于连接相等和 ev_time 列可用于在最接近的匹配加入。 在我们的例子中, event_1_1 可以加入 event_2_1 和 event_1_2 可以加入 event_2_3，但是 event_2_2 不能加入。

注

ASOF 加入是不支持在加入我们表引擎。

**分布式联接** 

有两种方法可以执行涉及分布式表的join:

当使用正常 JOIN，将查询发送到远程服务器。 为了创建正确的表，在每个子查询上运行子查询，并使用此表执行联接。 换句话说，在每个服务器上单独形成右表。
使用时 GLOBAL ... JOIN，首先请求者服务器运行一个子查询来计算正确的表。 此临时表将传递到每个远程服务器，并使用传输的临时数据对其运行查询。
使用时要小心 GLOBAL. 有关详细信息，请参阅 分布式子查询 科。


**处理空单元格或空单元格**

在连接表时，可能会出现空单元格。 设置 join_use_nulls 定义ClickHouse如何填充这些单元格。

如果 JOIN 键是 可为空 字段，其中至少有一个键具有值的行 NULL 没有加入。

语法 

在指定的列 USING 两个子查询中必须具有相同的名称，并且其他列必须以不同的方式命名。 您可以使用别名更改子查询中的列名。

该 USING 子句指定一个或多个要联接的列，这将建立这些列的相等性。 列的列表设置不带括号。 不支持更复杂的连接条件。

语法限制 

对于多个 JOIN 单个子句 SELECT 查询:

通过以所有列 * 仅在联接表时才可用，而不是子查询。
该 PREWHERE 条款不可用。
为 ON, WHERE，和 GROUP BY 条款:

任意表达式不能用于 ON, WHERE，和 GROUP BY 子句，但你可以定义一个表达式 SELECT 子句，然后通过别名在这些子句中使用它。
性能 
当运行 JOIN，与查询的其他阶段相关的执行顺序没有优化。 连接（在右表中搜索）在过滤之前运行 WHERE 和聚集之前。

每次使用相同的查询运行 JOIN，子查询再次运行，因为结果未缓存。 为了避免这种情况，使用特殊的 加入我们 表引擎，它是一个用于连接的准备好的数组，总是在RAM中。

在某些情况下，使用效率更高 IN 而不是 JOIN.

如果你需要一个 JOIN 对于连接维度表（这些是包含维度属性的相对较小的表，例如广告活动的名称）， JOIN 由于每个查询都会重新访问正确的表，因此可能不太方便。 对于这种情况下，有一个 “external dictionaries” 您应该使用的功能 JOIN. 有关详细信息，请参阅 外部字典 科。

内存限制 
默认情况下，ClickHouse使用 哈希联接 算法。 ClickHouse采取 <right_table> 并在RAM中为其创建哈希表。 在某个内存消耗阈值之后，ClickHouse回退到合并联接算法。

如果需要限制联接操作内存消耗，请使用以下设置:

max_rows_in_join — Limits number of rows in the hash table.
max_bytes_in_join — Limits size of the hash table.
当任何这些限制达到，ClickHouse作为 join_overflow_mode 设置指示。

例子 
示例:

```sql

set joined_subquery_requires_alias=0;

SELECT
    CounterID,
    hits,
    visits
FROM
(
    SELECT
        CounterID,
        count() AS hits
    FROM test.hits
    GROUP BY CounterID
) ANY LEFT JOIN
(
    SELECT
        CounterID,
        sum(Sign) AS visits
    FROM test.visits
    GROUP BY CounterID
) USING CounterID
ORDER BY hits DESC
LIMIT 10
```


┌─CounterID─┬───hits─┬─visits─┐
│   1143050 │ 523264 │  13665 │
│    731962 │ 475698 │ 102716 │
│    722545 │ 337212 │ 108187 │
│    722889 │ 252197 │  10547 │
│   2237260 │ 196036 │   9522 │
│  23057320 │ 147211 │   7689 │
│    722818 │  90109 │  17847 │
│     48221 │  85379 │   4652 │
│  19762435 │  77807 │   7026 │
│    722884 │  77492 │  11056 │
└───────────┴────────┴────────┘

#### 6.1.2.9. LIMIT BY子句

与查询 LIMIT n BY expressions 子句选择第一个 n 每个不同值的行 expressions. LIMIT BY 可以包含任意数量的 表达式.

例：

CREATE TABLE limit_by(id Int, val Int) ENGINE = Memory;
INSERT INTO limit_by VALUES (1, 10), (1, 11), (1, 12), (2, 20), (2, 21);

SELECT * FROM limit_by ORDER BY id, val LIMIT 2 BY id

SELECT * FROM limit_by ORDER BY id, val LIMIT 1, 2 BY id;


domainWithoutWWW(URL) AS domain,
domainWithoutWWW(REFERRER_URL) AS referrer,
device_type,
count() cnt
FROM hits
GROUP BY domain, referrer, device_type
ORDER BY cnt DESC
LIMIT 5 BY domain, device_type
LIMIT 100


#### 6.1.2.10. ORDER BY子句

ORDER BY 子句包含一个表达式列表，每个表达式都可以用 DESC （降序）或 ASC （升序）修饰符确定排序方向。 如果未指定方向, 默认是 ASC ，所以它通常被省略。 排序方向适用于单个表达式，而不适用于整个列表。 示例: ORDER BY Visits DESC, SearchPhrase


**特殊值的排序**

有两种方法 NaN 和 NULL 排序顺序:

* 默认情况下或与 NULLS LAST 修饰符：首先是值，然后 NaN（字符串），然后 NULL.
* 与 NULLS FIRST 修饰符：第一 NULL，然后 NaN，然后其他值。

第二种方式：

```sql
SELECT * FROM t_null_nan ORDER BY y NULLS FIRST
```

当对浮点数进行排序时，Nan与其他值是分开的。 无论排序顺序如何，Nan都在最后。 换句话说，对于升序排序，它们被放置为好像它们比所有其他数字大，而对于降序排序，它们被放置为好像它们比其他数字小。

**ORDER BY Expr WITH FILL Modifier**

此修饰符可以与 LIMIT … WITH TIES modifier 进行组合使用.

可以在ORDER BY expr之后用可选的FROM expr，TO expr和STEP expr参数来设置WITH FILL修饰符。
所有expr列的缺失值将被顺序填充，而其他列将被填充为默认值。

使用以下语法填充多列，在ORDER BY部分的每个字段名称后添加带有可选参数的WITH FILL修饰符。

ORDER BY expr [WITH FILL] [FROM const_expr] [TO const_expr] [STEP const_numeric_expr], ... exprN [WITH FILL] [FROM expr] [TO expr] [STEP numeric_expr]
WITH FILL 仅适用于具有数字（所有类型的浮点，小数，整数）或日期/日期时间类型的字段。
当未定义 FROM const_expr 填充顺序时，则使用 ORDER BY 中的最小 expr 字段值。
如果未定义 TO const_expr 填充顺序，则使用 ORDER BY 中的最大expr字段值。
当定义了 STEP const_numeric_expr 时，对于数字类型，const_numeric_expr 将 as is 解释为 days 作为日期类型，将 seconds 解释为DateTime类型。
如果省略了 STEP const_numeric_expr，则填充顺序使用 1.0 表示数字类型，1 day表示日期类型，1 second 表示日期时间类型。

例如下面的查询：

```sql
SELECT n, source FROM (
   SELECT toFloat32(number % 10) AS n, 'original' AS source
   FROM numbers(10) WHERE number % 3 = 1
) ORDER BY n
```

返回

┌─n─┬─source───┐
│ 1 │ original │
│ 4 │ original │
│ 7 │ original │
└───┴──────────┘
但是如果配置了 WITH FILL 修饰符

```sql
SELECT n, source FROM (
   SELECT toFloat32(number % 10) AS n, 'original' AS source
   FROM numbers(10) WHERE number % 3 = 1
) ORDER BY n WITH FILL FROM 0 TO 5.51 STEP 0.5
```

返回

┌───n─┬─source───┐
│   0 │          │
│ 0.5 │          │
│   1 │ original │
│ 1.5 │          │
│   2 │          │
│ 2.5 │          │
│   3 │          │
│ 3.5 │          │
│   4 │ original │
│ 4.5 │          │
│   5 │          │
│ 5.5 │          │
│   7 │ original │
└─────┴──────────┘
For the case when we have multiple fields ORDER BY field2 WITH FILL, field1 WITH FILL order of filling will follow the order of fields in ORDER BY clause.
对于我们有多个字段 ORDER BY field2 WITH FILL, field1 WITH FILL 的情况，填充顺序将遵循ORDER BY子句中字段的顺序。

示例:
```sql
SELECT
    toDate((number * 10) * 86400) AS d1,
    toDate(number * 86400) AS d2,
    'original' AS source
FROM numbers(10)
WHERE (number % 3) = 1
ORDER BY
    d2 WITH FILL,
    d1 WITH FILL STEP 5;
```
返回

┌───d1───────┬───d2───────┬─source───┐
│ 1970-01-11 │ 1970-01-02 │ original │
│ 1970-01-01 │ 1970-01-03 │          │
│ 1970-01-01 │ 1970-01-04 │          │
│ 1970-02-10 │ 1970-01-05 │ original │
│ 1970-01-01 │ 1970-01-06 │          │
│ 1970-01-01 │ 1970-01-07 │          │
│ 1970-03-12 │ 1970-01-08 │ original │
└────────────┴────────────┴──────────┘
字段 d1 没有填充并使用默认值，因为我们没有 d2 值的重复值，并且无法正确计算 d1 的顺序。
以下查询中ORDER BY 中的字段将被更改
```sql
SELECT
    toDate((number * 10) * 86400) AS d1,
    toDate(number * 86400) AS d2,
    'original' AS source
FROM numbers(10)
WHERE (number % 3) = 1
ORDER BY
    d1 WITH FILL STEP 5,
    d2 WITH FILL;
```
返回


┌───d1───────┬───d2───────┬─source───┐
│ 1970-01-11 │ 1970-01-02 │ original │
│ 1970-01-16 │ 1970-01-01 │          │
│ 1970-01-21 │ 1970-01-01 │          │
│ 1970-01-26 │ 1970-01-01 │          │
│ 1970-01-31 │ 1970-01-01 │          │
│ 1970-02-05 │ 1970-01-01 │          │
│ 1970-02-10 │ 1970-01-05 │ original │
│ 1970-02-15 │ 1970-01-01 │          │
│ 1970-02-20 │ 1970-01-01 │          │
│ 1970-02-25 │ 1970-01-01 │          │
│ 1970-03-02 │ 1970-01-01 │          │
│ 1970-03-07 │ 1970-01-01 │          │
│ 1970-03-12 │ 1970-01-08 │ original │
└────────────┴────────────┴──────────┘

#### 6.1.2.11. PREWHERE 子句

Prewhere是更有效地进行过滤的优化。 默认情况下，即使在 PREWHERE 子句未显式指定。 它也会自动移动 WHERE 条件到prewhere阶段。 PREWHERE 子句只是控制这个优化，如果你认为你知道如何做得比默认情况下更好才去控制它。

使用prewhere优化，首先只读取执行prewhere表达式所需的列。 然后读取运行其余查询所需的其他列，但只读取prewhere表达式所在的那些块 “true” 至少对于一些行。 如果有很多块，其中prewhere表达式是 “false” 对于所有行和prewhere需要比查询的其他部分更少的列，这通常允许从磁盘读取更少的数据以执行查询。

**手动控制Prewhere**

该子句具有与 WHERE 相同的含义，区别在于从表中读取数据。 当手动控制 PREWHERE 对于查询中的少数列使用的过滤条件，但这些过滤条件提供了强大的数据过滤。 这减少了要读取的数据量。

查询可以同时指定 PREWHERE 和 WHERE. 在这种情况下, PREWHERE 先于 WHERE.

如果 optimize_move_to_prewhere 设置为0，启发式自动移动部分表达式 WHERE 到 PREWHERE 被禁用。

**限制** 
PREWHERE 只有支持 *MergeTree 族系列引擎的表。

#### 6.1.2.12. SAMPLE 采样子句

启用数据采样时，不会对所有数据执行查询，而只对特定部分数据（样本）执行查询。 例如，如果您需要计算所有访问的统计信息，只需对所有访问的1/10分数执行查询，然后将结果乘以10即可。

近似查询处理在以下情况下可能很有用:

- 当你有严格的时间需求（如\<100ms），但你不能通过额外的硬件资源来满足他们的成本。
- 当您的原始数据不准确时，所以近似不会明显降低质量。
- 业务需求的目标是近似结果（为了成本效益，或者向高级用户推销确切结果）。

您只能使用采样中的表 MergeTree 族，并且只有在表创建过程中指定了采样表达式

下面列出了数据采样的功能:

- 数据采样是一种确定性机制。 同样的结果 SELECT .. SAMPLE 查询始终是相同的。
- 对于不同的表，采样工作始终如一。 对于具有单个采样键的表，具有相同系- 数的采样总是选择相同的可能数据子集。 例如，用户Id的示例采用来自不同表的所有可能的用户Id的相同子集的行。 这意味着您可以在子查询中使用采样 IN 此外，您可以使用 JOIN 。
- 采样允许从磁盘读取更少的数据。 请注意，您必须正确指定采样键。 有关详- 细信息，请参阅 创建MergeTree表.

为 SAMPLE 子句支持以下语法:

SAMPLE k	这里 k 是从0到1的数字。


查询执行于 k 数据的分数。 例如, SAMPLE 0.1 对10%的数据运行查询。 Read more SAMPLE n 这里 n 是足够大的整数。该查询是在至少一个样本上执行的 n 行（但不超过这个）。 例如, SAMPLE 10000000 在至少10,000,000行上运行查询。 Read more SAMPLE k OFFSET m 这里 k 和 m 是从0到1的数字。查询在以下示例上执行 k 数据的分数。 用于采样的数据由以下偏移 m 分数。

**SAMPLE K**

这里 k 从0到1的数字（支持小数和小数表示法）。 例如, SAMPLE 1/2 或 SAMPLE 0.5.

在一个 SAMPLE k 子句，样品是从 k 数据的分数。 示例如下所示:

```sql
SELECT
    Title,
    count() * 10 AS PageViews
FROM hits_distributed
SAMPLE 0.1
WHERE
    CounterID = 34
GROUP BY Title
ORDER BY PageViews DESC LIMIT 1000
```

在此示例中，对0.1(10%)数据的样本执行查询。 聚合函数的值不会自动修正，因此要获得近似结果，值 count() 手动乘以10。


**SAMPLE N**

这里 n 是足够大的整数。 例如, SAMPLE 10000000.

在这种情况下，查询在至少一个样本上执行 n 行（但不超过这个）。 例如, SAMPLE 10000000 在至少10,000,000行上运行查询。

由于数据读取的最小单位是一个颗粒（其大小由 index_granularity 设置），是有意义的设置一个样品，其大小远大于颗粒。

使用时 SAMPLE n 子句，你不知道处理了哪些数据的相对百分比。 所以你不知道聚合函数应该乘以的系数。 使用 _sample_factor 虚拟列得到近似结果。

该 _sample_factor 列包含动态计算的相对系数。 当您执行以下操作时，将自动创建此列 创建 具有指定采样键的表。 的使用示例 _sample_factor 列如下所示。

让我们考虑表 visits，其中包含有关网站访问的统计信息。 第一个示例演示如何计算页面浏览量:

SELECT sum(PageViews * _sample_factor)
FROM visits
SAMPLE 10000000
下一个示例演示如何计算访问总数:

SELECT sum(_sample_factor)
FROM visits
SAMPLE 10000000
下面的示例显示了如何计算平均会话持续时间。 请注意，您不需要使用相对系数来计算平均值。

SELECT avg(Duration)
FROM visits
SAMPLE 10000000
SAMPLE K OFFSET M 
这里 k 和 m 是从0到1的数字。 示例如下所示。

示例1

SAMPLE 1/10
在此示例中，示例是所有数据的十分之一:

[++------------]

示例2

SAMPLE 1/10 OFFSET 1/2
这里，从数据的后半部分取出10％的样本

#### 6.1.2.13. UNION ALL子句

你可以使用 UNION ALL 结合任意数量的 SELECT 来扩展其结果。 示例:

```sql
SELECT CounterID, 1 AS table, toInt64(count()) AS c
    FROM test.hits
    GROUP BY CounterID

UNION ALL

SELECT CounterID, 2 AS table, sum(Sign) AS c
    FROM test.visits
    GROUP BY CounterID
    HAVING c > 0
```

结果列通过它们的索引进行匹配（在内部的顺序 SELECT). 如果列名称不匹配，则从第一个查询中获取最终结果的名称。

对联合执行类型转换。 例如，如果合并的两个查询具有相同的字段与非-Nullable 和 Nullable 从兼容类型的类型，由此产生的 UNION ALL 有一个 Nullable 类型字段。

属于以下部分的查询 UNION ALL 不能用圆括号括起来。 ORDER BY 和 LIMIT 应用于单独的查询，而不是最终结果。 如果您需要将转换应用于最终结果，则可以将所有查询 UNION ALL 在子查询中 FROM 子句。

## ALTER

ALTER 仅支持 *MergeTree ，Merge以及Distributed等引擎表。
该操作有多种形式。

**改变表结构：**

ALTER TABLE [db].name [ON CLUSTER cluster] ADD|DROP|CLEAR|COMMENT|MODIFY COLUMN ...

在语句中，配置一个或多个用逗号分隔的动作。每个动作是对某个列实施的操作行为。

支持下列动作：

* ADD COLUMN — 添加列
* DROP COLUMN — 删除列
* CLEAR COLUMN — 重置列的值
* COMMENT COLUMN — 给列增加注释说明
* MODIFY COLUMN — 改变列的值类型，默认表达式以及TTL
* 
这些动作将在下文中进行详述。

**增加列**
ADD COLUMN [IF NOT EXISTS] name [type] [default_expr] [codec] [AFTER name_after]

ALTER TABLE visits ADD COLUMN browser String AFTER user_id

**删除列**
DROP COLUMN [IF EXISTS] name

ALTER TABLE visits DROP COLUMN browser

**清空列**

CLEAR COLUMN [IF EXISTS] name IN PARTITION partition_name

ALTER TABLE visits CLEAR COLUMN browser IN PARTITION tuple()

**增加注释**

COMMENT COLUMN [IF EXISTS] name 'comment'

ALTER TABLE visits COMMENT COLUMN browser 'The table shows the browser used for accessing the site.

**修改列**

MODIFY COLUMN [IF EXISTS] name [type] [default_expr] [TTL]

该语句可以改变 name 列的属性：

* Type
* Default expression
* TTL

ALTER TABLE visits MODIFY COLUMN browser Array(String)

改变列的类型是唯一的复杂型动作 - 它改变了数据文件的内容。对于大型表，执行起来要花费较长的时间。
该操作分为如下处理步骤：

* 为修改的数据准备新的临时文件
* 重命名原来的文件
* 将新的临时文件改名为原来的数据文件名
* 删除原来的文件

**key表达式的修改**

MODIFY ORDER BY new_expression

该操作仅支持 MergeTree 系列表 (含 replicated 表)。它会将表的 排序键变成 new_expression (元组表达式)。主键仍保持不变。

该操作是轻量级的，仅会改变元数据。

**跳过索引来更改数据**

该操作仅支持 MergeTree 系列表 (含 replicated 表)。
下列操作是允许的：

ALTER TABLE [db].name ADD INDEX name expression TYPE type GRANULARITY value AFTER name [AFTER name2] - 在表的元数据中增加索引说明

ALTER TABLE [db].name DROP INDEX name - 从表的元数据中删除索引描述，并从磁盘上删除索引文件

由于只改变表的元数据或者删除文件，因此该操作是轻量级的，也可以被复制到其它节点（通过Zookeeper同步索引元数据）

![20201211173641](https://liulv.work/images/img/20201211173641.png)



