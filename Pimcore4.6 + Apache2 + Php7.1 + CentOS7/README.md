# Installing Pimcore 4.6 on CentOS7 + Apache2 + PHP7.1
## 1	Scope
Pimcore is an Open Source content platform, and as described on their site https://www.pimcore.org most common features are for Product Information Management, Web Content Management, Digital Asset Management, and e-Commerce. Recently, I had to investigate an special configuration for an specific task, and I didn’t found a full guide to launch application with all requirements I needed. Then I decided to get most common parts that was successful, and create my own custom guide. 

This guide tries to help on installing Pimcore 4.6 platform, on a CentOS7, with Apache 2 server and PHP 7.1.

Please, note that this is not a production installation. If you want a production server you will need to improve security policies, permissions, etc.

## 2	Install Apache web server
First of all, we have to install Apache web server.

`sudo yum install httpd`

Start the web server and enable it to start at boot

`sudo systemctl start httpd`

`sudo systemctl enable httpd`

Now, we will set up Apache virtual hosting directive for Pimcore website.
Create a ‘/etc/httpd/conf.d/vhosts.conf’ file with the following content.

`IncludeOptional vhosts.d/*.conf`

Then, we should create a ‘/etc/httpd/vhosts.d’ directory where we will put all our virtual hosts.

`sudo mkdir /etc/httpd/vhosts.d`

Create a virtual host for your Pimcore domain. First, create file with some file editor.

`sudo nano /etc/httpd/vhosts.d/yourdomain.conf`

Then add VHosts config.

`<VirtualHost YOUR_SERVER_IP:80>`

`ServerAdmin webmaster@yourdomain.com`

`DocumentRoot "/var/www/html/pimcore"`

`ServerName yourdomain.com`

`ServerAlias www.yourdomain.com`

`ErrorLog "/var/log/httpd/yourdomain.com-error_log"`

`CustomLog "/var/log/httpd/yourdomain.com-access_log" combined`

`<Directory "/var/www/html/pimcore/">`

`DirectoryIndex index.php`

`Options FollowSymLinks`

`AllowOverride All`

`Require all granted`

`</Directory>`

`</VirtualHost>`

And restart Apache.

`sudo systemctl restart httpd`

## 3	Installing MariaDB
MariaDB is a fork of MySQL database and maintains compatibility between both DBs. To install MariaDB on your server, run:

`sudo yum -y install mariadb mariadb-server`

Then we have to run the following commands to start MariaDB and enable it to start at boot time.

`sudo systemctl start mariadb`

`sudo systemctl enable mariadb`

Now run the following commands to secure your MariaDB installation.

`sudo mysql_secure_installation`

Above command will run a script to secure fresh MariaDB installation. The script will ask for the existing root user password, we have just installed MariaDB, the root password is not set, just press enter to proceed further.

This script will ask you if you want to set a root password for your MariaDB installation, choose y and set a strong password for the installation. Most of the questions are self-explanatory and you should answer yes or y to all the questions. The output will look like shown below.

To create a database we will need to login to MySQL command line first. Run the following command for same.

`mysql -u root -p`

This command will login to MySQL shell of the root user, it will prompt for the password of the root user. Provide the password to login. Now run the following query to create a new database for your Pimcore installation.

`CREATE DATABASE <set pimcore db name> CHARACTER SET UTF8;`

The above query will create a new database named pimcore_data. Make sure that you use semicolon at the end of each query as the query always ends with a semicolon.

To create a new database user, run the following query.

`CREATE USER '<set pimcore db username>'@'localhost' IDENTIFIED BY '<set pimcore user pass>';`

Now provide the all the privileges to your database user over the database you have created. Run the following command.

`GRANT ALL PRIVILEGES ON <set pimcore db name>.* TO '<set pimcore db username>'@'localhost';`

Now run the following command to immediately apply the changes on the database privileges.

`FLUSH PRIVILEGES;`

Exit from MySQL prompt using the following command.

`EXIT;`

## 4	Installing PHP
Pimcore supports all the version of PHP greater than 5.6. But we want to install lasts available version of PHP 7.1. Installing the latest version of PHP will ensure the maximum security and performance of the application.

PHP7.1 is not on default’s CentOS repository, then we will need to add the Webtatic repository in our system.

Type the commands to install Webtatic repository.

`sudo rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm`

`sudo yum -y update`

Type the following command to install PHP 7.1, and also, required dependencies.

`sudo yum -y install php71w php71w-mysqli php71w-fpm php71w-gd php71w-cli php71w-iconv php71w-dom php71w-simplexml php71w-exif php71w-fileinfo php71w-mbstring php71w-zlib php71w-zip php71w-bz2 php71w-openssl php71w-opcache php71w-curl php71w-pecl-redis ImageMagick`

