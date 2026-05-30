---
title: "Essential MySQL Command-Line Utilities Every DBA Should Know - Part 2"
description: "Part 2 covers the remaining MySQL utilities for backup, restore, binary log analysis, server safety, performance diagnostics, and benchmarking."
date: 2025-07-19
categories: [Database]
tags: [MySQL, MySQL Utilities, MySQL Administration, MySQL DBA, Linux Administration, Database Administration, Oracle MySQL, mysqldump, mysqlpump, mysqlbackup, mysqlbinlog, mysqlslap, mysqldumpslow, mysqld_safe, mysql_config]
permalink: /posts/essential-mysql-command-line-utilities-part-2/
---

In Part 1, I covered the foundational utilities that DBAs use daily for interactive administration and metadata validation.

In this second part, I cover the remaining utilities focused on backup and restore operations, binary log analysis, server recovery options, and performance diagnostics.

## Utilities Covered In Part 2

- `mysqldump`
- `mysqlpump`
- `mysqlbackup` (Enterprise Edition)
- `mysqlbinlog`
- `mysqld_safe`
- `mysqldumpslow`
- `mysql_config`
- `mysqlslap`

## 1. mysqldump - Logical Backup Utility

`mysqldump` is one of the most widely used tools for creating logical backups in MySQL.

It is commonly used for:

- Single database backups
- Multiple database backups
- Schema-only exports
- Data-only exports
- Migration and restore workflows

Single database backup example:

```bash
mysqldump -u root -p --databases test1 > /backup/test1_dump.sql
```

Multiple database backup example:

```bash
mysqldump -u root -p --databases test1 test2 > /backup/test1_test2_dump.sql
```

All database backup example:

```bash
mysqldump -u root -p --all-databases > /backup/all_databases_dump.sql
```

Example terminal output:

```console
[root@mysqlhost01 ~]# mysqldump -u root -p --databases test1 > /backup/test1_dump.sql
Enter password:
[root@mysqlhost01 ~]# ls -lh /backup/test1_dump.sql
-rw-r--r-- 1 root root 4.2M Jul  9 09:12 /backup/test1_dump.sql
```

### Restore Scenarios

Restore a single database from dump:

```bash
mysql -u root -p test1 < /backup/test1_dump.sql
```

Restore using a full server-level dump:

```bash
mysql -u root -p < /backup/all_databases_dump.sql
```

Example restore output:

```console
[root@mysqlhost01 ~]# mysql -u root -p test1 < /backup/test1_dump.sql
Enter password:
[root@mysqlhost01 ~]# mysql -u root -p -e "SHOW DATABASES LIKE 'test1';"
Enter password:
+----------------+
| Database (test1) |
+----------------+
| test1          |
+----------------+
```

Useful options:

- `--no-data` for schema-only dumps
- `--no-create-info` for data-only dumps
- `--login-path` for secure, non-interactive authentication

> `mysqldump` remains the default logical backup tool for portability, migration, and routine restore validation.

## 2. mysqlpump - Parallel Logical Backup Utility

`mysqlpump` extends logical backup capabilities with parallelism and additional filtering options.

Example:

```bash
mysqlpump -u root -p --all-databases > /backup/all_databases_pump.sql
```

Example terminal output:

```console
[root@mysqlhost01 ~]# mysqlpump -u root -p --all-databases > /backup/all_databases_pump.sql
Enter password:
Dump progress: 35/36 tables, 8161740/32639699 rows
Dump progress: 35/36 tables, 9811740/32639699 rows
Dump progress: 35/36 tables, 11462740/32639699 rows
Dump completed in 21765
```

Key points:

- Supports parallel backup operations
- Useful for larger logical export workloads
- Produces dump output suitable for restore using `mysql`

Restore example:

```bash
mysql -u root -p < /backup/all_databases_pump.sql
```

> On newer MySQL versions, check current product guidance since `mysqlpump` behavior and support recommendations can vary by release.

## 3. mysqlbackup - Enterprise Physical Backup Utility

`mysqlbackup` (MySQL Enterprise Backup) is designed for enterprise-grade backup operations with capabilities beyond standard logical dumps.

Typical capabilities include:

- Incremental backups
- Compression and encryption
- Parallelized backup processing
- Backup validation workflows

Example backup command pattern:

```bash
mysqlbackup --user=backup_user --password --backup-dir=/backup/mysql --with-timestamp backup
```

Example validation pattern:

```bash
mysqlbackup --backup-image=/backup/mysql/mysql_data.mbi validate
```

Example validation output:

```console
[root@mysqlhost01 ~]# mysqlbackup --backup-image=/backup/mysql/mysql_data.mbi validate
231108 02:23:56 MAIN INFO: Validate operation completed successfully.
231108 02:23:56 MAIN INFO: Backup Image validation successful.
mysqlbackup completed OK!
```

Restore scenario (high-level):

- Prepare backup image
- Apply logs if required
- Copy-back or move-back data
- Start MySQL and validate objects

