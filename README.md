# HOW TO DEPLOY A DYNAMIC WEBSITE IN AN EC2 INSTANCE

                         

## Project Overview
This project demonstrates hosting a Dynamic website app on AWS using various resources like VPC, subnets, Internet Gateway, Nat Gateway, Bastion host, Application Load Balancer, EC2 instances, Auto Scaling Group, and Route 53. The project aims to achieve high availability, fault tolerance, scalability, and elasticity for the web app.

## Architecture
![Project Architecture](https://github.com/taofeeklanre/your-repo-name/blob/main/assets/vpc-architecture.jpg?raw=true)

VPC Configuration:

In our Reference Architecture above,
1.	Vpc- We will create our Virtual Private Cloud (VPC) where we will launch aws resources for the project.
   
3.	Internet Gateway-This will enable the resources in the VPC to connect with the internet. E.g ec2 instance in our subnets
   
5.	Availability Zone- In order for us to have high availability and fault tolerant, we will be using 2 number of availability zone.
   
7.	Public Subnet-This will host our resources such as internet gateway, bastion Host and Nat Gateway.
   
9.	Private App Subnet- This will host our webserver, ec2 instance connect endpoint and database server.
    
11.	Nat-Gateway-This will enable the resources in our private subnets (ec2 instance and MySQL database) to connect to the internet.
    
13.	Relational Database Service (RDS)- This will store our data as table, record and field mostly used for transaction database. We are using MySQL database.
    
15.	Webserver-This is the ec2 instance that will host our website.
    
17.	Application Load Balance-This is going to distribute our traffic to both webservers in multiple availability zones.
    
19.	Auto-Scaling Group-This will dynamically create and terminate our ec2 instances so that the website will be highly available, fault-tolerance and scalable.
    
21.	Amazon Route 53- This will create the domain name and manage and direct traffic to our website.
    
23.	AWS S3- This will store our application code (Web file)
    
25.	Identity Access Management (IAM) Role- The role created will allow the EC2 instance to be able to download the web file stored in the S3 bucket.
    
27.	Amazon Machine Image (AMI)- We will create an AMI from the EC2 instance we have install and configured the web file on.

## Deployment Script

  ## SHELL SCRIPT TO CONFIGURE THE WEBSERVER
    # This command indicates that the script should be interpreted and executed using the Bash shell
    #!/bin/bash

    # This command updates all the packages on the server to their latest versions
    sudo yum update -y

    # This series of commands installs the Apache web server, enables it to start on boot, and then starts the server immediately
    sudo yum install -y httpd
    sudo systemctl enable httpd 
    sudo systemctl start httpd

    # This command installs PHP along with several necessary extensions for the application to run
    sudo dnf install -y \
    php \
    php-pdo \
    php-openssl \
    php-mbstring \
    php-exif \
    php-fileinfo \
    php-xml \
    php-ctype \
    php-json \
    php-tokenizer \
    php-curl \
    php-cli \
    php-fpm \
    php-mysqlnd \
    php-bcmath \
    php-gd \
    php-cgi \
    php-gettext \
    php-intl \
    php-zip

    ## These commands Installs MySQL version 8
    To install MYSQL on the server, the following command will be executed.
    sudo dnf search mysql
    sudo dnf update
    sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-4.noarch.rpm
    sudo dnf install mysql80-community-release-el9-4.noarch.rpm
    dnf repolist enabled
    sudo dnf install mysql-community-server -y
    sudo mysql -V
    sudo systemctl start mysqld
    sudo systemctl enable mysqld
    sudo systemctl status mysqld
    
    # This command enables the 'mod_rewrite' module in Apache on an EC2 Linux instance. It allows the use of .htaccess files for URL rewriting and other directives in the '/var/www/html' directory
    sudo sed -i '/<Directory "\/var\/www\/html">/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/httpd/conf/httpd.conf
    
    # Environment Veriable
    S3_BUCKET_NAME=aosnote-shopwise-web-files
    
    # This command downloads the contents of the specified S3 bucket to the '/var/www/html' directory on the EC2 instance
    sudo aws s3 sync s3://taofeeklanre-nest-webfile /var/www/html

    # This command changes the current working directory to '/var/www/html', which is the standard directory for hosting web pages on a Unix-based server
    cd /var/www/html
    
    # This command is used to extract the contents of the application code zip file that was previously downloaded from the S3 bucket
    sudo unzip  nest-app.zip
    
    # This command recursively copies all files, including hidden ones, from the 'shopwise' directory to the '/var/www/html/'
    sudo cp -R nest-app/. /var/www/html/
    
    # This command permanently deletes the 'shopwise' directory and the 'shopwise.zip' file.
    sudo rm -rf nest-app nest-app.zip
    
    # This command set permissions 777 for the '/var/www/html' directory and the 'storage/' directory
    sudo chmod -R 777 /var/www/html
    sudo chmod -R 777 storage/
    
    # This command will open th vi editor and allow you to edit the .env file to add your database credentials 
    sudo vi .env

    # This command will restart the Apache server
    sudo service httpd restart
    
  ## SCRIPT TO MIGRATE FLYWAY
    #!/bin/bash

    # Update all packages
    sudo yum update -y
    
    # Download and extract Flyway
    
    sudo wget -qO- https://download.red-gate.com/maven/release/com/redgate/flyway/flyway-commandline/10.12.0/flyway-commandline-10.12.0-linux-x64.tar.gz | tar -xvz && sudo ln -s `pwd`/flyway-10.12.0/flyway /usr/local/bin 
    
    # OR seperate the symbolic link as below
    # Create a symbolic link to make Flyway accessible globally
    sudo ln -s $(pwd)/flyway-10.9.1/flyway /usr/local/bin

    # Create the SQL directory for migrations
    sudo mkdir sql
    
    # Download the migration SQL script from AWS S3
    sudo aws s3 cp s3://taofeeklanre-nest-sql-file/V1__nest.sql sql/
    
    # Run Flyway migration
    flyway -url=jdbc:mysql://rentzone-db.cf8emlrkavsw.us-east-1.rds.amazonaws.com:3306/applicatiodb \
      -user=taofeek \
      -password=rahmah2005 \
      -locations=filesystem:sql \
      migrate

## Deployment Steps

## Step 1: Set up VPC
Go to the AWS Management Console and navigate to the VPC dashboard.

Click on "Create VPC" and enter the desired details (e.g., VPC name, IPv4 CIDR block).

Click on "Create".

## Step 2: Create Subnets

In the VPC dashboard, navigate to "Subnets".

Click on "Create subnet" and select the VPC created in Step 1.

Enter the subnet details (e.g., name, availability zone, IPv4 CIDR block).

Click on "Create subnet" and repeat for the second subnet in a different availability zone.

## Step 3: Create Internet Gateway (IGW)

In the VPC dashboard, navigate to "Internet Gateways".

Click on "Create internet gateway" and enter a name for the IGW.

Select the IGW and click on "Attach to VPC", then select the VPC created in Step 1.

## Step 4: Configure Route Tables

In the VPC dashboard, navigate to "Route Tables".

Select the default route table for the VPC created in Step 1 and click on "Edit routes".

Add a route with destination 0.0.0.0/0 and target as the IGW created in Step 3.

Click on "Save routes".

In the VPC dashboard, navigate to "Route Tables".

Create a new route table and associate it with the private subnets
.
Edit the new route table and add a route with destination 0.0.0.0/0 and target as the NAT Gateway's ID.

## Step 5: Launch EC2 Instance

Go to the EC2 dashboard and click on "Launch instance".

Select an Amazon Machine Image (AMI) for your EC2 instance (e.g., Amazon Linux 2).

Choose an instance type, configure instance details (e.g., VPC, subnet, auto-assign public IP), and add storage as needed.

Configure security groups to allow inbound traffic on port 80 (HTTP) and 443 (HTTPS) for your web app.

Review and launch the instance, selecting or creating a key pair for SSH access.

## Step 6: Deploy Web App to EC2 Instance

1. SSH into your EC2 instance using the EC2 Instance Connect Endpoint or key pair you downloaded.
   
2. Update the package manager:

    sudo yum update -y

3. Install Apache HTTP Server:
   
    sudo yum install -y httpd

4. Download and deploy your web app files (e.g., from GitHub):

    cd /var/www/html
   
    wget https://github.com/taofeeklanre/static-website-Repository/raw/main/Speed%20Free%20Cycle.zip

     unzip Speed\ Free\ Cycle.zip 

     cp -r /var/www/html/speed/* /var/www/html

     cd /var/www/html

     rm -rf Speed\ Free\ Cycle.zip

6. Start the Apache service and enable it to start on boot:

    sudo systemctl start httpd

    sudo systemctl enable httpd

## Step 7: Create a NAT Gateway

In the VPC dashboard, navigate to "NAT Gateways".

Click on "Create NAT Gateway" and select the public subnet and Elastic IP address.

Click on "Create NAT Gateway".

## Step 8: Update Route Table for Private Subnets

In the VPC dashboard, navigate to "Route Tables".

Edit the route table associated with the private subnets.

Add a route with destination 0.0.0.0/0 and target as the NAT Gateway created in Step 7.

## Step 9: Create Application Load Balancer (ALB)

In the EC2 dashboard, navigate to "Load Balancers".

Click on "Create Load Balancer" and select "Application Load Balancer".

Configure the ALB with listeners (e.g., HTTP on port 80), availability zones, and security settings.

Add the EC2 instances from Step 5 to the ALB's target group.

## Step 10: Update Security Groups

Update the security group associated with the ALB to allow inbound traffic on port 80 (HTTP) and 443 (HTTPS).

Update the security group associated with the EC2 instances to allow inbound traffic on port 80 (HTTP) from the ALB's security group.

## Step 11: Update Route 53

In the Route 53 dashboard, click on "Hosted zones" and select your domain.

Click on "Create Record Set" and configure the record set to point to the ALB's DNS name.

Save the record set.

## Step 12: Access Your Website

Wait for the DNS changes to propagate (this may take some time).

Use your domain name to access your website.

By following these additional steps, you can set up Route 53 for DNS management, a NAT Gateway for internet access from private subnets, and an Application Load Balancer for distributing traffic to your EC2 instances.

## Repository Structure
deploy.sh: Bash script for deploying the website to EC2 instances.

html.png: Reference architecture diagram for the project.

finexo.zip: Application code in zip format

others: Application code in file and folder.

html1.png: Website displayed for the application.


README.md: This README file.

## Website displayed

## Additional Resources
AWS VPC Documentation

AWS EC2 Documentation

AWS Auto Scaling Documentation

AWS Route 53 Documentation

GitHub Guides

## Conclusion

This project demonstrates how to host a static website on AWS with high availability, fault tolerance, scalability, and elasticity using various AWS services and resources. The use of VPC, subnets, Internet Gateway, Nat Gateway, Bastion host, Application Load Balancer, EC2 instances, Auto Scaling Group, Route 53, and GitHub ensures a robust and reliable hosting environment for the web app.


