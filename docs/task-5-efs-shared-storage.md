# Task 5 – Amazon EFS Shared Storage

## Objective

The objective of this task was to understand and implement **Amazon Elastic File System (EFS)** as shared storage for multiple EC2 instances.

The main goal was to demonstrate how multiple application servers can access the same filesystem, which becomes important when an application is running across multiple EC2 instances behind a load balancer.

---

## 1. Created the EFS Filesystem

I created an Amazon EFS filesystem for the project.

The filesystem was named:

```text
techblog-efs
```

The purpose of this filesystem is to provide shared storage that can be accessed by multiple EC2 instances.

Instead of storing files independently on each EC2 instance:

```text
EC2-A → Local Storage
EC2-B → Local Storage
EC2-C → Local Storage
```

EFS provides a shared filesystem:

```text
              EFS
           /   |   \
          /    |    \
       EC2-A EC2-B EC2-C
```

---

## 2. Created an EFS Security Group

I created a dedicated security group for the EFS filesystem:

```text
techblog-efs-sg
```

The security group controls access to the EFS filesystem.

EFS uses the **NFS protocol**, which communicates over:

```text
TCP port 2049
```

Access to the filesystem is therefore controlled through the EFS security group.

---

## 3. Created the Mount Point

On the EC2 instance, I created the directory that would be used as the EFS mount point:

```bash
sudo mkdir /mnt/efs
```

The mount point chosen for this project was:

```text
/mnt/efs
```

This means that once EFS is mounted, applications can access the shared filesystem through:

```text
/mnt/efs
```

---

## 4. Configured `/etc/fstab`

I configured `/etc/fstab` so that the EFS filesystem could be mounted using the system's normal mount mechanism.

Initially, running:

```bash
mount -a
```

failed because the mount directory did not exist:

```text
mount.nfs4: mount point /mnt/efs does not exist
```

I resolved this by creating the directory:

```bash
mkdir /mnt/efs
```

I then ran:

```bash
mount -a
```

and the filesystem mounted successfully.

---

## 5. Verified the EFS Mount

I verified the mounted filesystem using:

```bash
df -h
```

The output showed:

```text
127.0.0.1:/    8.0E    0    8.0E    0%    /mnt/efs
```

This confirmed that the EFS filesystem was successfully mounted at:

```text
/mnt/efs
```

---

## 6. Tested Shared Storage Between EC2 Instances

The most important part of the task was proving that the filesystem was actually shared between multiple EC2 instances.

### EC2-A

I created a test file on the EFS filesystem:

```bash
echo "Hello from EC2-A through EFS" | sudo tee /mnt/efs/test.txt
```

I then verified the file:

```bash
cat /mnt/efs/test.txt
```

The result was:

```text
Hello from EC2-A through EFS
```

---

### EC2-B

I accessed the same EFS filesystem from another EC2 instance and ran:

```bash
cat /mnt/efs/test.txt
```

The file created by EC2-A was successfully accessible from EC2-B.

This demonstrated that both EC2 instances were accessing the same underlying EFS filesystem.

---

## 7. Tested Bidirectional Access

To further verify the shared filesystem, I created another file from EC2-B:

```bash
echo "Hello from EC2-B through EFS" | sudo tee /mnt/efs/test-b.txt
```

I then returned to EC2-A and ran:

```bash
cat /mnt/efs/test-b.txt
```

The file created by EC2-B was successfully accessible from EC2-A.

Therefore, the shared storage was confirmed in both directions:

```text
EC2-A
  |
  | write
  v
 EFS
  |
  | read
  v
EC2-B
```

and:

```text
EC2-B
  |
  | write
  v
 EFS
  |
  | read
  v
EC2-A
```

---

## 8. Why EFS Is Important in a Scaled Architecture

This exercise demonstrated why shared storage becomes important when an application runs on multiple EC2 instances.

Without shared storage, a file uploaded to EC2-A would exist only on EC2-A's local filesystem.

For example:

```text
User
  |
  v
ALB
  |
  v
EC2-A
  |
  └── /uploads/image.jpg
```

If the next request is routed to EC2-B:

```text
User
  |
  v
ALB
  |
  v
EC2-B
```

EC2-B would not automatically have access to the file stored on EC2-A.

With EFS:

```text
                 ALB
                  |
          +-------+-------+
          |       |       |
         EC2-A  EC2-B  EC2-C
          |       |       |
          +-------+-------+
                  |
                 EFS
```

all application servers can access the same shared filesystem.

This makes EFS useful for workloads that require shared files across multiple EC2 instances.

---

## Key Lessons

This task helped me understand:

* What Amazon EFS is.
* How EFS differs from EC2 local storage.
* How multiple EC2 instances can access the same filesystem.
* How NFS is used to access EFS.
* The importance of port `2049` for NFS traffic.
* How security groups control access to EFS.
* How to mount EFS on an EC2 instance.
* How `/etc/fstab` can be used for persistent mounting.
* Why shared storage can be important in horizontally scaled applications.

Most importantly, I learned this through practical testing rather than simply reading about EFS.

I created a file from one EC2 instance, accessed it from another instance, then performed the reverse operation. This confirmed that both instances were interacting with the same shared filesystem.

---

## Current Architecture

The architecture now includes shared storage alongside the load balancing and auto scaling infrastructure:

```text
                         Application Load Balancer
                                  |
                                  v
                            Target Group
                                  |
                                  v
                         Auto Scaling Group
                                  |
                  +---------------+---------------+
                  |               |               |
                  v               v               v
                EC2-A           EC2-B           EC2-C
              NestJS :3000    NestJS :3000    NestJS :3000
                  |               |               |
                  +---------------+---------------+
                                  |
                                  v
                           Amazon EFS
                         Shared Storage
```

## Status

**Task 5 – Amazon EFS Shared Storage: COMPLETED ✅**

