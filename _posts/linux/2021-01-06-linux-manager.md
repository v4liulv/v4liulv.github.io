---
title: "Linux管理手册"
subtitle: "Linux操作汇总"
layout: post
author: "LiuL"
header-style: text
tags:
  - Linux
  - Linux 命令
---

Linux管理手册
---

# 1. 查看linux服务器配置信息命令

## 1.1. 查看操作系统信息：

```powershell
uname -a

cat /etc/redhat-release 
```

CentOS Linux release 7.5.1804 (Core)


## 1.2. 查看是否是虚拟机还是物理机：

https://blog.csdn.net/yangniuer/article/details/105320619

dmesg | grep -i virtual

## 1.3. 查看 cpu信息：

cat /proc/cpuinfo

## 1.4. 查看内存信息：

free

free -g

grep MemTotal /proc/meminfo

## 1.5. 查看centos版本信息：

cat /etc/issue

## 1.6. 查看磁盘使用情况：

df -h

查看其它磁盘外设信息：
fdisk -l

smartctl --all /dev/sda
如果没有smartctl 工具可以用 yum install smartmontools安装

查看所有可用块设备的信息：
lsblk

## 1.7. linux查看磁盘类型（是否SSD盘）
介绍两种方法：

第一种：

cat /sys/block/sda/queue/rotational

注意：
命令中的sba是你的磁盘名称，可以通过df命令查看磁盘，然后修改成你要的
结果：
返回0：SSD盘
返回1：SATA盘

## 1.8. 使用磁盘是否为raid

https://www.cnblogs.com/JianGuoWan/p/7709971.html

cat /proc/mdstat

如果
Personalities : 
unused devices: <none>

则没做raid，或者是关闭的

## 1.9. 查看所有硬件信息
dmidecode |more
或：dmesg |more

## 1.10. 查看网卡信息

 ethtool eth0

# 2. 创建文件命令

直接vim 编辑保存


# 3. TCP端口

## 3.1. 查看端口

netstat -lnpt

netstat -anp |grep 6500

查看端口号被那个进程id占用，然后
kill -9

查看那些程序使用tcp的80端口：

fuser -n tcp 80

## 3.2. 防火墙下打开端口

### 3.2.1. 打开端口号

TBDS Portal必须开放端口：80 443 8081 8080 8088

iptables -A INPUT -ptcp --dport  443 -j ACCEPT
iptables -A INPUT -ptcp --dport  8081 -j ACCEPT
iptables -A INPUT -ptcp --dport  8080 -j ACCEPT
iptables -A INPUT -ptcp --dport  8088 -j ACCEPT

### 3.2.2. 查询端口是否已打开

firewall-cmd --query-port=7788/tcp

### 3.2.3. 关闭端口号

iptables -A OUTPUT -p tcp --dport 端口号-j DROP

### 3.2.4. 保存设置

service iptables save 
systemctl iptables save

# 4. 添加用户

```powershell
useradd tbds_student_liulv
passwd tbds_student_liulv
```

# 5. 日志查看

**跟踪最新日志**

```powershell
tailf /log/t.log
```

**跟踪最新日志10行**

```powershell
tail -n 10
```

**根据关键字查看日志**

- **根据关键字查看日志**

cat t.log | grep "Exception"

- **根据关键字查看后10行日志**

cat t.log | grep "Error" -A 10

- **根据关键字查看前10行日志**

cat t.log | grep "Error" -B 10

- **根据关键字查看前后10行日志，并显示出行号**

cat -n t.log | grep "Warn" -C 10

- **查看日志前 50 行**

cat t.log | head -n 50

- **查看日志后 50 行，并显示出行号**

cat -n t.log | tail -n 50

# 6. Linux 添加环境变量模版

```powershell
echo "export SCALA_HOME=/usr/local/scala/scala-2.11.11" >> /etc/profile
echo "export PATH=$SCALA_HOME/bin:$PATH" >> /etc/profile
cat /etc/profile
source /etc/profile
```

# 7. gradle-项目自动化建构工具命令

``` sh
gradle idea
gradlew assemble -Dorg.gradle.java.home=D:\gongji\Java\jdk-12.0.2
```

# 8. rmp

``` sh
yum groupinstall --downloadonly --downloaddir=/home/hadoop/desktop Desktop
rpm -ivh ./*.rpm --nodeps --force
rpm -qa | grep realvnc
rpm -e --nodeps realvnc-vnc-server-6.5.0.41730-1.x86_64
```

# 9. 磁盘查看：

``` sh
df -hl
```

查看所有磁盘使用率

``` sh
du -h --max-depth=1
```

查看当前目录下最大的文件或路径。结果发现是我的hadoop data 目录占用超大，还有自己测试用的数据，不想随便清理，所以查看hdfs路径:

``` sh
hadoop fs -du -h
```

此时发现占用空间最大的是 hbase的WAL日志占用比较大，直接删除，问题解决。

# 10. SCP命令

``` sh
scp -r /root root@43.224.34.73:/home/lk
```

1Hblsqt# 
132.232.90.182 172.27.16.31
172.27.16.104
172.27.16.5

# 11. Linux 重启关机

$ reboot -----------重启
$ halt -------------关机

# 12. 端口和应用查看

netstat -nptl | grep 10000
ps -ef|grep dse

# 13. 建立软连接ln -s

ln -s /usr/java/jdk1.8.0_151/jre/bin/java /usr/bin/java 
ln -s /opt/jdk1.8.0_172/bin/jps /opt/jdk1.8.0_172/jre/bin/jps
ln -s /usr/java/jdk1.8.0_172-amd64/bin/jps /usr/java/jdk1.8.0_172-amd64/jre/bin/jps

# 14. 网络配置

## 14.1. 网络配置方式1

安装完Linux系统后，无法上网 ping www.baidu.com 提示Network is unreachable
解决办法：
cd /etc/sysconfig/network-scripts
vi ifcfg-enp0s3 （后缀为本机自动生成）

TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=enp0s3
UUID=fadf194d-7c3e-4b65-9ddc-8a0e780fc234
DEVICE=enp0s3
ONBOOT=yes
IPADDR=192.168.99.101
NETWORK=192.168.99.16
NETSTAT=255.255.255.0
GATEWAY=192.168.10.1
DNS1=8.8.8.8
DNS2=114.114.114.114

2、重启网卡 service network restart

### 14.1.1. 新建网卡

cd /etc/sysconfig/network-scripts/

cp ifcfg-enp0s3 ifcfg-eth1

### 14.1.2. NetworkManager

systemctl start NetworkManager
systemctl status NetworkManager
systemctl enable NetworkManager

### 14.1.3. 网卡配置界面 el7

``` shell
nmtui
```

就添加个IP地址其他全部不变
IP4 ADDR=192.168.19.20 

ip配置

``` shell
nm-connection-editor
```

其中General勾选第一和第二个

IPV4 Settings选择
Mothod: Manual

地址配置如下

``` shell
IPADDR=192.168.19.20
GATEWAY=192.168.19.2
NETMASK=255.255.255.0
DNS Servers=192.168.19.2
DNS1=118.118.118.9
```

### 14.1.4. 设置IP

nmcli connection show

nmcli connection modify enp0s3 ipv4.addresses 192.168.99.101/24 ipv4.gateway 192.168.99.1 connection.autoconnect yes ipv4.method manual

nmcli connection up enp0s3

nmcli connection show <nmcliName>

nmcli connection modify enp0s3 ipv4.addresses 192.168.99.101/24

### 14.1.5. 网卡终端图形化工具

nm-connection-editor

### 14.1.6. 重启网络

``` shell
systemctl restart network
```

### 14.1.7. 检查IP地址是否正确

``` shell
ifconfig
```

# 15. 关闭防火墙并设置开机不启动

## 15.1. 临时关闭防火墙

el6:

``` shell
iptables -F
```

el7:

``` shell
firewalld -F
```

