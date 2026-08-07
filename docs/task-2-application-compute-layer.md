# AWS Cloud Infrastructure Project — Task 2 Report

## Task Completed

## Task 2 — Build the Application Compute Layer

## Date

August 7, 2026

---

# Objective

The goal of this task was to create the application compute layer of the planned AWS architecture.

The objective was to deploy multiple EC2 instances capable of running a backend application, creating the foundation required for future load balancing and auto scaling.

---

# Architecture Goal

The target design was:

```text
                    Future Stage

              Application Load Balancer

                         |
              ┌──────────┴──────────┐
              |                     |
              ▼                     ▼

            EC2-A                EC2-B

         NestJS API            NestJS API

          Port 3000             Port 3000
```

The idea is that both EC2 instances should be able to independently serve application requests.

---

# Concepts Reviewed

## EC2 Instance vs AMI

Before implementation, the difference between AMI and EC2 was clarified.

## AMI (Amazon Machine Image)

An AMI is a template used to create EC2 instances.

It contains:

* Operating system
* Initial configuration
* Software state

Example:

```text
Ubuntu AMI

      |
      |
      ▼

Creates EC2 Instance
```

---

## EC2 Instance

An EC2 instance is the running virtual machine created from an AMI.

It contains:

* CPU
* Memory
* Storage
* Operating system
* Installed applications
* Running services

---

# Compute Design Decision

Two EC2 instances were selected instead of one.

Reason:

* Simulate a production backend environment.
* Prepare for future load balancing.
* Improve availability.
* Allow traffic distribution between servers.

---

# EC2 Application Servers Created

Two application servers were configured:

## Application Server A

Purpose:

* Run the backend application.
* Act as one node in the compute layer.

---

## Application Server B

Purpose:

* Run an identical backend application.
* Provide a second application node.

---

# Technology Stack Selected

The backend technology used:

```text
Node.js + NestJS
```

Reason:

* Familiar development stack.
* Allows focus on AWS deployment concepts.
* Represents a realistic backend service.

---

# Server Configuration

Each EC2 instance was prepared with:

* Node.js runtime
* npm package manager
* NestJS application environment

---

# NestJS Application Creation

A simple NestJS application was created.

The purpose was not to build a complete product but to create a working backend service that could respond to requests.

The application exposed:

```text
GET /
```

endpoint.

---

# Application Server A

The first EC2 instance was configured to run a NestJS service.

Expected response:

```json
{
  "server": "Application Server A",
  "message": "Hello from EC2-A"
}
```

---

# Application Server B

The second EC2 instance was configured similarly.

The response was customized to identify the server:

```json
{
  "server": "Application Server B",
  "message": "Hello from EC2-B"
}
```

---

# Running the Application

The NestJS application was started using:

```bash
npm run start
```

Successful startup was confirmed through NestJS logs:

```text
Nest application successfully started
```

The application was running on:

```text
Port: 3000
```

---

# Testing Application Server B

The service was tested from inside the EC2 instance:

```bash
curl localhost:3000
```

Response:

```json
{
  "server": "Application Server B",
  "message": "Hello from EC2-B"
}
```

This confirmed:

* Node.js runtime works.
* NestJS application works.
* The EC2 instance can serve backend requests.
* The application is reachable internally.

---

# Current Architecture State

After completing Task 2:

```text
                 AWS Cloud

                     |
                     |

          ┌────────────────────┐
          │                    │
          ▼                    ▼

       EC2-A                EC2-B

       Node.js              Node.js

       NestJS               NestJS

       Port 3000            Port 3000

```

---

# Important Learning Points

## 1. Multiple EC2 instances need the same application capability

If a load balancer sends traffic to multiple servers, every server must be able to handle requests.

Example:

```text
Request 1 → EC2-A ✅

Request 2 → EC2-B ✅
```

---

## 2. Application servers should be stateless

The EC2 instances should not permanently store important user data.

Future services will handle this:

* S3 → Files and objects
* EFS → Shared filesystem
* RDS → Database storage

---

## 3. Manual configuration vs automation

The servers were manually configured for learning purposes.

In production, this process is usually automated using:

* AMIs
* Launch Templates
* User Data scripts
* Infrastructure as Code
* Configuration management tools

---

# Challenges Encountered

## Understanding AMI and EC2 relationship

Initial confusion:

"Is an AMI the virtual machine?"

Resolution:

* AMI = blueprint/template
* EC2 = running virtual machine

---

## Understanding why multiple servers are needed

Clarified that multiple instances allow:

* Redundancy
* Load balancing
* Scaling

---

# Task 2 Status

Completed successfully.

Completed:

✅ EC2 application servers created
✅ Node.js environment configured
✅ NestJS deployed
✅ Backend endpoint created
✅ Application tested successfully
✅ Two-server compute architecture prepared

---

# Next Task

## Task 3 — Networking and Load Distribution

Planned concepts:

* Security Groups
* Application Load Balancer
* Target Groups
* Traffic distribution
* Health checks

