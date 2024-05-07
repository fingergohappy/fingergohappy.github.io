---
title: Mysql配置主从同步
toc: true
date: 2024-05-07 19:48:38
tags: Mysql
categories: Mysql
cover: https://source.unsplash.com/a-desk-with-a-tv-and-a-remote-control-on-it-94xWLyKfRXo/1200x628
---
# master 配置


## 修改配置文件

```bash
[mysqld]
log-bin
server-id=1
replicate-do-db=tu-portal
## 下面可选
binlog_format=ROW          #复制模式
max_binlog_size=100M         #超过max_binlog_size或超过6小时会切换到下一序号文件
binlog_cache_size       = 16M
log_bin=/var/lib/mysql/mysql-bin.log   #默认路径可修改
expire_logs_days= 7                           #日志过期时间，设置为0则永不过期
binlog_cache_size=16M           #二进制日志缓冲大小,通过show status like 'binlog_%';查看调整写入磁盘的次数，写入磁盘为0最好max_binlog_cache_size   = 256M
relay_log_recovery  = 1            #当slave从库宕机后，假如relay-log损坏了，
#导致一部分中继日志没有处理，则自动放弃所有未执行的relay-log，
#并且重新从master上获取日志，这样就保证了relay-log的完整性。
sync_binlog= 1            #二进制日志（binary log）同步到磁盘的频率
innodb_flush_log_at_trx_commit = 1     #每次事务提交将日志缓冲区写入log file，并同时flush到磁盘。
```

## 创建从库用户

```sql

CREATE USER 'slave1'@'127.0.0.1' IDENTIFIED BY 'qw1234';
GRANT REPLICATION SLAVE ON *.* TO 'slave1'@'127.0.0.1';
FLUSH PRIVILEGES;
```
> 对于8.4以上的版本,要么启用`mysql_native_password`插件，要么使用`rsa`认证
我使用的启用`mysql_native_password`插件
{.is-warning}



修改`my.cnf`
```sql
[mysqld]
mysql_native_password=ON
```
上面的创建用户的`sql`改为:

```sql

CREATE USER 'slave1'@'127.0.0.1' IDENTIFIED with mysql_native_password BY 'qw1234';
GRANT REPLICATION SLAVE ON *.* TO 'slave1'@'127.0.0.1';
FLUSH PRIVILEGES;
```

## dump主库数据

```sql
# 给主库上锁
FLUSH TABLES WITH READ LOCK;
# dump数据
mysqldump -uroot -pqw1234 --databases tu-portal -h 127.0.0.1 > a.sql

```


## 查看主库状态

记录`File`和`Positon`两个字段的值
```sql
show master status;
# 对于mysql 8.4 以上版本
show binary log status;
```

## 释放锁

```sql
#释放锁
UNLOCK TABLES;
```

# 配置从库

## 修改从库配置文件

修改从库配置文件：

```conf
[mysqld]
log-bin
server-id=2
#replicate-do-db=tu-portal

```

重启数据库

## 向从库导入数据

```bash
nohup mysql -uroot -P 3307 -pqw1234 -h 127.0.0.1 < ~/temp/mysql-master-slave/a.sql &
```


## 配置salve


```sql
STOP SLAVE;
CHANGE MASTER TO MASTER_HOST='192.168.33.22',MASTER_USER='slave1',MASTER_PASSWORD='slavepass',MASTER_LOG_FILE='mysql-bin.000001',MASTER_LOG_POS=613;
START SLAVE;
```

对于8.4以上的版本

```sql
STOP REPLICA;
CHANGE REPLICATION SOURCE TO SOURCE_HOST='127.0.0.1',SOURCE_PORT=3306,SOURCE_USER='slave1',SOURCE_PASSWORD='qw1234',SOURCE_LOG_FILE='mysql-bin.000004',SOURCE_LOG_POS=459;
START REPLCIA;
```

