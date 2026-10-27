# PostgreSQL 18.6 and pgAdmin 4 Installation on Ubuntu 22.04

Complete installation and configuration guide for **PostgreSQL 18.6** and **pgAdmin 4 Desktop** on Ubuntu 22.04 LTS (Jammy).

## Table of Contents

1. [Environment and Prerequisites](#1-environment-and-prerequisites)
2. [Check Disk Space](#2-check-disk-space)
3. [Add PostgreSQL Repository](#3-add-postgresql-repository)
4. [Install PostgreSQL 18](#4-install-postgresql-18)
5. [Verify PostgreSQL Installation](#5-verify-postgresql-installation)
6. [Connect Using the Terminal](#6-connect-using-the-terminal)
7. [Install pgAdmin 4 Desktop](#7-install-pgadmin-4-desktop)
8. [Configure PostgreSQL Password](#8-configure-postgresql-password)
9. [Connect pgAdmin to PostgreSQL](#9-connect-pgadmin-to-postgresql)
10. [Useful PostgreSQL Commands](#10-useful-postgresql-commands)
11. [Create a Database and Application User](#11-create-a-database-and-application-user)
12. [Backup and Restore](#12-backup-and-restore)
13. [Troubleshooting](#13-troubleshooting)
14. [Official Documentation](#14-official-documentation)

---

## 1. Environment and Prerequisites

### System Requirements

- Operating System: Ubuntu 22.04 LTS
- Architecture: 64-bit x86_64
- Database: PostgreSQL 18
- GUI: pgAdmin 4 Desktop
- Default PostgreSQL port: `5432`

Check the Ubuntu version:

```bash
cat /etc/os-release
```

Check the system architecture:

```bash
uname -m
```

Expected architecture:

```text
x86_64
```

## 2. Check Disk Space

Before installing software, check available disk space:

```bash
df -h / /var /media/vdata
```

Ensure `/var` has enough free space. Ubuntu's APT package manager requires space for package indexes, downloaded packages, and package-management records.

If `/var` is full, package installation may fail.

**Important:** On systems with existing Docker applications or MySQL databases, do not delete Docker containers, images, containerd data, or MySQL files to free space without a specific, verified storage plan.

## 3. Add PostgreSQL Repository

Ubuntu's default repository might not provide the PostgreSQL version you want. Add the official PostgreSQL Global Development Group (PGDG) APT repository.

### Step 1: Install prerequisites

```bash
sudo apt update
sudo apt install -y postgresql-common ca-certificates
```

### Step 2: Configure the PostgreSQL repository

```bash
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
```

If prompted, press **Enter** to continue.

### Step 3: Refresh package indexes

```bash
sudo apt update
```

The repository setup script configures the PostgreSQL repository for your Ubuntu release.

## 4. Install PostgreSQL 18

Install the PostgreSQL server and command-line client:

```bash
sudo apt install -y postgresql-18 postgresql-client-18
```

This installs:

- PostgreSQL 18 database server
- `psql` command-line client
- Required PostgreSQL dependencies

### If you encounter `Hash Sum mismatch`

Clear cached package archives and refresh the package indexes:

```bash
sudo apt clean
sudo rm -rf /var/lib/apt/lists/*
sudo apt update
```

Retry the installation:

```bash
sudo apt install -y postgresql-18 postgresql-client-18
```

The `rm` command above removes cached APT index files only. The subsequent `apt update` downloads them again.

If the error persists, investigate repository mirror or proxy caching. Never disable package checksum or signature verification.

## 5. Verify PostgreSQL Installation

### Check the installed version

```bash
psql --version
```

Example output:

```text
psql (PostgreSQL) 18.6 (Ubuntu 18.6-1.pgdg22.04+2)
```

### Check PostgreSQL clusters

```bash
pg_lsclusters
```

Expected output resembles:

```text
Ver Cluster Port Status Owner    Data directory
18  main    5432 online postgres /var/lib/postgresql/18/main
```

The important values are:

- Version: `18`
- Cluster: `main`
- Port: `5432`
- Status: `online`
- Owner: `postgres`

### Check the service

```bash
sudo systemctl status postgresql
```

The umbrella service may display `active (exited)`. This can be normal. Use `pg_lsclusters` to verify that the actual PostgreSQL cluster is `online`.

## 6. Connect Using the Terminal

Connect as the PostgreSQL administrator:

```bash
sudo -u postgres psql
```

You should see a prompt similar to:

```text
postgres=#
```

Check the PostgreSQL version:

```sql
SELECT version();
```

List databases:

```text
\l
```

List database roles:

```text
\du
```

Exit the PostgreSQL terminal:

```text
\q
```

## 7. Install pgAdmin 4 Desktop

pgAdmin 4 provides a graphical interface for managing databases, tables, users, queries, and other PostgreSQL objects.

### Step 1: Install prerequisites

```bash
sudo apt install -y curl gnupg
```

### Step 2: Create the APT keyring directory

```bash
sudo install -d /etc/apt/keyrings
```

### Step 3: Add the pgAdmin repository signing key

```bash
curl -fsS https://www.pgadmin.org/static/packages_pgadmin_org.pub | sudo gpg --dearmor --yes -o /etc/apt/keyrings/packages-pgadmin-org.gpg
```

### Step 4: Add the pgAdmin repository

```bash
echo "deb [signed-by=/etc/apt/keyrings/packages-pgadmin-org.gpg] https://ftp.postgresql.org/pub/pgadmin/pgadmin4/apt/$(lsb_release -cs) pgadmin4 main" | sudo tee /etc/apt/sources.list.d/pgadmin4.list
```

### Step 5: Update package indexes

```bash
sudo apt update
```

### Step 6: Install pgAdmin Desktop

```bash
sudo apt install -y pgadmin4-desktop
```

### Step 7: Launch pgAdmin

1. Open the Ubuntu **Applications** menu.
2. Search for `pgAdmin 4`.
3. Launch the application.

## 8. Configure PostgreSQL Password

When connecting through pgAdmin, you may encounter:

```text
FATAL: password authentication failed for user "postgres"
```

This means PostgreSQL rejected the supplied password. Set a password for the PostgreSQL database role.

### Step 1: Open the PostgreSQL terminal

```bash
sudo -u postgres psql
```

### Step 2: Set the password

At the `postgres=#` prompt, execute:

```text
\password postgres
```

Enter your new password twice when prompted.

### Step 3: Exit

```text
\q
```

**Security note:** The PostgreSQL password is different from your Ubuntu login password. Use a strong password and do not commit it to GitHub.

## 9. Connect pgAdmin to PostgreSQL

Open pgAdmin 4.

1. Right-click **Servers**.
2. Select **Register**.
3. Select **Server**.

### General Tab

| Setting | Value |
|---|---|
| Name | Local PostgreSQL 18 |

### Connection Tab

| Setting | Value |
|---|---|
| Host name/address | `127.0.0.1` |
| Port | `5432` |
| Maintenance database | `postgres` |
| Username | `postgres` |
| Password | The password configured in Section 8 |
| Save password | Optional |

Click **Save**.

After connecting, expand:

```text
Servers
└── Local PostgreSQL 18
    └── Databases
        └── postgres
```

You can now create databases, execute SQL queries, manage tables, and administer PostgreSQL through the GUI.

## 10. Useful PostgreSQL Commands

### Start PostgreSQL

```bash
sudo systemctl start postgresql
```

### Stop PostgreSQL

```bash
sudo systemctl stop postgresql
```

### Restart PostgreSQL

```bash
sudo systemctl restart postgresql
```

### Enable PostgreSQL at system startup

```bash
sudo systemctl enable postgresql
```

### Check cluster status

```bash
pg_lsclusters
```

### Check whether port 5432 is listening

```bash
sudo ss -ltnp | grep 5432
```

### Connect as the PostgreSQL administrator

```bash
sudo -u postgres psql
```

### Useful commands inside `psql`

| Command | Description |
|---|---|
| `\l` | List databases |
| `\du` | List roles |
| `\dt` | List tables in the current schema |
| `\conninfo` | Show connection information |
| `\q` | Exit `psql` |

## 11. Create a Database and Application User

For application development, create a dedicated database user instead of connecting applications as the PostgreSQL superuser.

Open the PostgreSQL terminal:

```bash
sudo -u postgres psql
```

Create an application role:

```sql
CREATE ROLE appuser
WITH LOGIN PASSWORD 'Replace-With-A-Strong-Password';
```

Create a database owned by that role:

```sql
CREATE DATABASE appdb OWNER appuser;
```

Exit:

```text
\q
```

Replace the example password with a strong, unique password before executing the SQL. Do not use the example password in production.

## 12. Backup and Restore

### Create a database backup

Example for a database named `appdb`:

```bash
sudo -u postgres pg_dump -Fc appdb > appdb_backup.dump
```

The backup file is written to the current directory. Ensure the destination has sufficient space and appropriate permissions.

### Restore a database backup

Restore into an existing database with appropriate ownership and permissions:

```bash
pg_restore --clean --if-exists --no-owner -d appdb appdb_backup.dump
```

Test backup restoration regularly. For production, maintain a backup schedule, retention policy, and a copy stored separately from the database server.

## 13. Troubleshooting

### Issue 1: `Unable to locate package postgresql-18`

**Cause:** The PGDG repository may not be configured, or package indexes may be stale.

**Solution:**

```bash
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
sudo apt update
sudo apt install -y postgresql-18 postgresql-client-18
```

### Issue 2: `Hash Sum mismatch`

**Solution:**

```bash
sudo apt clean
sudo rm -rf /var/lib/apt/lists/*
sudo apt update
sudo apt install -y postgresql-18 postgresql-client-18
```

If it persists, investigate repository mirror or proxy caching. Do not bypass checksum validation.

### Issue 3: pgAdmin password authentication failed

**Solution:**

```bash
sudo -u postgres psql
```

Then:

```text
\password postgres
```

Exit with `\q`, and enter the new password in pgAdmin.

### Issue 4: PostgreSQL service shows `active (exited)`

This can be normal for the umbrella service.

Check the actual database cluster:

```bash
pg_lsclusters
```

Confirm that the `18 main` cluster status is `online`.

### Issue 5: `/var` is full

Check available space:

```bash
df -h /var
```

Identify the largest directories:

```bash
sudo du -xhd1 /var | sort -h
```

Inspect the results before deleting anything. Do not remove active Docker/containerd files, Docker images, the `itmsvas` container, or MySQL data. Plan storage changes carefully if a permanent capacity increase is required.

## 14. Official Documentation

- [PostgreSQL for Ubuntu](https://www.postgresql.org/download/linux/ubuntu/)
- [PostgreSQL APT Repository Wiki](https://wiki.postgresql.org/wiki/Apt)
- [Ubuntu Server: Install and Configure PostgreSQL](https://ubuntu.com/server/docs/install-and-configure-postgresql/)
- [pgAdmin 4 APT Installation](https://www.pgadmin.org/download/pgadmin-4-apt/)

---

**Installation completed:** PostgreSQL 18.6 and pgAdmin 4 Desktop on Ubuntu 22.04 LTS.

**Default PostgreSQL port:** `5432`

**PostgreSQL configuration directory:** `/etc/postgresql/18/main`

**PostgreSQL data directory:** `/var/lib/postgresql/18/main`
