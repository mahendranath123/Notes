# Fix SSH Login Delay and Secure Against Brute-Force Attacks

## Step 1: Fix the Login Delay

By default, SSH tries to resolve the connecting IP address to a hostname, and it attempts GSSAPI (Kerberos) authentication before moving to password authentication. Disabling these settings can speed up the login process.

### Open the SSH configuration file

```bash
vi /etc/ssh/sshd_config
```

### Update the following settings

Find these lines, uncomment them if needed, and set them to `no`:

```conf
UseDNS no
GSSAPIAuthentication no
```

### Restart the SSH service

```bash
systemctl restart sshd
```

---

# Step 2: Install and Configure Fail2Ban

Fail2Ban automatically blocks IP addresses that repeatedly fail SSH login attempts.

## Install EPEL Repository and Fail2Ban

For Rocky Linux, AlmaLinux, or other RHEL-based systems:

```bash
dnf install epel-release -y
dnf install fail2ban -y
```

---

## Create Fail2Ban Configuration

Create the configuration file:

```bash
cat << 'EOF' > /etc/fail2ban/jail.local

ignoreip = 127.0.0.1/8 ::1 103.25.44.211 
[DEFAULT]
# Ban IP for 24 hours (86400 seconds)
bantime = 86400
# Window of time to track failed attempts (10 minutes)
findtime = 600
# Number of failures before a ban is triggered
maxretry = 3

[sshd]
enabled = true
port = 4040
backend = systemd
EOF

```

### Configuration Explanation

- `bantime = 86400`  
  Ban IPs for 24 hours

- `findtime = 600`  
  Count failed attempts within 10 minutes

- `maxretry = 3`  
  Ban after 3 failed login attempts

---

## Enable and Start Fail2Ban

```bash
systemctl enable --now fail2ban
```

---

## Restart Fail2Ban

Now restart the service after creating the configuration:

```bash
systemctl restart fail2ban
```

---

## Verify Fail2Ban Status

Check whether the service started successfully:

```bash
systemctl status fail2ban
```

---

## Check Active SSH Jail

```bash
fail2ban-client status sshd
```
## Verify the IPs are whitelisted
```bash
fail2ban-client get sshd ignoreip
```

Example output:

```bash
Status for the jail: sshd
|- Filter
|  |- Currently failed: 0
|  |- Total failed: 15
|  `- File list: /var/log/secure
`- Actions
   |- Currently banned: 3
   |- Total banned: 3
   `- Banned IP list: 192.168.1.100
```

---

# Step 3: Secure SSH Configuration

## Change the Default SSH Port

Using a non-standard SSH port reduces automated scanning attempts.

### Edit SSH Configuration

```bash
vi /etc/ssh/sshd_config
```

Find:

```conf
#Port 22
```

Change it to:

```conf
Port 2244
```

Restart SSH:

```bash
systemctl restart sshd
```

Connect using:

```bash
ssh -p 2244 user@server-ip
```

---

# Step 4: Disable Root Password Login

Using passwords for root login is highly risky. SSH key authentication is strongly recommended.

## Edit SSH Configuration

```bash
vi /etc/ssh/sshd_config
```

Add or update:

```conf
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
```

Restart SSH:

```bash
systemctl restart sshd
```

---

# Step 5: Allow New SSH Port in Firewall

If firewalld is enabled, allow the new SSH port before disconnecting.

```bash
firewall-cmd --permanent --add-port=2244/tcp
firewall-cmd --reload
```

Optional: Remove old SSH port access after testing:

```bash
firewall-cmd --permanent --remove-service=ssh
firewall-cmd --reload
```

---

# Step 6: Configure SELinux for Custom SSH Port

If SELinux is enabled, allow the new SSH port:

```bash
semanage port -a -t ssh_port_t -p tcp 2244
```

If `semanage` is missing:

```bash
dnf install policycoreutils-python-utils -y
```

---

# Verify SSH Listening Port

```bash
ss -tulnp | grep ssh
```

Example output:

```bash
tcp   LISTEN  0  128  0.0.0.0:2244  0.0.0.0:*  users:(("sshd",pid=1234,fd=3))
```

---

# Recommended Security Practices

- Use SSH key authentication
- Disable root login
- Use Fail2Ban
- Change the default SSH port
- Keep the system updated regularly
- Allow only trusted IPs if possible

---

# Important Warning

Before closing your current SSH session:

1. Open a second terminal
2. Test login using the new SSH port
3. Confirm SSH key authentication works

This prevents accidental lockout from the server.
