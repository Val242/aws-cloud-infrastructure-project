# Task 3 — Application Load Balancer & Traffic Distribution

## Objective

Configure an AWS Application Load Balancer (ALB) to expose the NestJS backend and distribute incoming traffic across two EC2 application servers.

The goal was to replace the manual load-balancing role previously performed by Nginx in my Vagrant projects with AWS's managed Application Load Balancer.

---

## Architecture

```text
                         Internet
                            |
                            |
                         Browser
                            |
                       HTTP :80
                            |
                            ▼
                Application Load Balancer
                            |
                            ▼
                     Target Group
                         :3000
                       /       \
                      /         \
                     ▼           ▼
                  EC2-A        EC2-B
                  :3000        :3000
                    |             |
                  NestJS        NestJS
```

The ALB acts as the public entry point for the application.

The two EC2 instances run identical NestJS backend applications, but each returns a different response so that load balancing can be observed during testing.

---

## Target Group

Before configuring the ALB, a target group was created.

The target group contains the EC2 instances that can receive traffic from the ALB.

```text
techblog-backend-tg

├── EC2-A :3000
└── EC2-B :3000
```

The target group uses HTTP on port `3000`, which is the port on which the NestJS applications are running.

Health checks are used by the ALB to determine whether the targets are healthy and capable of receiving traffic.

---

## Application Load Balancer

An internet-facing Application Load Balancer was created.

The ALB listener accepts HTTP traffic on port `80`.

The listener forwards requests to the backend target group.

Therefore, the external and internal ports are different:

```text
Client → ALB :80 → Target Group :3000 → EC2 :3000
```

The browser does not need to know that the NestJS application is running on port `3000`.

---

## Security Groups

The EC2 security group was configured to allow TCP traffic on port `3000` from the security group associated with the ALB.

This means the application port does not need to be openly exposed to the entire internet.

Conceptually:

```text
ALB Security Group
        |
        | TCP :3000
        ▼
EC2 Security Group
        |
        ▼
NestJS Application
```

---

## Testing

The NestJS application on EC2-A was tested directly:

```bash
curl localhost:3000
```

Response:

```json
{
  "server": "Application Server A",
  "message": "Hello from EC2-A"
}
```

The application on EC2-B was also tested and returned its own response.

The ALB DNS name was then tested:

```bash
curl http://<ALB-DNS-NAME>
```

The request successfully reached EC2-A:

```json
{
  "server": "Application Server A",
  "message": "Hello from EC2-A"
}
```

A subsequent request reached EC2-B:

```json
{
  "server": "Application Server B",
  "message": "Hello from EC2-B"
}
```

The ALB therefore successfully distributed traffic between the two EC2 instances.

The ALB was also tested from a web browser successfully.

---

## Important Concepts Learned

### 1. ALB can replace Nginx for load balancing

In previous Vagrant projects, Nginx was used as a reverse proxy and load balancer:

```text
Client
  ↓
Nginx
  ↓
Application Servers
```

With AWS:

```text
Client
  ↓
Application Load Balancer
  ↓
Application Servers
```

Therefore, Nginx is not required simply to perform load balancing when an AWS ALB is being used.

---

### 2. Listener port and target port can be different

The ALB listens on port `80`, while the NestJS applications listen on port `3000`.

```text
ALB :80
   ↓
Target Group :3000
   ↓
EC2 :3000
```

The ALB handles the translation between the client-facing listener and the backend target.

---

### 3. Target groups define where traffic goes

The ALB does not randomly communicate with every EC2 instance in the account.

It forwards traffic to the targets registered in its target group.

The target group also provides health information so that unhealthy instances can be excluded from receiving traffic.

---

### 4. The ALB provides a single entry point

From the user's perspective, there is only one backend endpoint:

```text
Browser
   ↓
ALB DNS
```

The user does not need to know whether the request is eventually handled by EC2-A or EC2-B.

---

## Final Result

Task 3 successfully produced a working load-balanced backend:

```text
                         Browser
                            |
                            ▼
                    Application ALB
                       HTTP :80
                            |
                            ▼
                     Target Group
                       HTTP :3000
                      /          \
                     ▼            ▼
                  EC2-A         EC2-B
                  NestJS        NestJS
                  :3000         :3000
```

Both EC2 instances were successfully registered as healthy targets, and requests through the ALB were successfully distributed between them.

**Task 3 — COMPLETE ✅**

