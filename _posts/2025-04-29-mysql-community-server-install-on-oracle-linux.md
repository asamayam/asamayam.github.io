---
title: "Installing MySQL Community Server on Oracle Linux Using the MySQL Repository"
description: "Learn how to install MySQL Community Server on Oracle Linux using the official MySQL repository. This step-by-step guide covers repository configuration, package installation, service startup, password management, and security hardening."
date: 2025-04-29
categories: [Database]
tags: [MySQL, Oracle MySQL, MySQL Installation, Database Administration, MySQL DBA, MySQL Community Edition]
---

**Description**

Learn how to install MySQL Community Server on Oracle Linux using the official MySQL repository. This step-by-step guide covers repository configuration, package installation, service startup, password management, and security hardening.

## Introduction

One of the questions I get asked frequently by developers and junior DBAs is: What is the easiest and most reliable way to install MySQL on Linux?

Although MySQL supports multiple installation methods, I generally prefer using the official MySQL repository for Oracle Linux and Red Hat Enterprise Linux environments. Here is what I learned from my experience. Repository-based installations simplify dependency management, streamline future upgrades, and ensure that the server is installed using Oracle-supported packages.

In this article, I'll walk through the same installation approach I use when provisioning a fresh MySQL environment on Oracle Linux.

## Why Use the MySQL Repository?

MySQL can be installed using several methods:

* Repository installation
* RPM bundle installation
* Generic binary installation
* Container-based deployment
* Source code compilation

For most Linux environments, repository installation offers the best balance between simplicity and maintainability.

Some advantages include:

* Automatic package dependency resolution
* Simplified upgrades and patching
* Consistent package versions
* Easy integration with operating system package management

For this demonstration, I am using Oracle Linux, but the same approach applies to most RHEL-compatible distributions.

## Download the Repository Package

The first step is downloading the MySQL repository package from Oracle.

Navigate to the MySQL Community Downloads page and select the repository package that matches your Linux version.

```bash
wget https://dev.mysql.com/get/mysql80-community-release-el7-10.noarch.rpm
```

Once downloaded, verify that the file exists on the server.

```bash
ls -ltr
```

Example:

```text
-rw-r--r-- 1 root root 11472 mysql80-community-release-el7-10.noarch.rpm
```

Please see below for the terminal output:

```console
[root@mysql-a mysql_binaries]# wget https://dev.mysql.com/get/mysql80-community-release-el7-10.noarch.rpm

--2023-10-09 22:05:52--  https://dev.mysql.com/get/mysql80-community-release-el7-10.noarch.rpm

Resolving dev.mysql.com (dev.mysql.com)... 23.4.59.106, 2600:1404:6400:1691::2e31, 2600:1404:6400:1697::2e31

Connecting to dev.mysql.com (dev.mysql.com)|23.4.59.106|:443... connected.

HTTP request sent, awaiting response... 302 Moved Temporarily

Location: https://repo.mysql.com//mysql80-community-release-el7-10.noarch.rpm [following]

--2023-10-09 22:05:53--  https://repo.mysql.com//mysql80-community-release-el7-10.noarch.rpm

Resolving repo.mysql.com (repo.mysql.com)... 104.90.18.174, 2600:1404:6400:158b::1d68, 2600:1404:6400:1582::1d68

Connecting to repo.mysql.com (repo.mysql.com)|104.90.18.174|:443... connected.

HTTP request sent, awaiting response... 200 OK

Length: 11472 (11K) [application/x-redhat-package-manager]

Saving to: 'mysql80-community-release-el7-10.noarch.rpm'

100%[================================>] 11,472      --.-K/s   in 0s

2023-10-09 22:05:53 (390 MB/s) - 'mysql80-community-release-el7-10.noarch.rpm' saved [11472/11472]

[root@mysql-a mysql_binaries]# ls -ltr

total 12

-rw-r--r-- 1 root root 11472 Aug 23 12:38 mysql80-community-release-el7-10.noarch.rpm

[root@mysql-a mysql_binaries]#
```

## Install the Repository Package

Install the repository RPM using yum.

```bash
sudo yum install mysql80-community-release-el7-10.noarch.rpm
```

Once installation completes, verify that the MySQL repositories are enabled.

```bash
yum repolist enabled | grep mysql
```

Typical output will include repositories such as:

```text
mysql-connectors-community
mysql-tools-community
mysql80-community
```

Please see below for the terminal output:

