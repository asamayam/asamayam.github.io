---
title: "Practical MySQL Server Administration - Part 6: Storage Engine Conversions and Plugins"
description: "Part 6 covers creating tables with different storage engines, observing on-disk file layout, converting engines, understanding conversion errors, and loading an engine plugin."
date: 2025-10-16
categories: [Database]
tags: [MySQL, MySQL Administration, Storage Engines, InnoDB, MyISAM, CSV, MEMORY, Plugins]
permalink: /posts/practical-mysql-server-administration-part-6/
---

Part 6 covers practical storage-engine conversions in this 7-part MySQL administration series.

Understand storage engines conceptually first, then watch how file layout, table behavior, and conversion errors change in practice.

## Create Tables With Different Engines

A simple lab makes the differences visible quickly.

```sql
CREATE DATABASE storage_engines;
USE storage_engines;

CREATE TABLE tab1 (no int);
CREATE TABLE tab2 (no int NOT NULL) ENGINE=CSV;
CREATE TABLE tab3 (no int) ENGINE=MEMORY;
CREATE TABLE tab4 (no int) ENGINE=MyISAM;
CREATE TABLE tab5 (no int) ENGINE=BLACKHOLE;
CREATE TABLE tab6 (no int) ENGINE=ARCHIVE;
```

Example terminal output:

```console
mysql> SELECT TABLE_NAME, ENGINE FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA = 'storage_engines';
+------------+-----------+
| TABLE_NAME | ENGINE    |
+------------+-----------+
| tab1       | InnoDB    |
| tab2       | CSV       |
| tab3       | MEMORY    |
| tab4       | MyISAM    |
| tab5       | BLACKHOLE |
| tab6       | ARCHIVE   |
+------------+-----------+
6 rows in set (0.01 sec)
```

> Use a quick query against `INFORMATION_SCHEMA.TABLES` to verify engine assignment after bulk table creation.

## Observe the On-Disk Differences

After creating the tables, inspect the schema directory on disk.

Example terminal output:

```console
-bash-4.2$ ls -ltr
total 144
-rw-r----- 1 mysql mysql 114688 Aug 20 16:42 tab1.ibd
-rw-r----- 1 mysql mysql     35 Aug 20 16:44 tab2.CSM
-rw-r----- 1 mysql mysql      0 Aug 20 16:44 tab2.CSV
-rw-r----- 1 mysql mysql   1024 Aug 20 16:47 tab4.MYI
-rw-r----- 1 mysql mysql      0 Aug 20 16:47 tab4.MYD
-rw-r----- 1 mysql mysql     88 Aug 20 16:48 tab6.ARZ
```

> This example shows clearly that engine choice changes the physical representation of a table, not just a metadata flag.

## Convert Everything to InnoDB

To normalize the lab, convert the non-InnoDB tables back to the default engine.

```sql
ALTER TABLE tab2 ENGINE=InnoDB;
ALTER TABLE tab3 ENGINE=InnoDB;
ALTER TABLE tab4 ENGINE=InnoDB;
ALTER TABLE tab5 ENGINE=InnoDB;
ALTER TABLE tab6 ENGINE=InnoDB;
```

Example terminal output:

```console
mysql> SELECT TABLE_NAME, ENGINE FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA = 'storage_engines';
+------------+--------+
| TABLE_NAME | ENGINE |
+------------+--------+
| tab1       | InnoDB |
| tab2       | InnoDB |
| tab3       | InnoDB |
| tab4       | InnoDB |
| tab5       | InnoDB |
| tab6       | InnoDB |
+------------+--------+
6 rows in set (0.00 sec)
```

> After conversion, the schema directory reflects the change with `.ibd` files for the converted tables.

## Convert Back and Watch for Constraints

Converting back to the original engines works cleanly for empty tables in the example lab:

```sql
ALTER TABLE tab2 ENGINE=CSV;
ALTER TABLE tab3 ENGINE=MEMORY;
ALTER TABLE tab4 ENGINE=MyISAM;
ALTER TABLE tab5 ENGINE=BLACKHOLE;
ALTER TABLE tab6 ENGINE=ARCHIVE;
```

With real data and table structures, limitations start to appear.

Example terminal output:

```console
mysql> alter table orders engine=CSV;
ERROR 1069 (42000): Too many keys specified; max 0 keys allowed

mysql> alter table orders engine=archive;
ERROR 1069 (42000): Too many keys specified; max 1 keys allowed

mysql> alter table orders engine=memory;
ERROR 1163 (42000): The used table type doesn't support BLOB/TEXT columns
```

> These failures show why engine conversion should never be treated as a purely cosmetic change. Table structure compatibility matters.

## Load a Storage Engine Plugin

Because MySQL uses a pluggable engine architecture, supported engines can also depend on shared libraries installed on the server.

Check the plugin directory:

```bash
ls -lrth /usr/lib64/mysql/plugin
```

Example terminal output:

```console
[mysql@mysql-p ~]$ ls -lrth /usr/lib64/mysql/plugin
...
-rwxr-xr-x 1 root root 102K Jun 22 07:51 innodb_engine.so
-rwxr-xr-x 1 root root  57K Jun 22 07:51 ha_mock.so
-rwxr-xr-x 1 root root  46K Jun 22 07:51 ha_example.so
...
```

Install the example engine plugin:

```sql
INSTALL PLUGIN EXAMPLE SONAME 'ha_example.so';
SHOW ENGINES\G;
```

> In production, I would treat plugin loading as a controlled change because it modifies server capability at runtime.

## Final Thoughts

Storage engine changes look easy in syntax and complex in outcome. Administrators should always verify compatibility, table structure, on-disk changes, and workload expectations before converting anything at scale.

In Part 7, I close the series with a full walkthrough of running multiple MySQL instances on the same host using `mysqld_multi`.