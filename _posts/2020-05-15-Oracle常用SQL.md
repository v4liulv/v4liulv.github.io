---
title: Oracle常用SQL
tags: 数据库
---

## Oracle 创建用流程
```sql
CREATE TEMPORARY TABLESPACE PCS_JCGLPT_tmp
         TEMPFILE 'd:\oracle\app\oradata\gz\PCS_JCGLPT_tmp.dbf'
         SIZE 32m
         AUTOEXTEND ON 
         NEXT  32m
         EXTENT MANAGEMENT LOCAL;
         
CREATE TABLESPACE PCS_JKPT_DATA
         LOGGING
         DATAFILE 'D:\Oracle\app\oradata\gz\PCS_JKPT_DATA.dbf'
         SIZE 32M
         AUTOEXTEND ON
         NEXT 32M MAXSIZE UNLIMITED
         EXTENT MANAGEMENT LOCAL;

         
---CREATE USER pcs_sd_jzcq_pzk IDENTIFIED BY pcs_sd_jzcq_pzk;

---DROP USER pcs_sd_jzcq_pzk;
```

或者用户提示删除正在使用，那么通过下面方式删除
```sql
select username,sid,serial## from v$session; 
alter system kill session '66,294'; --66为删除用户的id,294为serial#的值
DROP USER pcs_yn_hiveIncrupdate_pzk cascade;


CREATE USER PCS_JCGLPT IDENTIFIED BY pcs_jcglpt
         ACCOUNT UNLOCK
         DEFAULT TABLESPACE PCS_JCGLPT
         TEMPORARY TABLESPACE PCS_JCGLPT_tmp;
         
GRANT CONNECT, RESOURCE TO PCS_JCGLPT;
GRANT UNLIMITED TABLESPACE to PCS_JCGLPT; --使用表空间
GRANT CREATE TABLE TO PCS_JCGLPT;--建表
GRANT DROP TABLE TO PCS_JCGLPT;
GRANT INSERT TABLE TO PCS_JCGLPT;
GRANT UPDATE TABLE TO PCS_JCGLPT;
GRANT ALTER SESSION TO PCS_JCGLPT;--修改会话
GRANT CREATE CLUSTER TO PCS_JCGLPT;--建立聚簇
GRANT CREATE DATABASE LINK TO PCS_JCGLPT;--建立数据库链接
GRANT CREATE SEQUENCE TO PCS_JCGLPT;--建立序列
GRANT CREATE SESSION TO PCS_JCGLPT;--建立会话
GRANT CREATE SYNONYM TO PCS_JCGLPT;--建立同义词
GRANT CREATE VIEW TO PCS_JCGLPT;--建立视图
GRANT CREATE CLUSTER TO PCS_JCGLPT;--建立聚簇
GRANT CREATE PROCEDURE TO PCS_JCGLPT; --建立过程
GRANT CREATE SEQUENCE TO PCS_JCGLPT; --建立序列
GRANT CREATE TRIGGER TO PCS_JCGLPT;--建立触发器
GRANT CREATE TYPE TO PCS_JCGLPT; --建立类型
```

## 用户被锁定，解决办法
```sql
alter user pcs_yn_hiveIncrupdate_pzk account unlock; 
```

## 创建数据库
- 开始 -> 程序 —>Oracle-Oracle11g_home1—>配置和移植工具—>Database Configuration Assistant命令，启动DBCA，出现“欢迎使用”窗口，
--> 点击下一步
>- 步骤1 创建数据库 点击下一步
>- 步骤2 数据库模版，默认使用一般用途和事务处理，点击下一步
>- 步骤3 数据库标识，并输入全局数据库名：dse.cs.sinobest，SID：dse
--> 后面全部下一步，然后完成

## 删除数据库
开始”—>“程序”—>Oracle-Oracle11g_home1—>配置和移植工具—>Database Configuration Assistant命令，启动DBCA，出现“欢迎使用”窗口，
--> 点击下一步
>- 步骤1 选择删除数据库 点击下一步
>- 步骤2 选择需要删除的数据库 点击下一步

## 创建表空间
> 1 . 创建用户之前要创建"临时表空间"，若不创建则默认的临时表空间为temp
**语法：**
```sql
CREATE TEMPORARY TABLESPACE db_temp
         TEMPFILE 'D:appAdministratororadataNewDBDB_TEMP.DBF'
         SIZE 32M
         AUTOEXTEND ON
         NEXT 32M MASIZE UNLIMITED
         EXTENT MANAGEMENT LOCAL;
```


