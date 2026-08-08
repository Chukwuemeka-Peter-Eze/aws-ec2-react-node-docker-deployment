# AWS EC2 Web Deployment Troubleshooting Guide

## 1. Purpose

This document provides a structured troubleshooting approach for the AWS EC2 web application deployment.

The deployment involves multiple layers:

```text
AWS Infrastructure
       ↓
EC2 Instance
       ↓
SSH Access
       ↓
Docker
       ↓
Container
       ↓
Application
       ↓
Security Group
       ↓
Browser Access
```

When a deployment fails, troubleshooting should proceed from the lower infrastructure layers toward the application layer rather than changing multiple components at the same time.

---

# 2. EC2 Instance Problems

## Symptom

The EC2 instance cannot be reached or the application is unavailable.

### Checks

Verify that the instance is in a running state.

Check:

* Instance state
* Public IPv4 address
* Public DNS
* Attached Security Group
* Key pair associated with the instance

If the instance is stopped or terminated, application access will not work.

---

# 3. SSH Connection Problems

## Symptom

SSH cannot establish a connection to the EC2 instance.

Example:

```text
Permission denied
```

or:

```text
Connection timed out
```

### Possible Causes

* Incorrect private key
* Incorrect SSH username
* Incorrect public IP address
* Private key permissions are too permissive
* SSH traffic is not permitted by the Security Group
* EC2 instance is not running

### Check Private Key Permissions

```bash
chmod 400 <private-key>.pem
```

### Check the Connection

```bash
ssh -i <private-key>.pem <username>@<ec2-public-ip>
```

### Troubleshooting Order

```text
EC2 Running?
     │
     ├── No → Resolve EC2 issue
     │
     ▼
Correct Public IP?
     │
     ├── No → Use current address
     │
     ▼
Correct Key?
     │
     ├── No → Use associated key
     │
     ▼
Correct Username?
     │
     ├── No → Use AMI-specific username
     │
     ▼
Security Group Allows SSH?
     │
     ├── No → Correct inbound rule
     │
     ▼
SSH Connection
```

---

# 4. Docker Installation Problems

## Symptom

The `docker` command is unavailable.

Example:

```text
docker: command not found
```

### Check

```bash
docker --version
```

If Docker is not installed, install it according to the operating system used by the EC2 instance.

After installation, verify again:

```bash
docker --version
```

---

# 5. Docker Service Problems

## Symptom

Docker is installed but containers cannot be started.

### Check Docker

```bash
docker --version
```

Check running containers:

```bash
docker ps
```

Check all containers:

```bash
docker ps -a
```

If the Docker service is not operating correctly, investigate the Docker service configuration for the operating system used by the EC2 instance.

---

# 6. Docker Image Problems

## Symptom

The required application image is unavailable on the EC2 instance.

### Check Local Images

```bash
docker images
```

If the image is missing, pull it from the private registry:

```bash
docker pull <registry>/<repository>:<tag>
```

### Verify

```bash
docker images
```

Confirm that:

* Repository name is correct.
* Image tag is correct.
* Registry address is correct.
* Authentication has succeeded.

---

# 7. Private Registry Authentication Problems

## Symptom

The EC2 instance cannot pull the application image from the private registry.

### Possible Causes

* Registry authentication has not been completed.
* Incorrect registry credentials.
* Incorrect repository name.
* Incorrect image tag.
* Image was never pushed successfully.
* Network connectivity to the registry is unavailable.

### Check the Image Source

Verify the image reference:

```text
<registry>/<repository>:<tag>
```

### Verify the Registry

Confirm that the image exists in the private registry.

### Authenticate Again

Use the authentication mechanism required by the registry and retry:

```bash
docker pull <registry>/<repository>:<tag>
```

The project workflow requires the application image to be pushed to a private Docker repository before it is deployed to EC2.

---

# 8. Docker Container Fails to Start

## Symptom

The image exists but the application container stops immediately.

### Check Running Containers

