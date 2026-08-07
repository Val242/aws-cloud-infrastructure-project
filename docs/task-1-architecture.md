# AWS Cloud Infrastructure Project — Task 1 Report

## Task Completed

## Task 1 — Architecture Design

## Date

August 7, 2026

---

# Objective

The goal of this task was to design a cloud architecture for a scalable backend application using AWS managed services.

The focus was understanding where each AWS service fits in a production-style architecture before implementing anything.

---

# Proposed Architecture Overview

The planned architecture is a highly available backend system using:

* EC2
* EBS
* EFS
* S3
* RDS
* CloudWatch
* Auto Scaling
* Application Load Balancer (future stage)

The architecture follows the principle of separating responsibilities:

* Compute layer → EC2
* Storage layer → EBS, EFS, S3
* Database layer → RDS
* Monitoring layer → CloudWatch
* Scaling layer → Auto Scaling

---

# High-Level Architecture

```text
                         Users

                           |
                           |
                    Domain Name System

                           |
                           |
              Application Load Balancer
                    (Future Stage)

                           |
              ┌────────────┴────────────┐
              |                         |
              ▼                         ▼

           EC2-A                    EC2-B

        NestJS App                NestJS App

              |                         |
              └────────────┬────────────┘

                           |
                           |
                         RDS

                    Relational Database


Shared Services:

        EFS  → Shared application files

        S3   → Object storage
              (images, documents, uploads)

        EBS  → Instance storage volumes

        CloudWatch → Monitoring
```

---

# AWS Services Selected

## 1. EC2 — Compute Layer

Purpose:

Run the backend application servers.

Design decision:

Multiple EC2 instances will be used instead of a single server.

Reason:

* High availability
* Better fault tolerance
* Ability to distribute traffic
* Foundation for Auto Scaling

The EC2 instances will run:

* Node.js runtime
* NestJS backend application
* Nginx reverse proxy (later stage)

---

# 2. EBS — Block Storage

Purpose:

Provide persistent block storage attached to EC2 instances.

Planned usage:

* Separate application-related storage from the operating system.
* Store data that requires block-level access.
* Use snapshots for backup and recovery.

Concept:

```text
EC2 Instance

    |
    |
   EBS Volume

    |
    |
Application Data
```

---

# 3. EFS — Shared File Storage

Purpose:

Provide shared storage accessible by multiple EC2 instances.

Reason:

EC2 instances should remain stateless.

Instead of:

```text
EC2-A
 |
Local Files

EC2-B
 |
Different Local Files
```

Use:

```text
          EFS

       /       \

    EC2-A    EC2-B
```

Both application servers can access the same files.

---

# 4. S3 — Object Storage

Purpose:

Store application objects such as:

* Images
* Documents
* User uploads
* Static files

Reason:

Large files should not be stored directly inside EC2 instances.

Benefits:

* Highly durable storage
* Independent from compute
* Easy scaling

---

# 5. RDS — Database Layer

Purpose:

Provide managed relational database services.

The application servers will communicate with RDS.

Architecture:

```text
EC2 Application Servers

          |
          |
          ▼

          RDS Database
```

Benefits:

* Managed database
* Automated backups
* Easier maintenance
* High availability options

---

# 6. CloudWatch — Monitoring

Purpose:

Monitor AWS resources and application health.

Planned monitoring:

* EC2 metrics
* CPU utilization
* Memory usage
* Application logs
* Resource usage

Purpose:

Prevent unexpected failures and control costs.

---

# 7. Auto Scaling

Purpose:

Automatically adjust the number of EC2 instances depending on demand.

Example:

Low traffic:

```text
EC2-A
```

High traffic:

```text
EC2-A
EC2-B
EC2-C
EC2-D
```

Benefits:

* Handles traffic spikes
* Reduces unnecessary cost
* Improves availability

---

# Frontend Decision

For this project:

A frontend application will not be built.

The focus is the backend infrastructure.

The architecture will simulate a backend API service.

---

# Backend Flow

The expected request flow:

```text
User

 |
 |
Domain

 |
 |
Load Balancer

 |
 |
EC2 Application Servers

 |
 |
RDS Database
```

Supporting services:

```text
EC2 → EBS
EC2 → EFS
EC2 → S3
CloudWatch → Monitoring
```

---

# Design Principles Applied

## Stateless Application Servers

The application servers should not store important data locally.

Reason:

Any EC2 instance should be replaceable.

---

## Separation of Concerns

Each AWS service has a clear responsibility:

| Service      | Responsibility      |
| ------------ | ------------------- |
| EC2          | Run application     |
| EBS          | Block storage       |
| EFS          | Shared filesystem   |
| S3           | Object storage      |
| RDS          | Database            |
| CloudWatch   | Monitoring          |
| Auto Scaling | Capacity management |

---

# Current Status

Task 1 Completed:

✅ Architecture designed
✅ AWS services selected
✅ Responsibilities defined
✅ Data flow understood

Next Task:

## Task 2 — Build Application Compute Layer

Planned work:

* Create EC2 instances
* Install Node.js
* Deploy NestJS application
* Prepare servers for future Load Balancing

