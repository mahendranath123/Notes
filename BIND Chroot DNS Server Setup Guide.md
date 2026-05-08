# BIND Chroot DNS Server Setup Guide

## What is `bind-chroot`?

`bind-chroot` runs the BIND DNS server inside a restricted directory environment called a **chroot jail**.

This improves security by isolating the DNS service from the rest of the operating system.

Default chroot directory:

```bash
/var/named/chroot
```

---

# Install BIND with Chroot Support

Install required packages:

```bash
sudo dnf install bind bind-utils bind-chroot -y
```

---

# Verify Chroot Package Installation

```bash
rpm -qa | grep bind
```

Example output:

```bash
bind
bind-utils
bind-chroot
```

---

# Force BIND to Use IPv4 Only

## Edit the Named Sysconfig File

```bash
sudo vi /etc/sysconfig/named
```

Update:

```conf
OPTIONS="-4"
```

---

# Understanding BIND Chroot Paths

When using `bind-chroot`, important files are stored inside:

```bash
/var/named/chroot/
```

Common paths:

| Normal Path | Chroot Path |
|---|---|
| `/etc/named.conf` | `/var/named/chroot/etc/named.conf` |
| `/var/named/` | `/var/named/chroot/var/named/` |
| `/var/run/named/` | `/var/named/chroot/var/run/named/` |

---

# Edit the BIND Configuration

## Open Configuration File

```bash
sudo vi /etc/named.conf
```

Example optimized configuration:

```conf
options {

        directory "/var/named";

        # Listen on IPv4
        listen-on port 53 { any; };

        # Allow DNS Queries
        allow-query { localhost; any; };

        # Forwarders (Optional)
        #forwarders {
        #       8.8.8.8;
        #       1.1.1.1;
        #};

        #forward only;

        # Cache Optimization
        max-cache-size 512m;
        minimal-responses yes;
        prefetch 2 9;

        # DNSSEC
        dnssec-validation no;

        # Allow Recursion
        allow-recursion { localhost; any; };

        # Rate Limiting
        rate-limit {
                responses-per-second 10;
                window 5;
        };
};
```

---

# Validate Configuration Syntax

Always check for syntax errors before restarting BIND.

```bash
sudo named-checkconf
```

If there is no output, the configuration is valid.

---

# Configure Firewall for DNS

## Start Firewalld

```bash
sudo systemctl start firewalld
```

---

## Enable Firewalld on Boot

```bash
sudo systemctl enable firewalld
```

---

## Allow DNS Service

```bash
sudo firewall-cmd --permanent --add-service=dns
```

---

## Reload Firewall Rules

```bash
sudo firewall-cmd --reload
```

---

# Start and Enable BIND Service

## Start Named Service

```bash
sudo systemctl start named
```

---

## Enable Named Service on Boot

```bash
sudo systemctl enable named
```

---

# Verify Named Service Status

```bash
sudo systemctl status named
```

---

# Verify Port 53 Listening

```bash
ss -tulnp | grep :53
```

Expected output:

```bash
udp   UNCONN 0      0      0.0.0.0:53      0.0.0.0:*
tcp   LISTEN 0      10     0.0.0.0:53      0.0.0.0:*
```

---

# Test DNS Resolution

## Using DIG

```bash
dig @127.0.0.1 google.com
```

---

## Using NSLOOKUP

```bash
nslookup google.com 127.0.0.1
```

---

# Verify Chroot Environment

Check chroot directory structure:

```bash
ls -l /var/named/chroot/
```

---

# Important Chroot Notes

- Always place custom zone files inside:

```bash
/var/named/chroot/var/named/
```

- Configuration files should remain synced with:

```bash
/etc/named.conf
```

- SELinux contexts must be correct for zone files.

Restore contexts if needed:

```bash
sudo restorecon -Rv /var/named
```

---

# Useful Troubleshooting Commands

## Check Logs

```bash
sudo journalctl -u named -f
```

---

## Check Configuration

```bash
sudo named-checkconf
```

---

## Check Zone Files

```bash
sudo named-checkzone example.com /var/named/example.com.zone
```

---

# Security Recommendations

- Restrict recursion to trusted networks
- Disable recursion for public authoritative servers
- Enable DNSSEC in production
- Use ACLs for internal DNS access
- Keep BIND updated regularly

---

# Restart BIND After Changes

```bash
sudo systemctl restart named
```
