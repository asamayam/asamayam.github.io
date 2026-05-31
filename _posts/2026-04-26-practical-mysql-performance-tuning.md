---
title: "Practical MySQL Performance Tuning"
description: "A practical guide to MySQL performance tuning, covering system sizing, InnoDB memory and redo settings, bottleneck analysis, slow query logging, Performance Schema, and index maintenance."
date: 2026-04-26
categories: [Database]
tags: [MySQL, Performance Tuning, InnoDB, Slow Query Log, Performance Schema, Indexes, Database Administration]
permalink: /posts/practical-mysql-performance-tuning/
---

Performance tuning matters because it affects the experience people feel at the application layer.

In OLTP systems, slow checkout flows and query latency affect revenue directly. In batch-oriented systems, the tuning goal shifts toward predictable throughput and completing work inside the available processing window. Good tuning starts with the workload, not with random parameter changes.

This article condenses the practical approach from the source material into one workflow:

- Set realistic baseline expectations
- Size the server for the workload
- Tune the MySQL configuration for the deployment type
- Identify bottlenecks before changing SQL
- Use built-in MySQL tools to validate changes
- Review index design and maintenance choices

## Start with the Right Goal

Do not start by tuning the loudest query you find.

Start by answering these questions:

- What performance does the application need?
- Where does the workload spend time today?
- Which resource becomes the bottleneck first: CPU, memory, disk, or SQL design?
- What does good enough look like for this system?

That framing matters because a query that runs slowly in isolation may not be the real problem. The real issue may come from memory pressure, I/O waits, missing indexes, or a database layout that does not match the workload.

## Size the Server for the Workload

Database performance depends on hardware resources and configuration together.

### CPU

CPU throughput affects concurrency, parsing, and execution speed. The chapter uses systems ranging from a handful of cores to larger deployments with many more. The right CPU choice depends on the transaction rate, query complexity, and the amount of parallel work the application generates.

### Memory

Memory drives cache efficiency. A larger buffer cache reduces disk reads and improves response time for repeated access patterns.

### Disk

Disk performance becomes critical in write-heavy systems and workloads with frequent modifications. SSDs or other high-throughput storage outperform spinning disks for most transactional systems.

If the workload modifies data heavily, storage latency quickly becomes visible in user-facing response time.

## Tune the Core Database Settings

MySQL defaults target small or moderate deployments. Larger or latency-sensitive systems need a deliberate configuration review.

### `innodb_dedicated_server`

Use `innodb_dedicated_server` only on hosts that exist primarily for MySQL. When enabled, MySQL configures the buffer pool and redo capacity automatically.

```ini
[mysqld]
innodb_dedicated_server=ON
```

Check the current value:

```sql
SHOW VARIABLES LIKE 'innodb_dedicated_server';
```

### `innodb_buffer_pool_size`

InnoDB buffer pool tuning ranks among the most important performance settings. A larger buffer pool keeps data and index pages in memory and reduces physical I/O.

In practice, the chapter recommends assigning a large share of server memory to the buffer pool on dedicated database hosts.

```ini
[mysqld]
innodb_buffer_pool_size=10G
```

Verify it with:

```sql
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';
```

### `innodb_buffer_pool_instances`

Multiple buffer pool instances can reduce contention on busy systems.

```ini
[mysqld]
innodb_buffer_pool_instances=24
```

The exact value depends on the deployment size and workload shape. Start with a sensible baseline and monitor the result.

### `innodb_log_buffer_size`

The log buffer affects transaction commit behavior and can help workloads that generate frequent changes.

```ini
[mysqld]
innodb_log_buffer_size=48M
```

### `innodb_flush_log_at_trx_commit`

This setting controls how aggressively InnoDB flushes log records at commit time.

```ini
[mysqld]
innodb_flush_log_at_trx_commit=1
```

Keep the value at `1` for the strongest durability behavior.

### `innodb_flush_method`

`innodb_flush_method` controls how MySQL flushes data to disk.

```sql
SHOW VARIABLES LIKE 'innodb_flush_method';
```

Linux and Unix deployments commonly use `fsync`. Some fast local storage systems perform better with `O_DIRECT`, which avoids extra buffering overhead.

### `innodb_file_per_table`

Use file-per-table tablespaces for most modern deployments.

```ini
[mysqld]
innodb_file_per_table=ON
```

That setting stores each table in its own `.ibd` file and simplifies some maintenance operations.

### `innodb_redo_log_capacity`

In MySQL 8.0.30 and later, `innodb_redo_log_capacity` replaces the older redo file sizing model.

```ini
[mysqld]
innodb_redo_log_capacity=32G
```

### `sort_buffer_size` and `join_buffer_size`

These buffers matter when the optimizer must sort or join without an efficient index path.

Use more memory here only when the optimizer must sort or join without an efficient index path; indexing still provides the better fix.

```sql
SHOW VARIABLES LIKE 'sort_buffer_size';
SHOW VARIABLES LIKE 'join_buffer_size';
```

### `read_buffer_size`

This setting matters less for InnoDB than for MyISAM, but the chapter includes it as part of the broader tuning review.

```sql
SHOW VARIABLES LIKE 'read_buffer_size';
```

## Use a Practical Baseline Configuration

The chapter gives two example configurations: one for dedicated servers and one for systems that do not dedicate all resources to MySQL.

On dedicated MySQL servers, allocate resources so InnoDB can use the machine effectively.

Example baseline:

```ini
[mysqld]
innodb_dedicated_server=1
innodb_buffer_pool_instances=24
innodb_log_buffer_size=48M
innodb_file_per_table=1
max_connections=500
slow-query-log=1
slow_query_log_file=/var/log/slow_query.log
```

