# RKHunter Installation and Scan Guide

RKHunter (Rootkit Hunter) is a security tool used to detect rootkits, backdoors, and possible local exploits on Linux systems.

## Installation

Install RKHunter using DNF:

```bash
sudo dnf install rkhunter -y
```

## Update Rootkit Definitions

Update the database of known rootkits:

```bash
sudo rkhunter --update
```

## Initialize File Properties Database

Create or update the file properties database:

```bash
sudo rkhunter --propupd
```

> Run this command only after verifying that the system is in a trusted state.

## Run a Full System Scan

Start a complete scan:

```bash
sudo rkhunter --check --sk
```

### Options

| Option    | Description                |
| --------- | -------------------------- |
| `--check` | Perform a full system scan |
| `--sk`    | Skip interactive prompts   |

## View Scan Results

Check the log file:

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

### Show Warning Report Only

```bash
sudo rkhunter --check --report-warnings-only
```

### Update File Database After System Updates

```bash
sudo rkhunter --propupd
```

## Important Files

| File                    | Description             |
| ----------------------- | ----------------------- |
| `/etc/rkhunter.conf`    | Main configuration file |
| `/var/log/rkhunter.log` | Scan log file           |

## Best Practices

* Keep RKHunter updated.
* Review warnings before taking action.
* Update the file properties database after legitimate system changes.
* Investigate unexpected file modifications immediately.
* Schedule regular scans using cron.

## Example Workflow

```bash
sudo dnf install rkhunter -y
sudo rkhunter --update
sudo rkhunter --propupd
sudo rkhunter --check --sk
```

## Author

System Security Documentation