```bash
docker ps
```

### Check Stopped Containers

```bash
docker ps -a
```

### Inspect Logs

```bash
docker logs <container-id>
```

The logs should be reviewed before changing the deployment configuration.

### Common Areas to Investigate

* Application startup failure
* Incorrect environment configuration
* Incorrect container command
* Missing application dependencies
* Incorrect application port
* Invalid image configuration

---

# 9. Application Container Is Running but Application Is Unavailable

## Symptom

`docker ps` shows the container as running, but the application cannot be reached through a browser.

This usually requires checking the deployment path layer by layer.

```text
Browser
   ↓
EC2 Public IP
   ↓
Security Group
   ↓
EC2 Host Port
   ↓
Docker Port Mapping
   ↓
Container Port
   ↓
Application
```

---

# 10. Docker Port Mapping Problems

## Symptom

The application is running inside the container but cannot be reached externally.

### Check

```bash
docker ps
```

Review the `PORTS` column.

The expected mapping should resemble:

```text
<host-port>:<container-port>
```

For example:

```text
0.0.0.0:<host-port>-><container-port>/tcp
```

If the wrong ports were mapped, stop the container and recreate it using the correct mapping.

Example:

```bash
docker run -d -p <host-port>:<container-port> <image>
```

---

# 11. Security Group Problems

## Symptom

The application works locally on the EC2 instance but cannot be accessed from a browser.

### Possible Cause

The EC2 Security Group does not allow inbound traffic to the application port.

The project checklist specifically requires configuring the Security Group firewall to allow browser access to the deployed web application.

### Check

Review the inbound rules attached to the EC2 instance.

Verify that the required application port is allowed.

Also confirm that the rule's source is appropriate for the intended access pattern.

Avoid opening unnecessary ports.

---

# 12. Application Works Inside EC2 but Not from Browser

## Symptom

The application responds from the EC2 host but not from an external browser.

### Troubleshooting Sequence

First verify that the container is running:

```bash
docker ps
```

Then verify the port mapping:

```bash
docker ps
```

Then verify that the Security Group permits the required port.

Finally, confirm that the correct EC2 public IP address is being used.

The troubleshooting path is:

```text
Container Running?
       │
       ▼
Correct Port Mapping?
       │
       ▼
Security Group Rule?
       │
       ▼
Correct Public IP?
       │
       ▼
Browser Access
```

---

# 13. EC2 Public IP Address Changed

## Symptom

The application was previously accessible but the old address no longer works.

### Cause

The public IPv4 address associated with an EC2 instance can change when the instance is stopped and started, depending on the configuration.

### Action

Check the current public IPv4 address in the EC2 console and use the current address when accessing the application.

Example:

```text
http://<current-ec2-public-ip>:<application-port>
```

---

# 14. Application Logs Show Errors

## Check Logs

```bash
docker logs <container-id>
```

For continuously updating logs:

```bash
docker logs -f <container-id>
```

Review the logs for:

* Application startup errors
* Missing configuration
* Dependency errors
* Port binding problems
* Runtime exceptions

Do not immediately rebuild the entire deployment. First identify the error reported by the application.

---

# 15. Container Stops Unexpectedly

## Symptom

The application container starts but later stops.

### Check

```bash
docker ps -a
```

Then inspect the logs:

```bash
docker logs <container-id>
```

Determine why the container exited before restarting it.

The exit behavior should be understood rather than repeatedly restarting the container without investigating the underlying problem.

---

# 16. Image Was Pushed but EC2 Cannot Find It

## Possible Causes

### Incorrect Repository

The image may have been pushed to a different repository.

### Incorrect Tag

The EC2 host may be attempting to pull a tag that does not exist.

### Incorrect Registry

The image reference may point to the wrong registry.

### Image Was Not Successfully Pushed

Verify the registry directly.

The expected image reference should follow:

```text
<registry>/<repository>:<tag>
```

---

# 17. Deployment Verification Commands