**例子：**
```sql
CREATE TEMPORARY TABLESPACE pcs_editor_tmp
         TEMPFILE 'd:\oracle\app\oradata\basic\pcs_editor_tpm.dbf'
         SIZE 32m
         AUTOEXTEND on
         NEXT 32m
         EXTENT MANAGEMENT local;
```

```sql
CREATE TEMPORARY TABLESPACE pcs_zstp_tmp
         TEMPFILE 'd:\oracle\app\oradata\zstp\pcs_zstp_tmp.dbf'
         SIZE 32m
         AUTOEXTEND ON 
         NEXT 32m
         EXTENT MANAGEMENT LOCAL;
```

> 2. 创建用户之前先要创建数据表空间，若没有创建则默认永久性表空间是system。
**语法：**
```sql
CREATE TABLESPACE db_data
         LOGGING
         DATAFILE 'D:appAdministratororadataNewDBDB_DATA.DBF'
         SIZE 32M
         AUTOEXTEND ON
         NEXT 32M MAXSIZE UNLIMITED
         EXTENT MANAGEMENT LOCAL;
```

**例：**
```sql
CREATE TABLESPACE pcs_editor_data
         LOGGING
         DATAFILE 'D:\Oracle\app\oradata\basic\pcs_editor_data.dbf'
         SIZE 32M
         AUTOEXTEND ON
         NEXT 32M MAXSIZE UNLIMITED
         EXTENT MANAGEMENT LOCAL;
```

其中**DB_DATA**和**DB_TEMP**是你自定义的数据表空间名称和临时表空间名称，可以任意取名；**D:appAdministratororadataNewDBDB_DATA.DBF**是数据文件的存放位置，**DB_DATA.DBF**文件名也是任意取；**size 32M**是指定该数据文件的大小，也就是表空间的大小。


```sql
CREATE TABLESPACE pcs_zstp_data
         LOGGING
         DATAFILE 'D:\Oracle\app\oradata\zstp\pcs_zstp_data.DBF'
         SIZE 32M
         AUTOEXTEND ON
         NEXT 32M MAXSIZE UNLIMITED
         EXTENT MANAGEMENT LOCAL;
```

## 创建修改用户

oracle 内部有两个建好的用户：system 和 sys。用户可直接登录到 system 用户以创建其他用户，因为system具有创建别 的用户的 权限。 在安装 oracle 时，用户或系统管理员首先可以为自己建立一个用户。

管理员 sys/123456 登录。

语法[创建用户]： create user 用户名 identified by 口令[即密码]；
例子使用默认表空间： 
```sql
CREATE USER test IDENTIFIED BY test;
CREATE USER yy_editor_csb IDENTIFIED BY 123456;
```
例子使用已经创建的表空间
```sql
CREATE USER yy_editor_csb IDENTIFIED BY 123456
         ACCOUNT UNLOCK
         DEFAULT TABLESPACE pcs_editor_data
         TEMPORARY TABLESPACE pcs_editor_tmp;
```


语法[更改用户]: alter user 用户名 identified by 口令[改变的口令];
例子： 
```sql
ALTER USER test IDENTIFIED BY 123456;
```

## 删除用户

语法：drop user 用户名;
例子：
```sql
DROP USER yy_editor_csb;
```
若用户拥有对象，则不能直接删除，否则将返回一个错误值。指定关键字cascade,可删除用户所有的对象，然后再删除用户。
语法： drop user 用户名 cascade;
例子：  删除test用户对象
```sql
DROP USER test cascade;
```

## 授权角色
oracle为兼容以前版本，提供三种标准角色（role）:connect/resource和dba.

### 三种标准角色
**connect role(连接角色)**
--临时用户，特指不需要建表的用户，通常只赋予他们connect role. 
--connect是使用oracle简单权限，这种权限只对其他用户的表有访问权限，包括select/insert/update和delete等。
--拥有connect role 的用户还能够创建表、视图、序列（sequence）、簇（cluster）、同义词(synonym)、回话（session）和其他  数据的链（link）。

**resource role(资源角色)**
--更可靠和正式的数据库用户可以授予resource role。
--resource提供给用户另外的权限以创建他们自己的表、序列（sequence）、过程(procedure)、触发器(trigger)、索引(index)和簇(cluster)。

**dba role(数据库管理员角色)**
--dba role拥有所有的系统权限
--包括无限制的空间限额和给其他用户授予各种权限的能力。system由dba用户拥有。

### 授权命令
语法：grant connect, resource to 用户名;
例子授权connect和resource权限：
```sql 
GRANT CONNECT, RESOURCE TO test;
GRANT CONNECT, RESOURCE TO yy_editor_csb;
```
例子授权DBA权限：
```sql
GRANT DBA TO test
```

