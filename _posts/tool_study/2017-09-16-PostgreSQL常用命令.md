---
layout: post
title: PostgreSQL常用命令总结
category: 工具学习
keywords: PostgreSQL, 命令
---

由于工作原因，日常工作中会经常接触到PostgreSQL，因此对于PostgreSQL的了解就显得尤为重要，这里将一些常用的命令记录下来，方便自己忘记的时候进行查阅，当前，由于自己的认知还处于浅薄的状态，所以，这个博客可能会随着时间不断进行更新。
## 2017年03月24日
### 初始化相关
```shell
//创建数据存储路径
binresist@linux-n8lv:~> sudo mkdir -p /usr/local/pgsql/data

//修改文件夹属主
binresist@linux-n8lv:~> sudo chown postgres /usr/local/pgsql/data

//修改默认用户postgres的密码
binresist@linux-n8lv:~> sudo passwd postgres

//初始化数据库，会同时生成两个模板数据库template1和template0，需要用postgres用户执行
postgres@linux-n8lv:/home/binresist> initdb -D /usr/local/pgsql/data
属于此数据库系统的文件宿主为用户 "postgres".
此用户也必须为服务器进程的宿主.
数据库簇将使用本地化语言 "zh_CN.UTF-8"进行初始化.
默认的数据库编码已经相应的设置为 "UTF8".
initdb: 无法为本地化语言环境"zh_CN.UTF-8"找到合适的文本搜索配置
缺省的文本搜索配置将会被设置到"simple"

禁止为数据页生成校验和.

修复已存在目录 /usr/local/pgsql/data 的权限 ... 成功
正在创建子目录 ... 成功
选择默认最大联接数 (max_connections) ... 100
选择默认共享缓冲区大小 (shared_buffers) ... 128MB
选择动态共享内存实现 ......posix
创建配置文件 ... 成功
在 /usr/local/pgsql/data/base/1 中创建 template1 数据库 ... 成功
初始化 pg_authid ...  成功
初始化dependencies ... 成功
创建系统视图 ... 成功
正在加载系统对象描述 ...成功
创建(字符集)校对规则 ... 成功
创建字符集转换 ... 成功
正在创建字典 ... 成功
对内建对象设置权限 ... 成功
创建信息模式 ... 成功
正在装载PL/pgSQL服务器端编程语言...成功
清理数据库 template1 ... 成功
拷贝 template1 到 template0 ... 成功
拷贝 template1 到 template0 ... 成功
同步数据到磁盘...成功

警告:为本地连接启动了 "trust" 认证.
你可以通过编辑 pg_hba.conf 更改或你下次
行 initdb 时使用 -A或者--auth-local和--auth-host选项.

成功. 您现在可以用下面的命令运行数据库服务器:

    postmaster -D /usr/local/pgsql/data
或者
    pg_ctl -D /usr/local/pgsql/data -l logfile start
```

### 启动数据库
```shell
postgres@linux-n8lv:/home/binresist> pg_ctl -D /usr/local/pgsql/data start
正在启动服务器进程
postgres@linux-n8lv:/home/binresist> 2017-03-24 10:01:29 CST   日志:  日志输出重定向到日志收集进程
2017-03-24 10:01:29 CST   提示:  后续的日志输出将出现在目录 "pg_log"中.
```

### 创建新用户
```shell
//使用postgres登录数据库
postgres@linux-n8lv:/home/binresist> psql postgres 
psql (9.4.9)
输入 "help" 来获取帮助信息.

//创建新用户pub并指定密码
postgres=# create user pub password 'pub';
CREATE ROLE

```

### 创建新数据库
```shell
//为pub用户创建数据库pub1
postgres=# create database pub1 owner pub;
CREATE DATABASE
```

### 登录数据库
```shell
//使用pub用户登录数据库pub1
postgres@linux-n8lv:/home/binresist> psql -U pub pub1
psql (9.4.9)
输入 "help" 来获取帮助信息.
```

### 创建schema
```shell
//创建schema
pub1=> create schema pub authorization pub;
CREATE SCHEMA
```

### 帮助
```shell
//查看帮助
pub1=> help
您正在使用psql, 这是一种用于访问PostgreSQL的命令行界面
键入： \copyright 显示发行条款
       \h 显示 SQL 命令的说明
       \? 显示 pgsql 命令的说明
       \g 或者以分号(;)结尾以执行查询
       \q 退出
```


### 快捷命令
```shell
//查看数据库
pub1=> \l
                                     资料库列表
   名称    |  拥有者  | 字元编码 |  校对规则   |    Ctype    |       存取权限        
-----------+----------+----------+-------------+-------------+-----------------------
 postgres  | postgres | UTF8     | zh_CN.UTF-8 | zh_CN.UTF-8 | 
 pub1      | pub      | UTF8     | zh_CN.UTF-8 | zh_CN.UTF-8 | 
 template0 | postgres | UTF8     | zh_CN.UTF-8 | zh_CN.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | zh_CN.UTF-8 | zh_CN.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
(4 行记录)

//切换数据库
pub1=> \c
您现在已经连线到数据库 "pub1",用户 "pub".
pub1=> \c postgres
您现在已经连线到数据库 "postgres",用户 "pub".
postgres=> \c template1
您现在已经连线到数据库 "template1",用户 "pub".
template1=> \c pub1
您现在已经连线到数据库 "pub1",用户 "pub".

//查看所有表
pub1=> \dt
找不到关联。

//查看定义
pub1=> \d pg_stat_activity 
         视观表 "pg_catalog.pg_stat_activity"
       栏位       |           型别           | 修饰词 
------------------+--------------------------+--------
 datid            | oid                      | 
 datname          | name                     | 
 pid              | integer                  | 
 usesysid         | oid                      | 
 usename          | name                     | 
 application_name | text                     | 
 client_addr      | inet                     | 
 client_hostname  | text                     | 
 client_port      | integer                  | 
 backend_start    | timestamp with time zone | 
 xact_start       | timestamp with time zone | 
 query_start      | timestamp with time zone | 
 state_change     | timestamp with time zone | 
 waiting          | boolean                  | 
 state            | text                     | 
 backend_xid      | xid                      | 
 backend_xmin     | xid                      | 
 query            | text                     | 


//查看索引
pub1=> \di
找不到关联。
```
