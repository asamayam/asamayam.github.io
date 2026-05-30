---
title: "Essential MySQL Command-Line Utilities Every DBA Should Know - Part 1"
description: "Part 1 covers core MySQL command-line utilities for interactive administration, quick validation, metadata inspection, integrity checks, and secure credential handling."
date: 2025-07-18
categories: [Database]
tags: [MySQL, MySQL Utilities, MySQL Administration, MySQL DBA, Linux Administration, Database Administration, Oracle MySQL]
permalink: /posts/essential-mysql-command-line-utilities/
---

One of the reasons I've always enjoyed working with MySQL is the rich collection of command-line utilities that are included with the installation. While most administrators spend a significant amount of time inside the MySQL client itself, there are several supporting utilities that simplify administration, troubleshooting, validation, and automation.

After installing MySQL, one of the first things I typically do is review the utilities available on the server.

This is Part 1 of a two-part utilities series.

On Linux systems, these utilities are usually installed under:

```text
/usr/bin
```

A typical installation includes utilities such as:

```text
mysql
mysqladmin
mysqlshow
mysqlcheck
mysql_config_editor
mysqldump
mysqlbinlog
mysqlslap
```

Example terminal output:

```console
[root@mysqlhost bin]# pwd
/usr/bin
[root@mysqlhost bin]# ls -lrt mysql*
-rwxr-xr-x 1 root root     6681 Jun 22 05:08 mysqldumpslow
-rwxr-xr-x 1 root root    29137 Jun 22 05:08 mysqld_safe
-rwxr-xr-x 1 root root     3380 Jun 22 05:08 mysqld_pre_systemd
-rwxr-xr-x 1 root root     4029 Jun 22 05:08 mysql_config-64
-rwxr-xr-x 1 root root      840 Jun 22 05:08 mysql_config
-rwxr-xr-x 1 root root  7510432 Jun 22 05:47 mysql_upgrade
-rwxr-xr-x 1 root root  6473592 Jun 22 05:47 mysqlrouter_plugin_info
-rwxr-xr-x 1 root root   662672 Jun 22 05:47 mysqlrouter
-rwxr-xr-x 1 root root  6650168 Jun 22 05:47 mysql_keyring_encryption_test
-rwxr-xr-x 1 root root  7485296 Jun 22 05:47 mysqldump
-rwxr-xr-x 1 root root  7933752 Jun 22 05:47 mysql_client_test
-rwxr-xr-x 1 root root  9350520 Jun 22 05:47 mysqlxtest
-rwxr-xr-x 1 root root    17144 Jun 22 05:47 mysqltest_safe_process
-rwxr-xr-x 1 root root  7413952 Jun 22 05:47 mysqlslap
-rwxr-xr-x 1 root root   188680 Jun 22 05:47 mysqlrouter_keyring
-rwxr-xr-x 1 root root  7492336 Jun 22 05:47 mysql_migrate_keyring
-rwxr-xr-x 1 root root 14047128 Jun 22 05:47 mysqlbackup
-rwxr-xr-x 1 root root  7717912 Jun 22 05:47 mysql
-rwxr-xr-x 1 root root  7417176 Jun 22 05:47 mysqlcheck
-rwxr-xr-x 1 root root  7874304 Jun 22 05:47 mysqlbinlog
-rwxr-xr-x 1 root root  6431568 Jun 22 05:47 mysql_tzinfo_to_sql
-rwxr-xr-x 1 root root  6555808 Jun 22 05:47 mysql_ssl_rsa_setup
-rwxr-xr-x 1 root root  7394216 Jun 22 05:47 mysqlshow
-rwxr-xr-x 1 root root   151392 Jun 22 05:47 mysqlrouter_passwd
-rwxr-xr-x 1 root root  7400104 Jun 22 05:47 mysqlimport
-rwxr-xr-x 1 root root  6520432 Jun 22 05:47 mysql_config_editor
-rwxr-xr-x 1 root root  7403792 Jun 22 05:47 mysqladmin
-rwxr-xr-x 1 root root  8736128 Jun 22 05:47 mysqltest
-rwxr-xr-x 1 root root  7387664 Jun 22 05:47 mysql_secure_installation
-rwxr-xr-x 1 root root  8026184 Jun 22 05:47 mysqlpump
```

> This listing confirms that the core MySQL CLI utilities are available under `/usr/bin`.

In this article, I'll focus on five utilities that I use regularly when administering MySQL environments.

## 1. mysql - The Primary Administrative Interface

The `mysql` client is the most commonly used utility in the MySQL ecosystem.

It provides interactive access to the database server and is typically the first tool administrators use after installation.

Typical activities include:

- Running SQL statements
- Creating databases and tables
- Managing users and roles
- Troubleshooting issues
- Validating configurations

Basic connection:

```bash
mysql -uroot -p
```

Example:

```console
[root@mysqlhost01 ~]# mysql -uroot -p
Enter password:
mysql>
```

Although MySQL allows passwords to be supplied on the command line, I generally avoid this approach in production environments.

Example:

```bash
mysql -uroot -pPassword@123
```

This generates the warning:

```text
mysql: [Warning] Using a password on the command line interface can be insecure.
```

### Why It Matters

The `mysql` client remains the fastest way to perform administrative operations, validate configuration changes, and troubleshoot problems directly on the server.

## 2. mysqladmin - Quick Server Health Checks

The `mysqladmin` utility provides a simple way to retrieve server information without launching the interactive MySQL client.

One of my favorite uses is validating server availability after maintenance activities.

Example:

```bash
mysqladmin -u root -p version
```

Typical output:

```text
Server version          8.0.34
Protocol version        10
Connection              Localhost via UNIX socket
UNIX socket             /var/lib/mysql/mysql.sock
```