### 授权某个用户访问另外用户权限
//把JBQDEDI读写权限 授权给JBQD
```sql
select 'Grant all on '||table_name||'to JBQD ;' from all_tables
where owner = upper('JBQDEDI');
```

//把当前登陆的用户表查询权限授权给b用户
```sql
select 'GRANT SELECT ON '||table_name||' to b;'  from user_tables
```
得到所有表的授权命名，如果需要授权那个表复制进行执行授权即可，如：
```sql
GRANT SELECT ON b_editor_abstract TO yy_editor_csb;
```


### 撤销权限
语法： revoke connect, resource from 用户名;
列子： 
```sql
REVOKE CONNECT, RESOURCE FROM test;
```

## 创建/授权/删除角色

除了前面讲到的三种系统角色----connect、resource和dba，用户还可以在oracle创建自己的role。用户创建的role可以由表或系统权限或两者的组合构成。为了创建role，用户必须具有create role系统权限。

### 1》创建角色

语法： CREATE ROLE 角色名;

例子： CREATE ROLE testRole;

2》授权角色

语法： GRANT SELECT ON CLASS to 角色名;

列子： GRANT SELECT ON CLASS TO testRole;

注：现在，拥有testRole角色的所有用户都具有对class表的select查询权限

3》删除角色

语法： DROP ROLE 角色名;

例子： DROP ROLE testRole;

注：与testRole角色相关的权限将从数据库全部删除

## 复制表数据
语法：insert into 插入表 select * from 复制来源表
例子：
```sql
INSERT INTO b_editor_abstract SELECT * FROM yy_gxksh_zsb.b_editor_abstract;
```

## 函数日期格式转换

```sql
CREATE OR REPLACE PACKAGE pkg_otou AS
/******************************************************************************
   NAME:       pkg_otou
   PURPOSE:

   REVISIONS:
   Ver        Date        Author           Description
   ---------  ----------  ---------------  ------------------------------------
   1.0        11/19/2014      defu       1. Created this package.
******************************************************************************/
  --传入日期型，返回unix型日期
  function otou(in_date IN DATE) return varchar2;
  --传入字符型日期，格式：yyyymmddhh24miss，返回unix型日期
  function otou(in_date IN varchar2) return varchar2;
   --传入数字型日期，格式：yyyymmddhh24miss，返回unix型日期
  function otou(in_date IN number) return varchar2;
    --传入字符型日期，格式：yyyy-mm-dd hh24:mi:ss，返回unix型日期
  function otou2(in_date IN varchar2) return varchar2;

  ----例子
  /* SELECT pkg_otou.otou(sysdate),
        pkg_otou.otou2('2014-11-10 10:01:02') ,
        pkg_otou.otou(20141110100102),
        pkg_otou.otou('20141110100102')
   FROM DUAL;*/

END pkg_otou;
```

## 触发器

### 检测某个表如果更新，则更新另外一个表
```sql
CREATE OR REPLACE TRIGGER tr_update_pkey_index
  --INSERT数据库插入会触发此触发器，after表示在数据库动作之后触发器执行;
AFTER UPDATE ON b_titan_propertykey
  --for each row：对表的每一行触发器执行一次。如果没有这一选项，则只对整个表执行一次。
FOR EACH ROW
  BEGIN
    --复制来源的表值可以使用 :new.字段名称
    UPDATE B_TITAN_INDEX SET
      PKEYS = :new.PKEY_NAME,
      INDEX_LABEL = :new.PKEY_LABEL,
      INDEX_LABEL_TYPE = :new.PKEY_TYPE,
      ISUNIQ = :new.PKEY_INDEX_UNIQ
    WHERE INDEX_NAME = :new.PKEY_NAME;
  END;
```

### 检测某个表如果插入，则插入另外一个表
```sql
  CREATE OR REPLACE TRIGGER TR_COPY_PKEY_INDEX
  --INSERT数据库插入会触发此触发器，after表示在数据库动作之后触发器执行;
after INSERT on B_TITAN_PROPERTYKEY
--for each row：对表的每一行触发器执行一次。如果没有这一选项，则只对整个表执行一次。
for each row
BEGIN
  INSERT INTO B_TITAN_INDEX(INDEX_NAME, PKEYS, INDEX_LABEL, INDEX_LABEL_TYPE, ISUNIQ)
  VALUES
    --复制来源的表值可以使用 :new.字段名称
    (:new.PKEY_NAME, :new.PKEY_NAME, :new.PKEY_LABEL, :new.PKEY_TYPE, :new.PKEY_INDEX_UNIQ);
END;
``