---
title: "Practical MySQL Backup Workflows: mysqldump, mydumper, and XtraBackup"
description: "A practical guide to MySQL backup and restore workflows using mysqldump, mydumper/myloader, and Percona XtraBackup, with examples for single databases, selective tables, compression, and recovery preparation."
date: 2026-04-12
categories: [Database]
tags: [MySQL, Backup, Restore, mysqldump, mydumper, myloader, XtraBackup, Database Administration]
permalink: /posts/practical-mysql-backup-workflows/
---

This article pulls the Chapter 8 material into one operational guide instead of splitting it into a series.

Production MySQL teams often start serious backup planning only after the first restore request arrives. Start earlier. Build a usable backup process around three elements: a tool that fits the workload, a repeatable command pattern, and a restore procedure you have already verified.

This guide walks through three practical backup paths covered in the source material:

- `mysqldump` for straightforward logical exports
- `mydumper` and `myloader` for faster multithreaded logical backup and restore
- `Percona XtraBackup` for hot physical backup and incremental recovery workflows

## 1. Use `mysqldump` for Simple Logical Exports

`mysqldump` remains the easiest way to capture MySQL objects as SQL statements that you can review, store, and replay later.

Many environments still benefit from `mysqldump` when the job calls for a simple export of one database, a few tables, or schema-only metadata.

### Single Database Backup

```bash
mysqldump -u root -p employees > single_db_bk_employees.sql
```

### Multiple Databases

```bash
mysqldump --databases -u root -p db2 db3 employees > multiple_db_bk.sql
```

### All Databases

```bash
mysqldump --all-databases -u root -p > alldbs.sql
```

### Schema-Only Backup

Use `--no-data` when you need DDL without row data.

```bash
mysqldump -u root -p --no-data employees > employees_metadata.sql
```

### Single Table Backup

```bash
mysqldump -u root -p db1 tab1 > db1_tab1_table.sql
```

### Table Schema Only

```bash
mysqldump -u root -p --no-data db1 tab1 > db1_emp_table_metadata.sql
```

### Table Data Only

Use `--no-create-info` when the target schema already exists and you only want row data.

```bash
mysqldump -u root -p --no-create-info db1 tab1 > db1_emp_data.sql
```

### Exclude a Table

```bash
mysqldump -u root -p db1 --ignore-table=db1.emp > db1_wo_emp_table.sql
```

### Compress During Backup

```bash
mysqldump -u root -p db1 | gzip > db1_gzip_compressed.sql.gz
```

### Add a Timestamp to the Output File

```bash
mysqldump -u root -p db1 > db1-$(date +%Y%m%d).sql
```

### Take a Global Read Lock During Backup

```bash
mysqldump -u root -p --lock-all-tables db1 > db1_global_readlock.sql
```

### Record Binary Log Coordinates

If you plan to use the dump for replication bootstrap or point-in-time recovery planning, include source log metadata.

```bash
mysqldump -u root -p --master-data db1 > db1_master_data.sql
```

## 2. Restore a `mysqldump` Backup Carefully

Treat a logical backup as incomplete until you can run a predictable restore.

Basic restore pattern:

```bash
mysql db1 < db1.sql > db1_restore.log
```

After the restore, validate the target database immediately:

```sql
SHOW DATABASES;
USE db1;
SHOW TABLES;
```

That quick verification step catches more issues than most teams expect, especially when the backup contains only part of the original schema.

## 3. Use `mydumper` and `myloader` for Faster Logical Backups

`mydumper` solves the biggest operational limitation of `mysqldump`: single-threaded execution. On larger datasets, multithreaded logical backup can reduce runtime significantly.

`mydumper` writes the dump files. `myloader` reads the backup set and restores the objects into MySQL.

### Install the Tools

The source workflow installs the upstream GitHub releases. After you install the packages, confirm that both binaries are available.

```bash
which mydumper
which myloader
```

### Back Up a Single Database

```bash
mydumper \
  --database=db1 \
  --host=localhost \
  --user=root \
  --password='<strong-password>' \
  --outputdir=mysql_backup/ \
  -G -E -R \
  --threads=4 \
  --rows=10
```

Important flags from the source material:

- `-G` dumps triggers
- `-E` dumps events
- `-R` dumps routines
- `--threads` controls parallelism
- `--rows` controls chunk sizing behavior

### Restore with `myloader`

```bash
myloader \
  --host=localhost \
  --user=root \
  --password='<strong-password>' \
  --database=db1 \
  --directory=/home/mysql/mysql_backup/mysql_backup \
  --queries-per-transaction=10 \
  --threads=4 \
  --verbose=3
```

The source examples validate restore success by dropping the database first, loading it back, and then checking the database and table inventory.

### Back Up Selected Databases with Regex

