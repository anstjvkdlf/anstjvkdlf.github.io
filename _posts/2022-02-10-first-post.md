---
title:  "Maria DB 설치방법"
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

## 1.1 ODBC

```[root@sdm1 ~] yum list installed | grep ODBC
unixODBC.x86_64                       2.3.1-14.el7                 installed
unixODBC-devel.x86_64                 2.3.1-14.el7                 installed
```


## 1.2. MariaDB

```MariaDB-client.x86_64                 10.5.13-1.el7.centos         @mariadb
MariaDB-common.x86_64                 10.5.13-1.el7.centos         @mariadb
MariaDB-compat.x86_64                 10.5.13-1.el7.centos         @mariadb
MariaDB-server.x86_64                 10.5.13-1.el7.centos         @mariadb
```


## 1.3 Connector

```[root@sdm1 ~]# ll | grep connector
-rw-r--r--. 1 root root 1991882 Jan 25 15:04 mariadb-connector-odbc-3.1.15-centos7-amd64.tar.gz
```


# 2. Install
## 2.1. [mariadb install]

- set Mariadb.repo

```[nssf-opm01] root@ /etc/yum.repos.d # vi  Mariadb.repo
MariaDB 10.5 RedHat repository list - created 2021-06-07 05:03 UTC
http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.5/rhel7-amd64
  gkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1

[nssf-opm01] root@ /etc/yum.repos.d # yum -y install MariaDB-server MariaDB-client

systemctl enable mariadb, systemctl restart mariadb
```


## 2.2. odbc install

```1. yum install unixODBC
yum install unixODBC-devel

2. tar download
tar 다운로드 https://downloads.mariadb.com/Connectors/odbc/connector-odbc-3.1.9/


3. tar zcvf *.tar.gz
```


# 3. Setting

## 3.1. MARAIDB SETTING

```$ mysql -u root {PW}
mysql> SELECT Host,User,plugin,authentication_string FROM mysql.user;
mysql> GRANT ALL PRIVILEGES ON *.* TO '{ID}'@'%' IDENTIFIED BY '{PWD}';
```

## 3.2. ODBC SETTING

~~~[nssf-opm01] root@ /etc # cat odbcinst.ini

[PostgreSQL]
Description     = ODBC for PostgreSQL
Driver          = /usr/lib/psqlodbcw.so
Setup           = /usr/lib/libodbcpsqlS.so
Driver64        = /usr/lib64/psqlodbcw.so
Setup64         = /usr/lib64/libodbcpsqlS.so
FileUsage       = 1


[MySQL]
Description     = ODBC for MySQL
Driver          = /usr/lib/libmyodbc5.so
Setup           = /usr/lib/libodbcmyS.so
Driver64        = /usr/lib64/libmyodbc5.so
Setup64         = /usr/lib64/libodbcmyS.so
FileUsage       = 1

[ODBC Drivers]
MariaDB ODBC 3.1 Driver = installed

[MariaDB ODBC 3.1 Driver]
Description=MariaDB Connector/ODBC v.3.1
Driver = /nssf/mariadb/lib64/libmaodbc.so
#Driver = /usr/lib64/libmaodbc.so


[nssf-opm01] root@ /etc # cat odbc.ini
[MariaDB-server]
Description=MariaDB server
Driver=MariaDB ODBC 3.1 Driver
SERVER=ipc_opm1
USER=root
PASSWORD=.dlfndhs
DATABASE=NSSF
PORT=3306
~~~

# 4. Problems
## 4.1. AutoCommit을 OFF로 두었을 때 생기는 문제

