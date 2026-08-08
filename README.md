# AWS EC2 Web Application Deployment

Deploy and host a containerized web application on an Amazon EC2 instance using Docker, SSH, and AWS Security Groups.

## Overview

This project demonstrates the deployment of a web application to **Amazon Elastic Compute Cloud (EC2)**.

The application is packaged as a Docker image and deployed manually to an EC2 instance. The EC2 instance provides the compute environment, while Docker is used to run the application container.

The deployment also demonstrates basic AWS networking and security configuration through an EC2 Security Group, allowing the deployed application to be accessed from a web browser.

This project focuses on understanding the fundamental workflow involved in deploying a containerized application to AWS EC2 before introducing automated CI/CD deployment.

---

## Architecture

The deployment follows this general flow:

```text
Developer
   │
   │ Build Docker Image
   ▼
Docker Image
   │
   │ Push
   ▼
Private Docker Registry
   │
   │ Pull Image
   ▼
Amazon EC2 Instance
   │
   ├── Docker
   │     │
   │     └── Application Container
   │
   └── Security Group
            │
            │ Allow Application Traffic
            ▼
        Web Browser
```

### Components

| Component               | Purpose                                                    |
| ----------------------- | ---------------------------------------------------------- |
| Amazon EC2              | Provides the compute environment for the application       |
| Docker                  | Packages and runs the application                          |
| Docker Image            | Contains the application and its runtime dependencies      |
| Private Docker Registry | Stores the Docker image before deployment                  |
| EC2 Security Group      | Controls inbound network traffic to the instance           |
| SSH                     | Provides remote administrative access to the EC2 instance  |
| Web Browser             | Used to verify that the deployed application is accessible |

---

## Project Objectives

The main objectives of this project were to:

* Create and configure an Amazon EC2 instance.
* Securely manage the private SSH key required for instance access.
* Build and push a Docker image to a private Docker registry.
* Connect to the EC2 instance using SSH.
* Install Docker on the EC2 instance.
* Retrieve and run the application Docker image.
* Configure the EC2 Security Group to allow access to the web application.
* Verify the deployed application through a web browser.
* Understand the basic workflow for manually deploying a containerized application to AWS.

---

## Technologies and Services

### AWS

* Amazon EC2
* EC2 Security Groups

### Containerization

* Docker

### Remote Access

* SSH
* SSH private key

### Container Registry

* Private Docker registry

---

## Deployment Workflow

The deployment consists of several stages.

### 1. Create the EC2 Instance

An EC2 instance is created to provide the compute environment where the application container will run.

The instance configuration includes the required:

* AMI
* Instance type
* Key pair
* Security Group
* Network configuration

The EC2 instance becomes the host for the Dockerized web application.

---

### 2. Configure SSH Access

The private key associated with the EC2 key pair is stored securely and used to establish an SSH connection to the instance.

Example:

```bash
chmod 400 <private-key>.pem
```

The permissions ensure that the private key is not unnecessarily accessible to other users on the system.

The instance can then be accessed through SSH using its public address.

---

### 3. Build the Docker Image

The application is packaged into a Docker image.

The image contains the application and the dependencies required to run it consistently in the target environment.

Example:

```bash
docker build -t <image-name> .
```

---

### 4. Push the Image to a Private Registry

After building the image, it is pushed to a private Docker registry.

This provides a central location from which the EC2 instance can retrieve the image during deployment.

The image is tagged with the appropriate registry and repository information before being pushed.

Example:

```bash
docker tag <image-name> <registry>/<repository>:<tag>

docker push <registry>/<repository>:<tag>
```

---

### 5. Connect to the EC2 Instance

SSH is used to connect to the newly created EC2 instance.

Example:

```bash
ssh -i <private-key>.pem <user>@<public-ip>
```

Once connected, the EC2 instance can be configured for application deployment.

---

### 6. Install Docker

Docker is installed on the EC2 instance so that the application image can be executed as a container.

After installation, Docker is used to retrieve the application image from the private registry.

---

### 7. Run the Application Container

The application image is pulled from the private registry and started on the EC2 instance.

Example:

```bash
docker pull <registry>/<repository>:<tag>

docker run -d -p <host-port>:<container-port> <image-name>
```

The port mapping exposes the application running inside the container through the EC2 instance.

---

### 8. Configure the Security Group

The EC2 Security Group acts as the instance-level firewall.

An inbound rule is configured to allow traffic required to access the deployed web application from a browser.

The checklist specifically identifies Security Group configuration as part of the deployment process.

Security rules should be configured deliberately rather than opening unnecessary ports or sources.

---

### 9. Verify the Deployment