To check if PHP is installed successfully, just run:

`php -v`

You should get output similar to this.

`[user@localhost ~]$ php –v`

`PHP 7.1.8 (cli) (built: Aug  9 2017 19:19:49) ( NTS )`

`Copyright (c) 1997-2017 The PHP Group`

`Zend Engine v3.1.0, Copyright (c) 1998-2017 Zend Technologies`

    `with Zend OPcache v7.1.8, Copyright (c) 1999-2017, by Zend Technologies`

Now we will need to set some configurations in PHP. To allow it, we need to open PHP’s configuration file (php.ini) using some text editor.

`sudo nano /etc/php.ini`

Find the following line, uncomment the line and set the timezone according to your region. For example:

`[Date]`

`; Defines the default timezone used by the date functions`

`; http://php.net/date.timezone`

`date.timezone = Europe/Madrid`

Further, search for the following lines:

`upload_max_filesize = 2M`

Change the value from **2M** to **100M**.

`post_max_size = 8M`

Change the value from **8M** to **100M**. Save the file and exit from the editor.

Then, we need to adjust PHP-FPM configuration editing file located at /etc/php-fpm.d/www.conf.

`nano /etc/php-fpm.d/www.conf`

Find the following lines:

`;listen.owner = nobody`

`;listen.group = nobody`

`;listen.mode = 0660`

Uncomment the above lines and change nobody to apache.

`listen = 127.0.0.1:9000`

Then comment out the above line and add the following line below.

`listen = /var/run/php-fpm/php-fpm.sock`

Now start PHP-FPM service, and enable it to automatically start at boot time.

`sudo systemctl start php-fpm`

`sudo systemctl enable php-fpm`

## 5	Installing Additional Server Software

Pimcore required additional server packages which are used to perform certain operations using Pimcore CMS. Few of the dependencies are only available in RPMFusion repository. We can install RPM Fusion repository using the following command.

`sudo rpm -Uvh https://download1.rpmfusion.org/free/el/rpmfusion-free-release-7.noarch.rpm`

Then we can install additional server’s software.

`sudo yum -y install ffmpeg libreoffice libreoffice-math xorg-x11-fonts-75dpi poppler-utils inkscape libXrender ghostscript fontconfig wkhtmltopdf`

Then, we have installed FFMPEG, LibreOffice, pdftotext, Inkscape and Wkhtmltoimage / Wkhtmltopdf.

### 5.1	Installation of composer

The installation process of Composer is relatively simple. First, let's get in the good habit of updating our server.

`sudo yum -y update`

Switch into the temp directory.

`cd /tmp`

Install Composer using cURL

`sudo curl -sS https://getcomposer.org/installer | php`

Want to make Composer globally accessible?

`mv composer.phar /usr/local/bin/composer`

# 6	Configure Permissions and Firewall

Now we  will need to provide application’s ownership to web server user.

`sudo chown -R apache:apache /var/www/html/<pimcore directory>`

Also, we need to allow HTTP traffic on port 80 through the firewall if you are running one. Note that this is a simple installation and we are not using HTTPS protocol.

`sudo firewall-cmd --zone=public --permanent --add-service=http`

`sudo firewall-cmd --reload`

We can temporally disable SELinux without restarting the server, run the following command.

`setenforce 0`

Not recommended, but we can completely disable the SELinux editing /etc/selinux/config.

`nano /etc/selinux/config`

Then, find following line:

`SELINUX=enforcing`

And change it to:

`SELINUX=disabled`

## 7	Installing and Configuring Pimcore

We have all dependencies ready, then we can now download install package from Pimcore website.

`cd /var/www/html`

Pimcore provides three different types of installation archive.

If you want to install Pimcore along with demo data which is suitable for beginners, use the following link to download Pimcore.

`sudo wget https://www.pimcore.org/download/pimcore-data.zip`

If you want to install the Pimcore core package only, use the following link to download, that will always download latest stable version.

`sudo wget https://www.pimcore.org/download/pimcore-latest.zip`

Third option is to install nightly build, which is for development purpose only.

Then we can extract zip archive using the following command.

`sudo unzip pimcore*.zip -d pimcore`

Change ownership recursively.

`sudo chown -R apache:apache /var/www/html/pimcore/`

In order to use scheduled features we need to create a cronjob.

`sudo nano /etc/crontab`

`*/5 * * * * /usr/bin/php /var/www/html/pimcore/pimcore/cli/maintenance.php`

### 7.1	Installation via composer
Pimcore also supports a Composer’s based install, if this is more suiteable for your development environment.

Result is the same and of course you're still able to install custom packages using Composer if choosing the package install.

I installed one version though Composer, to easily check that all required dependencies was installed. Then I didn’t completed installation through web browser. I removed installation directory, and proceeded as described before.

