---
title: DSEGraph-DataLoader使用手册
tags: 图库
---

## 安装GraphLoader
--

### 下载Graph Loader安装包

点击[下载地址](https://pan.baidu.com/s/1erorpQrwXr0wUHRMgVQnMA "下载地址")进行下载。

下载得到dse-grap-loader-1.0.1.zip的压缩文件

**注意：** 下载提取码为d4t4

### 解压安装

#### 步骤：

1. 在集群需要安装graph-loader的节点，如19.31节点上创建安装目录
```sh
mkdir -p /usr/share/dse-graph-loader
```

2. 将上面下载的dse-grap-loader-1.0.1.zip的压缩文件通过sftp方式拷贝到/usr/share/dse-graph-loader路径下，如下图

![](https://v4liulv.github.io/assets/image/1529315140715_74.png)

3. 执行解压命令
```
cd /usr/share/dse-graph-loader
unzip dse-graph-loader-1.0.1.zip
```

![](https://v4liulv.github.io/assets/image/1529315807911_42.png)

得到`/usr/share/dse-graph-loader/`的目录即为graph loader目录

4. 设置环境变量

执行下面命令
```sh
echo 'export DES_GLOADER_HOME=/usr/share/dse-graph-loader' >> /etc/profile
echo 'export PATH=$DES_GLOADER_HOME:$PATH' >> /etc/profile
```

```sh
source /etc/profile
chmod +755 /usr/share/dse-graph-loader/*.sh
```

![](https://v4liulv.github.io/assets/image/1529316233010_69.png)

#### 检查是否安装成功
此命令是查询dse loader和dse的版本信息
```
graphloader -v
```

如果出现下图所示证明安装成功:

![](https://v4liulv.github.io/assets/image/1529316305475_70.png)


## 配置参数

公共配置文件在`/usr/share/dse-graph-loader/scripts/loadJdbc/base`目录下的base.groovy

```groovy
// 此处调用DataLoaderImpl.config()
config create_schema: true, load_new: false, load_vertex_threads: 200, load_edge_threads:100, read_threads:100
//是否自动创建模型，可选默认为true
create_schema = true
//是否是数据库导入方式，需要配置为true
isDatabase = true

// Oracle 连接参数配置
inputDatabase = '192.168.1.110:1521:orcl'
user = 'YY_GXKSH_ZSB'
password = '123456'
configTable = 'B_DSE_LOADER_CONFIG'
datePattern = "yyyy-mm-dd hh24:mi:ss"
```

|  配置项 |  值 |  说明 |
| ------------ | ------------ | ------------ |
| preparation  | true  | 是一种有效的检查机制。如果preparation设置true，那么就分析该模式是否有效的数据示例  |
| create_schema  | true  | 模式是从数据导入过程中创建，如果为false则需要先手动创建模型，生成环境需要设置为false |
| load_new  | true  | 新图如果在加载过程的开始时，顶点记录还不存在，配置load_new可以显著加快加载过程。然而，重要的是用户保证顶点记录确实是新的，或者在图中可以创建重复的顶点。在同一个脚本中创建的边将使用新创建的顶点来表示外向的inV和outV  如果load_new被设置为false，并且载入的数据包含在图中已经存在的任何顶点，就会创建重复的顶点。|
| load_vertex_threads | 200 |加载顶点数据线程数为200 |
| load_edge_threads | 100 |加载边数据线程数为100 |
| isDatabase | true | 加载数据来源是否JDBC |
| inputDatabase | 192.168.1.110:1521:orcl | 数据抽取来源库URL  |
| user | YY_GXKSH_ZSB |  数据抽取来源库的用户名 |
| password | 123456 | 数据抽取来源库的密码 |
| configTable | B_DSE_LOADER_CONFIG | 数据抽取配置表 |
| datePattern | yyyy-mm-dd hh24:mi:ss | 时间字符格式 |

**注意**：其中数据库配置需要根据现场的数据抽取来源库进行配置修改，其他可选进行修改

## 数据抽取配置表

### 数据抽取配置表 - B_DSE_LOADER_CONFIG
[建表语句和插入sql](http:///dse/数据抽取/db/B_DSE_LOADER_CONFIG.sql "建表语句和插入sql")

字段说明：

|  字段名 |  字段类型 |  可为空 | 默认值 |  字段说明 |
| ------------ | ------------ | ------------ | ------------ | ------------ |
| SYSTEMID | VARCHAR2(32) |  | SYS_GUID() | 主键 |
| CATEGORY | VARCHAR2(50)  |  |  | 抽取分类，01代表抽取人轨迹关系抽取配置，可扩展 |
| ZY_YWMC | VARCHAR2(50) |  |  | 抽取资源英文名称 |
| ZY_ZWMC | VARCHAR2(100) | Y |  | 抽取资源中文名称 |
| ZY_QWK_VIEW | VARCHAR2(30) | Y |  | 全文库视图 |
| ZY_YYK_VIEW | VARCHAR2(30) | Y |  | 应用库视图 |
| ZY_WBZY_VIEW | VARCHAR2(30) | Y |  | 缓冲库库视图 |
| ZY_WBZY_VIEW | VARCHAR2(30) | Y |  | 缓冲库库视图 |
| ZY_KETTLE_MC | VARCHAR2(255) | Y |  | 配置kettle的转换名对应PCS_WBZY_DATA.TRANS_LOG传输日志的TRANSNAME，用于生成批次 |
| ZLCQ_KSSJ | DATE | Y |  | 增量抽取_开始时间 |
| ZLCQ_JSSJ | DATE | Y |  | 增量抽取_结束时间 |
| ZLCQ_ZT | VARCHAR2(2) |  | 0 | 增量抽取_抽取状态（0初始，1完成, 2抽取中） |
| ZLCQ_TYPE | VARCHAR2(2) |  |  | 增量配置标签，用于区分每个增量配置信息 |
| BLZD1-5 | VARCHAR2(30-1000) |  |  | 扩展字段1-5 |
| CREATE_USER | VARCHAR2(30-1000) |  | 'SYS' | 创建人 |
| CREATE_TIME | DATE |  | SYSDATE | 创建时间 |
| UPDATE_TIME | DATE | Y | SYSDATE | 更新时间 |
| SCBZ | NUMBER(2) |  | 0 | 删除标志，0代表未删除，1代表删除 |

### 数据抽取配置日志表 B_DSE_LOADER_LOG
[建表语句](http:///dse/数据抽取/db/B_DSE_LOADER_LOG.sql "建表语句和插入sql")

字段说明：

|  字段名 |  字段类型 |  可为空 | 默认值 |  字段说明 |
| ------------ | ------------ | ------------ | ------------ | ------------ |
| SYSTEMID | VARCHAR2(32) |  | SYS_GUID() | 主键 |
| CATEGORY | VARCHAR2(50)  |  |  | 抽取分类，01代表抽取人轨迹关系抽取配置，可扩展 |
| ZLCQ_TYPE | VARCHAR2(2) |  |  | 增量配置标签，用于区分每个增量配置信息 |
| ZLCQ_KSSJ | DATE | Y |  | 增量抽取_开始时间 |
| ZLCQ_JSSJ | DATE | Y |  | 增量抽取_结束时间 |
| ZLCQ_ZT | VARCHAR2(2) |  | 0 | 增量抽取_抽取状态（0初始，1完成, 2抽取中） |
| BLZD1-5 | VARCHAR2(30-1000) | Y |  | 扩展字段1-5 |
| VERTICES | NUMBER | Y | 0 | 顶点数 |
| EDGES |NUMBER | Y | 0 | 边数 |
| PROPERTIES |NUMBER | Y | 0 | 属性数 |
| ANONYMOUS |NUMBER | Y | 0 | 抽取无效数 |
| TOTAL |NUMBER | Y | 0 | 抽取总数 |
| TOTALTIME |NUMBER | Y | 0 | 抽取总耗时 |
| ERROR |NUMBER | Y | 0 | 如果抽取状态为0即为失败，那么存在抽取异常信息 |
| CREATE_USER | VARCHAR2(30-1000) |  | 'SYS' | 创建人 |
| CREATE_TIME | DATE |  | SYSDATE | 创建时间 |
| UPDATE_TIME | DATE | Y | SYSDATE | 更新时间 |
| SCBZ | NUMBER(2) |  | 0 | 删除标志，0代表未删除，1代表删除 |

注意：根据现场的资源名称修改调整配置表配置信息。

## 轨迹旅馆住宿关系图-配置抽取

### 抽取配置信息
B_DSE_LOADER_CONFIG表,插入配置数据
```sql
insert into B_DSE_LOADER_CONFIG (CATEGORY, ZY_ZWMC, ZY_YWMC, ZY_QWK_VIEW, ZY_YYK_VIEW, ZY_WBZY_VIEW, ZY_KETTLE_MC, ZLCQ_KSSJ, ZLCQ_JSSJ, ZLCQ_ZT, ZLCQ_TYPE, BLZD1, BLZD2, BLZD3, BLZD4, BLZD5, CREATE_USER,SCBZ)
values ('01', '旅馆住宿信息', 'T_BZ_LY_LGZSXX', 'T_BZ_LY_LGZSXX', 'T_BZ_LY_LGZSXX', 'T_BZ_LY_LGZSXX', 'ORACLE_DSE_T_BZ_LY_LGZSXX', to_date('10-10-2017 14:42:59', 'dd-mm-yyyy hh24:mi:ss'), to_date('10-10-2017 14:42:59', 'dd-mm-yyyy hh24:mi:ss'), '1', '01', null, null, null, null, null, 'SYS', 0);
commit;
```
说明：CATEGORY=01代表轨迹关系图类别，ZLCQ_TYPE=01代表估轨迹旅馆住宿关系图的增量抽取类别
注意现场导入数据后，需要在创建视图库中创建的资源名为T_BZ_LY_LGZSXX.

验证插入数据
```sql
select t.* from B_DSE_LOADER_CONFIG t where CATEGORY = 01 and ZLCQ_TYPE = 01;
```

### 现场视图数据视图准备
现场需要根据旅馆住宿信息表，创建视图库后，创建资源名为T_BZ_LY_LGZSXX的视图。

视图要求,必须包含字段

| 字段名  | 类型  |  可为空 | 说明  |
| ------------ | ------------ | ------------ | ------------ |
| SYSTEMID  | VARCHAR2  |   | 主键  |
| ZJHM  | VARCHAR2  |   | 身份证号码  |
| XM  | VARCHAR2  | Y  | 姓名  |
| LGID  | VARCHAR2  |   | 旅馆ID  |
| LGMC  | VARCHAR2  | Y  | 旅馆名称  |
| FJH  | VARCHAR2  |   | 房间号 |
| RZSJ  | NUMBER  |   | 入住时间,必须为毫秒格式 |
| TFSJ  | NUMBER  | Y  | 退房时间,必须为毫秒格式 |
| BZK_GXSJ_DATE  | DATA  |   | 增量时间，DATE类型 |

还有其他需要抽取进图的全部字段。根据现场的资源情况而定，最好是其他全部字段都配置进视图。


### 抽取脚本
模型是-人员旅馆住宿关系图

脚本文件：`/usr/share/dse-graph-loader/scripts/loadJdbc/oracle_model_lgzs.groovy`

脚本内容：
```grovy
//需要修改配置，根据抽取资源进行调整
category = "01" //轨迹分析类型
zlcqType='01' //需要调整,

bzkGxsjField='BZK_GXSJ_DATE'

//person node
personKey='zjhm' //key必须为小写字段名
personFieldes='ZJHM, XM' //需要导入图库人员信息节点属性的相关字段，必须包含Key字段
personLabel='person'

//guiji轨迹节点
guijiKey='lgid' //需要修改, key必须为小写字段名
guijiFieldes='LGID,LGMC' //需要修改，需要导入图库轨迹节点属性的相关字段
guijiLabel='hotel' //需要修改，轨迹节点标签名

//人的轨迹关系
relsKey='systemid' //key必须为小写字段名
//必须包含rels的主键字段和其属性字段，还需要包含startNode和endNode的主键字段
relsFieldes='SYSTEMID,ZJHM,LGID,FJH,RZSJ,TFSJ' //需要修改，需要导入的轨迹关系属性的相关字段，必须包含全部的主键
relsLabel='lgzs' //需要修改，轨迹关系标签名



// #################################################################################################################################
//以下内容为模版配置不需要修改调整，固定程序配置，可了解
...
```

### 旅馆住宿数据
如旅馆住宿信息表

Oracle数据必须字段
```
zjhm：证件号码
xm：姓名
lgid:旅馆ID
fjh：房间号
rzsj：入住时间
tfsj：退房时间
bzk_gxsj：标准库更新时间
bzk_gxsj_date：标准库更新时间（date类型)
```

注意： 根据现场的字段信息修改调整脚本中对应的personInputSql、personHotelInputSql、personHotelInputSql语句中的查询字段名。


### 执行脚本检查并生成Graph模型

```sh
cd /usr/share/dse-graph-loader/scripts/loadJdbc

graphloader oracle_model.groovy -graph test01 -address dse01 -username admin -password hnzx@123 -dryrun true
```
**注： graphloader命令参数详细解**
oracle_model.groovy ： 为脚本导入的脚本文件名称
-graph test01  ：代表导入的图表为test1,这个只是测试图表
-address dse01  ：DSE图数据库地址，这个dse01可以使用hostname，可以是集群的任何一个启动Graph的节点的ip地址和hostname,也可以是直接导入机器localhost
-username admin  ：DSE集群启动认证的用户名（默认的用户）
-passaword hnzx@123   ：DSE集群启动认证的用户密码（默认的默认）
-dryrun true 启动脚本检查测试，不会真正执行结果，只是测试脚本运行是否正常。

运行情况： 如果出现 DSE Loader Succes,那么证明执行成功
```
root@dse02 ~]# 
[root@dse02 ~]# 
[root@dse02 ~]# cd /usr/share/dse-graph-loader/scripts/loadJdbc
[root@dse02 loadJdbc]# 
[root@dse02 loadJdbc]# 
[root@dse02 loadJdbc]# graphloader oracle_model.groovy -graph test01 -address dse01 -username admin -password hnzx@123 -dryrun true
/usr/java/latest/bin/java -Xms1691M -Xmx1691M -Dio.netty.tmpdir=/tmp -Djava.io.tmpdir=/tmp -cp :/usr/share/dse-g-loader/dse-graph-loader-1.0.1/*:/usr/share/dse-g-loader/dse-graph-loader-1.0.1/lib/* com.datastax.dsegraphloader.cli.Executable oracle_model.groovy -graph test01 -address dse01 -username admin -password hnzx@123 -dryrun true
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration address=dse01
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration username=admin
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration graph=test01
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration password=<redacted>
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration dryrun=true
2018-06-18 17:45:03 INFO  ClockFactory:52 - Using native clock to generate timestamps.
2018-06-18 17:45:03 INFO  NettyUtil:83 - Did not find Netty's native epoll transport in the classpath, defaulting to NIO.
2018-06-18 17:45:04 WARN  ReplicationStrategy$NetworkTopologyStrategy:200 - Error while computing token map for keyspace system_auth with datacenter dc1: could not achieve replication factor 3 (found 2 replicas only), check your keyspace replication settings.
2018-06-18 17:45:04 INFO  DCAwareRoundRobinPolicy:95 - Using data-center name 'dc1' for DCAwareRoundRobinPolicy (if this is incorrect, please provide the correct datacenter name with DCAwareRoundRobinPolicy constructor)
2018-06-18 17:45:04 INFO  Cluster:1543 - New Cassandra host dse01/192.168.19.30:9042 added
2018-06-18 17:45:04 INFO  Cluster:1543 - New Cassandra host /192.168.19.31:9042 added
2018-06-18 17:45:07 INFO  JDBCSource:239 - Update [update B_DSE_LOADER_CONFIG set zlcq_zt = '2' where zlcq_type = '01'] date success, The updated data volume is 1
2018-06-18 17:45:07 INFO  Executable:155 - The groovy script reads to see if schema is created [create_schema:true]
2018-06-18 17:45:07 WARN  Executable:165 - As of DGL 6.0, transformation functions will be dep
...
...
...

2018-06-18 17:45:17 INFO  Executable:304 - DSE Loader Succes, TotalTime : 4 S
```

### 数据建模语句

以下为通过 -dryrun true生成的模型以及索引

```java
//旅馆住宿信息Schema
schema.propertyKey('systemid').Text().single().create()
schema.propertyKey('lgmc').Text().single().create()
schema.propertyKey('lgid').Text().single().create()
schema.propertyKey('xm').Text().single().create()
schema.propertyKey('fjh').Text().single().create()
schema.propertyKey('rzsj').Decimal().single().create()
schema.propertyKey('zjhm').Text().single().create()
schema.propertyKey('tfsj').Decimal().single().create()
schema.vertexLabel('person').properties('zjhm', 'xm').create()
schema.vertexLabel('hotel').properties('lgid', 'lgmc').create()
schema.edgeLabel('lgzs').properties('systemid', 'fjh', 'tfsj', 'rzsj').connection('person', 'hotel').create()

//Vertex Index
schema.vertexLabel('person').index('byXm').materialized().by('xm').add()
schema.vertexLabel('person').index('byZjhm').materialized().by('zjhm').add()
schema.vertexLabel('hotel').index('byLgid').materialized().by('lgid').add()
schema.vertexLabel('hotel').index('byLgmc').materialized().by('lgmc').add()

schema.vertexLabel('person').index('search').search().by('xm').by('zjhm').add()
schema.vertexLabel('hotel').index('search').search().by('lgid').by('lgmc').add()

//Edge Index
schema.vertexLabel('person').index('bySystemid').inE('lgzs').by('systemid').ifNotExists().add()
schema.vertexLabel('hotel').index('bySystemid').inE('lgzs').by('systemid').ifNotExists().add()
schema.vertexLabel('hotel').index('ratedBySystemid').outE('lgzs').by('systemid').ifNotExists().add()
schema.vertexLabel('person').index('ratedBySystemid').outE('lgzs').by('systemid').ifNotExists().add()
schema.vertexLabel('person').index('byFjh').inE('lgzs').by('fjh').ifNotExists().add()
schema.vertexLabel('hotel').index('byFjh').inE('lgzs').by('fjh').ifNotExists().add()
schema.vertexLabel('hotel').index('ratedByFjh').outE('lgzs').by('fjh').ifNotExists().add()
schema.vertexLabel('person').index('ratedByFjh').outE('lgzs').by('fjh').ifNotExists().add()
schema.vertexLabel('person').index('byRzsj').inE('lgzs').by('rzsj').ifNotExists().add()
schema.vertexLabel('hotel').index('byRzsj').inE('lgzs').by('rzsj').ifNotExists().add()
schema.vertexLabel('hotel').index('ratedByRzsj').outE('lgzs').by('rzsj').ifNotExists().add()
schema.vertexLabel('person').index('ratedByRzsj').outE('lgzs').by('rzsj').ifNotExists().add()
schema.vertexLabel('person').index('byTfsj').inE('lgzs').by('tfsj').ifNotExists().add()
schema.vertexLabel('hotel').index('byTfsj').inE('lgzs').by('tfsj').ifNotExists().add()
schema.vertexLabel('hotel').index('ratedByTfsj').outE('lgzs').by('tfsj').ifNotExists().add()
schema.vertexLabel('person').index('ratedByTfsj').outE('lgzs').by('tfsj').ifNotExists().add()
```

通过上面模型创建后，后续不自动创建模型，需要修改base.groovy配置参数
create_schema都修改为false


### 数据全量抽取

#### 修改配置表增量时间

```sql
UPDATE B_DSE_LOADER_CONFIG t
SET t.ZLCQ_KSSJ = to_date('1990-01-01 0:00:00','yyyy-mm-dd hh24:mi:ss'), t.ZLCQ_JSSJ = to_date('1990-01-01 0:00:00','yyyy-mm-dd HH24:MI:SS') WHERE CATEGORY = 01 and ZLCQ_TYPE = 01;
commit;
````

更新后查询验证
```sql
select t.ZLCQ_KSSJ, t.ZLCQ_JSSJ from B_DSE_LOADER_CONFIG t where CATEGORY = 01 and ZLCQ_TYPE = 01;
```

#### 执行抽取命令
```sh
cd /usr/share/dse-graph-loader/scripts/loadJdbc

graphloader oracle_model.groovy -graph test01 -address dse01 -username admin -password hnzx@123
```

相对于执行脚本检查命令只是少了个参数-dryrun true

**运行情况 **：
如果出现 DSE Loader Succes,那么证明执行成功,执行过程出现下面导入多少条记录信息
Current total additions: 7 vertices 6 edges 12 properties 0 anonymous
表示导入了7个顶点，6条边和12条属性记录。

```
[root@dse02 loadJdbc]# graphloader oracle_model.groovy -graph test01 -address dse01 -username admin -password hnzx@123
/usr/java/latest/bin/java -Xms1691M -Xmx1691M -Dio.netty.tmpdir=/tmp -Djava.io.tmpdir=/tmp -cp :/usr/share/dse-g-loader/dse-graph-loader-1.0.1/*:/usr/share/dse-g-loader/dse-graph-loader-1.0.1/lib/* com.datastax.dsegraphloader.cli.Executable oracle_model.groovy -graph test01 -address dse01 -username admin -password hnzx@123 -dryrun true
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration address=dse01
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration username=admin
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration graph=test01
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration password=<redacted>
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration dryrun=true
2018-06-18 17:45:03 INFO  ClockFactory:52 - Using native clock to generate timestamps.
2018-06-18 17:45:03 INFO  NettyUtil:83 - Did not find Netty's native epoll transport in the classpath, defaulting to NIO.
2018-06-18 17:45:04 WARN  ReplicationStrategy$NetworkTopologyStrategy:200 - Error while computing token map for keyspace system_auth with datacenter dc1: could not achieve replication factor 3 (found 2 replicas only), check your keyspace replication settings.
2018-06-18 17:45:04 INFO  DCAwareRoundRobinPolicy:95 - Using data-center name 'dc1' for DCAwareRoundRobinPolicy (if this is incorrect, please provide the correct datacenter name with DCAwareRoundRobinPolicy constructor)
2018-06-18 17:45:04 INFO  Cluster:1543 - New Cassandra host dse01/192.168.19.30:9042 added
2018-06-18 17:45:04 INFO  Cluster:1543 - New Cassandra host /192.168.19.31:9042 added

....
...
...
2018-06-18 17:47:43 INFO  Reporter:101 - Current total additions: 7 vertices 6 edges 12 properties 0 anonymous
2018-06-18 17:47:43 INFO  Reporter:104 - 25 total elements written
...
...
...
2018-06-18 17:47:45 INFO  Executable:304 - DSE Loader Succes, TotalTime : 11 S
```

### 数据增量抽取

#### 创建执行调度shell脚本

```sh
cd /usr/share/dse-graph-loader
mkdir graphloader_lgzs.sh
echo 'cd /usr/share/dse-graph-loader/scripts/loadJdbc' >> graphloader_lgzs.sh
echo '/usr/share/dse-graph-loader/graphloader oracle_model.groovy -graph test01 -address dse01 -username admin -password hnzx@123' >> graphloader_lgzs.sh
```

#### 定时调度

crontab添加前检查
```sh
sudo -uroot /usr/share/dse-graph-loader/1.0.1/graphloader_lgzs.sh
```

通过Linux的定时器crontab进行定时调度配置, 相关命令如下：

```
crontab -e
```
编辑添加定时器，每5分钟执行一次，或者每1小时执行一次

每5分钟执行一次
```sh
*/5 * * * * /usr/share/dse-graph-loader/graphloader_lgzs.sh > /var/log/graphloader/lgzs/graphloader_lgzs.log  2>&1 &
```

每1小时执行一次
```sh
 0 */1 * * *  /usr/share/dse-graph-loader/graphloader_lgzs.sh > /var/log/graphloader/lgzs/graphloader_lgzs.log  2>&1 &
```

设置日志修改 每周星期六晚上11点重新命名日志文件
```sh
0 23 * * 6 /usr/share/dse-graph-loader/renamefile.sh lgzs graphloader_lgzs.log >> /var/log/graphloader/lgzs/graphloader_lgzs_renamefile.log  2>&1 &
```

#### 抽取调度执行结果检查

可以检查/opt/graphloader_lgzs.log打印信息，也可以通过配置抽取log表检查
```sql
select t.*, t.rowid from B_DSE_LOADER_LOG t order by create_time desc;
```

如下图，比如我们设置每间隔1分钟执行一次，图下图日志情况

![](https://v4liulv.github.io/assets/image/1529338757622_48.png)


## 轨迹航班关系图-配置抽取

#### 抽取配置信息
B_DSE_LOADER_CONFIG表,插入配置数据
```sql
insert into B_DSE_LOADER_CONFIG (SYSTEMID, CATEGORY, ZY_ZWMC, ZY_YWMC, ZY_QWK_VIEW, ZY_YYK_VIEW, ZY_WBZY_VIEW, ZY_KETTLE_MC, ZLCQ_KSSJ, ZLCQ_JSSJ, ZLCQ_ZT, ZLCQ_TYPE, BLZD1, BLZD2, BLZD3, BLZD4, BLZD5, CREATE_USER, CREATE_TIME, UPDATE_TIME, SCBZ)
values ('A685BE72D5E84572A374CD80DECE1F3D', '01', '民航出入港信息', 'T_BZ_MH_CRGXX', 'T_BZ_MH_CRGXX', 'T_BZ_MH_CRGXX', 'T_BZ_MH_CRGXX', 'ORACLE_DSE_T_BZ_MH_CRGXX', to_date('10-10-2017 14:42:59', 'dd-mm-yyyy hh24:mi:ss'), to_date('10-10-2017 14:42:59', 'dd-mm-yyyy hh24:mi:ss'), '1', '02', null, null, null, null, null, 'SYS', to_date('26-05-2018 22:56:54', 'dd-mm-yyyy hh24:mi:ss'), to_date('26-05-2018 22:56:54', 'dd-mm-yyyy hh24:mi:ss'), 0);
commit;
```
说明：CATEGORY=01代表轨迹关系图类别，ZLCQ_TYPE=02代表乘坐航班关系图的增量抽取类别
注意现场导入数据后，需要在创建视图库中创建的资源名为T_BZ_MH_CRGXX.

验证插入数据
```sql
select t.* from B_DSE_LOADER_CONFIG t where CATEGORY = 01 and ZLCQ_TYPE = 02;
```

### 现场视图数据视图准备
现场需要根据旅馆住宿信息表，创建视图库后，创建资源名为T_BZ_LY_LGZSXX的视图。

视图要求,必须包含字段

| 字段名  | 类型  |  可为空 | 说明  |
| ------------ | ------------ | ------------ | ------------ |
| SYSTEMID  | VARCHAR2  |   | 主键  |
| ZJLX  | VARCHAR2  |   | 证件类型  |
| ZJHM  | VARCHAR2  |   | 证件号码  |
| CHN_NM  | VARCHAR2  | Y  | 中文名  |
| FST_NM  | VARCHAR2  | Y  | 英文名  |
| HBH  | VARCHAR2  |   | 航班号  |
| QFZ  | VARCHAR2  | Y  | 起飞站  |
| DDZ  | VARCHAR2  |   | 到达站 |
| HBRQ  | VARCHAR2  |   | 航班日期，格式为年月日如\:20160714 |
| QFSJ  | NUMBER  |   | 起飞时间,必须为毫秒格式进行计算 |
| DDSJ  | NUMBER  | Y  | 退房时间,必须为毫秒格式 |
| BZK_GXSJ_DATE  | DATA  |   | 增量时间，DATE类型 |

还有其他需要抽取进图的全部字段。根据现场的资源情况而定，最好是其他全部字段都配置进视图。


### 抽取脚本
模型是-人员旅馆住宿关系图

脚本文件：`/usr/share/dse-graph-loader/scripts/loadJdbc/oracle_model_czhb.groovy`

脚本内容：
```grovy
//需要修改配置，根据抽取资源进行调整
category = "01" //轨迹分析类型
zlcqType='02' //名航出入港"乘坐航班"关系图，需要调整

bzkGxsjField='BZK_GXSJ_DATE'

//person node
personKey='zjhm' //key必须为小写字段名
personFieldes='ZJHM, ZJLX, CHN_NM, FST_NM' //需要修改，需要导入图库人员信息节点属性的相关字段，必须包含Key字段
personLabel='person'

//guiji轨迹节点
guijiKey='hbh' //需要修改, key必须为小写字段名
guijiFieldes='HBH, QFZ, DDZ' //需要修改，需要导入图库轨迹节点属性的相关字段
guijiLabel='flight' //需要修改，轨迹节点标签名

//人的轨迹关系
relsKey='systemid' //key必须为小写字段名
//必须包含rels的主键字段和其属性字段，还需要包含startNode和endNode的主键字段
relsFieldes='SYSTEMID, ZJHM, HBH, QFZ, DDZ, HBRQ, QFSJ, DDSJ' //需要修改，需要导入的轨迹关系属性的相关字段，必须包含全部的主键
relsLabel='czhb' //需要修改，轨迹关系标签名

// #################################################################################################################################
//以下内容为模版配置不需要修改调整，固定程序配置，可了解
...
```

**航班信息数据**
名航出入港信息表

Oracle数据字段必须包含
```
ZJLX ： 证件类型
ZJHM ： 证件号码
CHN_NM ： 中文名
FST_NM ： 英文名
HBH ： 航班号
QFZ ： 起飞站
DDZ ： 到达站
HBRQ ： 航班日期
QFSJ ： 起飞时间
DDSJ ： 到达时间
bzk_gxsj_date：标准库更新时间（date类型)
```

注意： 根据现场的字段信息修改调整脚本中对应的personInputSql、personHotelInputSql、personHotelInputSql语句中的查询字段名。


### 执行脚本检查并生成Graph模型

```sh
cd /usr/share/dse-graph-loader/scripts/loadJdbc

graphloader oracle_model_czhb.groovy -graph test01 -address dse01 -username admin -password hnzx@123 -dryrun true
```
**注： graphloader命令参数详细解**
oracle_model.groovy ： 为脚本导入的脚本文件名称
-graph test01  ：代表导入的图表为test1,这个只是测试图表
-address dse01  ：DSE图数据库地址，这个dse01可以使用hostname，可以是集群的任何一个启动Graph的节点的ip地址和hostname,也可以是直接导入机器localhost
-username admin  ：DSE集群启动认证的用户名（默认的用户）
-passaword hnzx@123   ：DSE集群启动认证的用户密码（默认的默认）
-dryrun true 启动脚本检查测试，不会真正执行结果，只是测试脚本运行是否正常。

运行情况： 如果出现 DSE Loader Succes,那么证明执行成功
```
root@dse02 ~]# 
[root@dse02 ~]# 
[root@dse02 ~]# cd /usr/share/dse-graph-loader/scripts/loadJdbc
[root@dse02 loadJdbc]# 
[root@dse02 loadJdbc]# 
[root@dse02 loadJdbc]# graphloader oracle_model.groovy -graph test01 -address dse01 -username admin -password hnzx@123 -dryrun true
/usr/java/latest/bin/java -Xms1691M -Xmx1691M -Dio.netty.tmpdir=/tmp -Djava.io.tmpdir=/tmp -cp :/usr/share/dse-g-loader/dse-graph-loader-1.0.1/*:/usr/share/dse-g-loader/dse-graph-loader-1.0.1/lib/* com.datastax.dsegraphloader.cli.Executable oracle_model.groovy -graph test01 -address dse01 -username admin -password hnzx@123 -dryrun true
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration address=dse01
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration username=admin
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration graph=test01
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration password=<redacted>
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration dryrun=true
2018-06-18 17:45:03 INFO  ClockFactory:52 - Using native clock to generate timestamps.
2018-06-18 17:45:03 INFO  NettyUtil:83 - Did not find Netty's native epoll transport in the classpath, defaulting to NIO.
2018-06-18 17:45:04 WARN  ReplicationStrategy$NetworkTopologyStrategy:200 - Error while computing token map for keyspace system_auth with datacenter dc1: could not achieve replication factor 3 (found 2 replicas only), check your keyspace replication settings.
2018-06-18 17:45:04 INFO  DCAwareRoundRobinPolicy:95 - Using data-center name 'dc1' for DCAwareRoundRobinPolicy (if this is incorrect, please provide the correct datacenter name with DCAwareRoundRobinPolicy constructor)
2018-06-18 17:45:04 INFO  Cluster:1543 - New Cassandra host dse01/192.168.19.30:9042 added
2018-06-18 17:45:04 INFO  Cluster:1543 - New Cassandra host /192.168.19.31:9042 added
2018-06-18 17:45:07 INFO  JDBCSource:239 - Update [update B_DSE_LOADER_CONFIG set zlcq_zt = '2' where zlcq_type = '01'] date success, The updated data volume is 1
2018-06-18 17:45:07 INFO  Executable:155 - The groovy script reads to see if schema is created [create_schema:true]
2018-06-18 17:45:07 WARN  Executable:165 - As of DGL 6.0, transformation functions will be dep
...
...
...

2018-06-18 17:45:17 INFO  Executable:304 - DSE Loader Succes, TotalTime : 4 S
```


### 数据全量抽取

#### 修改配置表增量时间

```sql
UPDATE B_DSE_LOADER_CONFIG t
SET t.ZLCQ_KSSJ = to_date('1990-01-01 0:00:00','yyyy-mm-dd hh24:mi:ss'), t.ZLCQ_JSSJ = to_date('1990-01-01 0:00:00','yyyy-mm-dd HH24:MI:SS') WHERE CATEGORY = 01 and ZLCQ_TYPE = 02;
commit;
````

更新后查询验证
```sql
select t.ZLCQ_KSSJ, t.ZLCQ_JSSJ from B_DSE_LOADER_CONFIG t where CATEGORY = 01 and ZLCQ_TYPE = 02;
```

#### 执行抽取命令
```sh
cd /usr/share/dse-graph-loader/scripts/loadJdbc

graphloader oracle_model_czhb.groovy -graph test01 -address dse01 -username admin -password hnzx@123
```

相对于执行脚本检查命令只是少了个参数-dryrun true

**运行情况 **：
如果出现 DSE Loader Succes,那么证明执行成功,执行过程出现下面导入多少条记录信息
Current total additions: 7 vertices 6 edges 12 properties 0 anonymous
表示导入了7个顶点，6条边和12条属性记录。

```
[root@dse02 loadJdbc]# graphloader oracle_model.groovy -graph test01 -address dse01 -username admin -password hnzx@123
/usr/java/latest/bin/java -Xms1691M -Xmx1691M -Dio.netty.tmpdir=/tmp -Djava.io.tmpdir=/tmp -cp :/usr/share/dse-g-loader/dse-graph-loader-1.0.1/*:/usr/share/dse-g-loader/dse-graph-loader-1.0.1/lib/* com.datastax.dsegraphloader.cli.Executable oracle_model.groovy -graph test01 -address dse01 -username admin -password hnzx@123 -dryrun true
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration address=dse01
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration username=admin
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration graph=test01
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration password=<redacted>
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration dryrun=true
2018-06-18 17:45:03 INFO  ClockFactory:52 - Using native clock to generate timestamps.
2018-06-18 17:45:03 INFO  NettyUtil:83 - Did not find Netty's native epoll transport in the classpath, defaulting to NIO.
2018-06-18 17:45:04 WARN  ReplicationStrategy$NetworkTopologyStrategy:200 - Error while computing token map for keyspace system_auth with datacenter dc1: could not achieve replication factor 3 (found 2 replicas only), check your keyspace replication settings.
2018-06-18 17:45:04 INFO  DCAwareRoundRobinPolicy:95 - Using data-center name 'dc1' for DCAwareRoundRobinPolicy (if this is incorrect, please provide the correct datacenter name with DCAwareRoundRobinPolicy constructor)
2018-06-18 17:45:04 INFO  Cluster:1543 - New Cassandra host dse01/192.168.19.30:9042 added
2018-06-18 17:45:04 INFO  Cluster:1543 - New Cassandra host /192.168.19.31:9042 added

....
...
...
2018-06-18 17:47:43 INFO  Reporter:101 - Current total additions: 7 vertices 6 edges 12 properties 0 anonymous
2018-06-18 17:47:43 INFO  Reporter:104 - 25 total elements written
...
...
...
2018-06-18 17:47:45 INFO  Executable:304 - DSE Loader Succes, TotalTime : 11 S
```

### 数据增量抽取

#### 创建执行调度shell脚本

```sh
sudo cd /usr/share/dse-graph-loader
sudo mkdir graphloader_czhb.sh
sudo echo 'cd /usr/share/dse-graph-loader/scripts/loadJdbc' >> graphloader_czhb.sh
sudo echo '/usr/share/dse-graph-loader/graphloader oracle_model_czhb.groovy -graph test01 -address dse01 -username admin -password hnzx@123' >> graphloader_czhb.sh
```

#### 定时调度

创建定时调度前先创建日志文件目录
```sh
sudo mkdir -p /var/log/graphloader/czhb
```
注意：其中的czh为每个关系图标签名称

crontab添加前检查
```sh
sudo -uroot /usr/share/dse-graph-loader/graphloader_czhb.sh
```

通过Linux的定时器crontab进行定时调度配置, 相关命令如下：

```sh
crontab -e
```
编辑添加定时器，每5分钟执行一次，或者每1小时执行一次

每10分钟执行一次
```sh
*/10 * * * * /usr/share/dse-graph-loader/graphloader_czhb.sh > /var/log/graphloader/czhb/graphloader_czhb.log  2>&1 &
```

每1小时执行一次
```sh
 0 */1 * * *  /usr/share/dse-graph-loader/graphloader_czhb.sh > /var/log/graphloader/czhb/graphloader_czhb.log  2>&1 &
```

设置日志修改 每周星期六晚上11点重新命名日志文件
```sh
0 23 * * 6 /usr/share/dse-graph-loader/renamefile.sh czhb graphloader_czhb.log >> /var/log/graphloader/czhb/graphloader_czhb_renamefile.log  2>&1 &
```

#### 抽取调度执行结果检查

可以检查/opt/graphloader_lgzs.log打印信息，也可以通过配置抽取log表检查
```sql
select t.*, t.rowid from B_DSE_LOADER_LOG t where t.category = 01 and zlcq_type = 02 order by create_time desc;
```

如下图，比如我们设置每间隔10分钟执行一次，图下图日志情况

![](http://localhost:8081/assets/msg/upload/1529338757622_48.png)


## 轨迹火车关系图-配置抽取

### 抽取配置信息
B_DSE_LOADER_CONFIG表,插入配置数据
```sql
insert into B_DSE_LOADER_CONFIG (SYSTEMID, CATEGORY, ZY_ZWMC, ZY_YWMC, ZY_QWK_VIEW, ZY_YYK_VIEW, ZY_WBZY_VIEW, ZY_KETTLE_MC, ZLCQ_KSSJ, ZLCQ_JSSJ, ZLCQ_ZT, ZLCQ_TYPE, BLZD1, BLZD2, BLZD3, BLZD4, BLZD5, CREATE_USER, CREATE_TIME, UPDATE_TIME, SCBZ)
values ('533FA13A99F74ACDA805C5C2E0FC1E9D', '01', '乘坐火车信息', 'T_BZ_HCSK_GPXX', 'T_BZ_HCSK_GPXX', 'T_BZ_HCSK_GPXX', 'T_BZ_HCSK_GPXX', 'ORACLE_DSE_T_BZ_HCSK_GPXX', to_date('10-10-2017 14:42:59', 'dd-mm-yyyy hh24:mi:ss'), to_date('10-10-2010 14:42:59', 'dd-mm-yyyy hh24:mi:ss'), '1', '03', null, null, null, null, null, 'SYS', to_date('26-05-2018 22:56:54', 'dd-mm-yyyy hh24:mi:ss'), to_date('26-05-2010 22:56:54', 'dd-mm-yyyy hh24:mi:ss'), 0);
commit;
```
说明：CATEGORY=01代表轨迹关系图类别，ZLCQ_TYPE=03代表乘坐航班关系图的增量抽取类别
注意现场导入数据后，需要在创建视图库中创建的资源名为T_BZ_HCSK_GPXX

验证插入数据
```sql
select t.* from B_DSE_LOADER_CONFIG t where CATEGORY = 01 and ZLCQ_TYPE = 03;
```

如下：

![](https://v4liulv.github.io/assets/image/1530673721770_18.png)


### 现场视图数据视图准备
现场需要根据旅馆住宿信息表，创建视图库后，创建资源名为T_BZ_LY_LGZSXX的视图。

视图要求,必须包含字段

| 字段名  | 类型  |  可为空 | 说明  |
| ------------ | ------------ | ------------ | ------------ |
| SYSTEMID  | VARCHAR2  |   | 主键  |
| ZJLX  | VARCHAR2  |   | 证件类型  |
| ZJHM  | VARCHAR2  |   | 证件号码  |
| CHN_NM  | VARCHAR2  | Y  | 中文名  |
| FST_NM  | VARCHAR2  | Y  | 英文名  |
| CC  | VARCHAR2  |   | 车次  |
| SFZ  | VARCHAR2  | Y  | 始发站  |
| ZDZ  | VARCHAR2  |   | 终点站 |
| FCRQ  | VARCHAR2  |   | 发车日期，格式为年月日如\:20160714 |
| FCSJ  | NUMBER  |   | 发车时间,必须为毫秒格式进行计算 |
| DDSJ  | NUMBER  | Y  | 到达时间,必须为毫秒格式 |
| BZK_GXSJ_DATE  | DATA  |   | 增量时间，DATE类型 |

还有其他需要抽取进图的全部字段。根据现场的资源情况而定，最好是其他全部字段都配置进视图。


### 抽取脚本
模型是-人员旅馆住宿关系图

脚本文件：`/usr/share/dse-graph-loader/scripts/loadJdbc/oracle_model_czhc.groovy`

脚本内容：
```grovy
//需要修改配置，根据抽取资源进行调整
category = "01" //轨迹分析类型
zlcqType='03' //名航出入港"乘坐火车"关系图，需要调整

bzkGxsjField='BZK_GXSJ_DATE'

//person node
personKey='zjhm' //key必须为小写字段名
personFieldes='ZJHM, ZJLX, CHN_NM, FST_NM' //需要修改，需要导入图库人员信息节点属性的相关字段，必须包含Key字段
personLabel='person'

//guiji轨迹节点
guijiKey='hbh' //需要修改, key必须为小写字段名
guijiFieldes='CC, SFZ, ZDZ' //需要修改，需要导入图库轨迹节点属性的相关字段
guijiLabel='flight' //需要修改，轨迹节点标签名

//人的轨迹关系
relsKey='systemid' //key必须为小写字段名
//必须包含rels的主键字段和其属性字段，还需要包含startNode和endNode的主键字段
relsFieldes='SYSTEMID, ZJHM, CC, SFZ, ZDZ, FCRQ, FFSJ, DDSJ' //需要修改，需要导入的轨迹关系属性的相关字段，必须包含全部的主键
relsLabel='czhb' //需要修改，轨迹关系标签名

// #################################################################################################################################
//以下内容为模版配置不需要修改调整，固定程序配置，可了解
...
```

**航班信息数据**
名航出入港信息表

Oracle数据字段必须包含
```
ZJLX ： 证件类型
ZJHM ： 证件号码
CHN_NM ： 中文名
FST_NM ： 英文名
HBH ： 航班号
QFZ ： 起飞站
DDZ ： 到达站
HBRQ ： 航班日期
QFSJ ： 起飞时间
DDSJ ： 到达时间
bzk_gxsj_date：标准库更新时间（date类型)
```

注意： 根据现场的字段信息修改调整脚本中对应的personInputSql、personHotelInputSql、personHotelInputSql语句中的查询字段名。


### 执行脚本检查并生成Graph模型

```sh
cd /usr/share/dse-graph-loader/scripts/loadJdbc

graphloader oracle_model_czhc.groovy -graph test01 -address dse01 -username admin -password hnzx@123 -dryrun true
```
**注： graphloader命令参数详细解**
oracle_model.groovy ： 为脚本导入的脚本文件名称
-graph test01  ：代表导入的图表为test1,这个只是测试图表
-address dse01  ：DSE图数据库地址，这个dse01可以使用hostname，可以是集群的任何一个启动Graph的节点的ip地址和hostname,也可以是直接导入机器localhost
-username admin  ：DSE集群启动认证的用户名（默认的用户）
-passaword hnzx@123   ：DSE集群启动认证的用户密码（默认的默认）
-dryrun true 启动脚本检查测试，不会真正执行结果，只是测试脚本运行是否正常。

运行情况： 如果出现 DSE Loader Succes,那么证明执行成功
```
root@dse02 ~]# 
[root@dse02 ~]# 
[root@dse02 ~]# cd /usr/share/dse-graph-loader/scripts/loadJdbc
[root@dse02 loadJdbc]# 
[root@dse02 loadJdbc]# 
[root@dse02 loadJdbc]# graphloader oracle_model.groovy -graph test01 -address dse01 -username admin -password hnzx@123 -dryrun true
/usr/java/latest/bin/java -Xms1691M -Xmx1691M -Dio.netty.tmpdir=/tmp -Djava.io.tmpdir=/tmp -cp :/usr/share/dse-g-loader/dse-graph-loader-1.0.1/*:/usr/share/dse-g-loader/dse-graph-loader-1.0.1/lib/* com.datastax.dsegraphloader.cli.Executable oracle_model.groovy -graph test01 -address dse01 -username admin -password hnzx@123 -dryrun true
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration address=dse01
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration username=admin
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration graph=test01
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration password=<redacted>
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration dryrun=true
2018-06-18 17:45:03 INFO  ClockFactory:52 - Using native clock to generate timestamps.
2018-06-18 17:45:03 INFO  NettyUtil:83 - Did not find Netty's native epoll transport in the classpath, defaulting to NIO.
2018-06-18 17:45:04 WARN  ReplicationStrategy$NetworkTopologyStrategy:200 - Error while computing token map for keyspace system_auth with datacenter dc1: could not achieve replication factor 3 (found 2 replicas only), check your keyspace replication settings.
2018-06-18 17:45:04 INFO  DCAwareRoundRobinPolicy:95 - Using data-center name 'dc1' for DCAwareRoundRobinPolicy (if this is incorrect, please provide the correct datacenter name with DCAwareRoundRobinPolicy constructor)
2018-06-18 17:45:04 INFO  Cluster:1543 - New Cassandra host dse01/192.168.19.30:9042 added
2018-06-18 17:45:04 INFO  Cluster:1543 - New Cassandra host /192.168.19.31:9042 added
2018-06-18 17:45:07 INFO  JDBCSource:239 - Update [update B_DSE_LOADER_CONFIG set zlcq_zt = '2' where zlcq_type = '01'] date success, The updated data volume is 1
2018-06-18 17:45:07 INFO  Executable:155 - The groovy script reads to see if schema is created [create_schema:true]
2018-06-18 17:45:07 WARN  Executable:165 - As of DGL 6.0, transformation functions will be dep
...
...
...

2018-06-18 17:45:17 INFO  Executable:304 - DSE Loader Succes, TotalTime : 4 S
```


### 数据全量抽取

#### 修改配置表增量时间

```sql
UPDATE B_DSE_LOADER_CONFIG t
SET t.ZLCQ_KSSJ = to_date('1990-01-01 0:00:00','yyyy-mm-dd hh24:mi:ss'), t.ZLCQ_JSSJ = to_date('1990-01-01 0:00:00','yyyy-mm-dd HH24:MI:SS') WHERE CATEGORY = 01 and ZLCQ_TYPE = 03;
commit;
````

更新后查询验证
```sql
select t.ZLCQ_KSSJ, t.ZLCQ_JSSJ from B_DSE_LOADER_CONFIG t where CATEGORY = 01 and ZLCQ_TYPE = 03;
```

#### 执行抽取命令
```sh
cd /usr/share/dse-graph-loader/scripts/loadJdbc

graphloader oracle_model_czhc.groovy -graph test01 -address dse01 -username admin -password hnzx@123
```

相对于执行脚本检查命令只是少了个参数-dryrun true

**运行情况 **：
如果出现 DSE Loader Succes,那么证明执行成功,执行过程出现下面导入多少条记录信息
Current total additions: 7 vertices 6 edges 12 properties 0 anonymous
表示导入了7个顶点，6条边和12条属性记录。

```
[root@dse02 loadJdbc]# graphloader oracle_model_czhc.groovy -graph test01 -address dse01 -username admin -password hnzx@123
/usr/java/latest/bin/java -Xms1691M -Xmx1691M -Dio.netty.tmpdir=/tmp -Djava.io.tmpdir=/tmp -cp :/usr/share/dse-g-loader/dse-graph-loader-1.0.1/*:/usr/share/dse-g-loader/dse-graph-loader-1.0.1/lib/* com.datastax.dsegraphloader.cli.Executable oracle_model.groovy -graph test01 -address dse01 -username admin -password hnzx@123 -dryrun true
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration address=dse01
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration username=admin
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration graph=test01
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration password=<redacted>
2018-06-18 17:45:02 INFO  Executable:119 - Setting configuration dryrun=true
2018-06-18 17:45:03 INFO  ClockFactory:52 - Using native clock to generate timestamps.
2018-06-18 17:45:03 INFO  NettyUtil:83 - Did not find Netty's native epoll transport in the classpath, defaulting to NIO.
2018-06-18 17:45:04 WARN  ReplicationStrategy$NetworkTopologyStrategy:200 - Error while computing token map for keyspace system_auth with datacenter dc1: could not achieve replication factor 3 (found 2 replicas only), check your keyspace replication settings.
2018-06-18 17:45:04 INFO  DCAwareRoundRobinPolicy:95 - Using data-center name 'dc1' for DCAwareRoundRobinPolicy (if this is incorrect, please provide the correct datacenter name with DCAwareRoundRobinPolicy constructor)
2018-06-18 17:45:04 INFO  Cluster:1543 - New Cassandra host dse01/192.168.19.30:9042 added
2018-06-18 17:45:04 INFO  Cluster:1543 - New Cassandra host /192.168.19.31:9042 added

....
...
...
2018-06-18 17:47:43 INFO  Reporter:101 - Current total additions: 7 vertices 6 edges 12 properties 0 anonymous
2018-06-18 17:47:43 INFO  Reporter:104 - 25 total elements written
...
...
...
2018-06-18 17:47:45 INFO  Executable:304 - DSE Loader Succes, TotalTime : 11 S
```

### 数据增量抽取

#### 创建执行调度shell脚本

创建crontab执行命令文件graphloader_czhc.sh
```sh
cd /usr/share/dse-graph-loader/
mkdir graphloader_czhc.sh
echo 'cd /usr/share/dse-graph-loader/scripts/loadJdbc' >> graphloader_czhc.sh
echo '/usr/share/dse-graph-loader/graphloader oracle_model_czhc.groovy -graph test01 -address dse01 -username admin -password hnzx@123' >> graphloader_czhc.sh
```

#### 定时调度

crontab添加前检查
```sh
sudo -uroot /usr/share/dse-graph-loader/graphloader_czhc.sh
```

通过Linux的定时器crontab进行定时调度配置, 相关命令如下：

```sh
crontab -e
```
编辑添加定时器，每5分钟执行一次，或者每1小时执行一次

每5分钟执行一次
```sh
*/5 * * * * /usr/share/dse-graph-loader/graphloader_czhc.sh > /var/log/graphloader/czhb/graphloader_czhc.log  2>&1 &
```

每1小时执行一次
```sh
 0 */1 * * *  /usr/share/dse-graph-loader/graphloader_czhc.sh > /var/log/graphloader/czhb/graphloader_czhc.log  2>&1 &
```

设置日志修改 每周星期六晚上11点重新命名日志文件
```sh
0 23 * * 6 /usr/share/dse-graph-loader/renamefile.sh czhc graphloader_czhc.log >> /var/log/graphloader/czhc/graphloader_czhb_renamefile.log  2>&1 &
```

#### 抽取调度执行结果检查

可以检查/opt/graphloader_lgzs.log打印信息，也可以通过配置抽取log表检查
```sql
select t.*, t.rowid from B_DSE_LOADER_LOG t where t.category = 01 and zlcq_type = 03 order by create_time desc;
```

如下图，比如我们设置每间隔1分钟执行一次，图下图日志情况

![](https://v4liulv.github.io/assets/image/1529338757622_48.png)





END!
