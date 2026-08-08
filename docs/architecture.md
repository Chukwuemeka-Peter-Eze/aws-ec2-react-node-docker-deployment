# AWS EC2 Web Deployment Architecture

## 1. Overview

This document describes the architecture used to deploy a containerized web application on an Amazon EC2 instance.

The deployment follows a straightforward application delivery path:

```text
                    Developer
                        │
                        │ Build
                        ▼
                Docker Image
                        │
                        │ Push
                        ▼
              Private Docker Registry
                        │
                        │ Pull
                        ▼
              ┌─────────────────────┐
              │    Amazon EC2       │
              │                     │
              │      Docker         │
              │        │            │
              │        ▼            │
              │ Application         │
              │ Container           │
              └─────────┬───────────┘
                        │
                        │ Application Traffic
                        ▼
                EC2 Security Group
                        │
                        ▼
                  Web Browser
```

The architecture demonstrates how an application packaged as a Docker image can be deployed to AWS compute infrastructure and exposed for browser-based access.

---

## 2. Architecture Components

### Amazon EC2

Amazon Elastic Compute Cloud (EC2) provides the virtual server used to host the application.

The EC2 instance is the primary compute resource in this deployment.

The instance provides the environment in which Docker is installed and the application container is executed.

---

### Docker

Docker provides the container runtime used to execute the application.

Instead of installing and configuring the application's runtime environment directly on the EC2 host, the application is packaged into a Docker image.

The image can then be pulled and executed on the EC2 instance.

This provides a consistent application runtime between the development environment and the deployment environment.

---

### Private Docker Registry

The Docker image is stored in a private Docker registry before being deployed to EC2.

The registry provides the image distribution point between the environment where the image is built and the EC2 instance where the application runs.

The deployment flow is therefore:

```text
Build
  │
  ▼
Docker Image
  │
  ▼
Private Registry
  │
  ▼
EC2
  │
  ▼
Running Container
```

The AWS checklist identifies pushing the Docker image to a private Docker registry as part of the EC2 deployment workflow.

---

## 3. SSH Access

SSH is used to establish administrative access to the EC2 instance.

The EC2 key pair provides the private key required to authenticate to the instance.

The private key should be stored securely and should not be committed to the Git repository.

The basic access flow is:

```text
Administrator
     │
     │ SSH + Private Key
     ▼
Amazon EC2 Instance
```

Before establishing the connection, the private key is assigned appropriate permissions.

Example:

```bash
chmod 400 <private-key>.pem
```

The exact SSH username depends on the operating system and AMI used by the EC2 instance.

---

## 4. Network Security

The EC2 Security Group controls inbound traffic to the instance.

For this deployment, the Security Group must allow the traffic required to access the web application.

The project checklist specifically identifies configuring the Security Group firewall so that the deployed web application can be accessed through a browser.

Conceptually:

```text
Internet
   │
   │ Application Traffic
   ▼
Security Group
   │
   │ Allowed Traffic
   ▼
EC2 Instance
   │
   ▼
Docker Container
   │
   ▼
Web Application
```

Security Groups should be configured according to the actual application requirements rather than opening unnecessary ports.

---

## 5. Application Deployment Flow

The complete deployment process consists of the following stages.

### Stage 1 — Application

The web application is prepared for containerization.

```text
Application Source Code
          │
          ▼
       Dockerfile
```

---

### Stage 2 — Image Build

Docker builds an image containing the application and its runtime dependencies.

```text
Dockerfile
    │
    ▼
Docker Build
    │
    ▼
Docker Image
```

---

### Stage 3 — Image Distribution

The resulting image is tagged and pushed to a private Docker registry.

```text
Docker Image
     │
     │ docker push
     ▼
Private Registry
```

---

### Stage 4 — EC2 Preparation

An EC2 instance is created and accessed through SSH.

Docker is installed on the instance so that the application image can be executed.

```text
EC2 Instance
     │
     ├── SSH Access
     │
     └── Docker
```

---

### Stage 5 — Image Deployment

The EC2 host retrieves the application image from the private registry.

```text
Private Registry
       │
       │ docker pull
       ▼
EC2 Instance
       │
       ▼
Docker Image
```

The image is then started as a container.

```text
Docker Image
      │
      ▼
Docker Container
      │
      ▼
Web Application
```

---

### Stage 6 — External Access

The EC2 Security Group permits the required application traffic.

The application can then be reached using the EC2 instance's public address and configured application port.

