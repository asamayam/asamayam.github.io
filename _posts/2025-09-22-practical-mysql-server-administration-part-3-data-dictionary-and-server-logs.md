---
title: "Practical MySQL Server Administration - Part 3: Data Dictionary and Server Logs"
description: "Part 3 covers MySQL system tables, the data dictionary, object metadata categories, and the log files administrators use for troubleshooting and operational control."
date: 2025-09-22
categories: [Database]
tags: [MySQL, MySQL Administration, Data Dictionary, Server Logs, Error Log, Binary Log, Performance Schema]
permalink: /posts/practical-mysql-server-administration-part-3/
---

Part 3 covers the data dictionary and server logs in this 7-part MySQL administration series.

Every MySQL instance carries a large amount of metadata that tells the server which objects exist, which users can access each object, and how the server should record operational events. Administrators interact with this layer every day, often without thinking about it explicitly.

## Understand What the `mysql` Schema Stores

The `mysql` system schema contains the data dictionary and administrative tables the server depends on to run.

Key categories include:

- data dictionary tables for object metadata
- grant system tables for accounts and privileges
- object information tables for components, loadable functions, and plugins
- log system tables for general and slow query logging
- server-side help tables
- time zone tables
- replication support tables
- optimizer statistics and cost model tables

> When I troubleshoot metadata, account behavior, or server features, I usually start with this schema.

## Review the Available System Schemas

Run `SHOW DATABASES` to confirm the internal schemas present on the server.

Example terminal output:

```console
mysql> show databases;
+-------------------------------+
| Database                      |
+-------------------------------+
| db1                           |
| db2                           |
| db3                           |
| db4                           |
| information_schema            |
| mysql                         |
| mysql_innodb_cluster_metadata |
| performance_schema            |
| replication_test              |
| sys                           |
+-------------------------------+
10 rows in set (0.09 sec)
```

> This view quickly confirms whether the instance also includes extra administrative metadata, such as InnoDB Cluster-related schemas.

## Use the `sys` Schema for Faster Operational Insight

The `sys` schema makes Performance Schema data easier to interpret.

Example:

```sql
USE sys;
SHOW TABLES LIKE '%use%';
SELECT * FROM user_summary\G
```

Example terminal output:

```console
mysql> show tables like '%use%';
+-------------------------------------+
| Tables_in_sys (%use%)               |
+-------------------------------------+
| memory_by_user_by_current_bytes     |
| schema_unused_indexes               |
| user_summary                        |
| user_summary_by_file_io             |
| user_summary_by_statement_latency   |
| waits_by_user_by_latency            |
+-------------------------------------+
17 rows in set (0.02 sec)

mysql> select * from user_summary\G
*************************** 1. row ***************************
user: mysql_innodb_cluster_3
statements: 15
statement_latency: 9.52 h
current_connections: 1
total_connections: 7
current_memory: 33.50 KiB
total_memory_allocated: 1.09 MiB
...
*************************** 8. row ***************************
user: event_scheduler
statements: 0
current_connections: 1
current_memory: 16.27 KiB
total_memory_allocated: 16.27 KiB
8 rows in set (0.13 sec)
```

> Views like `user_summary` help correlate workload shape, connection activity, and memory usage without writing complex Performance Schema queries from scratch.

## Know the Main MySQL Log Types

MySQL administrators should be able to distinguish these log categories quickly:

- error log
- general query log
- binary log
- relay log
- slow query log
- DDL log

Each one answers a different operational question.

- The error log helps with startup, shutdown, warnings, and failures.
- The general log shows connection activity and received SQL.
- The binary log captures data-changing events.
- The relay log supports replication processing.
- The slow query log highlights expensive statements.
- The DDL log tracks metadata operations associated with DDL processing.

## Rotate and Flush Logs Carefully

As logs grow, back up each log file and rotate each file in a controlled way.

Example:

```bash
cd /var/lib/mysql
mv mysqld.log mysqld.log.old
mv mysql-slow.log mysql-slow.log.old
mysqladmin -uroot -p flush-logs
```

Example terminal output:

```console
[root@mysql-a ~]# mv mysql-slow.log mysql-slow.log.old
[root@mysql-a ~]# mysqladmin -uroot -p flush-logs
```

> Flushing logs closes the active files and opens new ones. In replicated environments, confirm that replicas have processed the required binary logs before rotation.

## Final Thoughts

If you understand the data dictionary and the purpose of each log family, MySQL administration becomes far more predictable. You know where to look when objects disappear, permissions misbehave, or operational events need to be traced.

In Part 4, I move into day-to-day operational control: startup, shutdown, authentication behavior, and connection management.