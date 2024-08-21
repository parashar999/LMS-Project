### **Before You Begin**

It's essential to note down the following credentials, as you'll need them throughout the installation and configuration process:

1. **Ubuntu Root Password:** The primary administrative password for your server.
2. **MySQL Username and Password:** These credentials will be used by Moodle to interact with the database.
3. **Moodle Admin Username and Password:** The main administrative account for Moodle.
4. **Additional Moodle Admin Username and Password:** A backup administrative account for Moodle.

### **Step 1: Install Ubuntu**

#### **Why Choose Ubuntu Server Over Desktop?**
- **Security:** A Command Line Interface (CLI) server is more secure and less vulnerable to attacks than a GUI-based server. For professional environments, a CLI server is recommended.
- **Ease of Use:** While a desktop environment might seem easier, it's unnecessary for a server that will primarily host Moodle. However, if you're using Moodle for local, experimental purposes, you might prefer the Ubuntu desktop version. 

If you start with a CLI server and later wish to add a GUI, you can install it using:
```bash
sudo tasksel
sudo apt install ubuntu-desktop
```
**Note:** Installing a GUI on a server is generally not recommended as it can complicate management and potentially cause issues.

**Alternative Instructions:**
- **LAMP Stack on Ubuntu 22.04:** [DigitalOcean Guide](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-ubuntu-22-04)
- **MariaDB and OpenLiteSpeed:** [DigitalOcean Guide](https://www.digitalocean.com/community/tutorials/how-to-install-linux-openlitespeed-mariadb-php-lomp-stack-ubuntu-22-04)

#### **Common Ubuntu Issues:**
- **/boot Directory Fill-Up:** Ubuntu's automatic updates can fill the `/boot` directory, causing issues with updates. Consider allocating around 5GB to the `/boot` directory and setting up automated cleanups.

### **Step 2: Install Apache, MySQL, and PHP (LAMP Stack)**

#### **Install Apache, MySQL, and PHP 7.4:**
```bash
sudo apt-get update
sudo apt install apache2 mysql-client mysql-server php7.4 libapache2-mod-php
```
**Important:** Ubuntu 20.04 installs PHP 8 by default. Specify PHP 7.4 to avoid compatibility issues with Moodle.

#### **Secure MySQL Installation:**
```bash
sudo mysql_secure_installation
```
This will prompt you to set the MySQL root password.

### **Step 3: Install Additional Software**

#### **Install Required Packages:**
```bash
sudo apt install graphviz aspell ghostscript clamav php7.4-pspell php7.4-curl php7.4-gd php7.4-intl php7.4-mysql php7.4-xml php7.4-xmlrpc php7.4-ldap php7.4-zip php7.4-soap php7.4-mbstring
```

#### **Restart Apache:**
```bash
sudo service apache2 restart
```

#### **Install Git:**
```bash
sudo apt install git
```
Git will help in managing and updating the Moodle code.

### **Step 4: Download Moodle**

#### **Set Up Local Repository and Download Moodle:**
```bash
cd /opt
sudo git clone git://git.moodle.org/moodle.git
```

#### **Switch to the Desired Moodle Branch:**
```bash
cd moodle
sudo git branch --track MOODLE_400_STABLE origin/MOODLE_400_STABLE
sudo git checkout MOODLE_400_STABLE
```

### **Step 5: Copy Moodle to the Web Directory**

```bash
sudo cp -R /opt/moodle /var/www/html/
sudo mkdir /var/moodledata
sudo chown -R www-data /var/moodledata
sudo chmod -R 777 /var/moodledata
sudo chmod -R 0755 /var/www/html/moodle
```

### **Step 6: Setup MySQL Server**

#### **Edit MySQL Configuration:**
```bash
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
```

Add the following lines under `[mysqld]`:

```bash
default_storage_engine = innodb
innodb_file_per_table = 1
innodb_file_format = Barracuda
```
Save and close the file with `:wq`, and restart MySQL:

```bash
sudo service mysql restart
```

#### **Create Moodle Database and User:**
```bash
sudo mysql -u root -p
CREATE DATABASE moodle DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'moodledude'@'localhost' IDENTIFIED BY 'passwordformoodledude';
GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,CREATE TEMPORARY TABLES,DROP,INDEX,ALTER ON moodle.* TO 'moodledude'@'localhost';
quit;
```

### **Step 7: Complete Moodle Setup**

#### **Set Permissions for Webroot:**
```bash
sudo chmod -R 777 /var/www/html/moodle
```
Navigate to `http://IP.ADDRESS.OF.SERVER/moodle` in your browser to complete the setup. After installation, revert the permissions:

```bash
sudo chmod -R 0755 /var/www/html/moodle
```

### **Step 8: Optimize Moodle**

#### **Set System Paths in Moodle:**
Go to `Site Administration > Server > System Paths` and set the following:
- **Path to du:** `/usr/bin/du`
- **Path to aspell:** `/usr/bin/aspell`
- **Path to dot:** `/usr/bin/dot`

#### **Setup ClamAV Antivirus (Optional):**
```bash
sudo mkdir /var/quarantine
sudo chown -R www-data /var/quarantine
```
In Moodle, enable ClamAV and set:
- **ClamAV Path:** `/usr/bin/clamscan`
- **Quarantine Directory:** `/var/quarantine`

### **Step 9: Enable Zend OPcache**

#### **Configure OPcache:**
```bash
sudo vi /etc/php7/apache2/conf.d/05-opcache.ini
```
Add the recommended settings, then restart Apache:

```bash
sudo service apache2 restart
```

### **Final Note: Customizing Apache Configuration**

If you want to avoid using `/moodle` in your URL, edit the Apache configuration:

```bash
sudo vi /etc/apache2/sites-available/000-default.conf
```

Change:
```bash
DocumentRoot /var/www/html/moodle
```
Restart Apache:
```bash
sudo service apache2 restart
```

### **Explanation**

Each step in this guide serves a specific purpose:

- **Step 1** ensures you have a secure, reliable operating system for hosting Moodle.
- **Step 2 and 3** set up the necessary server software (Apache, MySQL, PHP) to run Moodle.
- **Step 4 and 5** download and prepare Moodle for installation.
- **Step 6 and 7** configure the database and complete the Moodle setup.
- **Step 8 and 9** optimize your Moodle installation for performance and security.
