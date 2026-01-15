# PostgreSQL User and Database Setup Guide

This guide provides step-by-step commands to set up a PostgreSQL user and database, check the service status, and grant privileges.

---
## Mock Question Example

**Scenario:**
The Nautilus application development team plans to deploy a new application in Stratos DC, which uses PostgreSQL. The database server is already installed.

**Requirements:**

1. Create a database user `kodekloud_roy` with password `BruCstnMT5`.
2. Create a database `kodekloud_db1` and grant full permissions to `kodekloud_roy`.

**Commands:**

```sql
CREATE USER kodekloud_roy WITH LOGIN ENCRYPTED PASSWORD 'BruCstnMT5';
CREATE DATABASE kodekloud_db1;
GRANT ALL PRIVILEGES ON DATABASE kodekloud_db1 TO kodekloud_roy;
```

**Note:** Do not restart the PostgreSQL server service.

---

## 1. Log into DB Server

```bash
ssh peter@stdb01
```

## 2. Switch to Sudo User

```bash
sudo su -
```

## 3. Check PostgreSQL Service

Ensure the PostgreSQL service is running:

```bash
systemctl status postgresql
```

## 4. Access PostgreSQL Terminal

```bash
sudo -u postgres psql
```

## 5. Check Available Database Users

```sql
\du
```

## 6. See Available Databases

```sql
\l
```

## 7. Create Database User

**Option 1: Simple password**

```sql
CREATE USER kodekloud_cap WITH LOGIN PASSWORD '8FmzjvFU6S';
```

**Option 2: Encrypted password** (recommended)

```sql
CREATE USER kodekloud_cap WITH LOGIN ENCRYPTED PASSWORD '8FmzjvFU6S';
```

## 8. Verify Database Users

```sql
\du
```

## 9. Create Database

```sql
CREATE DATABASE kodekloud_db4;
```

## 10. Verify Available Databases

```sql
\l
```

## 11. Grant Full Privileges to Database User

```sql
GRANT ALL PRIVILEGES ON DATABASE kodekloud_db4 TO kodekloud_cap;
```

## 12. Verify Privileges

```sql
\l
```

## 13. Exit PostgreSQL Terminal

```sql
exit
```

---



