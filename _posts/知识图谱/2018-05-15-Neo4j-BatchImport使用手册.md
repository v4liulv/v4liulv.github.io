---
title:  2018-05-15-Neo4j-BatchImport使用手册
tags: 图库
---

# 提前准备

## 下载

下载3.3.1版本的[neo4j-batchimport-tool-3.3.1.zip](https://pan.baidu.com/s/1KWEBveZ4OostHOCePQHliQ "neo4j-batchimport-tool-3.3.1")压缩包

注意：压缩码为m084

## 解压

解压上面下载好的压缩包，到`D:\neo4j`文件夹下，会得到类似的路径`D:\neo4j\neo4j-batchimport-tool-3.3.1`，如下图1:

![](http://localhost:8081/images/neo4j-batch-import/batch-import-01.png)

## JDK

服务器必须安装JDK1.8.X以上版本, 本文推荐安装[1.8.0.172]( "dk-8u172-windows-x64.exe")

## Neo4j安装

已经安装Neo4j 3.3.1 版本的图数据库，如果您还没有进行进行安装，可以参考[Neo4j图库操作手册](/Neo4j图库操作手册.pdf)


## Neo4j-CSV数据准备

这里必须按照规范相关的CSV数据，具体详情可查看[Neo4j图库数据入库方案OLD](https://liulv.work/_posts/2018-08-15-Neo4j%E5%9B%BE%E5%BA%93%E6%95%B0%E6%8D%AE%E5%85%A5%E5%BA%93%E6%96%B9%E6%A1%88(OLD).pdf "Neo4j图库数据入库方案(OLD)")和[Neo4j图库数据入库方案KETTLE]( "Neo4j图库数据入库方案KETTLE")

**备注：** 如果只是测试使用，可以使用`D:\neo4j\neo4j-batchimport-tool-3.3.1\cvs`下的数据CSV文件进行测试


# BatchImport 数据导入

BatchImport是Neo4j进行比较大数据量的导入的一种批处理的导入CSV生成Neo4j数据库的一种方式，特别注意这里是生成那么就代表不能在已有的数据库的基础上进行处理，只能在用着初始化数据库的一种方式，不能用于增量处理，只能用于全量第一次初始化Neo4j数据库时候，执行BatchImport导入是通过预先生成CSV节点和关系数据文件，然后通过import命令进行生成相关的数据库的一种大数据量的数据入库创建数据库的方式。生成完成数据库后通过Neo4j切换数据库使用生成后的数据库。

## Import-CSV数据文件结构

import命令的CSV数据文件分为两种：
* 节点CSV文件
* 关系CSV文件

**备注：**CSV文件的分割都使用tab来进行分割

### 节点CSV数据文件结构

文件头：如user.csv文件：`1:label    zjhm:string:zjhm  name:string:name`
> `1:label`定义节点的标签，为固定不可修改
> `zjhm:string:zjhm` 定义节点的属性zjhm, 第一个zjhm代表属性的key，string代表类型，第二个zjhm代表创建索引名，需要在配置文件batch.properties添加相关索引配置，如果属性不需要创建索引则不需要`:zjhm`，如`zjhm:string`即可。
> name:string:name` 定义节点的属性姓名，说明和上面完全一致，属性是可以无限扩展的

例 ：user.csv

| 1:label  | zjhm:string:zjhm | xm:string:xm |
| ------------- | ------------- |
| User      | 520000000000000001 |  张三 |
| User      | 520000000000000001 |  李四 |
| User      | 520000000000000001 |  王二 |


### 关系CSV数据文件结构

文件头：`zjhm:string:zjhm  hotelId:string:hotelId  type:string:type createTime:string`

格式与节点CSV数据文件类似，其中第一行的`zjhm:string:zjhm`代表关系开始节点的指定zjhm.
第二行`hotelId:string:hotelId`代表结束节点（箭头指向节点）的hotelID
第三行`type:string:type`关系的类别也就是关系的标签
第四行代表 属性，属性可以配置多个，在后台添加相关的行即可

例 ：hotel-rels.csv

| zjhm:string:zjhm  | hotelId:string:hotelId | type:string:type | createTime:string |
| ------------- | ------------- | ------------- |
| 520000000000000004 | PCT52000 |  旅馆住宿 | 20170822144242 |
| 520000000000000005 | PCT52000 |  旅馆住宿 | 20170822144242 |

## Import 命令说明

命令格式： `import 参数1 参数2 参数3`


**参数1：** 生成数据库的文件路径，必须保证此路径最终文件夹不存在
比如`D:\neo4j\neo4j-community-3.3.1\data\dbtest.db`, 必须保证`dbtest.db` 不存在。

备注：此路径最好写在Neo4j安装路径的数据库文件夹下，如`..\neo4j-community-3.3.1\data\database\kshfx.db`,`kshfx.db`一定不能跟`database`文件夹下的其他文件夹重名，执行前必须检查下。

**参数2：** 节点CSV文件路径，如`csv\user.csv`
注意：必须保证csv文件存在，并且符合Neo4j 导入的规范，如果导入多个节点CSV文件可以使用','进行分割
如`csv\user.csv,csv\hotel.csv`

**参数3：** 关系CSV文件路径，如：`csv\hotel-rels.csv`，多个关系CSV文件导入可以使用','进行分割，
如：`csv\hotel-rels.csv,csv\train-rels.csv`

注意：关系CSV文件必须与节点的CSV文件对应，并且其中的数据也与节点CSV文件对应

## 索引提前配置

import 命令在导入CSV文件，如果CSV文件添加了相关索引配置如节点CSV文件中`zjhm:string:zjhm`那么代表创建名为zjhm的索引需要在配置`D:\neo4j\neo4j-batchimport-tool-3.3.1\batch.properties`文件中添加相关索引配置：`batch_import.node_index.zjhm=exact`


## 具体步骤：(Windows命令模式)

> 1. 进入cmd命令模式
```sh
cd D:\neo4j\neo4j-batchimport-tool-3.3.1
D:
```
> 2. 执行import命令进行csv数据导入
测试命令如下：
`import ..\neo4j-community-3.3.1\data\databases\dbtest.db csv\kshfx\user.csv,csv\kshfx\hotel.csv,csv\kshfx\train.csv csv\kshfx\hotel-rels.csv,csv\kshfx\train-rels.csv,csv\kshfx\flight.csv`

如果成功会提示创建多少个节点和关系。请看下图导入数据并创建数据库成功

[![](http://localhost:8081/images/neo4j-batch-import/batch-import-02.png)](http://localhost:8081/images/neo4j-batch-import/batch-import-02.png "BatchImport导入成功图")

查看是否生成数据库文件,下图可以看到已经生成kshfx.db数据库文件夹

![](http://localhost:8081/images/neo4j-batch-import/batch-import-03.png)]

## Neo4j数据库切换

* 找到Neo4j安装目录下的`conf/neo4j.conf`进行修改，如安装在`D:\neo4j\neo4j-community-3.3.1`，如下图
* 
![](http://localhost:8081/images/neo4j-batch-import/batch-import-04.png)]


* 编辑修改neo4j.conf内容，dbms.active_database修改为新生成的dbtest.db，如下
`dbms.active_database=kshfx.db`

* cmd命令重启Neo4j
`neo4j restart`

![](http://localhost:8081/images/neo4j-batch-import/batch-import-05.png)]

## 图库连接数据库数据验证

重启完成图库后，通过browser连接图库进行新生成的数据库的数据验证，如下图

![](http://localhost:8081/images/neo4j-batch-import/batch-import-06.png)]

如果出现此结果，那么整个Import导入流程就已经完成，如果您需要生成正式的数据，需要提供相关的CSV数据文件放到csv路径下，通过上面的流程进行简单的修改就可以进行生成正式的数据库！

End !!!
