---
title: "在Linux上安装部署Hadoop"
subtitle: "Hadoop安装"
layout: post
author: "LiuL"
header-style: text
tags:
  - Hadoop
  - Hadoop 安装
---

# 1. linux环境准备

## 1.1. 服务器环境

|操作系统  |操作系统版本  |CPU  |内存  |磁盘  |
|---------|---------|---------|---------|---------|
|Linux     |   Redhat7.6      | 8核8线程        | 8G        | 100G        |


## 1.2. 设置hosts和hostname

这里使用伪分布环境，单节点部署，所有只有一个节点

编辑`/etc/hosts`, 添加如下值

```powershell
127.0.0.1 hadoop01
```

编辑`/etc/hostname`,将其值删除，添加hadoop01

## 1.3. 关闭防火墙

```powershell
systemctl stop firewalld
systemctl disable firewalld 
systemctl status firewalld 
```

## 1.4. 关闭SELinux

``` shell
getenforce
```

返回
Enforcing

执行关闭SELinux

``` shell
setenforce 0
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

检查SELinux

``` shell
getenforce
```

返回
Disabled

## 1.5. 设置ssh免密钥登录

**生成无密码的密钥对**

``` shell
ssh-keygen -t rsa
```

(一路回车，生成无密码的密钥对。)

**将公钥添加到认证文件中**

``` shell
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

**设置authorized_keys的访问权限**

``` shell
chmod 600 ~/.ssh/authorized_keys
```

**ssh无密码证书发给其他节点**

```powershell
ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop01
```

**测试**

```powershell
ssh hadoop01 
```

# 2. JDK安装

**下载解压**

下载jdk-8u112-linux-x64.tar
`
上传到服务器`/usr/lib/java/`，执行下面命令解压得到`/usr/lib/java/jdk1.8.0_112`

```powershell
tar zxvf /usr/lib/java/jdk-8u112-linux-x64.tar -C /usr/lib/java
```

**设置环境变量**

```powershell
sudo echo 'export JAVA_HOME=/usr/lib/java/jdk1.8.0_112' >> /etc/profile
sudo echo 'export PATH=$PATH:$JAVA_HOME/bin' >> /etc/profil
sudo echo 'export CLASSPATH=.:$JAVA_HOME/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar' >> /etc/profil

source /etc/profile
```

**检查**

java -version

**如果存在openJDK，修改默认jdk为我们配置的JDK**

配置默认JDK，ubuntu下默认JDK是openJDK。。终端输入

```powershell
sudo update-alternatives --install /usr/bin/java java /usr/lib/java/jdk1.8.0_112 300
sudo update-alternatives --install /usr/lib/javac javac /usr/lib/java/jdk1.8.0_112/bin/javac 300
```


执行下面命令，配置需要默认的JDK, 会列出jdk版本, 选择为我们配置的JDK回车确定即可

```powershell
sudo update-alternatives --config java 
```

![20201216115811](https://liulv.work/images/img/20201216115811.png)

再一次执行检查

java -version

![20201216115958](https://liulv.work/images/img/20201216115958.png)

# 3. Hadoop安装

## 3.1. 下载hadoop包

请点击Appache Hadoop官方的[下载地址](https://archive.apache.org/dist/hadoop/common/)，查看对应版本，选择对应版本下载：

![20201215185002](https://liulv.work/images/img/20201215185002.png)

我们这里选择2.7.3版本进行下载，下载其中的hadoop-2.7.3.tar.gz

![20201215185115](https://liulv.work/images/img/20201215185115.png)

## 3.2. 拷贝解压

部署节点创建`/var/lib/hadoop`并并将下载的包拷贝其目录下

```powershell
mkdir -p /var/lib/hadoop
```

![1608029546(1)](https://liulv.work/images/img/1608029546(1).png)

使用下面命令进行解压

```java
cd /var/lib/hadoop
tar -zxvf hadoop-2.7.3.tar.gz
```

解压后得到`/var/lib/hadoop/hadoop-2.7.3`目录

## 3.3. 配置设置

**设置java_home**

在hadoop-env、mapred-env、yarn-env.sh文件中，注释掉`export JAVA_HOME=$JAVA_HOME`,新增`export JAVA_HOME=/usr/lib/java/jdk1.8.0_112`

![20201216141935](https://liulv.work/images/img/20201216141935.png)


**core-site设置**

```xml
<configuration>
<!--指定hdfs的唯一入口，以及namenode的地址-->
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://hadoop01:9000</value>
  </property>
  <!--配置hadoop的临时目录-->
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/home/hadoop/data/tpm</value>
  </property>
</configuration>
```

> 注意hadoop01为节点的hostname, 如果无`/home/hadoop/data/tpm`请执行下面命令创建

```powershell
mkdir -p /home/hadoop/data/tpm
```

**hdfs-site设置**

```xml
<configuration>
	<!--配置hdfs中文件块的副本数-->
	<property>
	<name>dfs.replication</name>
	<value>1</value>
	</property>
  <!--关闭hdfs的权限检查-->
	<property>
		<name>dfs.permissions.enabled</name>
		<value>false</value>
	</property>
</configuration>
```

**mapred-site设置**

```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>hadoop01:10020</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>hadoop01:19888</value>
    </property>
</configuration>
```

> 注意hadoop01为节点的hostname

**yarn-site设置**

```xml
<configuration>
	<!--用于配置日志聚集，保存7天-->
	<property>
		<name>yarn.log-aggregation-enable</name>
		<value>true</value>
	</property>
	<property>
		<name>yarn.log-aggregation.retain-seconds</name>
		<value>604800</value>
	</property>
</configuration>
```

## 设置环境变量

```powershell
echo 'export HADOOP_HOME=/var/lib/hadoop/hadoop-2.7.3' >> /etc/profile
echo 'export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin' >> /etc/profile
source /etc/profile
```

## 启动

**启动命令**

start-all.sh

**停止命令**

stop-all.sh

**访问**
hdfs http://192.168.56.101:50070/

yarn  http://192.168.56.101:8088/

-------




