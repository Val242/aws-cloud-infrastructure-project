# Task 6 – Amazon RDS PostgreSQL

## Objective

The objective of this task was to introduce a managed relational database into the AWS architecture using **Amazon RDS with PostgreSQL**.

The goal was not only to create an RDS database, but also to understand how an EC2 application server securely communicates with a managed database inside the VPC.

---

## 1. Created the RDS Database

I created an Amazon RDS database using PostgreSQL.

### Database

```text
Identifier: techblog-db
Engine: PostgreSQL
Database: techblogdb
Username: devops
Port: 5432
```

The RDS instance provides a managed PostgreSQL database without requiring me to install and maintain the PostgreSQL server directly on an EC2 instance.

---

## 2. Understanding RDS in the Architecture

Before RDS, the application architecture consisted primarily of:

```text
Internet
   |
   v
Application Load Balancer
   |
   v
Target Group
   |
   v
Auto Scaling Group
   |
   +---- EC2
   +---- EC2
   +---- EC2
```

RDS adds a centralized managed database:

```text
                 Application Load Balancer
                          |
                          v
                    Target Group
                          |
                          v
                   Auto Scaling Group
                    /      |      \
                   EC2    EC2    EC2
                    \      |      /
                     \     |     /
                       RDS PostgreSQL
```

The EC2 instances act as application servers, while RDS manages the database layer.

---

## 3. RDS Security Group Configuration

Initially, the RDS instance was associated with the wrong security group.

When I attempted to connect from EC2 to RDS, the connection timed out:

```text
EC2 → RDS:5432
        |
        X
     TIMEOUT
```

I investigated the issue by testing the PostgreSQL port from the EC2 instance.

After identifying the incorrect security group, I associated the correct security group with the RDS instance.

The RDS security group was configured to allow PostgreSQL traffic:

```text
Protocol: TCP
Port: 5432
Source: EC2/Application Security Group
```

This means that application servers belonging to the appropriate EC2 security group can connect to the database without exposing PostgreSQL unnecessarily to the public internet.

---

## 4. Testing Network Connectivity

I installed the `nc` networking utility on the EC2 instance:

```bash
sudo dnf install -y nmap-ncat
```

I then tested connectivity to the RDS endpoint:

```bash
nc -vz techblog-db.c4v0gi20c29q.us-east-1.rds.amazonaws.com 5432
```

Initially:

```text
Ncat: TIMEOUT.
```

This confirmed that the EC2 instance could not reach RDS on port 5432.

After correcting the RDS security group, the test succeeded:

```text
Ncat: Connected to 172.31.92.84:5432.
```

This proved that the network path between EC2 and RDS was working.

---

## 5. Installed the PostgreSQL Client

I installed the PostgreSQL client on the EC2 instance.

The PostgreSQL server itself did not need to be installed because RDS already manages the database server.

The architecture is therefore:

```text
EC2
 |
 | PostgreSQL client (psql)
 |
 | TCP 5432
 v
RDS
 |
 PostgreSQL database server
```

I verified the client and used `psql` to connect to the RDS database.

---

## 6. Configured SSL Connection

I downloaded the AWS RDS CA certificate bundle:

```bash
curl -o global-bundle.pem https://truststore.pki.rds.amazonaws.com/global/global-bundle.pem
```

I then configured the RDS endpoint:

```bash
export RDSHOST="techblog-db.c4v0gi20c29q.us-east-1.rds.amazonaws.com"
```

The connection command used SSL verification:

```bash
psql "host=$RDSHOST port=5432 dbname=techblogdb user=devops sslmode=verify-full sslrootcert=./global-bundle.pem"
```

The connection eventually succeeded.

The output confirmed:

```text
SSL connection (protocol: TLSv1.3, ...)
```

This demonstrated that the EC2 instance was communicating with RDS using an encrypted TLS connection.

---

## 7. Troubleshooting Authentication

After fixing the networking issue, the first authentication attempt failed:

```text
FATAL: password authentication failed for user "devops"
```

This demonstrated an important distinction:

* A network timeout indicates a connectivity/networking problem.
* A password authentication error indicates that the database was reachable but the credentials were incorrect.

After using the correct password, the connection succeeded:

```text
psql (15.18, server 18.3)
SSL connection (protocol: TLSv1.3, ...)
techblogdb=>
```