```console
[root@mysql-s mysql_binaries]# sudo yum install mysql80-community-release-el7-9.noarch.rpm

Loaded plugins: fastestmirror

Examining mysql80-community-release-el7-9.noarch.rpm: mysql80-community-release-el7-9.noarch

Marking mysql80-community-release-el7-9.noarch.rpm to be installed

Resolving Dependencies

--> Running transaction check

---> Package mysql80-community-release.noarch 0:el7-9 will be installed

--> Finished Dependency Resolution

Dependencies Resolved

=========================================================

Package                   Arch    Version  Repository                                  Size

=========================================================

Installing:

mysql80-community-release  noarch  el7-9    /mysql80-community-release-el7-9.noarch     12 k

Transaction Summary

========================================================

Install  1 Package

Total size: 12 k

Installed size: 12 k

Is this OK [y/d/N]: y

Downloading packages:

Running transaction check

Running transaction test

Transaction test succeeded

Running transaction

Warning: RPMDB altered outside of yum.

  Installing : mysql80-community-release-el7-9.noarch           1/1

  Verifying  : mysql80-community-release-el7-9.noarch           1/1

Installed:

  mysql80-community-release.noarch 0:el7-9

Complete!

[root@mysql-s mysql_binaries]# yum repolist enabled | grep "mysql.*-community.*"

mysql-connectors-community/x86_64 MySQL Connectors Community                 227

mysql-tools-community/x86_64      MySQL Tools Community                      100

mysql80-community/x86_64          MySQL 8.0 Community Server                 425

[root@mysql-s mysql_binaries]#
```

At this stage, the server is configured to retrieve MySQL packages directly from Oracle's repository.

## Install MySQL Community Server

With the repository configured, install the MySQL server package.

```bash
sudo yum install mysql-community-server
```

One thing worth noting is that the installation process automatically resolves and installs required dependencies. Depending on the operating system version, you may also notice MariaDB libraries being replaced by MySQL-compatible packages.

The installation pulls in several components including:

* mysql-community-server
* mysql-community-client
* mysql-community-common
* mysql-community-client-plugins
* mysql-community-libs

Please see below for the terminal output:

```console
[root@mysql-s mysql_binaries]# sudo yum install mysql-community-server

Loaded plugins: fastestmirror

Loading mirror speeds from cached hostfile

 * base: ohioix.mm.fcix.net

 * epel: veronanetworks.mm.fcix.net

 * extras: mirror.team-cymru.com

 * updates: veronanetworks.mm.fcix.net

Resolving Dependencies

--> Running transaction check

<<<<< output truncated for readability >>>>>

Dependencies Resolved

======================================================

 Package                           Arch      Version                     Repository            Size

======================================================

Installing:

 mysql-community-libs              x86_64    8.0.34-1.el7                mysql80-community    1.5 M

     replacing  mariadb-libs.x86_64 1:5.5.65-1.el7

 mysql-community-libs-compat       x86_64    8.0.34-1.el7                mysql80-community    669 k

     replacing  mariadb-libs.x86_64 1:5.5.65-1.el7

 mysql-community-server            x86_64    8.0.34-1.el7                mysql80-community     64 M

Installing for dependencies:

 mysql-community-client            x86_64    8.0.34-1.el7                mysql80-community     16 M

 mysql-community-client-plugins    x86_64    8.0.34-1.el7                mysql80-community    3.6 M

 mysql-community-common            x86_64    8.0.34-1.el7                mysql80-community    666 k

 mysql-community-icu-data-files    x86_64    8.0.34-1.el7                mysql80-community    2.2 M

 net-tools                         x86_64    2.0-0.25.20131004git.el7    base                 306 k

<<<<< output truncated for readability >>>>>

Installed:

  mysql-community-libs.x86_64 0:8.0.34-1.el7     mysql-community-libs-compat.x86_64 0:8.0.34-1.el7

  mysql-community-server.x86_64 0:8.0.34-1.el7

Dependency Installed:

  mysql-community-client.x86_64 0:8.0.34-1.el7 mysql-community-client-plugins.x86_64 0:8.0.34-1.el7

  mysql-community-common.x86_64 0:8.0.34-1.el7 mysql-community-icu-data-files.x86_64 0:8.0.34-1.el7

  net-tools.x86_64 0:2.0-0.25.20131004git.el7

Replaced:

  mariadb-libs.x86_64 1:5.5.65-1.el7

Complete!
```

Understanding these packages becomes useful later when troubleshooting installation issues or performing upgrades.

## Start the MySQL Service

Once installation completes, start the MySQL service.

```bash
systemctl start mysqld
```

Verify that the process is running.

```bash
ps -ef | grep mysqld
```

You should see a running mysqld process.

Next, verify the service status.

```bash
systemctl status mysqld
```

Please see below for the terminal output:

```console
[root@mysql-s mysql_binaries]# systemctl start mysqld

[root@mysql-s mysql_binaries]# ps -ef|grep mysqld

mysql  1598     1  5 22:40 ?        00:00:01 /usr/sbin/mysqld

root   1643  1338  0 22:41 pts/0    00:00:00 grep --color=auto mysqld

[root@mysql-s mysql_binaries]# systemctl status mysqld
```

The first startup performs several important initialization tasks:

* Creates the MySQL data directory
* Generates SSL certificates
* Creates system schemas
* Creates the root account
* Generates a temporary root password

This initialization happens automatically and usually completes within a few seconds.

## Retrieve the Temporary Root Password

During initialization, MySQL generates a temporary password for the root account and stores it in the error log.

Retrieve it using:

