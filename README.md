![Architecture_Reference_for_ Hosting_a_WordPress_Website_on_AWS](https://github.com/user-attachments/assets/da44aecc-dc12-412f-bd49-0b99ae53b965)

---
# **Deploy a Dynamic WordPress Website on AWS**


## **Project Description**
This repository contains resources and scripts to deploy a dynamic WordPress website on Amazon Web Services (AWS). The project uses a variety of AWS services to ensure the WordPress website is highly available, scalable, and secure. It covers essential AWS services, including a three-tier VPC setup with public and private subnets, EC2 instances, RDS for MySQL, EFS, an Application Load Balancer, Auto Scaling, Route 53, SSL certificates, and more.

## **Table of Contents**
- [Architecture Overview](#architecture-overview)
- [AWS Services Used](#aws-services-used)
- [Requirements](#requirements)
- [Deployment Scripts](#deployment-scripts)
  - [WordPress Installation Script](#wordpress-installation-script)
  - [Auto Scaling Group Launch Template Script](#auto-scaling-group-launch-template-script)
  - [HTTPS Listener Script](#https-listener-script)
- [How to Use](#how-to-use)
- [Contribution Guidelines](#contribution-guidelines)
- [License](#license)
- [Acknowledgments](#acknowledgments)

## **Architecture Overview**
1. **VPC**: A Virtual Private Cloud with public and private subnets across two Availability Zones (AZs) for fault tolerance and high availability.
2. **Internet Gateway**: Allows communication between instances within the VPC and the internet.
3. **Security Groups**: Acts as virtual firewalls controlling inbound and outbound traffic to EC2 instances.
4. **Public Subnets**: These are used for the NAT Gateway and Application Load Balancer to facilitate external access and load balancing.
5. **Private Subnets**: Hosts the web servers to enhance security.
6. **EC2 Instance Connect Endpoint**: Provides secure SSH access to the EC2 instances.
7. **Application Load Balancer (ALB)**: Distributes incoming traffic across multiple EC2 instances.
8. **Auto Scaling Group**: Adjusts the number of EC2 instances based on traffic, ensuring scalability and resilience.
9. **Amazon RDS**: A managed relational database service hosting the MySQL database.
10. **Amazon EFS**: Provides scalable, elastic file storage for the WordPress application.
11. **AWS Certificate Manager**: Manages SSL/TLS certificates for secure communication.
12. **AWS SNS**: Used for notifications related to Auto Scaling Group activities.
13. **Amazon Route 53**: Handles domain name registration and DNS management.

## **AWS Services Used**
Here is a brief description of each AWS service utilized in this project:

1. **Amazon VPC**: Virtual Private Cloud allows users to define a logically isolated network environment.
2. **EC2**: Elastic Compute Cloud provides scalable computing capacity in the AWS cloud.
3. **Amazon RDS**: Relational Database Service manages MySQL database instances, reducing database management overhead.
4. **Amazon EFS**: Elastic File System offers scalable file storage that will be shared across multiple instances.
5. **Application Load Balancer (ALB)**: Automatically distributes traffic across multiple EC2 instances, enhancing fault tolerance.
6. **Auto Scaling**: Automatically adjusts the number of EC2 instances to handle traffic surges.
7. **AWS Certificate Manager**: Simplifies SSL/TLS certificate management.
8. **Amazon SNS**: A Simple Notification Service that sends notifications related to Auto Scaling and other system events.
9. **Amazon Route 53**: A scalable DNS web service to route traffic to web applications.
10. **NAT Gateway**: This allows instances in a private subnet to connect to the Internet or other AWS services.
11. **Security Groups**: Act as firewalls for controlling traffic to EC2 instances.
12. **EC2 Instance Connect**: Provides SSH access to EC2 instances without an external key pair.

## **Requirements**
1. An AWS account.
2. Basic knowledge of AWS services navigation.
3. Familiarity with Linux, macOS, or Windows command line and PowerShell.
4. Ability to troubleshoot configuration issues.

## **Deployment Scripts**

### **1. WordPress Installation Script**

This script sets up a WordPress environment on an EC2 instance by installing Apache, PHP, MySQL, and mounting the EFS volume.

```bash
#!/bin/bash
# Update software packages
sudo yum update -y

# Install Apache and start the web server
sudo yum install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd

# Install PHP 8 and necessary extensions for WordPress
sudo dnf install -y php php-cli php-cgi php-curl php-mbstring php-gd php-mysqlnd php-gettext php-json php-xml php-fpm php-intl php-zip php-bcmath php-ctype php-fileinfo php-openssl php-pdo php-tokenizer

# Install MySQL 8
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf install -y mysql-community-server

# Start and enable MySQL
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Mount EFS to the /var/www/html directory
EFS_DNS_NAME=fs-0ce81a1bb251d5ead.efs.us-east-1.amazonaws.com
echo "$EFS_DNS_NAME:/ /var/www/html nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 0" >> /etc/fstab
sudo mount -a

# Set ownership and permissions
sudo chown apache:apache -R /var/www/html

# Restart Apache
sudo systemctl restart httpd
```

### **2. Auto Scaling Group Launch Template Script**

This script is included in the launch template for Auto Scaling Group instances, ensuring new EC2 instances are correctly configured.

```bash
#!/bin/bash
# Update software packages
sudo yum update -y

# Install Apache and start the web server
sudo yum install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd

# Install PHP 8 and necessary extensions for WordPress
sudo dnf install -y php php-cli php-cgi php-curl php-mbstring php-gd php-mysqlnd php-gettext php-json php-xml php-fpm php-intl php-zip php-bcmath php-ctype php-fileinfo php-openssl php-pdo php-tokenizer

# Install MySQL 8
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf install -y mysql-community-server

# Start and enable MySQL
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Mount EFS to the /var/www/html directory
EFS_DNS_NAME=fs-02d3268559aa2a318.efs.us-east-1.amazonaws.com
echo "$EFS_DNS_NAME:/ /var/www/html nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 0" >> /etc/fstab
sudo mount -a

# Set ownership and permissions
sudo chown apache:apache -R /var/www/html

# Restart Apache
sudo systemctl restart httpd
```

### **3. HTTPS Listener Script**

This script is added to the `wp-config.php` file to force SSL encryption between the user and the WordPress website.

```php
/* SSL Settings */
define('FORCE_SSL_ADMIN', true);

// Get true SSL status from AWS Load Balancer
if (isset($_SERVER['HTTP_X_FORWARDED_PROTO']) && $_SERVER['HTTP_X_FORWARDED_PROTO'] === 'https') {
  $_SERVER['HTTPS'] = '1';
}
```

## **How to Use**
1. Clone the repository to your local machine.
2. Review the scripts and customize any necessary parameters, such as EFS DNS or EC2 settings.
3. Deploy the resources using the provided scripts.
4. For details on each AWS service configuration, please take a look at the official AWS documentation ([documentation]( https://docs.aws.amazon.com/).
   
## **Contribution Guidelines**
We welcome contributions! Please follow these steps:
1. Fork this repository.
2. Create a new branch for your feature or bug fix.
3. Commit your changes and push your branch.
4. Submit a pull request for review.

## **License**
This project is licensed under the MIT License. Please have a look at the [LICENSE](LICENSE) file for details.

## **Acknowledgments**
Special thanks to:
- **Azzez Salu**, AWS Cloud/DevOps Engineer at AosNote ([AosNote](https://www.aosnote.com))
- **Lucy Wang**, AWS Solutions Architect ([Tech with Lucy](https://www.youtube.com/@TechwithLucy))
- **Amber Israelsen**, Software Developer and Technical Trainer ([Tiny Technical Tutorials](https://www.youtube.com/@TinyTechnicalTutorials))
