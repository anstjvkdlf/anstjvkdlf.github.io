---
title:  "사내 Maria DB 설치 Routine"
excerpt: "OS : Red Hat Enterprise Linux Server 7.5 (Maipo)
ODBC : unixODBC 2.3.1
DataBase : 10.5.13-MariaDB
Connector : MariaDB ODBC 3.1 Driver"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github, Git]

toc: true
toc_sticky: true
 
date: 2020-05-25
last_modified_at: 2020-05-25
---

# 1. Version Information

1.1 ODBC

[root@sdm1 ~]# yum list installed | grep ODBC
unixODBC.x86_64                       2.3.1-14.el7                 installed
unixODBC-devel.x86_64                 2.3.1-14.el7                 installed

1.2. MariaDB

MariaDB-client.x86_64                 10.5.13-1.el7.centos         @mariadb
MariaDB-common.x86_64                 10.5.13-1.el7.centos         @mariadb
MariaDB-compat.x86_64                 10.5.13-1.el7.centos         @mariadb
MariaDB-server.x86_64                 10.5.13-1.el7.centos         @mariadb

1.3 Connector

[root@sdm1 ~]# ll | grep connector
-rw-r--r--. 1 root root 1991882 Jan 25 15:04 mariadb-connector-odbc-3.1.15-centos7-amd64.tar.gz

# 2. Install

# 3. Setting

# 4. Problems
4.1. AutoCommit 이 OFF 일 때 생기는 문제

