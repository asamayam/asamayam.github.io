---
title: "Practical MySQL Tablespace and Partitioning - Part 3: Managing UNDO and Temporary Tablespaces"
description: "Part 3 covers relocating UNDO tablespaces and controlling InnoDB temporary tablespace growth with configuration-driven operational steps."
date: 2026-03-06
categories: [Database]
tags: [MySQL, InnoDB, UNDO Tablespace, Temporary Tablespace, innodb_undo_directory, MySQL Administration]
permalink: /posts/practical-mysql-tablespace-and-partitioning-part-3/
---

Part 3 covers UNDO and temporary tablespace management in MySQL 8.0.

These two areas are operationally important: UNDO affects transaction rollback and MVCC behavior, while temporary tablespace growth can become a capacity issue in busy environments.

## Move UNDO Tablespaces

Typical workflow:

1. Inspect current undo files.
2. Set `innodb_fast_shutdown=0`.
3. Stop MySQL.
4. Move undo files to target directory.
5. Configure `innodb-undo-directory`.
6. Start MySQL and validate.

Command pattern:

```bash
ls -lrth /var/lib/mysql/undo*
```

```sql
SHOW GLOBAL VARIABLES LIKE 'innodb_fast_shutdown';
SET GLOBAL innodb_fast_shutdown = 0;
```

```bash
sudo systemctl stop mysqld
sudo mv /var/lib/mysql/undo_* /var/lib/mysql/innodb/
sudo chown -R mysql:mysql /var/lib/mysql
```

Configuration snippet:

```ini
innodb-undo-directory=/var/lib/mysql/innodb/
```

## Validate UNDO Move

```bash
sudo systemctl start mysqld
```

Then check:

```sql
SHOW GLOBAL VARIABLES LIKE 'innodb_undo%';
```

## Resize Temporary Tablespace

If you need to cap or tune temporary tablespace growth, configure `innodb-temp-data-file-path`.

Example:

```ini
innodb-temp-data-file-path=ibtmp1:12M:autoextend:max:2G
```

Verification queries:

```sql
SELECT @@innodb_temp_tablespaces_dir;
SELECT @@innodb_temp_data_file_path;

SELECT file_name, tablespace_name, initial_size,
       total_extents * extent_size AS totalsizebytes,
       data_free, maximum_size
FROM information_schema.files
WHERE tablespace_name='innodb_temporary'\G
```

## Practical Guidance

- Use predictable directory standards for easier backup and incident response.
- Keep ownership/permissions checks in every move procedure.
- Always validate both config variables and actual file placement.

In Part 4, I move to file-per-table and general tablespace operations.