> `mysqlbackup` is available in Enterprise Edition and is best suited for production-grade backup, recovery, and compliance controls.

## 4. mysqlbinlog - Binary Log Inspection And Recovery Support

`mysqlbinlog` reads MySQL binary log files and helps DBAs understand the exact change history captured by the server.

It is useful for:

- Reviewing write activity
- Troubleshooting unexpected data changes
- Supporting point-in-time recovery workflows
- Validating replication-related events

Example:

```bash
mysqlbinlog /var/lib/mysql/binlog.000042
```

Example terminal output:

```console
[root@mysqlhost01 ~]# mysqlbinlog /var/lib/mysql/binlog.000042
# at 4
#231020  9:21:16 server id 1  end_log_pos 126
Start: binlog v 4, server v 8.0.34-commercial created 231020 9:21:16 at startup
# at 157
#231020  9:24:34 server id 1  end_log_pos 180
Stop
# End of log file
[root@mysqlhost01 ~]#
```

Sample lines you typically see include log positions, GTID context, and SQL event entries.

> `mysqlbinlog` is a critical tool for forensic analysis and recovery planning when binary logging is enabled.

## 5. mysqld_safe - Safe Server Startup Wrapper

`mysqld_safe` is commonly used to start MySQL with additional safety controls and custom startup parameters.

Typical use cases:

- Controlled startup during troubleshooting
- Passing temporary startup flags
- Recovering from authentication issues (for example, maintenance with grant tables skipped)

Example:

```bash
mysqld_safe --skip-grant-tables --skip-networking &
```

Example terminal output:

```console
[root@mysqlhost01 ~]# mysqld_safe --skip-grant-tables --skip-networking &
[1] 19442
[root@mysqlhost01 ~]# Logging to '/var/log/mysqld.log'.
[root@mysqlhost01 ~]# Starting mysqld daemon with databases from /var/lib/mysql
```

> Use this carefully and only during controlled maintenance windows.

## 6. mysqldumpslow - Slow Query Log Summarization

`mysqldumpslow` summarizes slow query log entries so you can quickly identify high-impact patterns.

Example:

```bash
mysqldumpslow -s t -t 10 /var/log/mysql/slow.log
```

Reverse-sort example:

```bash
mysqldumpslow -r /var/log/mysql/slow.log
```

Example terminal output:

```console
[root@mysqlhost01 ~]# mysqldumpslow -s t -t 10 /var/log/mysql/slow.log
Reading mysql slow query log from /var/log/mysql/slow.log
Count: 25  Time=2.11s (52s)  Lock=0.00s (0s)  Rows=1.0 (25), root[root]@localhost
SELECT * FROM orders WHERE order_id = N
```

> This is one of the fastest ways to prioritize performance tuning work from large slow-log files.

## 7. mysql_config - Build And Client Compilation Flags

`mysql_config` returns include paths, libraries, and compile flags required by client applications.

Example:

```bash
mysql_config --cflags
```

Compile example:

```bash
gcc -c $(mysql_config --cflags) example.c
```

Example terminal output:

```console
[root@mysqlhost1 ~]# mysql_config --cflags
-I/usr/include/mysql -m64
[root@mysqlhost1 ~]# gcc -c $(mysql_config --cflags) example.c
Compilation completed
```

> This utility is useful when building client-side tools or integrations against MySQL headers and libraries.

## 8. mysqlslap - Quick Benchmarking And Load Simulation

`mysqlslap` provides lightweight benchmarking by simulating concurrent client activity against MySQL.

It is commonly used to:

- Compare query behavior under different concurrency levels
- Validate performance changes after tuning
- Run quick pre-deployment performance checks

Example:

```bash
mysqlslap --concurrency=20 --iterations=5 --query="SELECT 1" --create-schema=test
```

Example terminal output:

```console
[root@mysqlhost01 ~]# mysqlslap --concurrency=20 --iterations=5 --query="SELECT 1" --create-schema=test
Benchmark
Average number of seconds to run all queries: 0.118 seconds
Minimum number of seconds to run all queries: 0.109 seconds
Maximum number of seconds to run all queries: 0.133 seconds
Number of clients running queries: 20
Average number of queries per client: 1
```

This executes the specified query under simulated concurrency and reports timing metrics.

> `mysqlslap` is useful when you need a quick, scriptable baseline instead of a full performance test harness.

## Final Thoughts

Together, these utilities cover a practical DBA workflow:

- `mysqldump` for logical backup and restore
- `mysqlpump` for parallel logical exports
- `mysqlbackup` for enterprise backup and recovery workflows
- `mysqlbinlog` for event-level visibility and recovery support
- `mysqld_safe` for controlled troubleshooting startup
- `mysqldumpslow` for slow-log analysis
- `mysql_config` for client build integration
- `mysqlslap` for lightweight load validation

With Part 1 and Part 2 combined, you have a strong command-line toolkit for day-to-day MySQL administration, troubleshooting, and operational readiness.
