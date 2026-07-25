Here's a clean README file for changing your Linux hostname:

---

# Change Linux Hostname (Permanent)

A quick guide to permanently change the system hostname on modern Linux distributions (CentOS, RHEL, Ubuntu, Debian).

---

## Prerequisites

- Root or `sudo` access
- A terminal session

---

## Steps

### 1. Set the New Hostname

Use `hostnamectl` to update the system hostname permanently across reboots:

```bash
hostnamectl set-hostname new-server-name
```

> Replace `new-server-name` with your desired hostname.

---

### 2. Update `/etc/hosts`

Map the new hostname to your local loopback address to prevent local network resolution errors.

```bash
vi /etc/hosts
```

Find the line starting with `127.0.0.1` (or `127.0.1.1` on Ubuntu/Debian) and append your new hostname:

```text
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 new-server-name
```

Save and exit.

---

### 3. Refresh Your Terminal

Your current shell session still shows the old hostname. Start a new bash session to see the change immediately:

```bash
exec bash
```

Your prompt should now display:

```text
[root@new-server-name ~]#
```

---

## Temporary Change (Optional)

If you only need to change the hostname **until the next reboot** without modifying system files:

```bash
hostname new-server-name
```

---

## Verification

Confirm the change with:

```bash
hostnamectl status
```

Or simply:

```bash
hostname
```

---

## Supported Distributions

- CentOS
- RHEL
- Ubuntu
- Debian
- Most modern systemd-based distributions

---

## Notes

- Always update `/etc/hosts` when setting a permanent hostname to avoid issues with services that rely on local name resolution.
- No reboot is required for the permanent change to take effect on new sessions.

---
