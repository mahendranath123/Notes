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
