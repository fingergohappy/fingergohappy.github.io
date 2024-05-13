---
title: proxysql配置读写分离
cover: https://source.unsplash.com/a-person-standing-on-the-side-of-a-road-at-night-k_oHn7XBhVA/1200x628
toc: true
date: 2024-05-13 16:32:54
tags: proxysql
categories: proxysql
---
`proxysql`配置读写分离

<!--more-->

# 配置mysql

## 添加用户(如果使用已存在的用户,无需添加用户)

```sql

-- 注意,这里的认证方式要使用mysql_native_password
create user `proxysql` identified with mysql_native_password by 'proxysql-password';

grant all privileges on proxysqldb.* to 'proxysql'@'%' identified with mysql_native_password by 'proxysql-password'
```

## 检查用户的认证方式

```sql
SELECT Host,User,plugin,authentication_string from  mysql.user;
```

如果用户使用的不是`mysql_native_password`加密方式,需要修改一下加密方式:

```sql
drop user ${mysql_username}@'%';

create user 'root'@'%' identified with mysql_native_password  BY '${msqyl_password}';

GRANT ALL PRIVILEGES ON *.* TO ${mysql_username}@'%' IDENTIFIED BY '${msqyl_password}' WITH GRANT OPTION;
```

## 检查autocommit

`proxysql`的配置读写规则后,如果开启了事务,那么在`commit`之前都会走同一个库,举个例子:
如果开启了`autocommit`.在执行玩`commit`之后,如果第一条语句是`select`,那么后面的所有语句都会走读库,包括`insert`和`update`语句.知道下一次`commit`为止.

所以要检查`my.cnf`里面的`autocommit`是否为`off`


# proxysql的基础配置

## 连接proxysql

在执行完`systemctl restart proxysql`之后,就可以使用`mysql`命令连接`proxysql`进行配置


```bash
mysql -uadmin -padmin -h127.0.0.1 -P6032
```

## 分层说明

