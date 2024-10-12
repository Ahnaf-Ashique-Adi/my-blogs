# Why Do We Need a Server ISO? Can’t We Make a Server Using a Desktop ISO?



## Introduction
When embarking on the journey of setting up a server, one of the first questions that arise is whether to use a server or desktop ISO.
 This topic is essential for anyone looking to create a robust server environment, especially for projects like personal cloud servers. In this blog post, I’ll explore the differences between Linux server and desktop versions and share my personal experience of setting up a Nextcloud server on an old Acer laptop running Ubuntu.

## Linux Server vs. Desktop: What’s the Difference?

### Purpose and Optimization

**Resource Usage: **

- **Server ISO: ** Designed to operate efficiently with minimal resources, making it ideal for hosting services. It often comes without a graphical user interface (GUI) by default, which helps conserve system resources.
- **Desktop ISO: ** Includes a GUI and a wide array of applications suited for general use, which may consume more resources. This can be unnecessary for a server setup.

**Software Packages: **

- **Server ISO: ** pre-installed packages focus on server functionalities (e.g., web servers, databases) and are optimized for performance and stability.
- **Desktop ISO: ** Comes with various applications and utilities aimed at end-user functionality, which might not be needed for a server.

**Maintenance and Updates: **

- **Server ISO: ** Typically receives updates that prioritize security and performance for server applications.
- **Desktop ISO: ** Updates can include features that are not relevant to a server environment, possibly introducing unnecessary complexities.

### Flexibility and Usability
While the server ISO is tailored for server tasks, I found that the desktop version could effectively create a server environment. As someone who recently undertook this challenge, the familiarity of the desktop interface was invaluable. It allowed me to perform various tasks beyond mere server operations, turning my journey into a unique learning experience.

## Conclusion on the Choice
Ultimately, the decision between a server and desktop ISO depends on the user's familiarity with Linux, the resources available, and the specific needs of the project. In many cases, particularly for personal projects, using a desktop ISO can be an excellent starting point.

## Setting Up a Personal Nextcloud Server on Ubuntu (Desktop Version)
After deciding to use the desktop version of Ubuntu for my personal cloud server, I embarked on the following steps:

### Step 1: Update Your System
I began by updating my system to ensure all packages were current. This is essential for security and performance.

```
sudo apt update
sudo apt upgrade
```

### Step 2: Install Required Packages
Next, I installed the packages required for Nextcloud, including Apache, MySQL, and PHP.

```
sudo apt install apache2 mariadb-server php libapache2-mod-php php-mysql php-xml php-curl php-zip php-gd php-mbstring php-intl php-bcmath
```

**Problem Encountered: PHP Compatibility Issues**

Initially, I tried to install PHP 8.3, which Nextcloud does not support. This led to the error message:
> This version of Nextcloud is not compatible with PHP>=8.3. You are currently running 8.3.6.

To resolve this, I researched the issue and discovered that Nextcloud requires PHP 8.1 or lower. I downgraded to PHP 8.1 by updating my package list and installing the correct version:

```
sudo apt install php8.1 libapache2-mod-php8.1 php8.1-mysql php8.1-xml php8.1-curl php8.1-zip php8.1-gd php8.1-mbstring php8.1-intl php8.1-bcmath
```

### Step 3: Configure the Database
I secured my MariaDB installation and created a database for Nextcloud.

```
sudo mysql_secure_installation
```

After securing the installation, I created a database:

```
CREATE DATABASE nextcloud;
CREATE USER 'nextclouduser'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextclouduser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```
*Note: Ensure you replace 'your_password' with a strong password.*

### Step 4: Download Nextcloud
I downloaded the latest version of Nextcloud from the official website and extracted it to the Apache web directory.

```

wget https://download.nextcloud.com/server/releases/nextcloud-x.y.z.zip
unzip nextcloud-x.y.z.zip
sudo mv nextcloud /var/www/html/
```

### Step 5: Configure Apache
To serve Nextcloud, I created a new virtual host configuration file.

```
sudo nano /etc/apache2/sites-available/nextcloud.conf
```

I added the following configuration (replacing `1xx.xxx.x.x` with your local IP):

```


<VirtualHost *:80>
    ServerName 1xx.xxx.x.x
    DocumentRoot /var/www/html/nextcloud

    <Directory /var/www/html/nextcloud>
        Options +FollowSymlinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/nextcloud_error.log
    CustomLog ${APACHE_LOG_DIR}/nextcloud_access.log combined
</VirtualHost>
```

After saving the file, I enabled the site and necessary Apache modules:

```


sudo a2ensite nextcloud
sudo a2enmod rewrite
sudo systemctl restart apache2
```

### Step 6: Access Nextcloud
I accessed Nextcloud through my browser using the local IP address. Here, I was greeted by the Nextcloud setup page.

### Step 7: Complete the Setup
On the setup page, I entered the database details and created an admin account. The installation took a few minutes. I was relieved to finally see the Nextcloud dashboard.

### Step 8: Security Configuration
To enhance security, I attempted to configure SSL using Let's Encrypt. However, I encountered an issue when trying to obtain a certificate for my private IP address:

> One or more of the entered domain names was not valid: 1xx.xxx.x.x

As Let's Encrypt does not issue certificates for bare IP addresses, I opted to create a self-signed certificate instead. This allows me to test the server securely without incurring costs. Here’s how I created a self-signed certificate:

```


sudo apt install openssl
sudo mkdir /etc/ssl/private
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/selfsigned.key -out /etc/ssl/certs/selfsigned.crt
```

During this process, I filled in the details as prompted. For the Organizational Unit, since it’s a personal project, I simply used “Personal Project.”

Next, I updated the Apache configuration to use the self-signed certificate:

```


<VirtualHost *:443>
    ServerName 1xx.xxx.x.x
    DocumentRoot /var/www/html/nextcloud

    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/selfsigned.crt
    SSLCertificateKeyFile /etc/ssl/private/selfsigned.key

    <Directory /var/www/html/nextcloud>
        Options +FollowSymlinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/nextcloud_error.log
    CustomLog ${APACHE_LOG_DIR}/nextcloud_access.log combined
</VirtualHost>
```

I then enabled the SSL module and restarted Apache:

```


sudo a2enmod ssl
sudo systemctl restart apache2
```

### Step 9: Mounting the External SSD for Storage
To expand my storage capacity, I decided to mount an external SSD. This step was crucial for enhancing the storage capabilities of my Nextcloud server. I mounted the SSD by following these commands:

**Identify the SSD:**

```


sudo fdisk -l
```

**Create a mount point: **

```


sudo mkdir /mnt/nextcloud_data
```

**Mount the SSD: **

```


sudo mount /dev/sdX1 /mnt/nextcloud_data
```
*(Replace `/dev/sdX1` with your actual SSD identifier.)*

To make the mount permanent, I added it to `/etc/fstab`:

```


echo '/dev/sdX1 /mnt/nextcloud_data ext4 defaults 0 2' | sudo tee -a /etc/fstab
```

## Final Steps: Testing and Future Improvements
After completing the setup, I tested Nextcloud by uploading files and checking functionalities like sharing and collaboration. Everything worked smoothly!

## Conclusion
Setting up a personal cloud server with Nextcloud on my Ubuntu laptop was a rewarding experience. Each challenge, from PHP compatibility issues to SSL certificate generation, provided valuable learning opportunities. Ultimately, this journey reinforced my technical skills and passion for exploring the possibilities within the Linux ecosystem.

By choosing the path less traveled—using a desktop ISO—I transformed an old laptop into a powerful cloud server, proving that with determination and adaptability, even the most daunting tasks can lead to remarkable achievement.