诊断域名：19c406791k.imwork.net 域名IP地址指向：47.98.161.190 转发服务器IP：47.98.161.190 域名已激活内网穿透功能，并与转发服务器IP指向一致   映射：19c406791k.imwork.net:31025 局域网服务器：
192.168.19.129:1723 连接转发服务器成功   局域网服务器连接成功

## 15.2. 关闭防火墙设置开机不启动

**el6**

``` shell
service iptables stop
chkconfig iptables off
```

**el7**

systemctl status firewalld 

``` shell
systemctl stop firewalld
systemctl disable firewalld 
```

# 16. 关闭SELinux

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
Permissive

# 17. 修改时区

## 17.1. 最新版本方法

tzselect：
执行tzselect命令-->选择Asia-->选择China-->选择east China - Beijing, Guangdong, Shanghai, etc-->然后输入1。
执行tzselect命令-->5-->9-->选择east China - Beijing, Guangdong, Shanghai, etc-->然后输入1。

TZ='Asia/Shanghai'; export TZ
date命令查看到为CST, 即中国标准时间

修改配置文件

```shell
vim /etc/sysconfig/clock

ZONE="Asia/Shanghai"
UTC=false
ARC=false
```

//linux的时区设置为上海时区

```shell
rm /etc/localtime
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

ln -sf /usr/share/zoneinfo/UTC /etc/localtime

timedatectl
```

//设置硬件时间和系统时间一致并校准
hwclock --systohc

## 17.2. 查看系统时区

```shell
date -R; date +%z
```

```sh
[root@hadoop05 ~]#  date -R; date +%z  
Thu, 10 Nov 2016 17:20:24 -08001
```

如IOS6.6版本的是-08001，而我们使用的市区是上海的+8000东8时区，这样时间就不准需要进行时区调整+8000东8时区

时区调整上海+8000东8时区

``` shell
cp /etc/localtime /etc/localtime.bak
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

再检查时区和时间是否准确

```sh
date -R; date +%z
date
```

```sh
 root@hadoop04 ~]# date -R; date +%z