This single command immediately confirms:

- Server availability
- Version information
- Socket configuration
- Connectivity

### Why It Matters

When performing upgrades or troubleshooting startup issues, `mysqladmin` often provides quicker validation than opening an interactive session.

## 3. mysqlshow - Exploring Database Objects Quickly

The `mysqlshow` utility provides a fast way to inspect databases, tables, and table structures.

Display tables in a database:

```bash
mysqlshow test -uroot -p
```

Example output:

```text
Database: test

+----------------------+
| Tables               |
+----------------------+
| employee             |
| company              |
| work                 |
| salary               |
+----------------------+
```

Example terminal output:

```console
[root@mysqlhost01 ~]# mysqlshow test -uroot -p
Enter password:
Database: test
+------------------------------------------------------+
|                        Tables                        |
+------------------------------------------------------+
| employee                                             |
| company                                              |
| test                                                 |
| org                                                  |
| work                                                 |
| salary                                               |
| salary_ex                                            |
-- output truncate for better visibility
| work_log                                             |
| tables_priv                                          |
| time_zone                                            |
| time_zone_leap_second                                |
| time_zone_name                                       |
| time_zone_transition                                 |
| time_zone_transition_type                            |
| user                                                 |
+------------------------------------------------------+
```

> This output provides a fast inventory of tables in the `test` schema without entering the interactive MySQL shell.

You can also inspect a specific table:

```bash
mysqlshow -t test example -uroot -p
```

```console
[root@mysqlhost01 ~]# mysqlshow -t test example -uroot -p
Enter password:
Database: test  Table: example
+-------+------+-----------+------+-----+---------+-------+--------------------------+---------+
| Field | Type | Collation | Null | Key | Default | Extra | Privileges               | Comment |
+-------+------+-----------+------+-----+---------+-------+--------------------------+---------+
| Host  | char(255) | ascii_general_ci | NO | PRI |       |       | select,insert,update,references | |
| Db    | char(64)  | utf8mb3_bin      | NO | PRI |       |       | select,insert,update,references | |
| User  | char(32)  | utf8mb3_bin      | NO | PRI |       |       | select,insert,update,references | |
-- output truncate for better visibility
| Event_priv |     | utf8mb3_bin | NO | PRI |       |       | select,insert,update,references | |
+-----------------------+---------------+--------------------+------+
```

> This output is useful when you need table metadata quickly during troubleshooting or validation.

This displays:

- Column names
- Data types
- Nullability
- Index information

### Why It Matters

I frequently use `mysqlshow` when I need quick metadata without connecting interactively and executing multiple SQL statements.

## 4. mysqlcheck - Validating Table Health

As databases grow, administrators occasionally need to validate table integrity.

The `mysqlcheck` utility performs several useful maintenance tasks:

- Table validation
- Corruption detection
- Table repair
- Optimization checks

Example:

```bash
/usr/bin/mysqlcheck --databases test -uroot -p
```

Sample output:

```text
test.Companies OK
test.EMPLOYEE OK
test.EMPLOYEE1 OK
test.TESTTABLE1 OK
```

Example terminal output:

```console
[root@mysqlhost01 ~]# /usr/bin/mysqlcheck --databases test -uroot -p
Enter password:
test.Companies                                     OK
test.EMPLOYEE                                      OK
test.EMPLOYEE1                                     OK
test.TESTTABLE1                                    OK
test.employee_table                                OK
```

> The `OK` status indicates each table check completed successfully with no immediate integrity issues detected.

The `OK` status indicates that the objects were successfully checked and no issues were detected.

### Common Use Cases

I typically use `mysqlcheck` after:

- Unexpected server crashes
- Storage failures
- Filesystem issues
- Database migrations

## 5. mysql_config_editor - Eliminating Plain Text Passwords

One utility that I believe deserves more attention is `mysql_config_editor`.

Many organizations still embed passwords directly inside backup scripts.

Example:

```bash
mysqldump -uroot -pPassword123
```

Besides generating security warnings, this exposes credentials to anyone with access to the script.

Instead, create a login path:

```bash
mysql_config_editor set \
--login-path=backupUser \
--host=localhost \
--user=root \
--password
```

This creates:

```text
/root/.mylogin.cnf
```

Connections can then be established using:

```bash
mysql --login-path=backupUser
```

Example terminal output:

```console
[root@mysqlhost ~]# mysql_config_editor set --login-path=backupUser --host=localhost --user=root --password
[root@mysqlhost ~]# file /root/.mylogin.cnf
/root/.mylogin.cnf: data
[root@mysqlhost ~]# mysql --login-path=backupUser
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3699
Server version: 8.0.16 MySQL Community Server - GPL

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

> This sequence demonstrates creating a secure login path and then connecting without exposing credentials on the command line.

### Why It Matters

Benefits include:

- Encrypted credential storage
- Simplified automation
- Reduced password exposure
- Cleaner backup scripts

This utility has become one of my preferred methods for securing automated MySQL administration tasks.

## Final Thoughts

The MySQL client utilities often receive less attention than newer tools and graphical interfaces, but they remain essential for database administration.

The five utilities covered in this article are among the ones I use most frequently:

| Utility | Primary Purpose |
| --- | --- |
| `mysql` | Interactive administration |
| `mysqladmin` | Server validation |
| `mysqlshow` | Metadata inspection |
| `mysqlcheck` | Table validation |
| `mysql_config_editor` | Secure credential management |

Mastering these utilities can significantly improve efficiency when managing MySQL environments and often eliminates the need for repetitive manual tasks.

In Part 2, I cover the remaining MySQL utilities focused on backup and restore operations, binary log analysis, server safety, and performance diagnostics.
