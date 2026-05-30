---
title: "Practical MySQL Tablespace and Partitioning - Part 6: COLUMNS, HASH, KEY, and Subpartitioning"
description: "Part 6 covers advanced partitioning patterns in MySQL, including COLUMNS, HASH, KEY, and composite subpartitioning with verification workflows."
date: 2026-01-31
categories: [Database]
tags: [MySQL, Partitioning, COLUMNS Partitioning, HASH Partitioning, KEY Partitioning, Subpartitioning, MySQL Administration]
permalink: /posts/practical-mysql-tablespace-and-partitioning-part-6/
---

Part 6 closes the Chapter 4 series with advanced partitioning patterns.

These methods are useful when RANGE/LIST alone do not match the distribution characteristics of your workload.

## 1. COLUMNS Partitioning

COLUMNS partitioning extends RANGE/LIST concepts to multiple columns and supports non-integer types such as date values.

Example pattern:

```sql
CREATE TABLE emp_columns (
  id INT NOT NULL,
  fname VARCHAR(30),
  lname VARCHAR(30),
  hired DATE NOT NULL DEFAULT '2023-01-01',
  position INT NOT NULL,
  fired VARCHAR(5) NOT NULL DEFAULT 'No',
  dep_id INT NOT NULL
)
PARTITION BY RANGE COLUMNS(fname,lname,hired) (
  PARTITION p1 VALUES LESS THAN ('a','a','2023-02-02'),
  PARTITION p2 VALUES LESS THAN ('z','z','2099-12-31')
);
```

## 2. HASH Partitioning

HASH distributes rows using a user-defined expression.

```sql
CREATE TABLE emp_hash (
  id INT NOT NULL,
  fname VARCHAR(30),
  lname VARCHAR(30),
  hired DATE NOT NULL DEFAULT '2023-01-01',
  position INT NOT NULL,
  fired VARCHAR(5) NOT NULL DEFAULT 'No',
  dep_id INT NOT NULL
)
PARTITION BY HASH(id)
PARTITIONS 5;
```

Check spread:

```sql
SELECT partition_name, table_rows
FROM information_schema.partitions
WHERE table_name='emp_hash';
```

## 3. KEY Partitioning

KEY partitioning uses MySQL's internal hashing logic instead of a user-defined hash expression.

```sql
CREATE TABLE emp_key (
  id INT NOT NULL PRIMARY KEY,
  fname VARCHAR(30),
  lname VARCHAR(30),
  hired DATE NOT NULL DEFAULT '2023-01-01',
  position INT NOT NULL,
  fired VARCHAR(5) NOT NULL DEFAULT 'No',
  dep_id INT NOT NULL
)
PARTITION BY KEY()
PARTITIONS 4;
```

## 4. Subpartitioning (Composite Partitioning)

Subpartitioning means partitions within partitions.

Example structure:

```sql
CREATE TABLE emp_subpart (
  bill_no INT,
  sale_date DATE,
  cust_code VARCHAR(15),
  amount DECIMAL(8,2)
)
PARTITION BY RANGE (YEAR(sale_date))
SUBPARTITION BY HASH (TO_DAYS(sale_date))
SUBPARTITIONS 4 (
  PARTITION p0 VALUES LESS THAN (1990),
  PARTITION p1 VALUES LESS THAN (2000),
  PARTITION p2 VALUES LESS THAN (2010),
  PARTITION p3 VALUES LESS THAN MAXVALUE
);
```

Inspect partition/subpartition metadata:

```sql
SELECT partition_name, table_rows
FROM information_schema.partitions
WHERE table_name='emp_subpart';
```

## Operational Wrap-Up

Across this chapter, the most useful pattern is consistent verification after each DDL change:

- `information_schema.partitions`
- `information_schema.files`
- `SHOW VARIABLES` checks
- `EXPLAIN` for partition-aware query behavior

This closes the practical tablespace and partitioning series. In the next chapter flow, I move to high availability and replication topics.