Fri, 11 Nov 2016 09:25:26 +0800
```

+0800时间也自动tongbu成功。和北京时间相同

## 17.3. 设置时区方式二(可选)

### 17.3.1. 将Linux系统时钟tongbu到远程NTP服务器

``` shell
timedatectl set-ntp true
```

### 17.3.2. Linux中设置本地时区

使用set-timezone开关，如下所示。
中国上海的时区：

``` shell
timedatectl set-timezone "Asia/Shanghai"
```

## 17.4. 取消订阅服务（el7）

订阅插件提示：This system is not registered with an entitlement server. You can use subscription-manager to register.

每次yum调用(不禁掉plugins的情况下)，都会更新此文件。
因此，为了不冲突，可以如下操作: 停止掉该插件的使用，在配置文件中把enable=0即可。

[root@micocube ~] vim /etc/yum/pluginconf.d/subscription-manager.conf
[main]
enabled=0           #将它禁用掉

## 17.5. 创建本地yum源

### 17.5.1. 配置yum仓库：本地yum源

```shell
rm -rf /etc/yum.repos.d/*
vim /etc/yum.repos.d/redhat7.repo
```

修改如下内容

```shell
[rhel]
name=rhel
baseurl=file:///media/cdrom
enabled=1
gpgcheck=0
```

### 17.5.2. 创建本地yum库目录

``` shell
mkdir /media/cdrom
```

df -h 检查镜像盘是否成功

``` shell
df -h
```

如下

```powershell
root@localhost root]# df -h
Filesystem             Size  Used Avail Use% Mounted on
/dev/mapper/rhel-root   50G  2.9G   48G   6% /
devtmpfs               3.8G     0  3.8G   0% /dev
tmpfs                  3.9G   88K  3.9G   1% /dev/shm
tmpfs                  3.9G  8.9M  3.9G   1% /run
tmpfs                  3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/mapper/rhel-home   42G   38M   42G   1% /home
/dev/sda1              497M  157M  340M  32% /boot
tmpfs                  781M   12K  781M   1% /run/user/0
/dev/sr0               3.8G  3.8G     0 100% /run/media/root/RHEL-7.2 Server.x86_64
```

## 17.6. 对镜像磁盘进行yum库目录挂盘

```powershell
mount /dev/sr0 /media/cdrom
```

## 17.7. YUM默认安装无法使用

### 17.7.1. 查看redhat 7.0系统本身所安装的那些yum软件包

```powershell
rpm -qa | grep yum

yum-utils-1.1.31-50.el7.noarch
yum-metadata-parser-1.1.4-10.el7.x86_64
yum-3.4.3-161.el7.noarch
PackageKit-yum-1.1.10-1.el7.x86_64
yum-rhn-plugin-2.0.1-10.el7.noarch
yum-langpacks-0.4.2-7.el7.noarch
```

## 17.8. 卸载这些软件包

```powershell
yum-updateonboot-1.1.31-52.el7.noarch
rpm -e yum-rhn-plugin-2.0.1-10.el7.noarch --nodeps
rpm -e yum-metadata-parser-1.1.4-10.el7.x86_64 --nodeps
rpm -e yum-3.4.3-163.el7.centos.noarch --nodeps
rpm -e yum-utils-1.1.31-50.el7.noarch --nodeps
rpm -e yum-updateonboot-1.1.31-52.el7.noarch --nodeps

rpm -e yum-langpacks-0.4.2-7.el7.noarch --nodeps

rpm -e python-kitchen-1.1.1-5.el7.noarch --nodeps
rpm -e python-chardet-2.2.1-1.el7_1.noarch --nodeps
```

## 17.9. 下载包并安装

```powershell
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/python-kitchen-1.1.1-5.el7.noarch.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/python-chardet-2.2.1-3.el7.noarch.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-3.4.3-163.el7.centos.noarch.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-metadata-parser-1.1.4-10.el7.x86_64.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-utils-1.1.31-52.el7.noarch.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-updateonboot-1.1.31-52.el7.noarch.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-plugin-fastestmirror-1.1.31-52.el7.noarch.rpm

rpm -ivh python-*
rpm -ivh yum-*
```

## 17.10. 配置NTP服务

外网环境：
$ yum install ntp

内网环境：
a. 同样系统的Linux外网服务器下载rmp包和依赖包
cd /home/hadoop/ntp
yumdownloader --resolve ntp
或
yum install all --downloadonly --downloaddir=/home/hadoop/ntp ntp

b. 拷贝/home/hadoop/ntp/包带内网服务器同样目录下

c. 内网安装
cd /home/hadoop/ntp/
rpm -ivh ./*.rpm --nodeps --force

## 17.11. 启动 ntp

$ service ntpd start

## 17.12. 设置开机启动:

$ chkconfig ntpd on

## 17.13. 同步时间服务器

vi /etc/ntp.conf

把其中的server全部注销，并添加远程时间服务器或者master节点

```powershell
# server 0.centos.pool.ntp.org iburst  
# server 1.centos.pool.ntp.org iburst  
# server 2.centos.pool.ntp.org iburst  
# server 3.centos.pool.ntp.org iburst

# 阿里云时间服务器
server ntp1.aliyun.com iburst
server ntp2.aliyun.com iburst
server ntp3.aliyun.com iburst
server ntp4.aliyun.com iburst
server ntp5.aliyun.com iburst
server ntp6.aliyun.com iburst
server ntp7.aliyun.com iburst

# 本地ntp服务器
server master.ip.host iburst
```

## 17.14. 重启ntp服务

$ service ntpd restart

检查是否成功，用ntpstat命令查看同步状态，出现以下状态代表启动成功：
$ ntpstat

# 18. 修改主机名hostname

``` shell
vim /etc/sysconfig/network
```

编辑为下面内容，其中Master为定义主机名，GATEWAY为本机的IP地址

``` 
NETWORKING=yes
HOSTNAME=Master
GATEWAY=192.168.19.20
```

# 19. 修改hosts

即ip与主机名的对应关系

``` shell
vim /etc/hosts
```

编辑为下面内容

``` 
192.168.18.205  des01
192.168.18.206  des02
192.168.18.207  des03
```

# 20. ssh无密码登陆

## 20.1. 生成无密码的密钥对

``` shell
ssh-keygen -t rsa
```

(一路回车，生成无密码的密钥对。)

## 20.2. 将公钥添加到认证文件中

``` shell
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

## 20.3. 设置authorized_keys的访问权限

``` shell
chmod 600 ~/.ssh/authorized_keys
```

## 20.4. ssh无密码证书发给其他节点

ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop01
ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop02
ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop03
ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop04

ssh-copy-id -i ~/.ssh/id_rsa.pub tbds-19.203

## 20.5. 关闭大透明压缩

echo never > /sys/kernel/mm/transparent_hugepage/defrag
echo never > /sys/kernel/mm/transparent_hugepage/enabled

然后写入/etc/rc.local

# 21. 禁止交换空间

# 22. Linux 系统设置sh文件开机启动

   工作中有一个linux下的服务需要启动，但是机器总是断电，导致需要反复启动，找了一下开机自启动的方法，解决了这个问题。Linux设置开机自启动非常简单，只要找到rc.local文件，将你需要自启动的文件加进去即可。我的linux服务器的rc.local文件在/etc文件夹下。rc.local文件没有修改之前是这样滴：

``` shell
#!/bin/sh
#
# This script will be executed *after* all the other init scripts.
# You can put your own initialization stuff in here if you don't
# want to do the full Sys V style init stuff.

touch /var/lock/subsys/local
```

设置rc.local是可执行 `chmod +x /etc/rc.d/rc.local`
在文件下方直接添加你需要自启动文件的路径，然后直接写上启动命令，修改后的文件如下：

``` shell
echo "cd /usr/share/dse-studio/datastax-studio-6.0.0" >> /etc/rc.local
echo "nohup bin/datastax.sh >> /var/log/datastaxstudio/output.log 2>&1 &" >> /etc/rc.local
```

``` shell
#!/bin/sh
#
# This script will be executed *after* all the other init scripts.
# You can put your own initialization stuff in here if you don't
# want to do the full Sys V style init stuff.

touch /var/lock/subsys/local

cd /usr/share/dse-studio/datastax-studio-6.0.0
nohup bin/datastax.sh >> /var/log/datastaxstudio/output.log 2>&1 &
```

# 23. 安装Yum并配置阿里源

> redhat系统中，如果你想要更新yum仓库，它会提示让你注册才能更新，因为centos和redhat基本相同，所以我把yum这一套全换成centos的。

1. 删除已安装的yum包(不必须)

cd /etc/rpm/

rm -rf ./m*

2. 下载centos的yum包(不必须)

wget https://mirrors.aliyun.com/centos/7/os/x86_64/Packages/yum-metadata-parser-1.1.4-10.el7.x86_64.rpm
wget https://mirrors.aliyun.com/centos/7/os/x86_64/Packages/yum-utils-1.1.31-40.el7.noarch.rpm
wget https://mirrors.aliyun.com/centos/7/os/x86_64/Packages/yum-3.4.3-158.el7.centos.noarch.rpm
wget https://mirrors.aliyun.com/centos/7/os/x86_64/Packages/yum-plugin-fastestmirror-1.1.31-45.el7.noarch.rpm
wget https://mirrors.aliyun.com/centos/7/os/x86_64/Packages/python-urlgrabber-3.10-8.el7.noarch.rpm

如果出现404错误，请去https://mirrors.aliyun.com/centos/7/os/x86_64/Packages/搜索关键词查询最新rpm包的链接。

3. 强制安装rpm包

--force --nodeps为忽略依赖检测的强制安装


rpm -ivh python-urlgrabber-3.10-8.el7.noarch.rpm --force --nodeps

rpm -ivh yum* --force --nodeps

4. 查看yum包是否安装好

[root@localhost yum.repos.d]# rpm -qa |grep yum
yum-metadata-parser-1.1.4-10.el7.x86_64
yum-utils-1.1.31-45.el7.noarch
yum-3.4.3-158.el7.centos.noarch
yum-plugin-fastestmirror-1.1.31-45.el7.noarch

5. 修改repo

下载源文件

curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
需要把Centos-7.repo文件中的$releasever全部替换为7

   67  cd /etc/yum.repos.d/
   68  ls
   69  vim CentOS-Base.repo
在vim中执行:%s/$releasever/7/g快速替换。保存退出。

6. 清空重载yum

```sql
yum clean all
yum update
# 最后查看一下yum列表
yum list
```


# 24. yum 需要的rpm包

el6 :
python-urlgrabber-3.9.1-9.el6.noarch
yum-utils-1.1.30-14.el6.noarch
yum-metadata-parser-1.1.2-16.el6.x86_64
yum-plugin-security-1.1.30-14.el6.noarch
yum-3.2.29-40.el6.noarch

Linux 下载方式一

wget https://mirrors.aliyun.com/centos/6/os/x86_64/Packages/python-urlgrabber-3.9.1-11.el6.noarch.rpm
wget https://mirrors.aliyun.com/centos/6/os/x86_64/Packages/yyum-utils-1.1.30-41.el6.noarch.rpm
wget https://mirrors.aliyun.com/centos/6/os/x86_64/Packages/yum-metadata-parser-1.1.2-16.el6.x86_64.rpm 
wget https://mirrors.aliyun.com/centos/6/os/x86_64/Packages/yum-plugin-security-1.1.30-41.el6.noarch.rpm  
wget https://mirrors.aliyun.com/centos/6/os/x86_64/Packages/yum-3.2.29-81.el6.centos.noarch.rpm

linux 下载方式二，离线下载拷贝，可看下面方式

> a. 外网yum install --downloadonly --downloaddir=/home/hadoop/yum/ yum*
> b. 然后u盘拷贝/home/hadoop/yum/目录下rmp到内网环境执行
> c. rpm -ivh ./*.rpm --nodeps --force

# 25. yum 外网服务器下载rpm包到内网（无法联网的服务器）

 自己找一台可以联网的机器，安装一个虚拟机，装上相同的操作系统，什么其他的软件都不要安装，如果你的内网环境rpm的时候，需要什么依赖，你就直接在可以联网的机器上执行，执行的命令为：

方式一：

* yum -install --downloadonly --downloaddir=/root/ 你的软件

方式二：

* yumdownloader --downloaddir=/home/hadoop/软件名 --resolve 软件名*

然后就会只下载，不安装，通过U盘拷贝到内网环境, 比如安装httpd

* cd /home/hadoop/http
* rpm -ivh ./*.rpm --nodeps --force

这条命令中 --nodeps 是不检查依赖 --force是覆盖安装，安装即可。

## 25.1.  

vim /etc/yum/pluginconf.d/subscription-manager.conf

[main]
enabled=0           #将它禁用掉

## 25.2. yum清楚缓存

``` shell
yum clean all
```

``` shell
yum list
```

# 26. yum安装外部远程VNC （可选）

yum install tigervnc-server

# 27. 安装httpd工具

选择其中一台服务器部署httpd，创建本地源

``` shell
mkdir /tmp/httpd
```

拷贝下面两个文件到/tmp/httpd文件目录下

``` 
httpd-2.2.15-39.el6.centos.x86_64.rpm
httpd-tools-2.2.15-39.el6.centos.x86_64.rpm
```

运行下面命令进行安装

el6 安装

``` shell
	yum localinstall –nogpgcheck httpd*
	或者
	rpm -ivh httpd-2.2.15-39.el6.centos.x86_64.rpm httpd-tools-2.2.15-39.el6.centos.x86_64.rpm
```

	

el7 安装
   需要依赖rpm文件

``` 
    apr-1.4.8-3.el7.x86_64
	apr-util-1.5.2-6.el7.x86_64
	httpd-tools-2.4.6-40.el7.x86_64
	mailcap-2.1.41-2.el7.noarch 
	httpd-2.4.6-40.el7.x86_64
```

安装命令：

``` 
    rpm -ivh httpd-2.4.6-40.el7.x86_64.rpm httpd-tools-2.4.6-40.el7.x86_64.rpm
```

	

创建http的rpm库地址/dse/enterprise/

``` 
mkdir -p /var/www/html/dse/enterprise/
```

授权目录

``` 
chmod -R ugo+rx /var/www/html/dse/enterprise/
```

启动httpd

``` 
service httpd start
```

检查启动情况
浏览器上使用http://IP/dse/enterprise/， 如http://192.168.19.32/dse/enterprise/验证

# 28. httpd设置开机启动

chkconfig httpd on

# 29. 安装VPN服务器PPTP

## 29.1. 软件安装
安装依赖软件
yum install -y ppp iptables perl

安装pptpd软件
https://pkgs.org/download/cpio下载包地址搜索pptpd，下载64位软件包到/home/hadoop/pptpd目录下pptpd-1.4.0-3.el6.x86_64.rpm
然后进行安装

``` sh
$ rpm -ivh ./*.rpm --nodeps --force
```

## 29.2. 配置文件

* 配置ppp选项

$ mv /etc/ppp/options.pptpd /etc/ppp/options.pptpd.bak
$ vi /etc/ppp/options.pptpd

输入下面内容：

``` 
name pptpd
refuse-pap
refuse-chap
refuse-mschap
require-mschap-v2
require-mppe-128
proxyarp
lock
nobsdcomp
novj
novjccomp
nologfd
idle 2592000
ms-dns 8.8.8.8
ms-dns 8.8.4.4
```

* 配置客户端连接用户信息

mv /etc/ppp/chap-secrets /etc/ppp/chap-secrets.bak
vi /etc/ppp/chap-secrets

输入以下内容：

``` 
#client server secret IP addresses
liulv  pptpd  password *
```

* 配置pptpd信息

$ mv /etc/pptpd.conf /etc/pptpd.conf.bak
$ vi /etc/pptpd.conf

输入以下内容：

``` 
option /etc/ppp/options.pptpd
logwtmp
localip 192.168.19.1
remoteip 192.168.19.11-30
```

* 配置启动路由转换

$ vi /etc/sysctl.conf
修改以下内容：

``` 
net.ipv4.ip_forward = 1
```

## 29.3. 启动pptp vpn服务端

$ service pptpd start

## 29.4. 关闭防火墙

iptables -F
chkconfig iptables off

## 29.5. 设置pptp vpn开机启动

$ chkconfig pptpd on

## 29.6. 日志信息

$ cat /var/log/messages

## 29.7. 客户端连接

可以查看下面“安装VPN客户端pptp-setup”信息

# 30. 安装VPN客户端pptp-setup（可选）

外网下载包：

``` 
$ yum install --downloadonly --downloaddir=/home/hadoop/pptp-setup pptp-setup
```

拷贝到内网，安装

``` sh
$ cd /home/hadoop/pptp-setup
$ rpm -ivh ./*.rpm --nodeps --force
```

连接vpn

``` sh
pptpsetup --create vpn --server 192.168.19.1 --username liulv --password 123456 --encrypt --start 
```

出现下面证明连接成功：

``` sh
Using interface ppp0
Connect: ppp0 <--> /dev/pts/2
CHAP authentication succeeded
MPPE 128-bit stateless compression enabled
local  IP address 192.168.1.182
remote IP address 192.168.1.180
```

但是此时发现，即使连上了vpn，还是不能上外网。这是因为路由的问题，查看路由
$ route -n 

``` 
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.1.180   0.0.0.0         255.255.255.255 UH    0      0        0 ppp0
192.168.19.1    0.0.0.0         255.255.255.255 UH    0      0        0 eth0
192.168.19.0    0.0.0.0         255.255.255.0   U     1      0        0 eth0
```

需要将ppp0的路由设为：将所有对外网络都通过ppp0路由。
$ route add -net 0.0.0.0 dev ppp0

查看路由
$ route -n 

``` 
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.1.180   0.0.0.0         255.255.255.255 UH    0      0        0 ppp0
192.168.19.1    0.0.0.0         255.255.255.255 UH    0      0        0 eth0
192.168.19.0    0.0.0.0         255.255.255.0   U     1      0        0 eth0
0.0.0.0         0.0.0.0         0.0.0.0         U     0      0        0 ppp0
```

# 31. 优化虚拟内存需求率

## 31.1. 检查虚拟内存需求率

``` shell
cat /proc/sys/vm/swappiness
```

显示如下：
30

## 31.2. 临时降低虚拟内存需求率

``` shell
sysctl vm.swappiness=0
```

## 31.3. 永久降低虚拟内存需求率

RedHat 6

``` shell
echo 'vm.swappiness = 0' > /etc/sysctl.d/swappiness.conf
```

并运行如下命令使生效

``` shell
sysctl -p
```

RedHat 7

``` shell
sysctl -w vm.swappiness=0
echo vm.swappiness = 0 >> /etc/sysctl.conf
```

# 32. 关闭透明大页面

## 32.1. 检查透明大页面

``` shell
cat /sys/kernel/mm/transparent_hugepage/defrag
```

如果显示为（未关闭）：

``` 
[always] madvise never
```

## 32.2. 临时关闭透明大页面

``` shell
echo never > /sys/kernel/mm/transparent_hugepage/defrag
```

确认配置生效：

``` shell
cat /sys/kernel/mm/transparent_hugepage/defrag
```s
显示为下面内容则关闭成功
```

always madvise [never]

``` 

## 32.3. 永久关闭透明大页面

开机自动关闭
```shell
echo 'echo never > /sys/kernel/mm/transparent_hugepage/defrag' >> /etc/rc.local
chmod +x /etc/rc.d/rc.local
```

# 33. 安装JDK

## 33.1. 检查Java是否安装

``` shell
rpm -qa | grep java
```

根据查询结果查看，如果有比如java-1.8.0-openjdk-headless-1.8.0.65-3.b17.el7.x86_64，命令如下：

``` shell
yum -y remove java-1.8.0-openjdk-headless-1.8.0.65-3.b17.el7.x86_64
```

如果需要卸载全部的openjdk, 运行下面命令

``` shell
yum -y remove java*openjdk*
```

## 33.2. 安装JDK - 源码压缩包方式安装(不推荐，如果使用此方式2.2 步骤忽略)

### 33.2.1. 下载安装压缩包

直接官网下载对应版本的JDK

使用wget方式下载，不推荐，因为下载不一定成功压缩包会出现异常

``` shell
mkdir /usr/java
cd /usr/java
wget http://download.oracle.com/otn-pub/java/jdk/8u151-b12/e758a0de34e24606bca991d704f6dcbf/jdk-8u151-linux-x64.tar.gz
```

### 33.2.2. 解压缩包

``` shell
mkdir /usr/java
cd /usr/java
tar -xzvf jdk-8u151-linux-x64.tar.gz
```

## 33.3. Rpm方式安装（推荐安装方式，如果使用此方式2.1步骤忽略）
步骤如下

下载需要版本的rpm包如jdk-8u171-linux-x64.rpm到/usr/java

Rpm安装

``` shell
cd /usr/java
rpm -ivh jdk-8u171-linux-x64.rpm
```

rpm安装完成后删除rpm包

``` shell
$ rm -r /tmp/jdk/jdk-8u171-linux-x64.rpm
```

切换Java版本

``` shell
sudo mv /usr/java/jdk1.8.0_171-amd64 /usr/java/jdk1.8.0_171
sudo alternatives --install /usr/bin/java java /usr/java/jdk1.8.0_171/bin/java 1
sudo update-alternatives --set java /usr/java/laster/jre/bin/java
```

配置jdk变量环境

``` shell
sudo echo 'export JAVA_HOME=/usr/java/jdk1.8.0_101' >> /etc/profile
sudo echo 'export JRE_HOME=${JAVA_HOME}/jre' >> /etc/profile
sudo echo 'export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib' >> /etc/profile
sudo echo 'export PATH=${JAVA_HOME}/bin:$PATH' >> /etc/profile
```

导入java环境变量

``` shell
source /etc/profile
```

测试jdk的配置

``` 
sudo java -version
sudo /usr/bin/java -version
```

结果：

``` 
[root@Master java]# java -version
java version "1.8.0_112"
Java(TM) SE Runtime Environment (build 1.8.0_112-b15)
Java HotSpot(TM) 64-Bit Server VM (build 25.112-b15, mixed mode)
```

# 34. 安装Mysql(Rhel-7)

参考地址el6：http://www.cnblogs.com/dianqijiaodengdai/p/7731562.html

参考地址el7: https://blog.csdn.net/xyang81/article/details/51759200

mysql授权远程用户

``` sql
grant all privileges on *.* to root@"192.168.19.1" identified by "@Wtt102501@" with grant option;
flush privileges;
```

``` sql
grant all privileges on *.* to root@"192.168.19.1" identified by "123456" with grant option;
flush privileges;
```

``` sql
grant all privileges on *.* to root@"hadoop02" identified by "123456" with grant option;
flush privileges;
```

查询用户远程权限

``` sql
select user,host from user;
```

# 35. HADOOP2.7.4 安装

## 35.1. 添加Hadoop环境变量

``` shell
echo 'export HADOOP_HOME=/usr/local/hadoop/hadoop-2.7.4'  >> /etc/profile
echo 'export PATH=$HADOOP_HOME/sbin:$HADOOP_HOME/bin:$PATH' >> /etc/profile
source /etc/profile
```

## 35.2. 创建数据和日志目录

``` shell
mkdir -p /data/hadoop
mkdir -p /data/hadoop/dfs
mkdir -p /data/hadoop/dfs/data
mkdir -p /data/hadoop/dfs/checkpoint
mkdir -p /data/hadoop/dfs/name
mkdir -p /data/hadoop/yarn
mkdir -p /data/hadoop/yarn/nodemanager
mkdir -p /data/hadoop/tmp
mkdir -p /data/hadoop/logs
```

## 35.3. namenode format

``` shell
cd /usr/local/hadoop/hadoop-2.7.4/bin
hadoop namenode -format
```

# 36. Spark安装

## 36.1. 配置环境变量

``` 
echo 'export SPARK_HOME=/usr/local/spark/spark-2.2.0' >> /etc/profile
echo 'export PATH=$SPARK_HOME/bin:$SPARK_HOME/sbin:$PATH'  >> /etc/profile
echo 'export PATH=$SPARK_HOME/sbin:$PATH'  >> /etc/profile
```

## 36.2. 同步文件的配置

``` shell
rsync -av /usr/local/spark/spark-2.2.0/conf Master1:/usr/local/spark/spark-2.2.0/conf
/usr/local/scala/scala-2.11.11
```

## 36.3. Scala环境变量

``` shell
echo 'export SCALA_HOME=/usr/local/scala/scala-2.11.11'  >> /etc/profile
echo 'export PATH=$SCALA_HOME/bin:$PATH'  >> /etc/profile
```

## 36.4. 环境变量生效

``` shell
source /etc/profile
```

## 36.5. Spark运行测试

``` shell
cd /usr/local/spark/spark-2.2.0/examples/jars
spark-submit \
--class org.apache.spark.examples.SparkPi \
--master spark://Master:7077 \
--executor-memory 1G \
--total-executor-cores 4 \
/usr/local/spark/spark-2.2.0/examples/jars/spark-examples_2.11-2.2.0.jar 100
```

## 36.6. Spark yarn检查

``` 
http://spark.apache.org/docs/latest/running-on-yarn.html
--master yarn
```

# 37. 扩展磁盘 - 基于源磁盘/dev/sda

## 37.1. 关机后VMware设置虚拟机然后点击磁盘以前100G, 扩展为200G

## 37.2. 开机后查看磁盘

``` shell
ll /dev/sd*
fdisk -l
```

# 38. 创建分区 sda已经有sda2了，那么创建为3

``` shell
fdisk /dev/sda
```

``` 
输入 p 打印现有分区情况（还没有分区）
输入 n 新建分区
输入 p 为建立主分区（此时的p是在n后的，不是打印）
输入 3 为建立第三个主分区，前面已经建立了2个分区了
分区起始位置可以直接回车，默认是1
默认回车则为全部，（分区最后位置为 650（因为每个柱面约8M，650柱面约是5G，本实验只用5G，剩余的做增加LV实验用））
输入 p 打印分区情况，发现已建立一个分区 /dev/sda3，但是 此分区为 Linux 格式

改变系统标识符
输入 t 改变分区3的属性
输入 L 查看有个属性对应的命令
输入 8e 改变分区1为 Linux LVM格式
输入 p 打印分区情况，发现建立的分区 /dev/sdb1 为 Linux LVM 格式
输入 w 保存创建的分区
```

## 38.1. 查看磁盘分区是否创建成功

``` shell
fdisk -l
```

## 38.2. 重启不需要reboot

使kernel重新读取分区表即可

``` shell
partprobe
```

## 38.3. 分区后 创建物理卷，创建卷组，创建逻辑卷，逻辑卷挂盘

特别注意：Rhel7已经有卷组了和逻辑卷等全部都有只是扩展，这里就不创建了

### 38.3.1. 创建物理卷

扫面系统PV：pvscan
创建物理卷PV
pvcreate /dev/sdb3
查看新添加的物理卷PV
pvdisplay

### 38.3.2. 创建物理卷组

vgscan
vgcreate vg_test /dev/sda3
vgdisplay

### 38.3.3. 创建逻辑卷

扫面系统LV：lvscan
创建LV：lvcreate -l 1274 -n lv_test vg_test（1274是VG中PE的个数）

创建逻辑卷根据物理卷100磁盘空间创建
lvcreate -l 100%VG -n lv_data vg_data

示例：
（1）创建一个指定大小的lv，并指定名字为lv_2
lvcreate -L 2G -n lv_2 vg_1
（2）创建一个占全部卷组大小的lv，并指定名字为lv_3（注意前提是vg并没有创建有lv）
lvcreate -l 100%VG -n lv_3 vg_1
（3）创建一个空闲空间80%大小的lv，并指定名字为lv_4(常用)
lvcreate -l 80%Free -n lv_4 vg_1

查看LV：lvdisplay
格式化刚刚创建的LV:
mkfs -t ext4 /dev/vg_test/lv_test

mkfs -t ext4 /dev/vg_data/lv_data

### 38.3.4. 创建目录并挂载

创建目录：mkdir /test
挂载逻辑卷到目录：mount /dev/vg_test/lv_test /test
mount /dev/vg_data/lv_data /data
查看：df -h

设置开机挂载：
/dev/mapper/vg_test-lv_test /test     ext4    defaults        1 2
 写入 /etc/fstab

tbds则需要修改为
/dev/mapper/vg_data-lv_data /data ext4 defaults, noatime 0 0

可执行以下命令应用修改的配置而无需重启系统:
mount -o remount /data

### 38.3.5. 对分区进行格式化

mkfs.ext4 /dev/sda3

### 38.3.6. 将物理卷加入到卷组

查看卷组信息
vgdisplay
返回结果
VG Name               rhel

### 38.3.7. 添加物理卷【/dev/sda3】到卷组【rhel】

vgextend rhel /dev/sda3

### 38.3.8. 添加成功，通过下面命令确认sdb1所属的卷组。

pvdisplay

## 38.4. 可选, 取消挂载:

取消挂载
umount /data
取消逻辑卷
lvremove /dev/vg_data/lv_data
取消卷组
vgremove vg_data
取消物理卷
pvremove /dev/sdb3
取消/etc/fstab配置文件中对应的挂载信息

### 38.4.1. 扩充逻辑卷的容量

先通过下面命令查看系统里有哪些逻辑卷。
lvdisplay
df -h
逻辑卷【/dev/rhel/root】挂在于根目录上，这个逻辑卷就是本次要扩充的对象
可以看到逻辑卷【/dev/rhel/root】和文件系统【/dev/mapper/rhel-root】的路径有所不同，但他们都对应相同的设备。

### 38.4.2. 通过下面的命令，扩充【/dev/rhel/root】的容量。

lvextend -L +40G /dev/rhel/root

### 38.4.3. 现在逻辑卷的容量扩充了，但是它对应的文件系统的容量还是原来的大小。

df -h

### 38.4.4. 需要通过下面的命令，扩充文件系统的大小。

xfs_growfs /dev/rhel/root

### 38.4.5. 查看文件系统，显示的空间增大了。

df -h

# 39. Dnsmasq搭建本地dns服务器上网

## 39.1. 、安装并启动Dnsmasq

yum remove dnsmasq
yum install -y dnsmasq
service dnsmasq start (rhle7 命令为：/bin/systemctl start  dnsmasq.service)

## 39.2. 、Dnsmasq配置

2.1 查看dnsmasq的【配置路径】
ll -d /etc/dnsmasq.conf 

### 39.2.1. 编辑vim /etc/dnsmasq.conf （conf-dir=/etc/dnsmasq.d）

【rhel 6】方式

``` 
resolv-file=/etc/resolv.dnsmasq.conf
strict-order
addn-hosts=/etc/dnsmasq.hosts
listen-address=127.0.0.1,192.168.1.244
```

【rhel 7】方式
vim /etc/dnsmasq.conf

``` 
listen-address=192.168.1.244,127.0.0.1
cache-size=10000
conf-dir=/etc/dnsmasq.d,.rpmnew,.rpmsave,.rpmorig
```

备注：resolv-file ---- /dnsmasq 会从这个文件中寻找上游dns服务器
strict-order  --- //去掉前面的#
addn-hosts --- //在这个目里面添加记录
listen-address ---//监听地址

### 39.2.2. 同步到其他节点

``` sh
rsync -av /etc/dnsmasq.conf hadoop02:/etc/
rsync -av /etc/dnsmasq.conf hadoop03:/etc/
rsync -av /etc/dnsmasq.conf hadoop04:/etc/
```

### 39.2.3. 修改/etc/resolv.conf

``` sh
echo 'nameserver 127.0.0.1' > /etc/resolv.conf
```

### 39.2.4. 创建resolv.dnsmasq.conf文件并添加上游dns服务器的地址

``` sh
touch /etc/resolv.dnsmasq.conf
echo 'nameserver 119.29.29.29' > /etc/resolv.dnsmasq.conf
```

提示：resolv.dnsmasq.conf中设置的是真正的Nameserver，可以用电信、联通等公共的DNS

### 39.2.5. 创建dnsmasq.hosts文件

``` sh
cp /etc/hosts /etc/dnsmasq.hosts
echo 'addn-hosts=/etc/dnsmasq.hosts' >> /etc/dnsmasq.conf
```

3. Dnsmasq启动

3.1 设置Dnsmasq开机启动并启动Dnsmasq服务
开机启动

``` sh
chkconfig dnsmasq on (rhle7 命令：systemctl enable dnsmasq)
```

启动Dnsmasq服务

``` sh
/etc/init.d/dnsmasq restart （(rhle7 命令为：systemctl restart  dnsmasq.service)）
```

3.2 查看Dnsmasq是否正常启动：

``` sh
netstat -tunlp|grep 53
```

返回下面信息：

``` sh
tcp        0      0 0.0.0.0:53                  0.0.0.0:*                   LISTEN      2491/dnsmasq        
tcp        0      0 :::53                       :::*                        LISTEN      2491/dnsmasq        
udp        0      0 0.0.0.0:53                  0.0.0.0:*                               2491/dnsmasq        
udp        0      0 :::53                       :::*                                    2491/dnsmasq
```        

3.3 测试
dig www.freehao123.com
第一次是没有缓存，所以时间是200多
第二次再次测试，因为已经有了缓存，所以查询时间已经变成了0.

3.4 域名检查
```sh
nslookup hadoop01.sinobest.com

root@hadoop01 ~]# nslookup hadoop01.sinobest.com
Server:		127.0.0.1
Address:	127.0.0.1#53

Name:	hadoop01.sinobest.com
Address: 192.168.18.201

[root@hadoop01 ~]# 
```

# 40. linux用户-相关命令

## 40.1. Linux里查看所有用户

　　(1)在终端里. 其实只需要查看 /etc/passwd文件就行了.
　　(2)看第三个参数:500以上的, 就是后面建的用户了. 其它则为系统的用户.
或者用cat /etc/passwd |cut -f 1 -d :

## 40.2. 用户管理命令

useradd 注：添加用户
adduser 注：添加用户
passwd 注：为用户设置密码
userdel user名 注：删除用户
groupdel user组名 注：删除用户组
usermod 注：修改用户命令，可以通过usermod 来修改登录名、用户的家目录等等; 
pwcov 注：同步用户从/etc/passwd 到/etc/shadow
pwck 注：pwck是校验用户配置文件/etc/passwd 和/etc/shadow 文件内容是否合法或完整; 
pwunconv 注：是pwcov 的立逆向操作，是从/etc/shadow和 /etc/passwd 创建/etc/passwd ，然后会删除 /etc/shadow 文件; 
finger 注：查看用户信息工具
id 注：查看用户的UID、GID及所归属的用户组
chfn 注：更改用户信息工具
su 注：用户切换工具
sudo 注：sudo 是通过另一个用户来执行命令(execute a command as another user)，su 是用来切换用户，然后通过切换到的用户来完成相应的任务，但sudo 能后面直接执行命令，比如sudo 不需要root 密码就可以执行root 赋与的执行只有root才能执行相应的命令; 但得通过visudo 来编辑/etc/sudoers来实现; 
visudo 注：visodo 是编辑 /etc/sudoers 的命令; 也可以不用这个命令，直接用vi 来编辑 /etc/sudoers 的效果是一样的; 
sudoedit 注：和sudo 功能差不多; 

## 40.3. 管理用户组(group)的工具或命令; 

groupadd 注：添加用户组; 
groupdel 注：删除用户组; 
groupmod 注：修改用户组信息
groups 注：显示用户所属的用户组
grpck
grpconv 注：通过/etc/group和/etc/gshadow 的文件内容来同步或创建/etc/gshadow ，如果/etc/gshadow 不存在则创建; 
grpunconv 注：通过/etc/group 和/etc/gshadow 文件内容来同步或创建/etc/group ，然后删除gshadow文件

给已有的用户增加工作组
usermod -G groupname username  （这个会把用户从其他组中去掉）
从组中删除用户
编辑/etc/group 找到GROUP1那一行，删除 A
或者用命令
gpasswd -d userA GROUP

## 40.4. 异常问题 useradd：hadoop 组已经存在 - 如果您想将此用户加入到该组，请使用 -g 参数

  解决办法:
  useradd hadoop -g hadoop

## 40.5. 异常问题：bash: /home/hadoop/.bashrc: 权限不够

  解决办法：
  chown -R hadoop:hadoop /home/hadoop
  
listen_address 设置本机IP值
192.168.10.30
native_transport_address 设置本机IP以外的值192.168.10.80
[lifecycle_manager]

## 40.6. 平台检查确保DSE的兼容性和支持。在自己的风险下禁用。默认值: False

disable_platform_check = True

Lifecycle Manager目前不支持管理DSE多实例节点

RetHad7  httpd 安装

## 40.7. 将下面下载好的rpm包拷贝到节点的/tmp/httpd上

apr-1.4.8-3.el7.x86_64
apr-util-1.5.2-6.el7.x86_64
httpd-tools-2.4.6-40.el7.x86_64
mailcap-2.1.41-2.el7.noarch 
httpd-2.4.6-40.el7.x86_64

## 40.8. httpd安装

``` shell
cd /tmp/httpd
rpm -ivh httpd-2.4.6-40.el7.x86_64.rpm httpd-tools-2.4.6-40.el7.x86_64.rpm
```

## 40.9. 创建自己需要的rmp源地址，并授权

比如test，可以多级，如/test/erj
$ mkdir /var/http/html/test
$ chmod -R ugo+rx /var/http/html/test

$ mkdir /var/http/html/test/erj
$chmod -R ugo+rx /var/http/html/test/erj

## 40.10. 启动httpd（注意关闭防火墙）

service httpd start

## 40.11. 检查启动状态

service httpd status

## 40.12. Web浏览器访问, 如果能访问成功那么HTTPD安装就已经成功了

http://id/test

## 40.13. 设置开机启动

el6:
chkconfig httpd on 

el7:
systemctl enable httpd.service 
如果此方式不行 进行下面命令设置开机启动
chkconfig httpd on

## 40.14. 检查开机启动

el7:
systemctl status httpd

# 41. 自定义开机启动服务

vim /etc/init.d/dse

``` shell
#!/bin/sh
#
# dse start/stop dse
#
# chkconfig: 2345 85 15
# description: start dse/dse at boot time

start(){
        echo "start dse"
		su cassandra
        dse cassandra -k -g -s
		exit 0;
}
stop(){
        echo "stop dse"
		su cassandra
        dse cassandra -k -g -s
		exit 0;
}

case "$1" in
start)
        start
        ;;
stop)
        stop
        ;;
*)
        echo "Usage: $0 {start|stop}"
        exit 1
        ;;
esac
```

开机启动

``` shell
service shadowsocks start
```

# 42. Shell编程

## 42.1. Hello World

打印信息，相当于System.out.println("Hello World"); ; 

``` 
#!/bin/bash
echo "Hello World !"
```

## 42.2. 变量

### 42.2.1. 定义变量

定义变量时，变量名不加美元符号（$，PHP语言中变量需要），如：

``` shell
your_name="runoob.com"
```

注意，变量名和等号之间不能有空格，这可能和你熟悉的所有编程语言都不一样。同时，变量名的命名须遵循如下规则：

  + 命名只能使用英文字母，数字和下划线，首个字符不能以数字开头。
  + 中间不能有空格，可以使用下划线（_）。
  + 不能使用标点符号。

  -不能使用bash里的关键字（可用help命令查看保留关键字）。

### 42.2.2. 使用变量, 通过 $ 加变量名 使用变量

``` shell
#!/bin/bash
your_name="qinjx"
echo $your_name
echo ${your_name}
```

加花括号是为了帮助解释器识别变量的边界，比如下面这种情况

``` shell
for skill in Ada Coffe Action Java; do
    echo "I am good at ${skill}Script"
done
```

推荐给所有变量加上花括号，这是个好的编程习惯。 

已定义的变量，可以被重新定义，如：

``` shell
#!/bin/bash

your_name="tom"
echo $your_name
your_name="alibaba"
echo $your_name
```

注意，第二次赋值的时候不能写$your_name="alibaba"，使用变量的时候才加美元符（$）。 

### 42.2.3. 只读变量

使用 readonly 命令可以将变量定义为只读变量，只读变量的值不能被改变。
下面的例子尝试更改只读变量，结果报错：

``` shell
#!/bin/bash
myUrl="http://www.w3cschool.cc"
readonly myUrl
myUrl="http://www.runoob.com"
```

报错信息：

``` 
[root@dse02 1.0.0]# ./renamefile.sh 
./renamefile.sh: line 4: myUrl: readonly variable
```

### 42.2.4. 删除变量

使用 unset 命令可以删除变量。语法：
unset variable_name

变量被删除后不能再次使用。unset 命令不能删除只读变量。

实例

``` shell
#!/bin/sh
myUrl="http://www.runoob.com"
unset myUrl
echo $myUrl
```

### 42.2.5. 获取变量字符串长度

``` shell
#!/bin/sh
string="abcd"
echo ${#string} #输出 4
```

### 42.2.6. 提取子字符串

以下实例从字符串第 2 个字符开始截取 4 个字符：

``` shell
#!/bin/sh
string="runoob is a great site"
echo ${string:1:4} # 输出 unoo
```

## 42.3. Shell 数组

bash支持一维数组（不支持多维数组），并且没有限定数组的大小。
类似与C语言，数组元素的下标由0开始编号。获取数组中的元素要利用下标，下标可以是整数或算术表达式，其值应大于或等于0。 

### 42.3.1. 定义数组

在Shell中，用括号来表示数组，数组元素用"空格"符号分割开。定义数组的一般形式为：
数组名=(值1 值2 ... 值n)

 例如：

array_name=(value0 value1 value2 value3)

或者
array_name=(
value0
value1
value2
value3
)

还可以单独定义数组的各个分量：
array_name[0]=value0
array_name[1]=value1
array_name[n]=valuen
可以不使用连续的下标，而且下标的范围没有限制。 

### 42.3.2. 读取数组

读取数组元素值的一般格式是：

${数组名[下标]}

例如：
valuen=${array_name[n]}

使用@符号可以获取数组中的所有元素，例如：
echo ${array_name[@]}

### 42.3.3. 获取数组的长度 

获取数组长度的方法与获取字符串长度的方法相同，例如：

``` shell
# 取得数组元素的个数
length=${#array_name[@]}
# 或者
length=${#array_name[*]}
# 取得数组单个元素的长度
lengthn=${#array_name[n]}
```

## 42.4. Shell 传递参数

我们可以在执行 Shell 脚本时，向脚本传递参数，脚本内获取参数的格式为：$n。n 代表一个数字，1 为执行脚本的第一个参数，2 为执行脚本的第二个参数，以此类推……

实例：
以下实例我们向脚本传递三个参数，并分别输出，其中 $0 为执行的文件名：

``` 
#!/bin/bash
# author:liulv
# url:

echo "Shell 传递参数实例！";
echo "执行的文件名：$0";
echo "第一个参数为：$1";
echo "第二个参数为：$2";
echo "第三个参数为：$3";
```

为脚本设置可执行权限，并执行脚本，输出结果如下所示：

``` 
$ chmod +x test.sh 
$ ./test.sh 1 2 3
Shell 传递参数实例！
执行的文件名：./test.sh
第一个参数为：1
第二个参数为：2
第三个参数为：3
```

另外，还有几个特殊字符用来处理参数：

``` 
| 参数处理  |  说明 |
| ------------ | ------------ |
| $#  | 传递到脚本的参数个数 |
| $*  | 以一个单字符串显示所有向脚本传递的参数。如"$*"用「"」括起来的情况、以"$1 $2 … $n"的形式输出所有参数。  |
| $$  | 脚本运行的当前进程ID号  |
| $!  | 后台运行的最后一个进程的ID号  |
| $@  | 与$*相同，但是使用时加引号，并在引号中返回每个参数。如"$@"用「"」括起来的情况、以"$1" "$2" … "$n" 的形式输出所有参数。   |
| $-  | 显示Shell使用的当前选项，与set命令功能相同。  |
| $?  | 显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。  |
```

``` shell
#!/bin/bash
# author:liulv
# url:

echo "Shell 传递参数实例！";
echo "第一个参数为：$1";

echo "参数个数为：$#";
echo "传递的参数作为一个字符串显示：$*";
```

$* 与 $@ 区别：

  相同点：都是引用所有参数。
  不同点：只有在双引号中体现出来。假设在脚本运行时写了三个参数 1、2、3，，则 " * " 等价于 "1 2 3"（传递了一个参数），而 "@" 等价于 "1" "2" "3"（传递了三个参数）。

``` shell
#!/bin/bash
# author:liulv
# url:

echo "-- \$* 演示 ---"
for i in "$*"; do
    echo $i
done

echo "-- \$@ 演示 ---"
for i in "$@"; do
    echo $i
done
```

## 42.5. Shell运算符

原生bash不支持简单的数学运算，但是可以通过其他命令来实现，例如 awk 和 expr，expr 最常用。
expr 是一款表达式计算工具，使用它能完成表达式的求值操作。 

例如，两个数相加(注意使用的是反引号 ` 而不是单引号 ')：

``` shell
#!/bin/bash
val= `expr 2 + 2`
echo "两数之和为 : $val"
```

执行脚本，输出结果如下所示：

``` 
两数之和为 : 4
```

两点注意：

  + 表达式和运算符之间要有空格，例如 2+2 是不对的，必须写成 2 + 2，这与我们熟悉的大多数编程语言不一样。
  + 完整的表达式要被 ` ` 包含，注意这个字符不是常用的单引号，在 Esc 键下边。

### 42.5.1. 关系运算符

下表列出了常用的算术运算符，假定变量 a 为 10，变量 b 为 20：
运算符   说明  举例

*   加法 `expr $a + $b` 结果为 30。
*   减法 `expr $a - $b` 结果为 -10。
*   乘法 `expr $a \* $b` 结果为  200。

/   除法 `expr $b / $a` 结果为 2。
%   取余 `expr $b % $a` 结果为 0。
=   赋值  a=$b 将把变量 b 的值赋给 a。
==  相等。用于比较两个数字，相同则返回 true。   [ $a == $b ] 返回 false。
!=  不相等。用于比较两个数字，不相同则返回 true。   [ $a != $b ] 返回 true。

注意：条件表达式要放在方括号之间，并且要有空格，例如: [$a==$b] 是错误的，必须写成 [ $a == $b ]。

实例

算术运算符实例如下:

``` shell
#!/bin/bash
a=10
b=20

val= `expr $a + $b`
echo "a + b : $val"

val= `expr $a - $b`
echo "a - b : $val"

val= `expr $a \* $b`
echo "a * b : $val"

val= `expr $b / $a`
echo "b / a : $val"

val= `expr $b % $a`
echo "b % a : $val"

if [ $a == $b ]
then
   echo "a 等于 b"
fi
if [ $a != $b ]
then
   echo "a 不等于 b"
fi
```

### 42.5.2. 布尔运算符

下表列出了常用的布尔运算符，假定变量 a 为 10，变量 b 为 20：

运算符   说明  举例
!   非运算，表达式为 true 则返回 false，否则返回 true。  [ ! false ] 返回 true。
-o  或运算，有一个表达式为 true 则返回 true。  [ $a -lt 20 -o $b -gt 100 ] 返回 true。
-a  与运算，两个表达式都为 true 才返回 true。  [ $a -lt 20 -a $b -gt 100 ] 返回 false。

``` shell
a=10
b=20

if [ $a != $b ]
then
   echo "$a != $b : a 不等于 b"
else
   echo "$a != $b: a 等于 b"
fi
if [ $a -lt 100 -a $b -gt 15 ]
then
   echo "$a 小于 100 且 $b 大于 15 : 返回 true"
else
   echo "$a 小于 100 且 $b 大于 15 : 返回 false"
fi
if [ $a -lt 100 -o $b -gt 100 ]
then
   echo "$a 小于 100 或 $b 大于 100 : 返回 true"
else
   echo "$a 小于 100 或 $b 大于 100 : 返回 false"
fi
if [ $a -lt 5 -o $b -gt 100 ]
then
   echo "$a 小于 5 或 $b 大于 100 : 返回 true"
else
   echo "$a 小于 5 或 $b 大于 100 : 返回 false"
fi
```

### 42.5.3. 逻辑运算符

以下介绍 Shell 的逻辑运算符，假定变量 a 为 10，变量 b 为 20:
运算符   说明  举例
&&  逻辑的 AND   [[ $a -lt 100 && $b -gt 100 ]] 返回 false
||  逻辑的 OR  [[ $a -lt 100 || $b -gt 100 ]] 返回 true

### 42.5.4. 字符串运算符

下表列出了常用的字符串运算符，假定变量 a 为 "abc"，变量 b 为 "efg"：
运算符   说明  举例
=   检测两个字符串是否相等，相等返回 true。  [ $a = $b ] 返回 false。
!=  检测两个字符串是否相等，不相等返回 true。   [ $a != $b ] 返回 true。
-z  检测字符串长度是否为0，为0返回 true。  [ -z $a ] 返回 false。
-n  检测字符串长度是否为0，不为0返回 true。   [ -n "$a" ] 返回 true。
str   检测字符串是否为空，不为空返回 true。   [ $a ] 返回 true。

### 42.5.5. 文件测试运算符

文件测试运算符用于检测 Unix 文件的各种属性。

属性检测描述如下：
操作符   说明  举例
-b file   检测文件是否是块设备文件，如果是，则返回 true。  [ -b $file ] 返回 false。
-c file   检测文件是否是字符设备文件，如果是，则返回 true。   [ -c $file ] 返回 false。
-d file   检测文件是否是目录，如果是，则返回 true。   [ -d $file ] 返回 false。
-f file   检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。  [ -f $file ] 返回 true。
-g file   检测文件是否设置了 SGID 位，如果是，则返回 true。  [ -g $file ] 返回 false。
-k file   检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。  [ -k $file ] 返回 false。
-p file   检测文件是否是有名管道，如果是，则返回 true。   [ -p $file ] 返回 false。
-u file   检测文件是否设置了 SUID 位，如果是，则返回 true。  [ -u $file ] 返回 false。
-r file   检测文件是否可读，如果是，则返回 true。  [ -r $file ] 返回 true。
-w file   检测文件是否可写，如果是，则返回 true。  [ -w $file ] 返回 true。
-x file   检测文件是否可执行，如果是，则返回 true。   [ -x $file ] 返回 true。
-s file   检测文件是否为空（文件大小是否大于0），不为空返回 true。   [ -s $file ] 返回 true。
-e file   检测文件（包括目录）是否存在，如果是，则返回 true。  [ -e $file ] 返回 true。

## 42.6. Linux压缩Zip和解压Zip

**压缩服务器上当前目录的内容为xxx.zip文件**

zip -r xxx.zip ./*

将当前目录下的所有文件和文件夹全部压缩成myfile.zip文件, －r表示递归压缩子目录下所有文件.

**其他**
  
zip -d myfile.zip smart.txt
删除压缩文件中smart.txt文件
zip -m myfile.zip ./rpm_info.txt
向压缩文件中myfile.zip中添加rpm_info.txt文件

**解压zip文件到当前目录**

unzip filename.zip

* unzip
unzip -o -d /home/sunny myfile.zip
把myfile.zip文件解压到 /home/sunny/
-o: 不提示的情况下覆盖文件；
-d:-d /home/sunny 指明将文件解压缩到/home/sunny目录下；

**有些服务器没有安装zip包执行不了zip和unzip命令，可使用yum安装**

```powershell
yum list | grep zip/unzip 
yum install zip
yum install unzip
```

## 42.7. linux压缩和解压缩命令

tar
  解包：tar zxvf filename.tar
  打包：tar -czvf rpm.tar rpm
zip命令
  解压：unzip filename.zip
  压缩：zip filename.zip dirname
gz命令
  解压1：gunzip filename.gz
  解压2：gzip -d filename.gz
  压缩：gzip filename
   .tar.gz 和  .tgz
   解压：tar zxvf filename.tar.gz
   压缩：tar zcvf filename.tar.gz dirname
   压缩多个文件：tar zcvf filename.tar.gz dirname1 dirname2 dirname3.....
bz2命令
  解压1：bzip2 -d filename.bz2
  解压2：bunzip2 filename.bz2
  压缩：bzip2 -z filename.tar.bz2
  解压：tar jxvf filename.tar.bz2
  压缩：tar jcvf filename.tar.bz2 dirname
bz命令
   解压1：bzip2 -d filename.bz
   解压2：bunzip2 filename.bz
      .tar.bz
      解压：tar jxvf filename.tar.bz

z命令
   解压：uncompress filename.z
   压缩：compress filename
      .tar.z
         解压：tar zxvf filename.tar.z
         压缩：tar zcvf filename.tar.z dirname
