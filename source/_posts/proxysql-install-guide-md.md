---
title: proxysql安装指引
toc: true
date: 2024-05-13 16:26:39
tags: proxysql
categories: proxysql
cover: https://source.unsplash.com/a-lake-surrounded-by-rocks-and-grass-with-mountains-in-the-background-1eZfCWs8IM8/1200x628
---
proxysql安装指引

<!--more --> 


# 下载proxysql

进入`proxysql`的[github releases](https://github.com/sysown/proxysql/releases)下载最下`rpm`


# 安装


在安装路径执行命令`rpm -ivh proxy-sql.rpm`执行安装,可能会缺少`proxy-sql`所需的依赖,根据情况安装即可

所需依赖可能需要安装的包,一下无需全部安装,根据上面命令输出,按需安装:

- gnutls
- perl-DBD-MySQL
- nettle
- perl-Compress-Raw-Bzip2
- perl-Compress-Raw-Zlib
- perl-DBI 
- perl-Data-Dumper
- perl-IO-Compress
- perl-Net-Daemon
- perl-PlRPC
- trousers


## 无网络安装

>如果你的网络环境比较特殊,无法使用`yum` 安装,可以下载以上`rpm`包
{.is-info}

`rpm`可以去这个[网站](https://rpmfind.net/linux/rpm2html/search.php)

然后执行命令

```bash
rpm -ivh gnutls-3.3.29-9.el7_6.x86_64.rpm \
trousers-0.3.14-2.el7.x86_64.rpm \
nettle-2.7.1-8.el7.x86_64.rpm \
perl-Compress-Raw-Bzip2-2.061-3.el7.x86_64.rpm \
perl-Compress-Raw-Zlib-2.061-4.el7.x86_64.rpm  \
perl-Net-Daemon-0.48-5.el7.noarch.rpm \
perl-IO-Compress-2.061-2.el7.noarch.rpm \
perl-Compress-Raw-Bzip2-2.061-3.el7.x86_64.rpm \
perl-PlRPC-0.2020-14.el7.noarch.rpm \
perl-Data-Dumper-2.145-3.el7.x86_64.rpm \
perl-DBI-1.627-4.el7.x86_64.rpm \
perl-DBD-MySQL-4.023-6.el7.x86_64.rpm \
```

# 安装mysql客户端

`proxysql`虽然底层的存储是文件和`sqllite`,但是`proxysql`重写了协议,可以使用`mysql`的客户端进行配置

```bash
yum localinstall  mysql80-community-release-el7-5.noarch.rpm && yum -y install mysql-community-client
```


## 无网络安装

如果你的服务器无法使用`yum`进行安装,可以下载对应的安装包,`mysql`客户端需要的`rpm`如下:

- mysql84-community-release-el7-1.noarch.rpm [下载地址](https://dev.mysql.com/downloads/repo/yum/)
- mysql-community-libs-8.3.0-1.el7.x86_64.rpm [下载地址](https://dev.mysql.com/downloads/mysql/5.5.html?os=31&version=5.1)
- mysql-community-common-8.3.0-1.el7.x86_64.rpm [下载地址](https://dev.mysql.com/downloads/mysql/5.5.html?os=31&version=5.1)
- mysql-community-client-plugins-8.3.0-1.el7.x86_64.rpm [下载地址](https://dev.mysql.com/downloads/mysql/5.5.html?os=31&version=5.1)
- mysql-community-client-8.3.0-1.el7.x86_64.rpm [下载地址](https://dev.mysql.com/downloads/mysql/5.5.html?os=31&version=5.1)


执行命令:
```bash
rpm -ivh \
mysql84-community-release-el7-1.noarch.rpm \
mysql-community-libs-8.3.0-1.el7.x86_64.rpm \
mysql-community-common-8.3.0-1.el7.x86_64.rpm \
mysql-community-client-plugins-8.3.0-1.el7.x86_64.rpm \
mysql-community-client-8.3.0-1.el7.x86_64.rpm \
```


# 安装验证

执行命令`systemctl status proxysql`

如果输出:

```bash
   Loaded: loaded (/etc/systemd/system/proxysql.service; enabled; vendor preset: disabled)
   Active: inactive (dead)

```

则证明安装成功

# 启动

执行命令`systemctl start proxysql`

## 启动验证

`systemctl status proxysql`如果输出:

```bash
● proxysql.service - High Performance Advanced Proxy for MySQL
   Loaded: loaded (/etc/systemd/system/proxysql.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2023-12-13 11:53:41 CST; 30s ago
  Process: 29610 ExecStart=/usr/bin/proxysql --idle-threads -c /etc/proxysql.cnf $PROXYSQL_OPTS (code=exited, status=0/SUCCESS)
 Main PID: 29612 (proxysql)
   CGroup: /system.slice/proxysql.service
           ├─29612 /usr/bin/proxysql --idle-threads -c /etc/proxysql.cnf
           └─29613 /usr/bin/proxysql --idle-threads -c /etc/proxysql.cnf

Dec 13 11:53:40 jira-node01 systemd[1]: Starting High Performance Advanced Proxy for MySQL...
Dec 13 11:53:40 jira-node01 proxysql[29610]: 2023-12-13 11:53:40 [INFO] Using config file /etc/proxysql.cnf
Dec 13 11:53:40 jira-node01 proxysql[29610]: 2023-12-13 11:53:40 [INFO] Current RLIMIT_NOFILE: 102400
Dec 13 11:53:40 jira-node01 proxysql[29610]: 2023-12-13 11:53:40 [INFO] Using OpenSSL version: OpenSSL 3.1.0 14 Mar 2023
Dec 13 11:53:40 jira-node01 proxysql[29610]: 2023-12-13 11:53:40 [INFO] No SSL keys/certificates found in datadir (/var/lib/proxysql). Generating ne...ificates.
Dec 13 11:53:41 jira-node01 systemd[1]: Started High Performance Advanced Proxy for MySQL.

```
则证明启动成功