[官方文档](https://proxysql.com/documentation/configuring-proxysql/)

┌─────────────────────────────────┐
│                                 │
│            Runtime              │
│                                 │
└─────────────────────────────────┘
                                   
                                   
                                   
┌─────────────────────────────────┐
│                                 │
│            Memory               │
│                                 │
└─────────────────────────────────┘
                                   
                                   
                                   
┌─────────────────────────────────┐
│                                 │
│    Disk & Configuration File    │
│                                 │
└─────────────────────────────────┘


简单来说`proxysql`一共有3层,我们通过`mysql`客户端连接到`proxysql`进行配置后,是在`Memory`层进行操作,所有想使修改生效/持久化,要使用`laod/save`命令

## global variables 配置

[表说明](https://proxysql.com/documentation/main-runtime/#global_variables)


### 修改proxysql提供的msyql服务的地址以及端口(可选)

```sql

-- 配置proxysql的mysql服务任意地址访问3306端口访问
SET mysql-interfaces='0.0.0.0:3306';

-- 注意修改这个变量要重启proxysql 服务
SAVE MYSQL VARIABLES TO DISK;
proxysql restart;
```

### 配置proxysql提供给应用的mysql版本号(可选)

```sql
-- 根据你的应用要求以及后端数据库自行配置自行配置
update global_variables set variable_value = '8.0.33' where variable_name = 'mysql-server_version';
LOAD MYSQL VARIABLES TO RUNTIME;
SAVE MYSQL VARIABLES TO DISK;
```

## 添加后端服务器

[表说明](https://proxysql.com/documentation/main-runtime/#mysql_servers)

```sql
INSERT INTO mysql_servers(hostgroup_id,hostname,port) VALUES (10,'10.0.0.1',3306);
INSERT INTO mysql_servers(hostgroup_id,hostname,port) VALUES (20,'10.0.0.2',3306);

load mysql servers to runtime;
save mysql servers to disk;
```

## 配置监控

监控的主要作用是监控后端服务器的健康状态
[表说明](https://proxysql.com/documentation/main-runtime/#global_variables)

```sql

-- 根据你的定义自行修改
set mysql-monitor_username=proxysqlusername;
set mysql-monitor_password=proxysqlpasword;

LOAD MYSQL VARIABLES TO RUNTIME;
SAVE MYSQL VARIABLES TO DISK;
```

### 检查检查日志


配置好监控和`servers`之后就可以查看监控日志了,如果`error`列为`NULL`,就是没有问题

```sql
select * from monitor.mysql_server_connect_log order by time_start_us desc limit 3;
select * from monitor.mysql_server_ping_log order by time_start_us desc limit 3;
```

## 配置分组信息

[表说明](https://proxysql.com/documentation/main-runtime/#mysql_group_replication_hostgroups)

```sql
-- 注意这里的writer_hostgroup和reader_hostgroup 要和上面的mysql_servers 里面的hostgroup_id对应上
insert into mysql_replication_hostgroups(writer_hostgroup, reader_hostgroup, comment) values (10,20,'proxy');
load mysql servers to runtime;
save mysql servers to disk;
```
## 配置访问MYSQL的用户

[表说明](https://proxysql.com/documentation/main-runtime/#mysql_users)

```sql
-- 根据你的配置自行修改
INSERT INTO mysql_users(username,password,default_hostgroup) VALUES ('proxysql','proxysqlpassword',1);
load mysql user to runtime;
save mysql user to disk;
```

## 查看统计信息

```sql
show tables from stats;
```

### stats_mysql_connection_pool

查看`mysql`后端的连接信息

### stats_mysql_commands_counters

查看各种`sql`语句分类的统计信息

### stats_mysql_query_digest

查看`sql`的`digest`的统计信息

# 配置读写分离

**`proxysql`默认所有的`sql`全部会走写库**

`proxysql`默认不建议简单粗暴的将所有的`select`走读库,其他的操作走写库的配置,而是根据慢`sql`日志或者上面的`stats_mysql_query_digest`日志来进行配置

## 三种规则

### match digest

正则匹配`sql`语句的`digest text`
这里要先说明什么是`sql`的`digest`,就是将一个`sql`语句进行摘要,规则如下:

- 关键字大写
- 标识符都被加反引号`
- 常量包括数值常量和字符串常量都被替换成?

可以在任意`mysql`客户端查看一条`sql`语句的`digest text`:

```sql
SELECT STATEMENT_DIGEST_TEXT('SELECT * FroM T1 WHERE c1 > 10 and c2="abc" AND 1+SIN(3) > 0;') AS label;

+------------------------------------------------------------------------+
| label                                                                  |
+------------------------------------------------------------------------+
| SELECT * FROM `T1` WHERE `c1` > ? AND `c2` = ? AND ? + `SIN` (?) > ? ; |
+------------------------------------------------------------------------+
1 row in set (0.00 sec)
```


### digest

`digest`就是将上面获得的`digest text`哈希运算后得到的`hash`字符串,可以在任意`mysql`客户端查看:

```sql
mysql> SELECT STATEMENT_DIGEST('SELECT * FROM T1 WHERE c1 > 2;') AS label;
+------------------------------------------------------------------+
| label                                                            |
+------------------------------------------------------------------+
| 3e2db85c14a4bc5662e406bedb24bc3b47bc98ee99f28bba846e32980a09448d |
+------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> SELECT STATEMENT_DIGEST('SelecT   * FROM T1 WHERE c1 > 2;') AS label;
+------------------------------------------------------------------+
| label                                                            |
+------------------------------------------------------------------+
| 3e2db85c14a4bc5662e406bedb24bc3b47bc98ee99f28bba846e32980a09448d |
+------------------------------------------------------------------+
1 row in set (0.00 sec)
```

### match_pattern

这个最好理解,就是`sql`语句的正则

## mysql_query_rules

[表说明](https://proxysql.com/documentation/main-runtime/#mysql_query_rules)

根据`mysql`的慢`sql`查询或者上面的`stats_mysql_query_digest`表的统计信息结果,就可以配置读写规则了

```sql
-- 注意这里只是例子,要根据场面的统计信息来进行配置然后在测试环境进行验证
-- 注意这里的destination_hostgroup 要和 mysql_replication_hostgroups里面的对应上
insert into mysql_query_rules(rule_id,active,match_digest,destination_hostgroup,apply) values (2,1,'^select.*for update$',10,1);
insert into mysql_query_rules(rule_id,active,match_digest,destination_hostgroup,apply) values (3,1,'^select',20,1);

load mysql query rules to runtime;
save mysql query rules to disk;
```


## 检查是否sql执行是否遵循规则

```sql
select * from stats_mysql_query_digest;
```

# 参考文档

[How to configure ProxySQL for the first time](https://proxysql.com/documentation/ProxySQL-Configuration/)
[How to set up ProxySQL Read/Write Split](https://proxysql.com/documentation/proxysql-read-write-split-howto/)
[Main (runtime tables definition)](https://proxysql.com/documentation/main-runtime/#mysql_query_rules)