```text
Web Browser
     │
     │ HTTP
     ▼
EC2 Public Address
     │
     ▼
Security Group
     │
     ▼
Application Container
```

---

## 6. End-to-End Architecture

Combining all components produces the following architecture:

```text
                         ┌─────────────────┐
                         │    Developer    │
                         └────────┬────────┘
                                  │
                                  │ docker build
                                  ▼
                         ┌─────────────────┐
                         │   Docker Image  │
                         └────────┬────────┘
                                  │
                                  │ docker push
                                  ▼
                    ┌──────────────────────────┐
                    │ Private Docker Registry  │
                    └────────────┬─────────────┘
                                 │
                                 │ docker pull
                                 ▼
                    ┌──────────────────────────┐
                    │       Amazon EC2         │
                    │                          │
                    │  ┌────────────────────┐  │
                    │  │       Docker       │  │
                    │  │         │          │  │
                    │  │         ▼          │  │
                    │  │ Application        │  │
                    │  │ Container          │  │
                    │  └────────────────────┘  │
                    │                          │
                    └────────────┬─────────────┘
                                 │
                                 │
                         ┌───────▼────────┐
                         │ Security Group │
                         └───────┬────────┘
                                 │
                                 │ Allowed
                                 │ Application
                                 │ Traffic
                                 ▼
                         ┌─────────────────┐
                         │   Web Browser   │
                         └─────────────────┘
```

---

## 7. Security Boundaries

The architecture contains several important security boundaries.

### SSH Access

SSH provides administrative access to the EC2 host and should be restricted to trusted sources whenever possible.

### Security Group

The Security Group acts as the network access control layer for the EC2 instance.

Only required inbound traffic should be permitted.

### Private Registry

The application image is stored in a private registry rather than being exposed as a public image.

Authentication is required before the image can be retrieved.

### Private Credentials

The following types of information should never be committed to source control:

* EC2 private keys
* Docker registry credentials
* AWS access keys
* Passwords
* Application secrets
* Environment-specific credentials

---

## 8. Deployment Dependencies

The deployment depends on the following components being correctly configured:

| Dependency              | Requirement                                         |
| ----------------------- | --------------------------------------------------- |
| AWS Account             | Required to provision EC2 resources                 |
| EC2 Instance            | Provides the application host                       |
| EC2 Key Pair            | Provides SSH authentication                         |
| Security Group          | Controls inbound network access                     |
| Docker                  | Runs the application container                      |
| Private Docker Registry | Stores the application image                        |
| Application Image       | Contains the deployable application                 |
| Network Connectivity    | Required for image retrieval and application access |

---

## 9. Operational Verification

After deployment, each layer should be verified independently.

### Infrastructure

Confirm that the EC2 instance is running.

### Access

Confirm that SSH access to the instance works.

### Docker

Confirm that Docker is installed and operational.

```bash
docker --version
```

### Container

Confirm that the application container is running.

```bash
docker ps
```

### Application

Confirm that the application is listening on the expected port.

### Network

Confirm that the Security Group allows the required application traffic.

### End-to-End

Access the application from a browser using the EC2 public address and configured port.

---

## 10. Architecture Principles Demonstrated

This project demonstrates several foundational cloud and DevOps principles:

### Infrastructure as a Host

EC2 provides the underlying compute infrastructure required to run the application.

### Immutable Application Packaging

Docker packages the application and its dependencies into an image that can be distributed and executed consistently.

### Separation of Image Storage and Compute

The Docker registry stores application artifacts while EC2 provides the compute environment.

### Network-Level Access Control

Security Groups determine which inbound traffic can reach the EC2 instance.

### Secure Administrative Access

SSH key-based authentication provides controlled access to the EC2 host.

### Layered Troubleshooting

The architecture makes it possible to troubleshoot failures by layer:

```text
Infrastructure
     ↓
Network
     ↓
SSH Access
     ↓
Docker
     ↓
Container
     ↓
Application
```

This layered approach helps identify whether a failure originates from AWS infrastructure, network configuration, container runtime, or the application itself.

---

## 11. Architecture Summary

The architecture is intentionally simple and represents a foundational AWS deployment pattern:

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
     ▼
Docker Container
     │
     ▼
Security Group
     │
     ▼
Web Application
     │
     ▼
Browser
```

The project establishes the fundamental concepts required for deploying containerized applications to AWS EC2.

These concepts become the foundation for the next stage of the AWS Services work, where the manual deployment process is automated through Jenkins CI/CD.
