EC2 to RDS MySQL Connectivity Lab (AWS)
Overview

This lab demonstrates secure, private connectivity between an Amazon EC2 instance and an Amazon RDS MySQL database within a VPC. The architecture follows AWS best practices by keeping the database private and restricting access using security group references instead of public CIDR ranges.

Architecture

EC2: Amazon Linux 2023

RDS: MySQL 8.0 (db.t3.micro)

Networking:

RDS deployed in private subnets

EC2 deployed in a VPC with outbound access

Security:

RDS not publicly accessible

RDS Security Group allows inbound MySQL (3306) only from EC2 Security Group

No 0.0.0.0/0 access

Key AWS Services Used

Amazon EC2

Amazon RDS (MySQL)

Amazon VPC

Security Groups (SG → SG reference)

AWS Secrets Manager

EC2 Instance Connect

Steps Performed
1. Provisioned RDS

Engine: MySQL 8.0

Instance class: db.t3.micro

Private DB subnet group (multiple AZ-capable)

Credentials stored in AWS Secrets Manager

2. Provisioned EC2

Amazon Linux 2023

Connected using EC2 Instance Connect

Installed MariaDB/MySQL-compatible client

3. Security Configuration

RDS inbound rule:

Port: 3306

Source: EC2 Security Group

No public database exposure

4. Verified Connectivity

Connected from EC2 to RDS using:

mysql -h <rds-endpoint> -P 3306 -u admin -p

5. Database Validation
SHOW DATABASES;
CREATE DATABASE armprojectdb;
USE armprojectdb;

CREATE TABLE test_table (
  id INT AUTO_INCREMENT PRIMARY KEY,
  message VARCHAR(100)
);

INSERT INTO test_table (message)
VALUES ('EC2 connected to RDS successfully');

SELECT * FROM test_table;


Successful query output confirmed end-to-end connectivity.

Outcome

✅ Private EC2 → Private RDS connectivity

✅ Secure SG-to-SG access

✅ Database operations validated

✅ Production-aligned AWS architecture

Cleanup

All resources were safely terminated after validation to prevent ongoing costs.

Skills Demonstrated

AWS networking fundamentals

Secure database access patterns

Linux administration (Amazon Linux 2023)

MySQL database operations

AWS security best practices
