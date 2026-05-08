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

## Step 2: Stop Brute-Force Attacks with Fail2Ban

If your server is exposed to the internet, automated bots constantly attempt SSH logins. Fail2Ban helps block repeated failed login attempts automatically.

### Install EPEL Repository and Fail2Ban

For Rocky Linux, AlmaLinux, or other RHEL-based systems:

```bash
dnf install epel-release -y
dnf install fail2ban -y
```

### Enable and Start Fail2Ban

```bash
systemctl enable --now fail2ban
```

### Verify Fail2Ban Status

```bash
systemctl status fail2ban
```

### Check Active Bans

```bash
fail2ban-client status sshd
```

---

## Step 3: Secure SSH Configuration

### Change the Default SSH Port

Using a non-standard SSH port reduces automated scanning attempts.

Edit the SSH configuration file:

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

## Step 4: Disable Root Password Login

Using passwords for root login is highly risky. SSH key authentication is strongly recommended.

### Disable Password Authentication

Edit the SSH configuration:

```bash
vi /etc/ssh/sshd_config
```

Update or add:

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

## Step 5: Allow New SSH Port in Firewall

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

## Step 6: SELinux Configuration for Custom SSH Port

If SELinux is enabled, allow the new SSH port:

```bash
semanage port -a -t ssh_port_t -p tcp 2244
```

If `semanage` is missing:

```bash
dnf install policycoreutils-python-utils -y
```

---

## Verify SSH Listening Port

```bash
ss -tulnp | grep ssh
```

Example output:

```bash
tcp   LISTEN  0  128  0.0.0.0:2244  0.0.0.0:*  users:(("sshd",pid=1234,fd=3))
```

---

## Recommended Security Practices

- Use SSH key authentication
- Disable root login
- Use Fail2Ban
- Change the default SSH port
- Keep the system updated regularly
- Allow only trusted IPs if possible

---

## Important Warning

Before closing your current SSH session:

1. Open a second terminal
2. Test login using the new SSH port
3. Confirm SSH key authentication works

This prevents accidental lockout from the server.
