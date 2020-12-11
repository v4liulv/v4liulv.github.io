---
title: "腾讯云EMR-Alluxio"
subtitle: ""
layout: post
author: "LiuL"
header-style: text
tags:
  - EMR
  - Alluxio
---

# 1. Alluxio 简介

读音 /aluxshel/ 

## 1.1. Alluxio 是什么

Alluxio是一个基于内存的分布式文件系统，它是架构在底层分布式文件系统和上层分布式计算框架之间的一个中间件，主要职责是以文件形式在内存或其
储设施中提供数据的存取服务。

Alluxio的前身为Tachyon。

## 1.2. Alluxio 应用场景

 Alluxio居于传统大数据存储（如：Amazon S3，Apache HDFS和OpenStack Swift等）和大数据计算框架（如Spark，Hadoop Mapreduce）之间，如下图所示：

 ![20201127162820](https://liulv.work/images/img/20201127162820.png)

  在大数据领域，最底层的是分布式文件系统，如Amazon S3、Apache HDFS等，而较高层的应用则是一些分布式计算框架，如Spark、MapReduce、HBase、Flink等，这些分布式框架，往往都是直接从分布式文件系统中读写数据，效率比较低，性能消耗比较大。而如果我们将其架构与底层分布式文件系统与上层分布式计算框架之间，以文件的形式在内存中对外提供读写访问服务的话，那么Alluxio可以为那些大数据应用提供一个数量级的加速，而且它只要提供通用的数据访问接口，就能很方便的切换底层分布式文件系统。

### 1.2.1. Alluxio系统架构

与其他诸如HDFS、HBase、Spark等大数据相关框架一致，Alluxio也是一个主从结构的系统。它的主节点为Master，负责管理全局的文件系统元数据，比如文件系统树等，而从节点为Worker，负责管理本节点数据存储服务。另外，Alluxio还有一个组件为Client，为用户提供统一的文件存取服务接口。

当应用程序需要访问Alluxio时，通过客户端先与主节点Master通讯，或许对应文件的元数据，然后再和对应Worker节点通讯，进行实际的文件存取操作。所有的Worker会周期性地发送心跳给Master，维护文件系统元数据信息和确保自己被Master感知扔在集群中正常提供服务，而Master不会主动发起与其他组件的通信，它只是以回复请求的方式与其他组件进行通信。这与HDFS、HBase等分布式系统设计模式是一致的。

# 2. Alluxio 开发

## 2.1. 背景

Alluxio 通过文件系统接口提供对数据的访问。Alluxio 中的文件提供一次写入语义：它们在被完整写下之后不可变，在完成之前不可读。Alluxio 提供了两种不同的文件系统 API：Alluxio API 和 Hadoop 兼容的 API。Alluxio API 提供了额外的功能，而 Hadoop 兼容的 API 为用户提供了无需修改现有代码使用 Hadoop API 的灵活性。

所有使用 Alluxio Java API 的资源都是通过 AlluxioURI 指定的路径实现。

## 2.2. 获取文件系统客户端

要使用 Java 代码获取 Alluxio 文件系统客户端，请使用：

```java
FileSystem fs = FileSystem.Factory.get();
```

## 2.3. 创建一个文件

所有的元数据操作，以及打开一个文件读取或创建一个文件写入都通过 FileSystem 对象执行。由于 Alluxio 文件一旦写入就不可改变，创建文件的惯用方法是使用FileSystem#createFile(AlluxioURI)，它返回一个可用于写入文件的流对象。例如：

```java
FileSystem fs = FileSystem.Factory.get();
AlluxioURI path = new AlluxioURI("/myFile");
// Create a file and get its output stream
FileOutStream out = fs.createFile(path);
// Write data
out.write(...);
// Close and complete file
out.close();
```

## 2.4. 指定操作选项

对于所有的文件系统操作，可以指定一个额外的 options 字段，它允许用户可以指定操作的非默认设置。例如：

```java
FileSystem fs = FileSystem.Factory.get();
AlluxioURI path = new AlluxioURI("/myFile");
// Generate options to set a custom blocksize of 128 MB
CreateFileOptions options = CreateFileOptions.defaults().setBlockSize(128 * Constants.MB);
FileOutStream out = fs.createFile(path, options);
```

## 2.5. IO 选项

Alluxio 使用两种不同的存储类型：Alluxio 管理存储和底层存储。Alluxio 管理存储是分配给 Alluxio worker 的内存 SSD 或 HDD。底层存储是由在最下层的存储系统（如 S3、Swift 或 HDFS）管理的资源。用户可以指定通过 ReadType 和 WriteType 与 Alluxio 管理的存储交互。ReadType 指定读取文件时的数据读取行为。WriteType 指定数据编写新文件时写入行为，例如，数据是否应该写入 Alluxio Storage。

下面是 ReadType 的预期行为表。读取总是偏好 Alluxio 存储优先于底层存储。


|Read Type  |Behavior  |
|---------|---------|
|CACHE_PROMOTE     | 如果读取的数据在 Worker 上时，该数据被移动到 Worker 的最高层。如果该数据不在本地 Worker 的 Alluxio 存储中，那么就将一个副本添加到本地 Alluxio Worker 中。如果alluxio.user.file.cache.partially.read.block设置为 true，没有完全读取的数据块也会被全部存到 Alluxio 内。相反，一个数据块只有完全被读取时，才能被缓存。        |
|CACHE     | 如果该数据不在本地 Worker 的 Alluxio 存储中，那么就将一个副本添加到本地 Alluxio Worker 中。
如果alluxio.user.file.cache.partially.read.block设置为 true，没有完全读取的数据块也会被全部存到 Alluxio 内。相反，一个数据块只有完全被读取时，才能被缓存。        |
|NO_CACHE     | 仅读取数据，不在 Alluxio 中存储副本。        |

下面是 WriteType 的预期行为表。


|Write Type  |Behavior  |
|---------|---------|
|CACHE_THROUGH     |数据被同步地写入到 Alluxio 的 Worker 和底层存储系统。         |
|MUST_CACHE     | 数据被同步地写入到 Alluxio 的 Worker。但不会被写入到底层存储系统。这是默认写类型。        |
|THROUGH     |  数据被同步地写入到底层存储系统。但不会被写入到 Alluxio 的 Worker。       |
|ASYNC_THROUGH     |  数据被同步地写入到 Alluxio 的 Worker，并异步地写入到底层存储系统。处于实验阶段。      |


## 位置策略

Alluxio 提供了位置策略来选择要存储文件块到哪一个 worker。

使用 Alluxio 的 Java API，用户可以设置策略在 CreateFileOptions 中向 Alluxio 写入文件和在 OpenFileOptions 中读取文件。

用户可以轻松地覆盖默认的策略类配置文件中的属性alluxio.user.file.write.location.policy.class。内置的策略包括：

- LocalFirstPolicy (alluxio.client.file.policy.LocalFirstPolicy)
首先返回本地节点，如果本地 worker 没有足够的块容量，它从活动 worker 列表中随机选择一名 worker。这是默认的策略。
- MostAvailableFirstPolicy (alluxio.client.file.policy.MostAvailableFirstPolicy)
返回具有最多可用字节的 worker。
- RoundRobinPolicy (alluxio.client.file.policy.RoundRobinPolicy)
以循环方式选择下一个 worker，跳过没有足够容量的 worker。
- SpecificHostPolicy (alluxio.client.file.policy.SpecificHostPolicy)
返回具有指定节点名的 worker。此策略不能设置为默认策略。


Alluxio 支持自定义策略，所以您也可以通过实现接口alluxio.client.file.policyFileWriteLocationPolicy制定适合自己的策略。

> **注意：**
默认策略必须有一个空的构造函数。并使用 ASYNC_THROUGH 写入类型，所有块的文件必须写入同一个 worker。

Alluxio 允许客户在向本地 worker 写入数据块时选择一个层级偏好。目前这种策略偏好只适用于本地 worker 而不是远程 worker；远程 worker 会写到最高层。

默认情况下，数据被写入顶层。用户可以通过alluxio.user.file.write.tier.default配置项修改默认设置，或通过FileSystem#createFile(AlluxioURI)API调用覆盖它。

对现有文件或目录的所有操作都要求用户指定 AlluxioURI。使用 AlluxioURI，用户可以使用 FileSystem 中的任何方法来访问资源。

AlluxioURI 可用于执行 Alluxio FileSystem 操作，例如修改文件元数据、ttl 或 pin 状态，或者获取输入流来读取文件。例如，要读取一个文件：

```java
FileSystem fs = FileSystem.Factory.get();
AlluxioURI path = new AlluxioURI("/myFile");
// Open the file for reading
FileInStream in = fs.openFile(path);
// Read data
in.read(...);
// Close file relinquishing the lock
in.close();
```





