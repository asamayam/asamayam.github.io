---
title: "Practical MySQL Tablespace and Partitioning - Part 5: RANGE and LIST Partitioning"
description: "Part 5 introduces RANGE and LIST partitioning with hands-on examples, partition-boundary errors, and maintenance patterns such as MAXVALUE partitions."
date: 2026-01-16
categories: [Database]
tags: [MySQL, Partitioning, RANGE Partitioning, LIST Partitioning, MySQL Administration, Database Performance]
permalink: /posts/practical-mysql-tablespace-and-partitioning-part-5/
---

Part 5 starts the partitioning half of the series.

Partitioning helps when large datasets need predictable pruning boundaries and easier lifecycle operations. In this part, I focus on RANGE and LIST because they are usually the first practical patterns teams adopt.

## RANGE Partitioning Basics

Example table definition:

```sql
CREATE TABLE emp_range (
  id INT NOT NULL,
  fname VARCHAR(30),
  lname VARCHAR(30),
  hired DATE NOT NULL DEFAULT '2023-01-01',
  position INT NOT NULL,
  fired VARCHAR(5) NOT NULL DEFAULT 'No'
)
PARTITION BY RANGE (id) (
  PARTITION p0 VALUES LESS THAN (5),
  PARTITION p1 VALUES LESS THAN (10),
  PARTITION p2 VALUES LESS THAN (15),
  PARTITION p3 VALUES LESS THAN (20)
);
```

Inspect distribution:

```sql
SELECT partition_name, table_rows
FROM information_schema.partitions
WHERE table_name='emp_range';
```

## Common RANGE Error and Fix

Inserting value outside defined ranges can fail:

```text
ERROR 1526 (HY000): Table has no partition for value 23
```

Add catch-all partition:

```sql
ALTER TABLE emp_range
ADD PARTITION (PARTITION p4 VALUES LESS THAN MAXVALUE);
```

## LIST Partitioning Basics

LIST uses explicit value sets per partition.

```sql
CREATE TABLE emp_list (
  id INT NOT NULL,
  fname VARCHAR(30),
  lname VARCHAR(30),
  hired DATE NOT NULL DEFAULT '2023-01-01',
  position INT NOT NULL,
  fired VARCHAR(5) NOT NULL DEFAULT 'No',
  dep_id INT NOT NULL
)
PARTITION BY LIST(dep_id) (
  PARTITION first_dep VALUES IN (3,5,20),
  PARTITION second_dep VALUES IN (25,50,75),
  PARTITION third_dep VALUES IN (80,85,90,100,120,140,150)
);
```

Out-of-list values trigger the same class of partition-missing error.

## Validation Workflow

For both RANGE and LIST:

```sql
SELECT partition_name, table_rows
FROM information_schema.partitions
WHERE table_name IN ('emp_range','emp_list');

EXPLAIN SELECT * FROM emp_range;
EXPLAIN SELECT * FROM emp_list;
```

## Practical Design Notes

- Use RANGE for natural ordered growth keys.
- Use LIST for explicit business bucket values.
- Decide early how you will handle future values to avoid frequent emergency DDL.

In Part 6, I cover COLUMNS, HASH, KEY, and subpartitioning.