For a non-dedicated system, set the memory explicitly:

```ini
[mysqld]
innodb_buffer_pool_size=10G
innodb_buffer_pool_instances=24
innodb_redo_log_capacity=32G
innodb_log_buffer_size=48M
innodb_file_per_table=1
max_connections=500
slow-query-log=1
slow_query_log_file=/var/log/slow_query.log
```

## Analyze Bottlenecks Before Changing SQL

Performance issues usually span the OS, the database, and the application. If you only inspect SQL, you may miss the real bottleneck.

### Check CPU

Use `top` or similar tools to watch CPU saturation, run queue pressure, and the percentage of idle time.

### Check Memory

Use `free -g` or `free -gt` to look for swap activity and low available memory.

### Check I/O

Use `iostat` or similar tools to find disk bottlenecks, write pressure, and elevated I/O wait.

Example commands:

```bash
top
free -gt
iostat 2 3
```

Focus on the resource that limits the system first; the command matters less than the signal it exposes.

## Turn on Slow Query Logging

MySQL can record the statements that run longer than a chosen threshold.

```ini
[mysqld]
slow-query-log=1
slow_query_log_file=/var/log/slow_query.log
long_query_time=1
```

The slow query log gives you a practical starting point for workload analysis.

Do not assume that every query in the log deserves tuning. Use it as a signal, then validate with execution plans, row counts, and access patterns.

## Use Performance Schema for Deeper Analysis

Performance Schema ships with MySQL and stores runtime metrics, waits, locks, and statement history.

Verify that MySQL enables it:

```sql
SHOW VARIABLES LIKE 'performance_schema';
```

Then inspect the available tables:

```sql
SELECT TABLE_NAME, TABLE_ROWS
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_SCHEMA = 'performance_schema';
```

The useful tables include:

- `events_statements_summary_by_digest`
- `events_waits_summary_global_by_event_name`
- `data_locks`
- `metadata_locks`
- `threads`
- `table_io_waits_summary_by_table`

That schema gives you the raw material for spotting lock contention, hot statements, and wait behavior.

## Use Maintenance Tools Carefully

The chapter treats maintenance tools as targeted utilities, not as blanket fixes.

Run maintenance tools during non-peak hours because some operations take locks or pause writes.

### `ANALYZE TABLE`

`ANALYZE TABLE` refreshes statistics that the optimizer uses to choose access paths.

```sql
ANALYZE TABLE EMPLOYEE1;
```

Use it after large DML changes or when execution plans no longer match reality.

### `OPTIMIZE TABLE`

`OPTIMIZE TABLE` reorganizes physical storage and can reclaim space in some cases.

```sql
OPTIMIZE TABLE EMPLOYEE1;
```

The chapter shows the common MySQL behavior where InnoDB may recreate and analyze the table instead of performing a classic optimization path.

### `CHECK TABLE`

`CHECK TABLE` helps validate table and index integrity.

```sql
CHECK TABLE EMPLOYEE1;
```

Use it when you suspect corruption, compatibility issues, or index problems.

## Review Table Statistics

Performance work often depends on understanding table size, row estimates, and index footprint.

The chapter uses `information_schema.INNODB_TABLESTATS` to inspect statistics for a table:

```sql
SELECT *
FROM information_schema.INNODB_TABLESTATS
WHERE NAME='test/EMPLOYEE1'\G
```

Useful fields include:

- `TABLE_ID`
- `NAME`
- `STATS_INITIALIZED`
- `NUM_ROWS`
- `CLUST_INDEX_SIZE`
- `OTHER_INDEX_SIZE`
- `MODIFIED_COUNTER`
- `AUTOINC`
- `REF_COUNT`

This information helps you understand whether MySQL has collected usable statistics and how much storage the clustered and secondary indexes consume.

## Index Design Still Matters

Good tuning usually comes back to indexes.

An index can reduce scans, shorten response time, and eliminate expensive sorts or joins. When you cannot add a useful index, buffer settings may help temporarily, but the index problem remains.

### Add a Non-Unique Index

```sql
ALTER TABLE tablename ADD INDEX (colname);
CREATE INDEX indexname ON tablename (colname);
```

### Add a Unique Index

```sql
ALTER TABLE tablename ADD UNIQUE (colname);
CREATE UNIQUE INDEX indexname ON tablename (colname);
```

### Add a Primary Key

Use a primary key constraint instead of `CREATE INDEX`.

```sql
ALTER TABLE tablename ADD PRIMARY KEY (col1, col2);
```

### Add a Functional Index

```sql
ALTER TABLE tablename ADD INDEX ((func(colname)));
CREATE INDEX indexname ON tablename ((func(colname)));
```

### Drop an Index

```sql
ALTER TABLE tablename DROP INDEX indexname;
DROP INDEX indexname ON tablename;
```

## Practical Tuning Flow

If you need a repeatable tuning sequence, use this order:

1. Measure the workload and capture the symptom.
2. Check CPU, memory, and storage behavior on the host.
3. Enable or review the slow query log.
4. Inspect Performance Schema for waits, locks, and hot statements.
5. Refresh table statistics with `ANALYZE TABLE` when needed.
6. Review indexes before raising buffer sizes.
7. Only then adjust configuration or SQL.

That order prevents guesswork and keeps changes tied to observed behavior.

## Final Takeaway

Treat performance tuning as a process that matches hardware, configuration, indexes, and workload behavior to application needs.

If you size the server correctly, configure InnoDB intentionally, watch the right bottlenecks, and validate query plans and index choices, you will solve more performance problems than you would by chasing one slow statement at a time.