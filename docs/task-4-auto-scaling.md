# Task 4 – Auto Scaling Group

## Objective

The objective of this task was to introduce **Auto Scaling** into the AWS infrastructure so that application servers can be launched automatically, configured consistently, and scaled according to workload.

The main goal was to move from manually created EC2 application servers toward a more automated and reproducible infrastructure.

---

## 1. Created a Launch Template

I created a Launch Template to define the configuration that should be used whenever the Auto Scaling Group launches a new EC2 instance.

The Launch Template contains the configuration required for a new application server, including:

* Amazon Machine Image (AMI)
* Instance configuration
* Key pair
* Security group
* User Data provisioning script

The purpose of the Launch Template is to ensure that every new EC2 instance launched by the Auto Scaling Group starts with a consistent configuration.

---

## 2. Configured User Data Provisioning

One of the major lessons from this task was the importance of automatically provisioning newly launched instances.

Initially, newly created instances became unhealthy because the NestJS application was not running on port `3000`.

I therefore added a User Data script to the Launch Template.

The script automatically:

1. Updates the Amazon Linux system.
2. Installs Node.js and npm.
3. Installs the NestJS CLI.
4. Creates a new NestJS application.
5. Builds the application.
6. Starts the NestJS application.

The provisioning script used was:

```bash
#!/bin/bash

dnf update -y

dnf install -y nodejs npm

npm install -g @nestjs/cli

nest new app --package-manager npm --skip-git

cd app

npm run build

nohup npm run start:prod > /var/log/nest-app.log 2>&1 &
```

The NestJS application is created under:

```text
/root/app
```

and runs on port:

```text
3000
```

---

## 3. Verified Automatic Provisioning

I initially suspected that the User Data script had failed because some of the newly launched instances were unhealthy.

Instead of assuming the script was broken, I inspected the cloud-init output:

```bash
sudo cat /var/log/cloud-init-output.log
```

The logs showed that:

* Node.js was successfully installed.
* npm was successfully installed.
* NestJS CLI was successfully installed.
* The NestJS application was successfully created.
* Dependencies were installed.
* The application was successfully built.
* Cloud-init completed successfully.

I also verified that the NestJS production process was running:

```text
node dist/main
```

This confirmed that the User Data provisioning was working automatically on a newly created EC2 instance.

---

## 4. Understanding ALB Health Checks

During the task, several targets initially appeared as unhealthy.

The reason was that the NestJS application was not running on the port configured in the Target Group.

The Target Group was configured to communicate with the application on:

```text
EC2:3000
```

Therefore, the service listening on port `3000` must be running for the ALB health check to succeed.

The request flow is:

```text
Application Load Balancer
        |
        | HTTP :3000
        v
EC2 Instance
        |
        v
NestJS Application :3000
        |
        v
HTTP Response
        |
        v
Healthy Target
```

This helped reinforce the relationship between:

* Load Balancer
* Target Group
* Target port
* Application service
* Health checks

---

## 5. Created the Auto Scaling Group

I created an Auto Scaling Group and configured it with:

```text
Minimum capacity: 2
Desired capacity: 2
Maximum capacity: 5
```

This means the Auto Scaling Group attempts to maintain at least two application servers while allowing the infrastructure to scale up to five instances when required.

The Auto Scaling Group uses the Launch Template to create new instances.

Therefore, newly created instances don't need to be manually configured.

---

## 6. Availability Zone Distribution

I reviewed the Availability Zone distribution options when configuring the Auto Scaling Group.

The purpose is to distribute application instances across Availability Zones so that the application does not depend on a single Availability Zone.

This improves the resilience of the infrastructure.

---

## 7. Scale-In Configuration

I left scale-in enabled.

This means the Auto Scaling Group can remove instances when they are no longer required.

The intended behavior is:

```text
High workload
     |
     v
Scale Out
     |
     v
More EC2 instances

Low workload
     |
     v
Scale In
     |
     v
Fewer EC2 instances
```

This prevents the infrastructure from permanently retaining unnecessary instances.

---

## 8. Configured Target Tracking Scaling Policy

I configured a **Target Tracking** scaling policy using:

```text
Metric: Average CPU Utilization
Target: 50%
```

The purpose is to allow the Auto Scaling Group to automatically adjust the number of EC2 instances based on CPU utilization.

The intended behavior is:

```text
Average CPU rises
        |
        v
Scaling policy detects increased demand
        |
        v
ASG launches additional EC2 instance
        |
        v
Launch Template provisions instance
        |
        v
NestJS starts on port 3000
        |
        v
ALB health check
        |
        v
Healthy target
```

When demand decreases, the Auto Scaling Group can scale back down while respecting the configured minimum capacity.

---

## 9. Important Lesson: Reproducible Infrastructure

One of the biggest lessons from this task was that an Auto Scaling Group cannot depend on manually configured servers.

Initially, I already had two manually configured EC2 instances running the NestJS application.

When the Auto Scaling Group launched additional instances, those new instances did not automatically contain the application environment.

This caused the new targets to fail their health checks.

The solution was to make the application server **reproducible** through the Launch Template and User Data.

Instead of:

```text
Create EC2
     |
SSH into EC2
     |
Install Node
     |
Install NestJS
     |
Start application
```

the infrastructure can now do:

```text
ASG launches EC2
       |
       v
Launch Template
       |
       v
User Data
       |
       v
Install dependencies
       |
       v
Create NestJS application
       |
       v
Build and start application
       |
       v
ALB health check
```

This is an important foundation for automated cloud infrastructure.

---

## 10. Cleanup

At the end of the session, I cleaned up the running resources so I could continue the project later without unnecessarily keeping the EC2 instances running.

I reduced the Auto Scaling Group capacity and stopped the EC2 instances rather than terminating the project infrastructure.

The configuration was preserved so that the project can be continued later.

---

## What I Achieved

By the end of this task, I had:

* Created a Launch Template.
* Added automated User Data provisioning.
* Created an Auto Scaling Group.
* Configured minimum, desired, and maximum capacities.
* Connected the ASG to the Target Group.
* Configured Availability Zone distribution.
* Enabled scale-in.
* Configured a target-tracking scaling policy.
* Set a 50% average CPU utilization target.
* Troubleshot unhealthy ALB targets.
* Verified that User Data successfully provisions a fresh EC2 instance.
* Verified that a newly provisioned instance can run a NestJS application on port `3000`.
* Gained a better understanding of how Auto Scaling, Launch Templates, User Data, Target Groups, health checks, and Application Load Balancers work together.

## Current Architecture

```text
                         Application Load Balancer
                                  |
                                  v
                            Target Group
                                  |
                                  v
                         Auto Scaling Group
                                  |
                     +------------+------------+
                     |                         |
                     v                         v
                  EC2 #1                   EC2 #2
                NestJS :3000             NestJS :3000
                     |                         |
                     +------------+------------+
                                  |
                           Launch Template
                                  |
                             User Data
                                  |
                     Automated provisioning
```

## Next Step

The next step is to **test the scaling behavior** by generating workload and verifying that the Auto Scaling Group can automatically launch an additional instance, provision the NestJS application, pass the ALB health check, and become part of the active target pool.

After verifying scale-out and scale-in behavior, the Auto Scaling task can be considered fully tested.

