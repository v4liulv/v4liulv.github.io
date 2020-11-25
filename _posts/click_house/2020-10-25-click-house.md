---
title: "ClickHouse入门教程"
subtitle: ""ClickHouse安装部署启动
layout: post
author: "LiuL"
header-style: text
tags:

  + ClickHouse

---

# 1. 安装ClickHouse

## 1.1. Docker方式安装

要在Docker中运行ClickHouse，请遵循Docker的指南。那些镜像使用官方的deb包。

### 1.1.1. 使用源码安装 

Client: programs/clickhouse-client
Server: programs/clickhouse-server

命令来查看安装可用版本：

``` shell
docker search clickhouse-server
docker search clickhouse-client
```

查询得到可用的镜像有yandex/开头的，那么使用下面命令下载镜像到本地

``` powershell
docker pull yandex/clickhouse-server:latest
docker pull yandex/clickhouse-client:latest
```

下载安装完成后，查看已经安装的镜像

``` powershell
docker images
```

在服务器中为数据创建如下目录：

/opt/clickhouse/data/default/
/opt/clickhouse/metadata/default/

(它们可以在server config中配置。)
为需要的用户运行’chown’

日志的路径可以在server config (src/programs/server/config.xml)中配置。

# 2. 启动ClickHouse 

可以运行如下命令在后台启动服务：

sudo service clickhouse-server start
可以在/var/log/clickhouse-server/目录中查看日志。

如果服务没有启动，请检查配置文件 /etc/clickhouse-server/config.xml。

你也可以在控制台中直接启动服务：

clickhouse-server --config-file=/etc/clickhouse-server/config.xml
在这种情况下，日志将被打印到控制台中，这在开发过程中很方便。
如果配置文件在当前目录中，你可以不指定’–config-file’参数。它默认使用’./config.xml’。

你可以使用命令行客户端连接到服务：

clickhouse-client
默认情况下它使用’default’用户无密码的与localhost:9000服务建立连接。
客户端也可以用于连接远程服务，例如：

clickhouse-client --host=example.com
有关更多信息，请参考«Command-line client»部分。

# 3. 检查ClickHouse是否工作

milovidov@hostname:~/work/metrica/src/src/Client$ ./clickhouse-client
ClickHouse client version 0.0.18749.
Connecting to localhost:9000.
Connected to ClickHouse server version 0.0.18749.

:) SELECT 1

SELECT 1

┌─1─┐
│ 1 │
└───┘

1 rows in set. Elapsed: 0.003 sec.

:)
恭喜，系统已经工作了!

为了继续进行实验，你可以尝试下载测试数据集。
