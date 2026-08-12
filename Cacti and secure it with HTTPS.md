Here is the complete, start-to-finish solution. If you were configuring a brand new server, following these exact steps in this specific order will completely prevent all three of those errors from happening.

This provides a clean, single-pass setup to route your domain to Cacti and secure it with HTTPS.

### 1. Install All Required Packages First

Install the EPEL repository, Certbot, and the Apache plugin right at the beginning. This prevents the **Missing Certbot Module** error.

```bash
dnf install epel-release -y
dnf install certbot python3-certbot-apache -y

```

### 2. Generate the Dummy SSL Certificate

Before Apache even tries to start, generate the default certificates. This satisfies Enterprise Linux requirements and prevents the **Apache SSL Crash**.

```bash
openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 \
  -subj "/CN=localhost" \
  -keyout /etc/pki/tls/private/localhost.key \
  -out /etc/pki/tls/certs/localhost.crt

```

### 3. Create the Virtual Host Configuration

Set up the configuration file immediately so Apache knows exactly where Cacti is installed. This prevents the **Missing Virtual Host** error and stops the default test page from loading.

```bash
vi /etc/httpd/conf.d/mrtg.conf

```

Paste this configuration (adjusting `/usr/share/cacti` if your installation path is different):

```apache
<VirtualHost *:80>
    ServerName mrtg.jeebr.net
    DocumentRoot /usr/share/cacti
    
    <Directory /usr/share/cacti>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>

```

### 4. Configure the Firewall

Ensure your web ports are permanently open, while explicitly allowing SSH so you don't get locked out.

```bash
systemctl start firewalld
systemctl enable firewalld
firewall-cmd --permanent --add-service=ssh
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --reload

```

### 5. Validate and Start Apache

Now that everything is perfectly in place, verify the syntax and start the web server.

```bash
apachectl configtest
systemctl restart httpd
systemctl enable httpd

```

### 6. Generate the Let's Encrypt Certificate

With Apache running flawlessly and fully aware of your domain, run the final Certbot command to secure the site and force the HTTPS redirect.

```bash
certbot --apache -d mrtg.jeebr.net

```

*(Select Option 2 when prompted to redirect all HTTP traffic to HTTPS).*

Yes, you can easily automate the SSL renewal. Let's Encrypt certificates are valid for 90 days, but Certbot has a built-in command to renew them automatically before they expire.

Since you are on Rocky Linux, the best way to handle this is by using the built-in systemd timer, or by adding a simple line to your crontab.

Here is how to set up and verify the automatic renewal:

### 1. Test the Renewal Process (Dry Run)

Before automating it, it is best practice to run a "dry run." This tests the renewal process without actually changing your live certificate to ensure there are no errors.

```bash
certbot renew --dry-run

```

*If this command outputs `Congratulations, all simulated renewals succeeded`, you are ready to automate.*

### 2. Enable the Auto-Renewal Timer (Recommended)

Rocky Linux automatically installs a background timer for Certbot. We just need to make sure it is permanently enabled and actively running.

Run these commands in order:

```bash
# Enable the timer to start automatically if the server reboots
systemctl enable certbot-renew.timer

# Start the timer right now
systemctl start certbot-renew.timer

# Verify it is actively running
systemctl status certbot-renew.timer

```

*You should see the status as **Active: active (waiting)**. This timer wakes up twice a day, checks if the certificate is within 30 days of expiration, and renews it automatically if needed.*

### 3. Alternative Method: Use Crontab

If you prefer to explicitly control the renewal schedule using a standalone crontab entry (which works perfectly alongside your SSL monitoring script), you can do so manually.

Open your crontab:

```bash
crontab -e

```

Add this line to run the renewal check daily at 2:00 AM. The `--quiet` flag ensures it only outputs a message if an error occurs:

```text
0 2 * * * /usr/bin/certbot renew --quiet

```

With either the system timer or the crontab method active, your SSL certificate will automatically renew itself before the 90-day expiration window, ensuring your HTTPS connection never drops.