---

## 8. Verified the Database Connection

Inside PostgreSQL, I verified the current database:

```sql
SELECT current_database();
```

Result:

```text
techblogdb
```

I also verified the authenticated user:

```sql
SELECT current_user;
```

Result:

```text
devops
```

---

## 9. Performed Database Operations

To prove that the database was fully usable, I created a test table:

```sql
CREATE TABLE test_users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);
```

I verified the table:

```sql
\dt
```

Result:

```text
Schema |    Name    | Type  | Owner
-------+------------+-------+-------
public | test_users | table | devops
```

I then inserted a record:

```sql
INSERT INTO test_users (name)
VALUES ('AWS RDS Test');
```

Finally, I queried the table:

```sql
SELECT * FROM test_users;
```

Result:

```text
id |     name
---+--------------
1  | AWS RDS Test
```

This confirmed that the EC2 instance could successfully perform database operations on the RDS PostgreSQL instance.

---

## 10. Important Troubleshooting Lessons

This task provided several practical lessons.

### Security groups affect database connectivity

The initial connection failed because the RDS instance was associated with the wrong security group.

The troubleshooting process was:

```text
psql hangs
   ↓
Test with nc
   ↓
Port 5432 TIMEOUT
   ↓
Check RDS security group
   ↓
Correct security group
   ↓
Port 5432 accessible
```

### Network connectivity and authentication are different problems

After fixing the network issue, authentication initially failed:

```text
Network: ✅
Port 5432: ✅
Authentication: ❌
```

After correcting the password:

```text
Network: ✅
Port 5432: ✅
Authentication: ✅
SSL: ✅
SQL operations: ✅
```

This is an important troubleshooting pattern when working with distributed systems.

---

## 11. RDS vs Running PostgreSQL on EC2

One of the key concepts learned from this task is the difference between running a database yourself and using a managed service.

With a self-managed database:

```text
EC2
 └── PostgreSQL Server
```

I would be responsible for installing, configuring, patching, backing up, monitoring, and maintaining PostgreSQL.

With RDS:

```text
EC2 Application
       |
       v
RDS PostgreSQL
```

AWS manages much of the underlying database infrastructure while the application communicates with the database through the network.

---

## 12. RDS and EFS Have Different Roles

This task also helped clarify the difference between database storage and shared file storage.

### RDS

Stores structured application data and metadata:

```text
User
Image metadata
Product
Order
Account
```

### EFS

Provides shared filesystem storage:

```text
/mnt/efs/images/
/mnt/efs/uploads/
```

For example, an application could store:

```text
RDS:
image_id = 42
filename = profile.jpg
path = /mnt/efs/images/profile.jpg
```

while the actual image file is stored on EFS.

EFS is therefore a shared filesystem rather than a database.

---

## Architecture After Task 6

The project architecture now contains:

```text
                         Internet
                            |
                            v
                  Application Load Balancer
                            |
                            v
                      Target Group
                            |
                            v
                   Auto Scaling Group
                     /      |      \
                    EC2    EC2     EC2
                     |      |       |
                     |      |       |
                     +------+-------+
                            |
                 +----------+----------+
                 |                     |
                 v                     v
                EFS                   RDS
          Shared Filesystem      PostgreSQL
                                  Port 5432
```

The responsibilities are separated:

```text
ALB
→ distributes incoming traffic

ASG
→ manages EC2 capacity

EC2
→ runs the NestJS application

EFS
→ provides shared filesystem storage

RDS
→ provides managed PostgreSQL database storage
```

---

## Key Lessons

Through this task, I learned:

* How to create an RDS PostgreSQL database.
* Why RDS is preferable to manually running a database server on EC2 for many workloads.
* How RDS communicates with EC2 through private networking.
* How security groups control database access.
* Why PostgreSQL uses port `5432`.
* How to troubleshoot network timeouts using `nc`.
* How to distinguish networking problems from authentication problems.
* How to connect to RDS using the PostgreSQL `psql` client.
* How SSL/TLS protects the database connection.
* How to verify the active database and authenticated user.
* How to perform basic SQL operations against RDS.
* The difference between RDS database storage and EFS shared file storage.
* How RDS fits into a horizontally scaled EC2 architecture.

---

## Status

**Task 6 – Amazon RDS PostgreSQL: COMPLETED ✅**