`cd /var/www/html`

`sudo composer create-project pimcore/pimcore ./your-project-name`

`cd your-project-name`

`sudo composer dumpautoload -o`

To install specific release or the nightly build you can use:

`sudo composer create-project -s dev pimcore/pimcore ./your-project-name dev-master`

## 8	Install Pimcore
Now, we need to lunch the web installer accessing http://yourdomain.com, enter your MariaDB information and create administrator user.

You will be welcomed by a page.

Choose your preferred database adapter, provide required details and proceed. 

Installation will just take few seconds to complete. 

`9	Install Redis`

Using Redis is highly recommended on production environment, to improve performance through caching.

In this section you’ll add the EPEL repository, and then use it to install Redis.

Add the EPEL repository, and update YUM to confirm your change:

`sudo yum install epel-release`

`sudo yum update`

Install Redis:

`sudo yum install redis`

Start Redis:

`sudo systemctl start redis`

To automatically start Redis on boot:

`sudo systemctl enable redis`

Verify the Installation

Verify that Redis is running with redis-cli:

`redis-cli ping`

If Redis is running, it will return:

`PONG`

### 9.1	Configure Redis
In this section, we will configure some basic persistence and tuning options for Redis.

**Persistence Options**
Redis provides two options for disk persistence:

Point-in-time snapshots of the dataset, made at specified intervals (RDB).

Append-only logs of all the write operations performed by the server (AOF).

Each option has its own pros and cons which are detailed in the Redis documentation. For the greatest level of data safety, consider running both persistence methods.

Because the Point-in-time snapshot persistence is enabled by default, you only need to set up AOF persistence:
Make sure that the following values are set for the appendonly and appendfsync settings in redis.conf:

`nano /etc/redis.conf`

`appendonly yes`

`appendfsync everysec`

Restart Redis:

`sudo systemctl restart redis`

**Basic System Tuning**

To improve Redis performance, set the Linux kernel overcommit memory setting to 1:

`sudo sysctl vm.overcommit_memory=1`

This immediately changes the overcommit memory setting, but the change will not persist across reboots. To make it permanent, add vm.overcommit_memory = 1 to /etc/sysctl.conf:

`nano /etc/sysctl.conf`

`vm.overcommit_memory = 1`

### 9.2	Additional Swap
Depending upon your usage, you may find it necessary to add extra swap disk space. You can add swap by resizing your disk in the Linode Manager. The Redis documentation recommends the size of your swap disk match the amount of memory available to your system.

### 9.3	Secure the Redis Installation
Since Redis is designed to work in trusted environments and with trusted clients, you should control access to the Redis instance. Some recommended security steps include:

* Set up a firewall using iptables.
* Encrypt Redis traffic using an SSH tunnel, or the methods described in the Redis Security documentation.

Additionally, to ensure that no outside traffic accesses your Redis instance, we suggest that you only listen for connections on the localhost interface or your Linode’s private IP address.

### 9.4	Use Password Authentication
For an added layer of security, use password authentication to secure the connection between your master and slave Linodes.

On your master Linode, uncomment the requirepass line in your Redis configuration and replace master_password with a secure password:

`/etc/redis.conf`

`requirepass master_password`

Save your changes, and apply them by restarting Redis on the master Linode:

`sudo systemctl restart redis`

On your slave Linode, add the master password to your Redis configuration under masterpass, and then create a unique password for the slave Linode with requirepass:

`nano /etc/redis.conf`

`masterpass  master_password`

`requirepass slave_password`

Replace master_password with the password you configured on your master, and replace slave_password with the password to use for your slave Linode.

Save your changes, and restart Redis on your slave Linode:

`sudo systemctl restart redis`

Connect to redis-cli on your master Linode, and use AUTH to authenticate with your master password:
`redis-cli`

`127.0.0.1:6379> AUTH master_password`

Once you’ve authenticated, you can view details about your Redis configuration by running INFO. This provides a lot of information, so you can specifically request the “Replication” section in your command:

`127.0.0.1:6379> INFO replication`

Output should be similar to the following:

`# Replication`

`role:master`

`connected_slaves:1`

`slave0:ip=192.0.2.105,port=6379,state=online,offset=1093,lag=1`

It should confirm the master role of your Linode, as well as how many slave Linodes are connected to it.

From your slave Linode, connect to redis-cli and authenticate using your slave password:

`redis-cli`

`127.0.0.1:6379> AUTH slave_password`

Once you’ve authenticated, use INFO to confirm your slave Linode’s role, and its connection to the master server:

`127.0.0.1:6379> INFO replication`

`# Replication`

`role:slave`

`master_host:192.0.2.100`

`master_port:6379`

`master_link_status:up`

