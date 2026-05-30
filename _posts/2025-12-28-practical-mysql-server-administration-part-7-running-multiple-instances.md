---
title: "Practical MySQL Server Administration - Part 7: Running Multiple MySQL Instances on One Host"
description: "Part 7 covers running multiple MySQL instances on one server, validating ports and sockets, checking instance health, and managing instances with mysqld_multi."
date: 2025-12-28
categories: [Database]
tags: [MySQL, MySQL Administration, mysqld_multi, Multiple Instances, mysqladmin, Linux Administration]
permalink: /posts/practical-mysql-server-administration-part-7/
---

Part 7 closes this 7-part practical MySQL administration series.

Run multiple MySQL instances on the same host for testing, environment isolation, version comparisons, or dedicated instance-level workloads. Once you do that, clean separation becomes critical. Every instance needs its own data directory, socket, port, log file, and operational controls.

## Verify Instance Processes and Ports

Start by confirming the active `mysqld` processes.

Example terminal output:

```console
[root@mysql-b ~]# ps -ef | grep mysql
mysql 1430 1232 0 19:31 pts/0 00:00:03 /usr/local/mysql/bin/mysqld --defaults-file=/mysql/3306/my.cnf --user=mysql
mysql 1476 1232 0 19:31 pts/0 00:00:02 /usr/local/mysql/bin/mysqld --defaults-file=/mysql/3307/my.cnf --user=mysql
mysql 1520 1232 0 19:32 pts/0 00:00:02 /usr/local/mysql/bin/mysqld --defaults-file=/mysql/3308/my.cnf --user=mysql
```

> Each process points to a separate configuration file. That pattern keeps multi-instance deployments much cleaner.

If needed, install `net-tools` and confirm the listening ports:

```console
[root@mysql-b ~]# sudo yum install net-tools
...
Complete!

[root@mysql-b ~]# netstat -ntl
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address    Foreign Address         State
tcp6       0      0 :::3306          :::*                    LISTEN
tcp6       0      0 :::3307          :::*                    LISTEN
tcp6       0      0 :::3308          :::*                    LISTEN
tcp6       0      0 :::33060         :::*                    LISTEN
```

> This confirms that each MySQL instance accepts client traffic on its assigned port.

## Validate Instance Health Per Port

Use `mysqladmin status` against each port.

```console
[root@mysql-b ~]# /usr/local/mysql/bin/mysqladmin -h127.0.0.1 -uroot -p -P3306 status
Enter password:
Uptime: 648  Threads: 2  Questions: 10  Slow queries: 0  Opens: 152  Flush tables: 3  Open tables: 68  Queries per second avg: 0.015

[root@mysql-b ~]# /usr/local/mysql/bin/mysqladmin -h127.0.0.1 -uroot -p -P3307 status
Enter password:
Uptime: 637  Threads: 2  Questions: 8  Slow queries: 0  Opens: 148  Flush tables: 3  Open tables: 64  Queries per second avg: 0.012

[root@mysql-b ~]# /usr/local/mysql/bin/mysqladmin -h127.0.0.1 -uroot -p -P3308 status
Enter password:
Uptime: 623  Threads: 2  Questions: 9  Slow queries: 0  Opens: 148  Flush tables: 3  Open tables: 64  Queries per second avg: 0.014
```

> Use this fast operational check to confirm live instances without opening interactive sessions against each one.

## Build a `mysqld_multi` Configuration

To manage several instances together, create a consolidated configuration.

Example terminal output:

```console
[root@mysql-b mysql]# cat multi.cnf
[mysqld_multi]
mysqld     = /usr/local/mysql/bin/mysqld_safe
mysqladmin = /usr/local/mysql/bin/mysqladmin
user       = multi_admin
password   = Welcome@123

[mysqld1]
port=3306
basedir=/usr/local/mysql/
datadir=/mysql/3306/data
socket=/tmp/mysql_3306.sock
log_error=/mysql/3306/data/mysql06.log

[mysqld2]
port=3307
basedir=/usr/local/mysql/
datadir=/mysql/3307/data
socket=/tmp/mysql_3307.sock
log_error=/mysql/3307/data/mysql07.log

[mysqld3]
port=3308
basedir=/usr/local/mysql/
datadir=/mysql/3308/data
socket=/tmp/mysql_3308.sock
log_error=/mysql/3308/data/mysql08.log
```

> The instance blocks must stay distinct. Reusing ports, sockets, or datadirs across blocks will create an operational mess very quickly.

## Create the Administrative Shutdown User

`mysqld_multi` needs an account with shutdown privileges on each managed server.

Example:

```sql
CREATE USER 'multi_admin'@'localhost' IDENTIFIED BY 'Welcome@123';
GRANT SHUTDOWN ON *.* TO 'multi_admin'@'localhost';
```

Repeat that setup on every instance you want `mysqld_multi` to control.

## Start and Stop All or One Instance

After configuring `mysqld_multi`, control all instances together or target one by number.

Example terminal output:

```console
[root@mysql-b ~]# /usr/local/mysql/bin/mysqld_multi stop
[root@mysql-b ~]# ps -ef | grep mysql
root 1428 1232 0 19:31 pts/0 00:00:00 grep --color=auto mysql

[root@mysql-b ~]# /usr/local/mysql/bin/mysqld_multi start
[root@mysql-b ~]# ps -ef | grep mysql
mysql 1430 1232 2 19:31 pts/0 00:00:01 /usr/local/mysql/bin/mysqld --defaults-file=/mysql/3306/my.cnf --user=mysql
mysql 1476 1232 4 19:31 pts/0 00:00:01 /usr/local/mysql/bin/mysqld --defaults-file=/mysql/3307/my.cnf --user=mysql
mysql 1520 1232 35 19:32 pts/0 00:00:01 /usr/local/mysql/bin/mysqld --defaults-file=/mysql/3308/my.cnf --user=mysql

[root@mysql-b ~]# /usr/local/mysql/bin/mysqld_multi stop 2
[root@mysql-b ~]# ps -ef | grep mysql
mysql 1430 1232 2 19:31 pts/0 00:00:01 /usr/local/mysql/bin/mysqld --defaults-file=/mysql/3306/my.cnf --user=mysql
mysql 1520 1232 35 19:32 pts/0 00:00:01 /usr/local/mysql/bin/mysqld --defaults-file=/mysql/3308/my.cnf --user=mysql
```

> `mysqld_multi` remains useful in lab and test environments because it simplifies coordinated control without forcing every instance into the same runtime context.

## Final Thoughts

Running multiple MySQL instances on one host stays completely manageable when you keep the boundaries clean: separate configuration files, separate ports, separate sockets, separate data directories, and a deliberate control strategy.

That closes this MySQL server administration series. Together, these seven posts cover the operational foundation administrators need before moving into more advanced topics like tablespace management, partitioning, backup strategy, and high availability.