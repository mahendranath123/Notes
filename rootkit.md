# Linux Security Check and Safe Data Extraction Guide

## Purpose

This guide explains how to:

1. Install and run RKHunter to check for rootkits and malware.
2. Safely extract databases and application data from a potentially compromised server.
3. Transfer backups to a clean system for recovery.

---

# Part 1: RKHunter Security Scan

## Install RKHunter

```bash
sudo dnf install rkhunter -y
```

## Update Rootkit Definitions

```bash
sudo rkhunter --update
```

## Initialize File Properties Database

```bash
sudo rkhunter --propupd
```

> Run this command only after verifying that the system is in a trusted state.

## Run a Full Security Scan

```bash
sudo rkhunter --check --sk
```

### Command Options

| Option    | Description                    |
| --------- | ------------------------------ |
| `--check` | Perform a complete system scan |
| `--sk`    | Skip interactive prompts       |

## Review Scan Results

View log file:

```bash
sudo less /var/log/rkhunter.log
```

Show warnings only:

```bash
grep Warning /var/log/rkhunter.log
```

## Useful Commands

### Check Version

```bash
rkhunter --version
```

### Validate Configuration

```bash
rkhunter --config-check
```

### Generate Warning Report Only

```bash
sudo rkhunter --check --report-warnings-only
```

### Update File Database After Legitimate Changes

```bash
sudo rkhunter --propupd
```

## Important Files

| File                    | Purpose                 |
| ----------------------- | ----------------------- |
| `/etc/rkhunter.conf`    | Main configuration file |
| `/var/log/rkhunter.log` | Scan results            |

---

# Part 2: Safe Data Extraction from a Potentially Compromised Server

## Important Safety Rule

Only extract:

* Database dumps (.sql)
* Text files
* Images
* Documents
* Media files

Do NOT copy or execute:

* Shell scripts (`.sh`)
* Python scripts (`.py`)
* Perl scripts (`.pl`)
* PHP files (`.php`) without review
* System directories such as:

  * `/bin`
  * `/usr`
  * `/etc`
  * `/root`
  * `/sbin`

Never run files copied from a compromised server without reviewing their contents first.

---

## Step 1: Dump Databases Safely

Create a plain-text SQL backup.

Backup all databases:

```bash
mysqldump -u root -p --all-databases > /tmp/safe_database_backup.sql
```

Backup a specific database:

```bash
mysqldump -u root -p DATABASE_NAME > /tmp/safe_database_backup.sql
```

Verify backup:

```bash
ls -lh /tmp/safe_database_backup.sql
```

---

## Step 2: Archive Application Data

Compress application files while excluding hidden files and common script types.

Example:

```bash
tar \
--exclude='.*' \
--exclude='*.sh' \
--exclude='*.py' \
--exclude='*.pl' \
-czvf /tmp/safe_app_backup.tar.gz /var/www/html/
```

> Replace `/var/www/html/` with the actual application directory.

Verify archive:

```bash
ls -lh /tmp/safe_app_backup.tar.gz
```

---

## Step 3: Verify Backups

Check that backup files exist and contain data.

```bash
ls -lh /tmp/safe_*
```

Expected output:

```text
/tmp/safe_database_backup.sql
/tmp/safe_app_backup.tar.gz
```

---

## Step 4: Transfer Files Off the Server

### Option A: Download Using SFTP

Use a client such as:

* WinSCP
* FileZilla
* Cyberduck

Connect to the server and download:

```text
/tmp/safe_database_backup.sql
/tmp/safe_app_backup.tar.gz
```

to your local computer.

---

### Option B: Copy to a Clean Linux Server

Use SCP:

```bash
scp \
/tmp/safe_database_backup.sql \
/tmp/safe_app_backup.tar.gz \
user@clean_server_ip:/home/user/
```

Replace:

* `user` with the destination username
* `clean_server_ip` with the destination server IP

---

# Immediate Response Checklist After a Compromise

* [ ] Run RKHunter scan.
* [ ] Preserve database backups.
* [ ] Preserve application data.
* [ ] Verify backup integrity.
* [ ] Transfer backups to a trusted system.
* [ ] Record logs and indicators of compromise.
* [ ] Power off the compromised server.
* [ ] Build a new server from trusted installation media.
* [ ] Restore only verified data.
* [ ] Change all passwords and SSH keys.
* [ ] Review firewall and service configurations.

---

# Example Workflow

```bash
sudo dnf install rkhunter -y

sudo rkhunter --update

sudo rkhunter --propupd

sudo rkhunter --check --sk

mysqldump -u root -p --all-databases > /tmp/safe_database_backup.sql

tar \
--exclude='.*' \
--exclude='*.sh' \
--exclude='*.py' \
--exclude='*.pl' \
-czvf /tmp/safe_app_backup.tar.gz /var/www/html/

ls -lh /tmp/safe_*
```

---

# Author

Linux Security and Incident Response Documentation
