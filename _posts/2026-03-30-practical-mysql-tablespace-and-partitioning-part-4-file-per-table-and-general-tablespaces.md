---
title: "Practical MySQL Tablespace and Partitioning - Part 4: File-per-Table and General Tablespaces"
description: "Part 4 demonstrates file-per-table and general tablespace operations, including external tablespace paths, table moves, and key constraints for partitioned tables."
date: 2026-03-30
categories: [Database]
tags: [MySQL, InnoDB, File-per-Table, General Tablespace, innodb_directories, MySQL Administration]
permalink: /posts/practical-mysql-tablespace-and-partitioning-part-4/
---

Part 4 explores practical file-per-table and general tablespace operations.

This is where logical design and physical placement intersect. You can control where table data lives, but you must stay within InnoDB rules.

## Confirm File-per-Table Mode

```sql
SHOW VARIABLES LIKE 'innodb_file_per_table';
```

With `ON`, each InnoDB table typically maps to its own `.ibd` file.

## Configure External Directories for General Tablespaces

If you create a general tablespace outside `datadir`, configure allowed directories.

```ini
innodb-directories=/var/lib/tbs/
```

Then restart and validate:

```sql
SHOW VARIABLES LIKE 'innodb_directories';
```

## Create General Tablespace and Move Table

Example pattern:

```sql
CREATE TABLESPACE db1_tbs ADD DATAFILE '/var/lib/tbs/db1_tbs.ibd';
```

Inspect metadata:

```sql
SELECT name
FROM information_schema.innodb_tablespaces;

SELECT file_name, tablespace_name, extent_size, initial_size, autoextend_size
FROM information_schema.files
WHERE tablespace_name='db1_tbs'\G
```

Move a non-partitioned table:

```sql
ALTER TABLE emp_range TABLESPACE db1_tbs;
```

## Important Constraint: Partitioned Tables

A partitioned table cannot be placed in a shared general tablespace in this context.

You can encounter errors like:

```text
ERROR 1478 (HY000): InnoDB: A partitioned table is not allowed in a shared tablespace.
```

So evaluate table design first, then decide tablespace strategy.

## Operational Checklist

- Validate directory ownership before creating tablespace files.
- Keep naming conventions clear for tablespace files.
- Use metadata queries after each structural change.

In Part 5, I begin partitioning with RANGE and LIST patterns.
