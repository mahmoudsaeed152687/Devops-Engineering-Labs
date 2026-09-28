

## Problem

MariaDB was down on the database server, causing the Nautilus application to lose database connectivity.

## How to Think

The MariaDB process runs as the `mysql` Linux user and needs to create:

```text
/run/mariadb/mariadb.pid
```

The directory was owned by `root:mysql`, and the `mysql` user did not have write permission.

check logs >>>> sudo tail -50 /var/log/mariadb/mariadb.log


## Solution

Check the directory ownership:

```bash
ls -ld /run/mariadb
```

Fix the ownership:

```bash
sudo chown mysql:mysql /run/mariadb
```

Start MariaDB:

```bash
sudo systemctl start mariadb
```

## Verification

```bash
sudo systemctl status mariadb
```

Expected:

```text
Active: active (running)
```

## Key Takeaway

When a service fails with `Permission denied`, check the service user's permissions on the files and directories it needs to access.
