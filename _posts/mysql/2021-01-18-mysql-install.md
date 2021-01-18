---
title: "MYSQL 8 安装部署"
subtitle: "「windows-linux」- 安装"
layout: post
author: "LiuL"
header-style: text
tags:
  - Mysql
  - Mysql 安装
---

<H1>MYSQL Windows 和 Linux系统安装部署</H1>
---

# 1. MySQL8.0 Windows安装

## 1.1. 下载

> MySQL8.0 For Windows zip包下载方式:

我的百度网盘-我的网盘-RDMS-mysql下载mysql-8.0.11-winx64.zip

![20200806233051](https://liulv.work/images/img/20200806233051.png)

## 1.2. 解压

> **注意**
> 解压目录不能含中文

解压到`D:\gongji\mysql\mysql-8.0.11-winx64`

![20200806233338](https://liulv.work/images/img/20200806233338.png)

## 1.3. 配置环境变量

添加MYSQL_HOME变量值为Mysql安装目录,如下：-
MYSQL_HOME=D:\gongji\mysql\mysql-8.0.11-winx64

![20200806233738](https://liulv.work/images/img/20200806233738.png)

PATH添加;%MYSQL_HOME%\bin;

![20200806233803](https://liulv.work/images/img/20200806233803.png)

## 1.4. 配置my.ini文件

 我们发现解压后的目录并没有my.ini文件，没关系可以自行创建。在安装根目录下添加 my.ini（新建文本文件，将文件类型改为.ini），写入基本配置：

```properties
[mysqld]
# 设置3306端口
port=3306

# 设置mysql的安装目录
# 切记此处一定要用双斜杠\\，单斜杠我这里会出错，不过看别人的教程，有的是单斜杠。自己尝试吧
basedir="D:\\rdbms\\mysql-8.0.11-winx64"

# 设置mysql数据库的数据的存放目录
# 此处同上
datadir="D:\\rdbms\\mysql-8.0.11-winx64\\data"

# 允许最大连接数
max_connections=200

# 允许连接失败的次数。这是为了防止有人从该h,5']主机试图攻击数据库系统
max_connect_errors=10

# 服务端使用的字符集默认为UTF8
character-set-server=utf8

# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB

# 默认使用“mysql_native_password”插件认证
default_authentication_plugin=mysql_native_password

[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8

[client]
# 设置mysql客户端连接服务端时默认使用的端口
port=3306
default-character-set=utf8
```

注意：其中的data目录不需要创建，下一步初始化工作中会自动创建

## 1.5. 安装Mysql

在安装时，必须以管理员身份运行cmd，否则在安装时会报错，会导致安装失败的情况

### 1.5.1. 初始化数据库

在MySQL安装目录的 bin 目录下执行命令：

mysqld --initialize --console

执行完成后，会打印 root 用户的初始默认密码，比如`l/%8wqerkUFX`：

```powershell
2020-08-17T02:00:26.509749Z 0 [System] [MY-013169] [Server] D:\gongji\mysql\mysql-8.0.11-winx64\bin\mysqld.exe (mysqld 8.0.11) initializing of server in progress as process 13968
2020-08-17T02:00:30.393776Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: Hu18AslT)rqq
2020-08-17T02:00:32.683490Z 0 [System] [MY-013170] [Server] D:\gongji\mysql\mysql-8.0.11-winx64\bin\mysqld.exe (mysqld 8.0.11) initializing of server has completed
```

 注意！执行输出结果里面有一段： [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: rI5rvf5x5G,E 其中root@localhost:后面的“rI5rvf5x5G,E”就是初始密码（不含首位空格）。在没有更改密码前，需要记住这个密码，后续登录需要用到。

 要是你手贱，关快了，或者没记住，那也没事，删掉初始化的 datadir 目录，再执行一遍初始化命令，又会重新生成的。当然，也可以使用安全工具，强制改密码，用什么方法，自己随意。

### 1.5.2. 安装服务

在MySQL安装目录的 bin 目录下执行命令：

```sql
mysqld --install 服务名
````

后面的服务名可以不写，默认的名字为 mysql。当然，如果你的电脑上需要安装多个MySQL服务，就可以用不同的名字区分了，比如 mysql5 和 MySQL8。

我这里使用MySQL8.

```shell
mysqld --install MySQL8
```

Install/Remove of the Service Denied!


### 1.5.3. 启动服务

```powershell
net start MySQL8
```

可停止服务(可選擇命令)

```powershell
net stop MySQL8
```

或卸载服务(可選擇命令)
```powershell
sc delete MySQL8/mysqld -remove
```

> **注意**
> 如果同时安装了57 和80版本
> 那么需要停止掉57才能使用80，需要哪个版本就停止掉哪个服务，启动另外一个服务，常用服务设置为自动启动，另外一个版本服务设置为手动启动服务

比如我后续就使用8版本，那么8设置为自动启动，5设置为手动启动，如下图

![20200806235033](https://liulv.work/images/img/20200806235033.png)


# 2. Mysql8 Linux安装

## 2.1. Linux服务器信息

Red Hat Enterprise Linux Server release 7.6 (Maipo)

## 2.2. 官网下载

https://dev.mysql.com/downloads/mysql/

![20210118125704](https://liulv.work/images/img/20210118125704.png)

点击archives选择版本和平台，e.g 我们选择其中的8.0.11版本，Linux-Generic

![1610946057(1)](https://liulv.work/images/img/1610946057(1).png)

我们选择下载mysql-8.0.11-linux-glibc2.12-x86_64.tar.gz



## 2.3. wget下载

mkdir /var/lib/mysql


wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.11-linux-glibc2.12-x86_64.tar.gz


解压

tar -zxvf mysql-8.0.11-linux-glibc2.12-x86_64.tar.gz

重命名
mv mysql-8.0.11-linux-glibc2.12-x86_64 mysql-8.0.11

## 2.4. 创建mysql用户

 useradd mysql
 
useradd -g mysql mysql

 mysql目录改为mysql读写权限

 chown -R mysql:mysql /var/lib/mysql/mysql-8.0.11

## 2.5. 初始化

cd /var/lib/mysql/mysql-8.0.11

./bin/mysqld --user=mysql --basedir=/var/lib/mysql/mysql-8.0.11/ --datadir=/var/lib/mysql/mysql-8.0.11/data/ --initialize

```log
root@hadoop01 mysql-8.0.11]# ./bin/mysqld --user=mysql --basedir=/var/lib/mysql/mysql-8.0.11/ --datadir=/var/lib/mysql/mysql-8.0.11/data/ --initialize
2021-01-18T06:06:28.857370Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
2021-01-18T06:06:28.857548Z 0 [System] [MY-013169] [Server] /var/lib/mysql/mysql-8.0.11/bin/mysqld (mysqld 8.0.11) initializing of server in progress as process 6657
2021-01-18T06:06:39.837190Z 5 [Note] [MY-010454] [Server] A temporary password is generated for  root@localhost: *8sFy+-orL&
2021-01-18T06:06:55.781113Z 0 [System] [MY-013170] [Server] /var/lib/mysql/mysql-8.0.11/bin/mysqld (mysqld 8.0.11) initializing of server has completed
[root@hadoop01 mysql-8.0.11]# 
```

生成临时密码： root@localhost: *8sFy+-orL&

## 2.6. Mysql配置

vim /etc/my.cnf

![20210118141014](https://liulv.work/images/img/20210118141014.png)

```properties
basedir=/var/lib/mysql/mysql-8.0.11
datadir=/var/lib/mysql/mysql-8.0.11/data
socket=/tmp/mysql.sock
```

vi /var/log/mariadb/mariadb.log

chown -R mysql:mysql /var/log/mariadb/*

## 2.7. 建立MySQL服务

添加Mysql到系统服务, 若mysqld，以下mysql相应的修改mysqld,如下图所示

    cp -a ./support-files/mysql.server /etc/init.d/mysql 
    chmod +x /etc/init.d/mysql 
    chkconfig --add mysql

检查服务是否生效

    chkconfig --list mysql

（chkconfig命令备：不是此处执行）删除mysql服务

    chkconfig --del mysql

（chkconfig命令备：不是此处执行）设置mysl开机启动

    chkconfig mysql on

（chkconfig命令备：不是此处执行）关闭mysl开机启动

    chkconfig mysql off

## 2.8. 启动Mysql服务

启动
```powershell
service mysql start

# yum安装mysql启动方式
systemctl start mysqld
```
   

查看启动状态
```powershell
    service mysql status;
```

## 2.9. 登录

```powershell
mysql -uroot -p

# 如提示无mysql命令执行下面命令
ln -s /var/lib/mysql/mysql-8.0.11/bin/mysql /usr/bin

# 再执行
mysql -uroot -p
```

# 3. Mysql配置调整

## 3.1. root密码不能登录

编辑/etc/my.cnf(注：windows下修改的是my.ini)某一行添加skip-grant-tables

![20210118151138](https://liulv.work/images/img/20210118151138.png)

重启mysql

service mysql restart

mysql 命令直接进入控制台，更新root密码

```sql
-- 老版本5
update user set password=password("123456") where user="root";

--新版本8
ALTER user 'root'@'localhost' IDENTIFIED BY '123456';

flush privileges;

quit;
```

编辑/etc/my.cnf(注：windows下修改的是my.ini)去掉skip-grant-tables

重启
service mysql restart

登录
mysql -uroot -p123456

![20210118151739](https://liulv.work/images/img/20210118151739.png)


## 3.2. 更改密码

在MySQL安装目录的 bin 目录下执行命令：
```sql
mysql -u root -pl/%8wqerkUFX

mysql -h 127.0.0.1 -u root -pl/%8wqerkUFX
```

进入MySQL命令模式, 在MySQL中执行命令：
```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';
```

修改密码，注意命令尾的；一定要有，这是mysql的语法

## 3.3. 修复数据库时区

**临时修改全局会话时区**

```sql
set global time_zone = '+8:00';
set time_zone = '+8:00';
flush privileges;
```

查询数据库时区
```sql
show variables like "%time_zone%";
```

但是如果数据库重启还是不信，需要配置my.ini中添加

```properties
# 设置时区
default-time_zone='+8:00'
```

![20201013175958](https://liulv.work/images/img/20201013175958.png)

重启mysql服务

```powershell
net stop MySQL8
net start MySQL8
```

## 3.4. 去掉ONLY_FULL_GROUP_BY

异常：
```log
Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated colum 
```

原因
: MySQL 5.7.5及以上功能依赖检测功能。如果启用了ONLY_FULL_GROUP_BY SQL模式（默认情况下），MySQL将拒绝选择列表，HAVING条件或ORDER BY列表的查询引用在GROUP BY子句中既未命名的非集合列，也不在功能上依赖于它们。

**临时解决办法：**

```sql
select @@global.sql_mode;
```

查询出来值去掉ONLY_FULL_GROUP_BY，重新设置值。

```sql
set @@global.sql_mode=
'STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION'
```

**永久解决办法：**

但是重启后，还是会出现问题，彻底解决办法mysql安装跟目录下my.ini添加sql_mode配置

```conf
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
```

![20201013175156](https://liulv.work/images/img/20201013175156.png)

重启mysql服务

```powershell
net stop MySQL8
net start MySQL8
```

## 3.5. 授权外部访问

root登进MySQL之后

```sql
use mysql

# 更新域属性，'%'表示允许外部访问
update user set host='%' where user ='root';

FLUSH PRIVILEGES;

# 再执行授权语句
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'WITH GRANT OPTION;

FLUSH PRIVILEGES;
```