The following commands are useful during investigation.

### Check Docker Version

```bash
docker --version
```

### List Images

```bash
docker images
```

### List Running Containers

```bash
docker ps
```

### List All Containers

```bash
docker ps -a
```

### View Container Logs

```bash
docker logs <container-id>
```

### Follow Container Logs

```bash
docker logs -f <container-id>
```

### Inspect a Container

```bash
docker inspect <container-id>
```

---

# 18. Layered Troubleshooting Model

Use the following model when diagnosing deployment problems:

```text
┌───────────────────────────┐
│       Application         │
└─────────────┬─────────────┘
              │
┌─────────────▼─────────────┐
│         Container         │
└─────────────┬─────────────┘
              │
┌─────────────▼─────────────┐
│           Docker          │
└─────────────┬─────────────┘
              │
┌─────────────▼─────────────┐
│       EC2 Instance        │
└─────────────┬─────────────┘
              │
┌─────────────▼─────────────┐
│      Security Group       │
└─────────────┬─────────────┘
              │
┌─────────────▼─────────────┐
│        AWS Network        │
└───────────────────────────┘
```

Start at the layer where the failure is first observed and work downward or upward as required.

---

# 19. Diagnostic Checklist

## Infrastructure

* [ ] EC2 instance exists
* [ ] EC2 instance is running
* [ ] Correct public IP identified
* [ ] Correct Security Group attached

## SSH

* [ ] Correct private key
* [ ] Correct key permissions
* [ ] Correct SSH username
* [ ] SSH inbound rule available
* [ ] SSH connection successful

## Docker

* [ ] Docker installed
* [ ] Docker command available
* [ ] Docker service operational

## Image

* [ ] Docker image built
* [ ] Image tagged correctly
* [ ] Image pushed successfully
* [ ] Image exists in private registry
* [ ] EC2 authenticated to registry
* [ ] Image successfully pulled

## Container

* [ ] Container created
* [ ] Container running
* [ ] Correct port mapping
* [ ] Application logs reviewed

## Network

* [ ] Required Security Group rule exists
* [ ] Application port is correct
* [ ] Correct public IP is being used

## Application

* [ ] Application starts successfully
* [ ] Application listens on expected port
* [ ] Browser can reach application

---

# 20. Troubleshooting Principles

### Change One Variable at a Time

Avoid modifying EC2, Docker, Security Groups, and application configuration simultaneously.

Make one change, test it, and observe the result.

### Verify Before Assuming

Use commands such as:

```bash
docker ps
docker ps -a
docker images
docker logs <container-id>
```

to establish the current state before making changes.

### Troubleshoot from the Bottom Up

A useful sequence is:

```text
EC2
 ↓
SSH
 ↓
Docker
 ↓
Image
 ↓
Container
 ↓
Port Mapping
 ↓
Security Group
 ↓
Application
```

### Preserve Evidence

When troubleshooting, capture relevant:

* Error messages
* Docker output
* Application logs
* EC2 configuration
* Security Group configuration
* Screenshots

This creates a useful incident record and makes future troubleshooting faster.

---

# 21. Final Troubleshooting Flow

When the deployed application is unavailable, follow this sequence:

```text
Application unavailable
          │
          ▼
Is EC2 running?
     │           │
    No          Yes
     │           │
  Fix EC2        ▼
             SSH works?
              │      │
             No     Yes
              │      │
           Fix SSH   ▼
                  Docker works?
                   │       │
                  No      Yes
                   │       │
               Fix Docker  ▼
                       Container running?
                        │        │
                       No       Yes
                        │        │
                    Check logs   ▼
                            Correct port mapping?
                              │       │
                             No      Yes
                              │       │
                         Reconfigure  ▼
                              Security Group?
                               │       │
                              No      Yes
                               │       │
                           Fix rule    ▼
                                Browser access
```

This systematic approach reduces guesswork and helps isolate failures to the infrastructure, access, container, networking, or application layer.