- 프로그램 특성 상 Transaction이 AutoCommit되지 않도록 해야할 때 [SQLSetConnectAttr](https://docs.microsoft.com/en-us/sql/odbc/reference/syntax/sqlsetconnectattr-function?view=sql-server-ver15) 함수에 아래와 같은 옵션을 주어, AutoCommit을 OFF할 수 있다.

~~~SQLSetConnectAttr ( *pHdbc , SQL_ATTR_AUTOCOMMIT, **SQL_AUTOCOMMIT_OFF**, 0 ) ;
~~~
***
세션2에서 세션1의 내용을 바꾸면 당연히 세션 1에서 바뀐내용이 적용되어야 한다. 그런데 이 옵션을 사용하면 아래와 같은 문제가 필연적으로 발생한다.


먼저 우리가 기대하는 결과는 아래와 같다.


~~~
mysql session 1$ create table test (id integer unsigned primary key) engine=innodb;
Query OK, 0 rows affected (0.01 sec)

mysql session 1$ set autocommit=1;
Query OK, 0 rows affected (0.00 sec)

mysql session 1$ select * from test;
Empty set (0.01 sec)

  mysql session 2$ begin;
  Query OK, 0 rows affected (0.00 sec)

  mysql session 2$ insert into test values (1);
  Query OK, 1 row affected (0.05 sec)

  mysql session 2$ commit;
  Query OK, 0 rows affected (0.00 sec)

mysql session 1$ select * from test;
+----+
| id |
+----+
|  1 |
+----+
1 row in set (0.00 sec)
~~~


만약 autocommit을 OFF 하면 어떻게 될까?

세션1의 내용을 세션2에서 바꾸고 commit을 했지만, 세션 1에는 반영이 안되고있다.
기대했던 결과가 아니다.


~~~
mysql session 1$ set autocommit=0;
Query OK, 0 rows affected (0.00 sec)

mysql session 1$ select * from test;
+----+
| id |
+----+
|  1 |
+----+
1 row in set (0.00 sec)

  mysql session 2$ begin;
  Query OK, 0 rows affected (0.00 sec)

  mysql session 2$ insert into test values (2);
  Query OK, 1 row affected (0.05 sec)

  mysql session 2$ commit;
  Query OK, 0 rows affected (0.00 sec)

mysql session 1$ select * from test;
+----+
| id |
+----+
|  1 |
+----+
1 row in set (0.00 sec)

mysql session 1$ commit;
Query OK, 0 rows affected (0.00 sec)

mysql session 1$ select * from test;
+----+
| id |
+----+
|  1 |
|  2 |
+----+
2 rows in set (0.00 sec)
~~~

이는 트랜잭션의 격리수준 (isolation level) 때문에 나타나는 현상이다.

아래와 같이 트랜잭션의 격리수준을 read commited로 설정해주면 문제가 해결된다.

~~~
mysql session 1$ set autocommit=0;
Query OK, 0 rows affected (0.00 sec)

mysql session 1$ set session transaction isolation level read committed;
Query OK, 0 rows affected (0.03 sec)

mysql session 1$ select * from test;
+----+
| id |
+----+
|  1 |
|  2 |
+----+
2 rows in set (0.01 sec)

  mysql session 2$ begin;
  Query OK, 0 rows affected (0.00 sec)

  mysql session 2$ insert into test values (3);
  Query OK, 1 row affected (0.05 sec)

  mysql session 2$ commit;
  Query OK, 0 rows affected (0.00 sec)

mysql session 1$ select * from test;
+----+
| id |
+----+
|  1 |
|  2 |
|  3 |
+----+
3 rows in set (0.00 sec)
~~~


**트랜잭션의 격리수준이란?**
- 트랜잭션이 처리 될 때, 특정 트랜잭션이 다른 트랜잭션에서 변경하거나 조회하는 데이터를 볼 수 있도록 허용할지 말지 결정하는 것이다.


**트랜잭션의 격리수준**
- READ UNCOMMITTED
- READ COMMITTED
- REPEATABLE READ
- SERIALIZABLE

Maria DB는 트랜잭션의 격리수준을 디폴트로 REPEATABLE READ 로 잡고 있다.
~~~
MariaDB [(none)]> SELECT @@GLOBAL.tx_isolation, @@tx_isolation;
+-----------------------+-----------------+
| @@GLOBAL.tx_isolation | @@tx_isolation  |
+-----------------------+-----------------+
| REPEATABLE-READ       | REPEATABLE-READ |
+-----------------------+-----------------+
1 row in set (0.000 sec)
~~~

격리 수준이 REPEATABLE READ 일 때, Transaction을 제대로 관리 하지 않으면 여러 문제가 일어날 수 있다.

REPEATABLE READ는 아래 그림처럼 Transaction을 ID별로 구분하기 때문에 COMMIT을 제대로 해주지 않는 다면 바뀐 DB내용이 아닌 해당 Transaction ID가 가진 Snapshot 내용을 계속읽게 된다.

![Repeatable Read](.././_img/Repeatable_read.png)
그림출처: doooyeon.github.io

---

## 4.1.S [Solution]
~~~
SQLSetConnectAttr ( *pHdbc , SQL_ATTR_AUTOCOMMIT, **SQL_AUTOCOMMIT_ON**, 0 ) ;
~~~

5.2 my.cnf 파일에 설정을 추가해준다.
~~~
[mysqld]
transaction-isolation           = READ-COMMITTED
~~~

5.3. Transaction 관리를 철저하게 해준다.
Transaction이 열렸을 때, 항시 commit해서 닫아줄 수 있도록 한다.
~~~
SQLRETURN SQLEndTran(  
     SQLSMALLINT   HandleType,  
     SQLHANDLE     Handle,  
     SQLSMALLINT   CompletionType);
~~~
*HandleType*

[Input] Handle type identifier. Contains either SQL_HANDLE_ENV (if Handle is an environment handle) or SQL_HANDLE_DBC (if Handle is a connection handle).

*Handle*

[Input] The handle, of the type indicated by HandleType, indicating the scope of the transaction. See "Comments" for more information.

*CompletionType*

[Input] One of the following two values:

SQL_COMMIT SQL_ROLLBACK
