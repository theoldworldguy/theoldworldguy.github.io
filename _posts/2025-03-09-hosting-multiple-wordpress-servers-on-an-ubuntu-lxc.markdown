---
layout: post
title:  "Hosting Multiple WordPress Servers On An Ubuntu LXC"
date:   2025-03-09 03:55:36 +1300
categories: linux guide install
---

### Create and configure the container

I used a Ubuntu 24.04 LXC with 3 cores, 2048 MB RAM, 2048 MB Swap and 30 G of disk space.

Update the container.
```bash
apt update && apt upgrade
```

Install Apache2 
```bash
apt install apache2
```

### Install and create the databases and users

Install MySQL, you could use MariaDB but ill use MySQL.
```bash
apt install mysql-server
```

Setup up the database.
```bash
mysql_secure_installation
```
PRIVILEGES
Access MySQL command prompt.
```bash
mysql -u root -p
```

Create the WordPress users
```mysql
CREATE DATABASE wpdbkarl;
CREATE DATABASE wpdbbekah;
```

Create the WordPress Database and grant privileges to the user
```mysql
CREATE USER 'wpdbkarl'@'localhost' IDENTIFIED BY 'G1xIP5FGqTHqLO5cJ48t';
CREATE USER 'wpdbbekah'@'localhost' IDENTIFIED BY 'zQWB5FGHDSHqLO5cJ48t';
GRANT ALL PRIVILEGES ON wpdbkarl.* TO 'wpdbkarl'@'localhost';
GRANT ALL PRIVILEGES ON wpdbbekah.* TO 'wpdbbekah'@'localhost';

FLUSH PRIVILEGES;
EXIT;
```

Install the required PHP files.
```bash
apt install php libapache2-mod-php php-mysql
```

Create the web root folders for each web server.
```bash
mkdir /var/www/wp.nastray.com
mkdir /var/www/wp.fabricatedfoundations.com
```

Create an Apache config file for each site.
```bash
nano /etc/apache2/sites-available/wp.nastray.com.conf
nano /etc/apache2/sites-available/wp.fabricatedfoundations.com.conf
```


---
Add the following into each conf file changing the settings as needed.
This one would not allow multiple hosting behind the same network
```bash
<VirtualHost *:80>
    ServerName wp.nastray.com
    ServerAlias www.wp.nastray.com
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/wp.nastray.com
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

I used ports to separate the virtual hosts
```bash
<VirtualHost *:9090>
    DocumentRoot /var/www/wp.nastray.com
    ServerName wp.nastray.com

    <Directory /var/www/wp.nastray.com>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

I added the ports i used for each web server in 
```bash
nano /etc/apache2/ports.conf
```


```bash
Listen 9191
Listen 9090
```

---

Enable the each sites virtual hosts file.
```bash
a2ensite wp.nastray.com.conf 
a2ensite wp.fabricatedfoundations.com.conf 
```

Download and extract the latest version of WordPress.
```bash
apt install curl
cd /tmp
curl -LO https://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
```

Create a backup copy of the WordPress config.php.
```bash
cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
```

Move the WordPress files to each sites root directory.
```bash
sudo cp -a /tmp/wordpress/. /var/www/wp.nastray.com
sudo cp -a /tmp/wordpress/. /var/www/wp.fabricatedfoundations.com
```

Generate some random keys using the WordPress key generator to add into the wp-config.php file. Might need to install curl.
```bash
curl -s https://api.wordpress.org/secret-key/1.1/salt/
```

Copy the generated keys and paste them over the default keys in wp-config.php.
```bash
nano /var/www/wp.nastray/wp-config.php
nano /var/www/wp.fabricatedfoundations/wp-config.php
```

Add the database credentials to the wp-config.php file in the appropriate areas.
```bash
define( 'DB_NAME', 'database_name_here' );

/** Database username */
define( 'DB_USER', 'username_here' );

/** Database password */
define( 'DB_PASSWORD', 'password_here' );
```

Navigate to each servers domain name in your web browser to finish the installation.
```
http://wp.nastray.com/wordpress
```


NOTES

added define('FS_METHOD', 'direct'); to the wp-config.php file on each site to prevent an auth pop up each time you want to upload or dl
```bash
/* Add any custom values between this line and the "stop editing" line. */
define('FS_METHOD', 'direct');
```


Fixing permision problems to folders that wordpress needs to write to 
```bash
sudo chown -R www-data:www-data wp-content/plugins/
sudo chmod 775 wp-content
sudo chown -R www-data:www-data wp-content/
```


```
apt install php8.3-xmlrpc
apt install php8.3-xml

```