```bash
mydumper \
  --host=localhost \
  --user=root \
  --password='<strong-password>' \
  --outputdir=/home/mysql/mysql_backup/mysql_backup \
  --rows=50000 \
  -G -E -R \
  --threads=4 \
  --regex '^(db3\.|db4\.)' \
  -L /tmp/mydumper-logs.txt
```

### Back Up Selected Tables

```bash
mydumper \
  --host=localhost \
  --user=root \
  --password='<strong-password>' \
  --outputdir=/home/mysql/mysql_backup/mysql_backup \
  --rows=50000 \
  -G -E -R \
  --threads=8 \
  --regex '^(db1\.emp$|db1\.country$)' \
  -L /tmp/mydumper-logs.txt
```

### Back Up a Single Table with Compression

```bash
mydumper \
  --host=localhost \
  --user=root \
  --password='<strong-password>' \
  --outputdir=/home/mysql/mysql_backup/mysql_backup \
  --rows=50000 \
  -G -E -R \
  --threads=4 \
  --regex '^(db1\.mgr$)' \
  --compress \
  --verbose 3 \
  -L /tmp/mydumper-logs.txt
```

### Restore the Compressed Table Backup

```bash
myloader \
  --host=localhost \
  --user=root \
  --password='<strong-password>' \
  -B db1 \
  --directory=/home/mysql/mysql_backup/mysql_backup \
  --queries-per-transaction=50000 \
  --threads=4 \
  --verbose=3 \
  --overwrite-tables
```

Use this pattern when you want selective logical restore without replaying a full database export.

## 4. Use Percona XtraBackup When You Need Hot Physical Backups

Although XtraBackup does not produce logical dumps, this guide includes it because many MySQL backup strategies combine logical and physical methods.

Use XtraBackup when you need:

- Hot backups against active InnoDB workloads
- Faster recovery of large datasets
- Incremental backup support
- A backup that preserves physical storage state and binlog position metadata

### Install and Verify XtraBackup

The source workflow installs the Percona release package, enables the repository, and then installs `percona-xtrabackup-80`.

Validation commands:

```bash
rpm -qa | grep percona-xtra
xtrabackup --version
```

### Take a Full Backup

```bash
xtrabackup --backup \
  --user=root \
  --password='<strong-password>' \
  --target-dir=/var/lib/backup/
```

After the backup completes, inspect the output directory and metadata files such as:

- `xtrabackup_info`
- `xtrabackup_checkpoints`
- `xtrabackup_binlog_info`
- `backup-my.cnf`

Those files tell you whether the backup completed successfully and what binlog position it captured.

### Take an Incremental Backup

After the full backup, point the next backup to the full backup directory as the base.

```bash
xtrabackup --backup \
  --user=root \
  --password='<strong-password>' \
  --target-dir=/var/lib/incremental_backup/ \
  --incremental-basedir=/var/lib/backup/
```

The resulting directory contains `.delta` and `.meta` files that represent changed pages relative to the full backup.

## 5. Prepare and Restore an XtraBackup Recovery Set

The source material simulates data loss by creating tables, taking an incremental backup, dropping those tables, and then restoring the prepared backup set.

### Prepare the Full Backup

```bash
xtrabackup --prepare=TRUE --apply-log-only=TRUE --target-dir=/var/lib/backup/
```

### Apply the Incremental Backup

```bash
xtrabackup --prepare=TRUE \
  --target-dir=/var/lib/backup/ \
  --incremental-dir=/var/lib/incremental_backup/
```

### Stop MySQL and Recreate the Target Directory

```bash
sudo systemctl stop mysqld
cd /var/lib/
mv mysql mysql_old
mkdir mysql
chown -R mysql:mysql /var/lib/mysql
```

### Copy Back the Prepared Backup

```bash
xtrabackup --copy-back --target-dir=/var/lib/backup/
```

### Start MySQL and Verify Recovery

```bash
sudo systemctl start mysqld
mysql -u root -p
```

Then validate the restored schema and tables:

```sql
SHOW DATABASES;
USE employees;
SHOW TABLES;
```

That validation step closes the loop. Skip it, and you only know that the process copied files; you still do not know whether the restored dataset is usable.

## How to Choose Between These Tools

Use `mysqldump` when you need portability, simple exports, schema-only dumps, or targeted object extraction.

Use `mydumper` and `myloader` when logical backup remains the right fit but `mysqldump` takes too long for the dataset size or restore window.

Use XtraBackup when you need hot physical backup, incremental capture, or faster recovery on larger MySQL environments.

In practice, many teams combine these approaches:

- Logical dumps for selective export and object-level recovery
- Physical backups for full-server protection and faster restore objectives

## Final Takeaway

Do not look for one backup tool to win every scenario. Match the backup method to the recovery objective.

If you only script the backup and never test the restore, the process remains unfinished. A working MySQL backup strategy includes verified recovery steps, metadata inspection, and enough operational discipline to reproduce the workflow under pressure.