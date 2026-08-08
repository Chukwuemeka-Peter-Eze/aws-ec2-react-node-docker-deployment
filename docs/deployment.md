# AWS EC2 Web Application Deployment Runbook

## 1. Purpose

This document describes the deployment procedure for running a containerized web application on an Amazon EC2 instance.

The deployment is performed manually using:

* Amazon EC2
* SSH
* Docker
* A private Docker registry
* EC2 Security Groups

The goal is to build a Docker image, make it available through a private registry, deploy it to an EC2 instance, and verify that the application can be accessed through a browser.

---

## 2. Deployment Flow

The deployment follows this sequence:

```text
Application Source
       │
       ▼
Build Docker Image
       │
       ▼
Push Image to Private Registry
       │
       ▼
Create EC2 Instance
       │
       ▼
Connect through SSH
       │
       ▼
Install Docker
       │
       ▼
Pull Docker Image
       │
       ▼
Run Application Container
       │
       ▼
Configure Security Group
       │
       ▼
Access Application
       │
       ▼
Browser Verification
```

---

# 3. Prerequisites

Before beginning the deployment, ensure the following are available:

* An active AWS account
* Access to the AWS Management Console
* An EC2 key pair
* A web application
* A Dockerfile for the application
* Docker installed on the development machine
* Access to a private Docker registry
* SSH client
* The required AWS permissions

The deployment should be performed using credentials with only the permissions required for the work.

---

# 4. Create the EC2 Instance

Navigate to the Amazon EC2 service in the AWS Management Console.

Create a new EC2 instance with the required configuration.

The main components include:

* Amazon Machine Image (AMI)
* Instance type
* Key pair
* Security Group
* Network configuration

The selected instance must provide sufficient resources for the application being deployed.

---

# 5. Configure the Key Pair

An EC2 key pair is required for SSH access to the instance.

After obtaining the private key, store it in a secure location.

For Linux-based environments, ensure that the private key has appropriate permissions:

```bash
chmod 400 <private-key>.pem
```

The private key should never be committed to Git or uploaded to a public repository.

Example `.gitignore` entry:

```gitignore
*.pem
```

---

# 6. Identify the EC2 Public Address

After the EC2 instance has entered the running state, identify its public IP address or public DNS address.

This address will be used to establish the SSH connection and later to access the deployed application.

---

# 7. Connect to the EC2 Instance

Use SSH to connect to the instance.

Example:

```bash
ssh -i <private-key>.pem <username>@<ec2-public-ip>
```

Replace the placeholders with the values corresponding to the EC2 instance.

Once connected, verify that you are operating on the intended EC2 host.

---

# 8. Install Docker

Docker must be available on the EC2 instance before the application image can be executed.

Install Docker according to the operating system of the selected EC2 AMI.

After installation, verify that Docker is available:

```bash
docker --version
```

If Docker is installed correctly, the command should return the installed Docker version.

---

# 9. Prepare the Application Image

Before deploying to EC2, build the application's Docker image.

From the application project directory:

```bash
docker build -t <image-name> .
```

Verify that the image exists locally:

```bash
docker images
```

The image should appear in the local Docker image list.

---

# 10. Tag the Docker Image

The image must be tagged using the appropriate private registry repository and image tag.

Example:

```bash
docker tag <image-name> <registry>/<repository>:<tag>
```

Example structure:

```text
<registry>/<repository>:<tag>
```

The exact registry URL, repository name, and tag should match the registry being used for the project.

---

# 11. Authenticate with the Private Registry

Authenticate with the private Docker registry before pushing the image.

The exact authentication command depends on the registry being used.

Once authenticated, verify that the Docker client can communicate with the registry.

---

# 12. Push the Docker Image

Push the tagged image to the private registry:

```bash
docker push <registry>/<repository>:<tag>
```

The AWS project checklist identifies building and pushing the Docker image to a private Docker repository as part of the EC2 deployment workflow.

After the push completes, verify that the image is available in the registry.

---

# 13. Authenticate from EC2

Connect to the EC2 instance through SSH if you are not already connected.

The EC2 host must be authenticated against the private Docker registry before it can pull the private image.

The authentication mechanism depends on the registry being used.

---

# 14. Pull the Application Image

Once authenticated, retrieve the application image:

```bash
docker pull <registry>/<repository>:<tag>
```

Verify that the image was downloaded:

```bash
docker images
```

The expected repository and tag should appear in the image list.

---

# 15. Run the Application Container

Start the application using Docker.

Example:

```bash
docker run -d -p <host-port>:<container-port> <registry>/<repository>:<tag>
```

The `-d` option runs the container in detached mode.

The `-p` option maps the EC2 host port to the application port inside the container.

For example:

```text
EC2 Host Port
      │
      ▼
Container Port
      │
      ▼
Web Application
```

Use the actual port configuration required by the application.

---

# 16. Verify the Running Container

Check the running containers:

```bash
docker ps
```

The application container should appear in the output.

For additional information:

```bash
docker ps -a
```

This also displays stopped containers.

---

# 17. Inspect Application Logs

If the application does not behave as expected, inspect the container logs:

```bash
docker logs <container-id>
```