```bash
grep 'temporary password' /var/log/mysqld.log
```

Example:

```text
A temporary password is generated for root@localhost
```

Make a note of this password as it will be required for the first login.

## Change the Root Password

Login using the temporary password.

```bash
mysql -u root -p
```

Immediately change the password.

```sql
ALTER USER 'root'@'localhost'
IDENTIFIED BY 'XXXXXXX';
```

Choose a password that complies with your organization's password policy.

A successful password change confirms that the installation and initialization completed correctly.

Please see below for the terminal output:

```console
[root@mysql-s mysql_binaries]# sudo grep 'temporary password' /var/log/mysqld.log

2023-07-29T03:40:53.483206Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: XXXXXXXX

[root@mysql-s mysql_binaries]# mysql -uroot -p

Enter password:

mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'XXXXX';

Query OK, 0 rows affected (0.00 sec)
```

## Secure the Installation

One step I strongly recommend for every new MySQL deployment is running the built-in security utility.

```bash
mysql_secure_installation
```

The utility guides you through several security-related tasks, including:

* Removing anonymous users
* Restricting remote root access
* Removing the test database
* Reloading privilege tables

Please see below for the terminal output:

```console
[root@mysql-s mysql]# mysql_secure_installation

Securing the MySQL server deployment.

Enter password for user root:

VALIDATE PASSWORD COMPONENT can be used to test passwords

and improve security. It checks the strength of password

and allows the users to set only those passwords which are

secure enough. Would you like to setup VALIDATE PASSWORD component?

Press y|Y for Yes, any other key for No: y

The password validation component is not available. Proceeding with the further steps without the component.

Using existing password for root.

Change the password for root ? ((Press y|Y for Yes, any other key for No) : n

 ... skipping.

By default, a MySQL installation has an anonymous user, allowing anyone to log into MySQL without having to have a user account created for them. This is intended only for testing, and to make the installation go a bit smoother. You should remove them before moving into a production environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) : y

Success.

Normally, root should only be allowed to connect from

'localhost'. This ensures that someone cannot guess at

the root password from the network.

 Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y

Success.

 By default, MySQL comes with a database named 'test' that anyone can access. This is also intended only for testing, and should be removed before moving into a production environment.

Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y

 - Dropping test database...

Success.

 - Removing privileges on test database...

Success.

Reloading the privilege tables will ensure that all changes

made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y

Success.

All done!

[root@mysql-s mysql]#
```

Although many administrators skip this step in development environments, I recommend including it in every installation checklist.

## Verify the MySQL Configuration File

The MySQL configuration file is located at `/etc/my.cnf` by default.

Please see below for the terminal output:

```console
[root@mysql-s mysql_binaries]# cat /etc/my.cnf

# For advice on how to change settings please see

# http://dev.mysql.com/doc/refman/8.0/en/server-configuration-defaults.html

[mysqld]

-- output truncated for better visibility

datadir=/var/lib/mysql

socket=/var/lib/mysql/mysql.sock

log-error=/var/log/mysqld.log

pid-file=/var/run/mysqld/mysqld.pid

[root@mysql-s mysql_binaries]#
```

## Validate the Installation

Now that the server is secured, perform a few basic validation checks. Please note that the installation creates a user named mysql and a group named mysql on the system.

Verify the server version:

```bash
mysqladmin -u root -p version
```

Connect to the database:

```bash
mysql -u root -p
```

Display the available databases:

```sql
SHOW DATABASES;
```

You should see:

```text
information_schema
mysql
performance_schema
sys
```

Please see below for the terminal output:

```console
[root@mysql-s mysql]# su - mysql

-bash-4.2$ mysql -u root -p

mysql> SHOW DATABASES;

+--------------------+

| Database           |

+--------------------+

| information_schema |

| mysql              |

| performance_schema |

| sys                |

+--------------------+

4 rows in set (0.00 sec)

 mysql> exit

[root@mysql-s mysql]# mysqladmin -u root -p version

Enter password:

mysqladmin  Ver 8.0.34 for Linux on x86_64 (MySQL Community Server - GPL)

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its

affiliates. Other names may be trademarks of their respective

owners.

Server version          8.0.34

Protocol version        10

Connection              Localhost via UNIX socket

UNIX socket             /var/lib/mysql/mysql.sock

Uptime:                 32 min 55 sec

Threads: 2  Questions: 7  Slow queries: 0  Opens: 130  Flush tables: 3  Open tables: 46  Queries per second avg: 0.003
```

At this point, the MySQL server is ready for application onboarding and further configuration.

## Final Thoughts

Installing MySQL using the official repository remains one of the simplest and most reliable deployment methods available on Linux.

While the installation itself only takes a few minutes, I always spend additional time validating the service startup, reviewing initialization logs, securing user accounts, and understanding the generated configuration. Those small checks often prevent larger issues later during application deployment.

In the next article, I'll look beyond the installation process and examine what actually happens behind the scenes when MySQL starts for the first time, including data directory creation, SSL certificate generation, system schema creation, and InnoDB initialization.