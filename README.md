EC2 → RDS MySQL Private Connectivity Lab (AWS Console)
What this demonstrates (why a recruiter should care)

This lab proves I can deploy a private RDS MySQL database and securely connect to it from an EC2 instance inside the same VPC using Security Group–to–Security Group access (no public database exposure, no open CIDR ranges).

In real environments, this pattern is used for:

Web/app servers connecting to private databases

Removing public attack surface (no public DB)

Least-access networking using SG references instead of 0.0.0.0/0

Target outcome (success criteria)

✅ EC2 can connect to RDS using the private endpoint
✅ RDS is not publicly accessible
✅ RDS inbound rule allows MySQL 3306 only from EC2 Security Group
✅ I can run real SQL operations: create DB, create table, insert, select

Architecture (high level)

EC2: Amazon Linux 2023

RDS: MySQL 8.0 (db.t3.micro)

Networking:

RDS deployed into private subnets

DB Subnet Group includes 2 subnets in different AZs

Security:

RDS inbound: MySQL 3306 from EC2 Security Group only

No public database access

Step-by-step (AWS Console)
Step 1 — Create/confirm your private DB subnet group

Why: RDS must live in private subnets for secure architecture (no public exposure).

Go to RDS → Subnet groups → Create DB subnet group

Name: armproj-rds-private-subnet-group

Select your Lab VPC

Select ONLY private subnets

Must include at least 2 subnets in different AZs

Create subnet group

✅ Result: RDS can be placed privately and multi-AZ-capable.

Step 2 — Create the RDS MySQL instance (private)

Why: Database should never be directly exposed to the internet.

Go to RDS → Databases → Create database

Engine: MySQL

Template: Free tier / Dev/Test (lab use)

Instance class: db.t3.micro (cheapest for this lab)

Storage: default is fine

Connectivity:

Select your Lab VPC

DB subnet group: armproj-rds-private-subnet-group

Public access: NO ✅

Authentication:

Master username: admin

Store credentials in Secrets Manager (recommended)

Create database

✅ Result: RDS shows Status: Available and Publicly accessible: No

Step 3 — Create the security groups (secure DB access)

Why: The correct best-practice is SG → SG access, not CIDR blocks.

3A) EC2 Security Group (example)

Inbound:

SSH (22) from your IP (or Instance Connect handles this)
Outbound:

Allow all outbound (default)

3B) RDS Security Group (critical rule)

Inbound:

Type: MySQL/Aurora

Port: 3306

Source: EC2 Security Group (NOT 0.0.0.0/0)

✅ Result: Only the EC2 instance can talk to the database.

Note: If the rule was previously created with a CIDR like 0.0.0.0/0, AWS won’t let you “convert” it to a security group reference. The fix is to delete the CIDR rule and recreate it as SG → SG.

Step 4 — Confirm RDS connection details (endpoint + port)

Why: The endpoint is how your app/EC2 knows where the database lives.

Go to RDS → Databases → (your DB) → Connectivity & security

Copy:

Endpoint: database-armproject.cijmucwoo5ud.us-east-1.rds.amazonaws.com

Port: 3306

Step 5 — Connect to EC2 using EC2 Instance Connect (browser SSH)

Why: Fast access without local SSH key configuration.

Go to EC2 → Instances → (your instance) → Connect

Choose EC2 Instance Connect

Click Connect

✅ Result: You see a terminal prompt like:

ec2-user@ip-10-x-x-x:~$

Step 6 — Install MySQL client on Amazon Linux 2023

Why: Amazon Linux 2023 doesn’t have a package named mysql. MariaDB client is wire-compatible.

Attempting:

sudo dnf install -y mysql


Expected:

No match for argument: mysql

Correct install:

sudo dnf install -y mariadb105


Verify:

mysql --version


✅ Result:

mysql  Ver 15.1 Distrib 10.5.x-MariaDB...

Step 7 — Retrieve DB password from Secrets Manager

Why: Credentials should be stored securely, not hardcoded.

Go to Secrets Manager

Open the secret (example name pattern):

rds!db-...

Click Retrieve secret value

Copy the value from the JSON:

"password": "..."

✅ Important: The secret name is not the password.

Step 8 — Connect from EC2 to RDS (final connectivity test)

Run:

mysql \
-h database-armproject.cijmucwoo5ud.us-east-1.rds.amazonaws.com \
-P 3306 \
-u admin \
-p


Paste the password when prompted.

✅ Success looks like:

Welcome to the MariaDB monitor...
MySQL [(none)]>

Step 9 — Prove database operations work (SQL validation)

Run these inside the MySQL [(none)]> prompt:

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


✅ Expected output includes the inserted row, proving read/write capability.

Results / Proof

✅ EC2 → private RDS connectivity established

✅ RDS not publicly accessible

✅ Access restricted via SG → SG reference

✅ DB operations successful (create DB/table, insert/select)

Cleanup (to avoid costs)

Recommended teardown order:

Delete RDS instance (largest cost driver)

Terminate EC2 instance

Delete RDS security group (if lab-specific)

Delete DB subnet group (if lab-specific)

Delete VPC resources (if dedicated lab VPC)

Skills demonstrated

AWS VPC + subnet architecture for private databases

RDS provisioning and secure configuration

Security group referencing (least access)

EC2 Instance Connect operations

Linux package management (Amazon Linux 2023)

MySQL administration and validation queries

Secrets Manager credential handling