After the container is running and the appropriate Security Group rule has been configured, the application can be accessed through the EC2 instance's public address and application port.

Example:

```text
http://<ec2-public-ip>:<application-port>
```

Successful browser access confirms that:

1. The EC2 instance is running.
2. The container is running.
3. The application is listening on the expected port.
4. The required network traffic is permitted by the Security Group.

---

## Verification

The deployment should be verified at multiple levels.

### EC2

Confirm that the instance is running.

```bash
aws ec2 describe-instances
```

### Docker

Confirm that the application container is running:

```bash
docker ps
```

Inspect the container when troubleshooting:

```bash
docker logs <container-id>
```

### Network Access

Confirm that the required Security Group rule is present.

### Application

Open the EC2 public address and configured application port in a browser.

```text
http://<ec2-public-ip>:<application-port>
```

---

## Security Considerations

This project demonstrates several basic security practices that should be considered when working with EC2.

### Protect Private Keys

The EC2 private key should be stored securely and should not be committed to Git.

Never add `.pem` files or other private credentials to the repository.

A `.gitignore` file should include sensitive files such as:

```gitignore
*.pem
.env
```

### Security Groups

Security Groups should expose only the traffic required by the application.

Avoid unnecessarily opening administrative ports to the entire internet.

### Credentials

AWS credentials, registry credentials, private keys, and other secrets should never be hard-coded into application source code or committed to version control.

---

## Troubleshooting

### Application Cannot Be Accessed

If the application cannot be reached through the browser, check:

1. The EC2 instance is running.
2. The Docker container is running.
3. The application is listening on the expected container port.
4. Docker port mapping is configured correctly.
5. The EC2 Security Group allows the required inbound traffic.
6. The correct public IP address is being used.

Useful command:

```bash
docker ps
```

---

### Container Is Not Running

Check the running containers:

```bash
docker ps
```

Check all containers:

```bash
docker ps -a
```

Review the container logs:

```bash
docker logs <container-id>
```

---

### Cannot Connect Through SSH

Verify:

* The EC2 instance is running.
* The correct public IP address is being used.
* The correct private key is being used.
* The private key has appropriate permissions.
* The Security Group allows SSH traffic from the required source.

---

### Docker Image Cannot Be Pulled

Check:

* The registry repository exists.
* The image name is correct.
* The image tag is correct.
* Authentication to the private registry has been completed.
* The EC2 instance has network connectivity to the registry.

---

## Repository Structure

A typical repository structure for this project can be organized as follows:

```text
Aws-ec2-web-deployment/
│
├── README.md
├── .gitignore
│
├── app/
│   └── ...
│
├── Dockerfile
│
├── docs/
│   ├── architecture.md
│   ├── deployment.md
│   └── troubleshooting.md
│
└── screenshots/
    └── ...
```

The exact structure should reflect the files and evidence actually included in the repository.

---

## What This Project Demonstrates

This project demonstrates practical understanding of the relationship between application packaging, compute infrastructure, networking, and deployment.

The deployment flow can be summarized as:

```text
Application
    │
    ▼
Docker Image
    │
    ▼
Private Registry
    │
    ▼
Amazon EC2
    │
    ├── Docker
    │
    └── Application Container
            │
            ▼
      Security Group
            │
            ▼
       Web Browser
```

Rather than treating EC2 as an isolated AWS service, the project demonstrates how EC2 works as part of a broader application deployment workflow.

---

## Key Takeaways

* EC2 provides the compute infrastructure required to host the application.
* Docker provides a consistent packaging and runtime environment.
* A container registry provides a location for storing and retrieving application images.
* SSH provides administrative access to the EC2 instance.
* Security Groups control network access to the instance.
* Application availability depends on both the container configuration and the underlying network configuration.
* Manual deployment provides the foundation for understanding the automated CI/CD deployment introduced in the subsequent AWS Jenkins project.

---

## Related AWS Projects

This repository is part of a broader AWS-focused engineering portfolio:

1. **AWS EC2 Web Deployment**
   Manual deployment of a containerized web application to Amazon EC2.

2. **AWS Jenkins CI/CD Pipeline**
   Automating application build and deployment using Jenkins and AWS.

3. **AWS ECR Docker Registry**
   Working with Amazon Elastic Container Registry for private Docker image storage.

4. **AWS CLI Automation**
   Automating AWS infrastructure and service operations using the AWS CLI.

---

## Project Status

**Status:** Completed

The project covers the core workflow of manually deploying a Dockerized web application to an Amazon EC2 instance, including EC2 provisioning, SSH access, Docker configuration, image deployment, Security Group configuration, and browser-based application verification.