Logs can help identify application startup failures, configuration problems, or other runtime issues.

---

# 18. Configure the EC2 Security Group

The EC2 Security Group controls inbound traffic reaching the instance.

Configure an inbound rule for the port required by the web application.

The project checklist explicitly requires the Security Group firewall to be configured so that the web application can be accessed from a browser.

The required rule should correspond to the actual port exposed by the Docker container.

Avoid opening unnecessary ports.

---

# 19. Verify Network Configuration

Before testing the application, verify:

* The EC2 instance is running.
* The Security Group is attached to the instance.
* The required application port is allowed.
* The Docker container is running.
* Docker is mapping the expected host and container ports.

The deployment depends on all of these layers working together.

---

# 20. Access the Application

Open a browser and navigate to:

```text
http://<ec2-public-ip>:<application-port>
```

The application should load successfully.

A successful response confirms that the request can travel through the deployment path:

```text
Browser
   │
   ▼
EC2 Public Address
   │
   ▼
Security Group
   │
   ▼
EC2 Host Port
   │
   ▼
Docker Container
   │
   ▼
Application
```

---

# 21. Deployment Verification Checklist

Use the following checklist after deployment.

### EC2

* [ ] EC2 instance created
* [ ] Instance is running
* [ ] Correct public address identified
* [ ] Correct key pair configured

### SSH

* [ ] Private key stored securely
* [ ] Private key permissions configured
* [ ] SSH connection successful

### Docker

* [ ] Docker installed
* [ ] Docker version verified
* [ ] Application image available
* [ ] Container started successfully

### Registry

* [ ] Image tagged correctly
* [ ] Image pushed to private registry
* [ ] EC2 authenticated to registry
* [ ] Image successfully pulled

### Networking

* [ ] Security Group attached
* [ ] Required application port allowed
* [ ] No unnecessary inbound ports exposed

### Application

* [ ] Container appears in `docker ps`
* [ ] Application is listening on expected port
* [ ] Application logs checked
* [ ] Application accessible through browser

---

# 22. Troubleshooting Procedure

When the application is unavailable, troubleshoot from the infrastructure layer upward.

## Step 1 — Check EC2

Confirm that the instance is running.

If the instance is stopped or unavailable, resolve the EC2 issue before continuing.

---

## Step 2 — Check SSH

Confirm that the EC2 instance can be reached through SSH.

If SSH fails, verify:

* Public IP address
* Key pair
* Private key permissions
* SSH username
* Security Group rules

---

## Step 3 — Check Docker

Verify that Docker is available:

```bash
docker --version
```

Then check running containers:

```bash
docker ps
```

---

## Step 4 — Check the Container

If the container is not running:

```bash
docker ps -a
```

Then inspect the logs:

```bash
docker logs <container-id>
```

---

## Step 5 — Check the Image

Verify that the required image exists:

```bash
docker images
```

If it is missing, pull it again:

```bash
docker pull <registry>/<repository>:<tag>
```

---

## Step 6 — Check Port Mapping

Inspect the running container:

```bash
docker ps
```

Verify that the host port maps to the correct container port.

---

## Step 7 — Check the Security Group

Verify that the required application port is permitted by the EC2 Security Group.

If the container is running but the application cannot be reached externally, network configuration should be investigated.

---

## Step 8 — Test Again

After correcting the issue, access:

```text
http://<ec2-public-ip>:<application-port>
```

Confirm that the application responds successfully.

---

# 23. Security Practices

This deployment should follow basic cloud security practices.

### Never Commit Private Keys

Do not commit EC2 `.pem` files to Git.

### Protect Credentials

Do not store AWS credentials or registry passwords directly in source code.

### Minimize Network Exposure

Only expose the ports required by the application.

### Use Least Privilege

AWS permissions should provide only the access required for the task.

### Protect the Registry

Private application images should require authentication before they can be retrieved.

---

# 24. Cleanup

After completing testing, resources created specifically for the deployment should be reviewed and removed when they are no longer required.

This helps prevent unnecessary resource consumption and unexpected AWS charges.

Review:

* EC2 instances
* Security Groups
* Key pairs
* Storage resources
* Registry images

Before deleting anything, confirm that the resource is not being used by another project or deployment.

---

# 25. Deployment Result

A successful deployment results in:

```text
                    AWS
                     │
                     ▼
              ┌─────────────┐
              │    EC2      │
              │             │
              │   Docker    │
              │      │      │
              │      ▼      │
              │ Application │
              │  Container  │
              └──────┬──────┘
                     │
              Security Group
                     │
                     ▼
                Web Browser
```

The application is packaged as a Docker image, distributed through a private registry, executed on EC2, and exposed through the configured network path.

---

## 26. Final Outcome

This deployment establishes the foundation for automated application delivery on AWS.

The manual workflow covered:

1. Provisioning EC2 infrastructure.
2. Establishing secure SSH access.
3. Building the application Docker image.
4. Publishing the image to a private registry.
5. Installing Docker on EC2.
6. Pulling the application image.
7. Running the application container.
8. Configuring Security Group access.
9. Verifying the application through a browser.

The next stage of the AWS Services work builds on this manual deployment process by introducing Jenkins-based CI/CD automation.
