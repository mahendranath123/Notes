📖 NOC Runbook: Cacti & Spine Production Deployment
Phase 1: The Fortress (Security Hardening)
Before installing any web services, the server must be locked down to prevent unauthorized access and automated brute-force attacks.

1. Update System and Install Security Tools

Bash
sudo dnf update -y
sudo dnf install epel-release fail2ban nano tar wget -y
2. Lock Down SSH (Key-Based Authentication Only)
Do not allow password logins.

Bash
sudo nano /etc/ssh/sshd_config
Change PasswordAuthentication yes to PasswordAuthentication no.

Save and exit, then restart SSH: sudo systemctl restart sshd.

3. Configure Fail2Ban
Protect SSH from automated brute-force attacks.

Bash
sudo systemctl enable --now fail2ban
sudo systemctl status fail2ban
4. Configure Firewalld
Open only the ports required for the NOC dashboard and SNMP polling.

Bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-port=161/udp
sudo firewall-cmd --reload
Phase 2: Core Prerequisites & Production Tuning
1. Install the LEMP/LAMP Stack

Bash
sudo dnf install httpd mariadb-server rrdtool net-snmp net-snmp-utils cronie -y
sudo dnf module enable php:8.1 -y
sudo dnf install php php-mysqlnd php-gd php-gmp php-intl php-ldap php-mbstring php-pdo php-xml php-cli php-snmp php-process -y
2. Production PHP Tuning
Allocate enough memory and execution time for the poller to process hundreds of devices without crashing.

Bash
sudo nano /etc/php.ini
Change these lines:

memory_limit = 800M

max_execution_time = 300

date.timezone = Asia/Kolkata (Adjust to your local timezone)

3. Production MariaDB Tuning (Crucial for Speed)

Bash
sudo nano /etc/my.cnf.d/mariadb-server.cnf
Add this block under [mysqld]:

Ini, TOML
[mysqld]
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
max_heap_table_size=128M
max_allowed_packet=16M
tmp_table_size=128M
join_buffer_size=128M
innodb_file_format=Barracuda
innodb_large_prefix=1
innodb_buffer_pool_size=4G  # Set to ~50% of total server RAM
innodb_buffer_pool_instances=4
innodb_flush_log_at_timeout=3
innodb_read_io_threads=32
innodb_write_io_threads=16
innodb_io_capacity=5000
innodb_io_capacity_max=10000
Start and enable core services:

Bash
sudo systemctl enable --now httpd mariadb snmpd crond chronyd
Phase 3: Database Preparation
1. Create the Cacti Database

Bash
sudo mysql -u root
SQL
CREATE DATABASE cacti;
CREATE USER 'cactiuser'@'localhost' IDENTIFIED BY 'YOUR_SECURE_PASSWORD';
GRANT ALL PRIVILEGES ON cacti.* TO 'cactiuser'@'localhost';
GRANT SELECT ON mysql.time_zone_name TO 'cactiuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
2. Import System Timezones
Cacti requires system timezones to draw graphs accurately.

Bash
sudo mysql -u root mysql < /usr/share/mariadb/mysql_test_data_timezone.sql
Phase 4: Data Migration (Export & Import)
Follow this section if you are moving data from an old Cacti server to this new one.

Part A: Exporting from the OLD Server
1. Export the Database

Bash
mysqldump -u root -p cacti > cacti_database_export.sql
2. Export the Graph Data (RRD Files)
Warning: Do not target /usr/share/cacti/rra/ as it is a symlink and will result in a 1KB empty backup.

Bash
tar -czvf cacti_graphs_export.tar.gz /var/lib/cacti/rra/
Transfer both files to the /root/ directory of your NEW server.

Part B: Importing to the NEW Server
1. Inject the Database

Bash
mysql -u root cacti < /root/cacti_database_export.sql
2. Extract the Graphs

Bash
tar -xzvf /root/cacti_graphs_export.tar.gz -C /
Phase 5: Cacti & Spine Installation
1. Install Packages and Fonts
(Fonts are required to prevent blank graphs).

Bash
sudo dnf install cacti cacti-spine dejavu-sans-fonts dejavu-sans-mono-fonts -y
2. Configure Cacti Web Application
Update the database credentials to match your new MariaDB user.

Bash
sudo nano /usr/share/cacti/include/config.php
Ensure these match exactly:

PHP
$database_username = 'cactiuser';
$database_password = 'YOUR_SECURE_PASSWORD';
3. Configure Spine Engine
Update Spine's dedicated configuration file.

Bash
sudo nano /etc/spine.conf
Change DB_Pass to match your new secure password.

4. Set Directory Permissions & SELinux Contexts
You must give Apache permission to write to the data folders, and tell SELinux it is safe.

Bash
sudo chown -R apache:apache /var/lib/cacti/rra/ /usr/share/cacti/log/
sudo chmod -R 775 /var/lib/cacti/rra/ /usr/share/cacti/log/
sudo restorecon -Rv /var/lib/cacti/rra/ /usr/share/cacti/log/ /usr/share/cacti/
5. Enable the Automated Poller
Uncomment the cron job to start the 5-minute polling cycle.

Bash
sudo nano /etc/cron.d/cacti
Remove the # at the start of the line:
*/5 * * * * apache /usr/bin/php /usr/share/cacti/poller.php > /dev/null 2>&1

Bash
sudo systemctl restart crond
6. Unlock the Web Dashboard
Allow remote access to the web UI.

Bash
sudo nano /etc/httpd/conf.d/cacti.conf
Change Require host localhost to Require all granted in both directory blocks.

Bash
sudo systemctl restart httpd
Phase 6: Final UI Configuration
Navigate to your server IP: http://SERVER_IP/cacti/

Log in with your admin credentials.

Go to Configuration > Settings > Paths.

Change Spine Binary File Location to: /usr/bin/spine

Click Save (Verify you see a Green Checkmark).

Go to Configuration > Settings > Poller.

Change Poller Type to: spine

Click Save.
