To completely remove MySQL from your Termux Proot-distro Ubuntu and install fresh, follow these steps:

Step 1: Stop MySQL Service

```bash
service mysql stop
```

Step 2: Remove MySQL Packages

```bash
# For Ubuntu/Debian in proot-distro
apt-get remove --purge mysql-server mysql-client mysql-common
```

Step 3: Remove All MySQL Files

```bash
# Remove configuration files
apt-get remove --purge mysql*

# Remove data directories
rm -rf /var/lib/mysql
rm -rf /var/log/mysql

# Remove configuration directories
rm -rf /etc/mysql
```

Step 4: Clean Up Dependencies

```bash
# Auto-remove unused dependencies
apt-get autoremove

# Auto-clean
apt-get autoclean

# Update package list
apt-get update
```

Step 5: Remove MySQL User (Optional)

```bash
# Check if mysql user exists
getent passwd mysql

# Remove mysql user (if exists)
userdel -r mysql
```

Step 6: Install MySQL Fresh

```bash
# Update package list
apt-get update

# Install MySQL server
apt-get install mysql-server -y

# For Termux proot, you might need MariaDB instead:
# apt-get install mariadb-server mariadb-client -y
```

Step 7: Start and Secure MySQL

```bash
# Start service
service mysql start

# Run secure installation
mysql_secure_installation
```

Alternative: Use MariaDB (Recommended for Termux Proot)

MariaDB often works better in proot environments:

```bash
# Remove existing MySQL/MariaDB
apt-get remove --purge mariadb-server mysql-server

# Install MariaDB
apt-get install mariadb-server mariadb-client -y

# Start service
service mariadb start

# Secure installation
mysql_secure_installation
```

Important Notes for Termux Proot:

1. Service Management: In proot, use service mysql start not systemctl
2. Permissions: You might need to run as root in proot
3. Storage: Check available storage: df -h
4. MariaDB vs MySQL: MariaDB is often more compatible with Termux proot

Complete Clean Install Script:

```bash
#!/bin/bash
# Stop services
service mysql stop 2>/dev/null
service mariadb stop 2>/dev/null

# Remove everything
apt-get remove --purge mysql-server mariadb-server mysql-client mariadb-client mysql-common -y
apt-get autoremove -y
apt-get autoclean -y

# Clean directories
rm -rf /var/lib/mysql
rm -rf /var/lib/mariadb
rm -rf /etc/mysql
rm -rf /etc/mariadb
rm -rf /var/log/mysql

# Install fresh (choose one)
# Option 1: MariaDB
apt-get update
apt-get install mariadb-server mariadb-client -y
service mariadb start
mysql_secure_installation

# Option 2: MySQL
# apt-get install mysql-server -y
# service mysql start
# mysql_secure_installation
```


---

If you want MySQL specifically, not MariaDB. Here's how to install and set up MySQL in Termux proot-distro Ubuntu:

Step 1: Clean Previous Installations First

```bash
# Remove any existing MySQL/MariaDB
apt-get remove --purge mysql* mariadb* -y
apt-get autoremove -y
apt-get autoclean -y

# Clean directories
rm -rf /var/lib/mysql
rm -rf /etc/mysql
rm -rf /var/log/mysql
rm -rf /tmp/mysql*
```

Step 2: Install MySQL

```bash
# Update package list
apt update
apt upgrade -y

# Install MySQL server and client
apt install mysql-server mysql-client -y
```

Step 3: For Termux Proot Issues - Manual Setup

Since MySQL service might not work normally in proot, we need to set it up manually:

```bash
# Initialize MySQL data directory
mysqld --initialize --user=mysql

# Or for older MySQL versions:
mysql_install_db --user=mysql

# Set permissions
chown -R mysql:mysql /var/lib/mysql
```

Step 4: Start MySQL Manually

```bash
# Start MySQL daemon in background
mysqld --user=mysql --skip-grant-tables --skip-networking &

# Wait a few seconds for it to start
sleep 5
```

Step 5: Set Root Password

```bash
# Connect to MySQL (no password initially)
mysql -u root

# Inside MySQL prompt, run:
USE mysql;

# For MySQL 5.7.6+:
ALTER USER 'root'@'localhost' IDENTIFIED BY 'your_password_here';
FLUSH PRIVILEGES;
EXIT;
```

Step 6: Restart MySQL Normally

```bash
# Stop the current instance
mysqladmin -u root -p shutdown

# Start MySQL with password
mysqld --user=mysql &
```

Step 7: Create a Service Script for Easy Management

```bash
# Create MySQL service script
cat > /usr/local/bin/start-mysql << 'EOF'
#!/bin/bash
echo "Starting MySQL..."
if [ ! -d "/var/run/mysqld" ]; then
    mkdir -p /var/run/mysqld
    chown mysql:mysql /var/run/mysqld
fi
mysqld --user=mysql --datadir=/var/lib/mysql --socket=/var/run/mysqld/mysqld.sock &
echo "MySQL started!"
EOF

# Make it executable
chmod +x /usr/local/bin/start-mysql

# Create stop script
cat > /usr/local/bin/stop-mysql << 'EOF'
#!/bin/bash
echo "Stopping MySQL..."
mysqladmin -u root -p shutdown
EOF
chmod +x /usr/local/bin/stop-mysql
```

Step 8: Alternative - Install Specific MySQL Version

If the default MySQL doesn't work, try a specific version:

```bash
# Check available versions
apt-cache search mysql-server

# Install specific version (example)
apt install mysql-server-8.0 -y
```

Step 9: Test MySQL Installation

```bash
# Start MySQL
/usr/local/bin/start-mysql

# Connect
mysql -u root -p

# Check version
SELECT VERSION();

# Exit
EXIT;
```

Step 10: Run mysql_secure_installation

Now you can run:

```bash
mysql_secure_installation
```

Complete Installation Script for MySQL:

```bash
#!/bin/bash
echo "=== MySQL Installation for Termux Proot ==="

# Clean previous
pkill mysql 2>/dev/null
apt remove --purge mysql* -y
apt autoremove -y
rm -rf /var/lib/mysql /etc/mysql /var/log/mysql

# Install
apt update
apt install mysql-server mysql-client -y

# Initialize
mysqld --initialize-insecure --user=mysql
chown -R mysql:mysql /var/lib/mysql

# Start
mysqld --user=mysql --skip-grant-tables &
sleep 3

# Set password
mysql -u root << EOF
ALTER USER 'root'@'localhost' IDENTIFIED BY 'rootpassword';
FLUSH PRIVILEGES;
EOF

# Restart properly
pkill mysql
sleep 2
mysqld --user=mysql &

echo "MySQL installed! Connect with: mysql -u root -p"
echo "Password: rootpassword"
```

Troubleshooting:

If you still get errors, try:

1. Check if MySQL is running:

```bash
ps aux | grep mysql
netstat -tlnp | grep 3306
```

1. Check error logs:

```bash
cat /var/log/mysql/error.log
```

1. Start with verbose logging:

```bash
mysqld --user=mysql --console
```

1. Try socket connection:

```bash
mysql -u root --socket=/tmp/mysql.sock
```

