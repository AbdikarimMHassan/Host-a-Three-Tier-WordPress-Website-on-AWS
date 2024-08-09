# Three-Tier WordPress Website on AWS

##   Project Overview
This repository contains resources and scripts to deploy a WordPress website on Amazon Web Services (AWS). The project leverages various AWS services to ensure high availability, scalability, and security for the WordPress application.

##   Architecture

The WordPress website is hosted on EC2 instances within a highly available and secure architecture that includes:

- **Virtual Private Cloud (VPC):** A VPC with public and private subnets across two Availability Zones (AZs) for fault tolerance and high availability.
- **Internet Gateway:** Allows communication between instances in the VPC and the internet.
- **Security Groups:** Acts as a virtual firewall to control inbound and outbound traffic.
- **Public Subnets:** Used for the NAT Gateway and Application Load Balancer, facilitating external access and load balancing.
- **Private Subnets:** Enhance security by hosting web servers in these subnets.
- **EC2 Instance Connect Endpoint:** For secure SSH access to the instances.
- **Application Load Balancer:** Distributes incoming web traffic across multiple EC2 instances.
- **Auto Scaling Group:** Automatically adjusts the number of EC2 instances based on traffic, ensuring scalability and resilience.
- **Amazon RDS:** Managed relational database service for the WordPress application.
- **Amazon EFS:** Scalable, elastic file storage system for shared file storage across EC2 instances.
- **AWS Certificate Manager:** Manages SSL/TLS certificates for secure communication.
- **AWS Simple Notification Service (SNS):** Sends notifications related to Auto Scaling Group activities.
- **Amazon Route 53:** Domain name registration and DNS management.

## Deployment Scripts

### WordPress Installation Script

This script is used for the initial setup of the WordPress application on an EC2 instance. It includes steps for installing Apache, PHP, MySQL, and mounting the Amazon EFS to the instance.

```bash
# Switch to root user
sudo su

# Update the software packages on the EC2 instance 
sudo yum update -y

# Create an HTML directory 
sudo mkdir -p /var/www/html

# Set the environment variable for EFS DNS Name
EFS_DNS_NAME=fs-064e9505819af10a4.efs.us-east-1.amazonaws.com

# Mount the EFS to the HTML directory 
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport "$EFS_DNS_NAME":/ /var/www/html

# Install the Apache web server, enable it to start on boot, and start the server
sudo yum install -y httpd
sudo systemctl enable httpd 
sudo systemctl start httpd

# Install PHP 8 along with necessary extensions for WordPress
sudo dnf install -y \
php \
php-cli \
php-cgi \
php-curl \
php-mbstring \
php-gd \
php-mysqlnd \
php-gettext \
php-json \
php-xml \
php-fpm \
php-intl \
php-zip \
php-bcmath \
php-ctype \
php-fileinfo \
php-openssl \
php-pdo \
php-tokenizer

# Install the MySQL 8 community repository and server
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm 
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm 
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server 

# Start and enable the MySQL server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Set permissions for Apache and EC2 user
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;

# Set ownership of HTML directory
chown apache:apache -R /var/www/html 

# Download and extract WordPress files
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo cp -r wordpress/* /var/www/html/

# Create the wp-config.php file
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php

# Edit the wp-config.php file as needed
sudo vi /var/www/html/wp-config.php

# Restart the web server
sudo service httpd restart
```

### Auto Scaling Group Launch Template Script

This script is included in the launch template for the Auto Scaling Group, ensuring that new instances are configured correctly with the necessary software and settings.

```bash
#!/bin/bash
# Update the software packages on the EC2 instance 
sudo yum update -y

# Install the Apache web server, enable it to start on boot, and start the server
sudo yum install -y httpd
sudo systemctl enable httpd 
sudo systemctl start httpd

# Install PHP 8 along with necessary extensions for WordPress
sudo dnf install -y \
php \
php-cli \
php-cgi \
php-curl \
php-mbstring \
php-gd \
php-mysqlnd \
php-gettext \
php-json \
php-xml \
php-fpm \
php-intl \
php-zip \
php-bcmath \
php-ctype \
php-fileinfo \
php-openssl \
php-pdo \
php-tokenizer

# Install the MySQL 8 community repository and server
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm 
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm 
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server 

# Start and enable the MySQL server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Set the environment variable for EFS DNS Name
EFS_DNS_NAME=fs-02d3268559aa2a318.efs.us-east-1.amazonaws.com

# Mount the EFS to the HTML directory 
echo "$EFS_DNS_NAME:/ /var/www/html nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 0" >> /etc/fstab
mount -a

# Set ownership of HTML directory
chown apache:apache -R /var/www/html

# Restart the web server
sudo service httpd restart
```

--- 


