# AWS EC2 Web Deployment — Lessons Learned

## 1. Overview

This project provided practical experience deploying a containerized web application to Amazon EC2.

The deployment required several components to work together:

* Amazon EC2
* SSH
* Docker
* Private Docker registry
* EC2 Security Groups
* Web application

The most important lesson from the project is that application deployment is not a single operation.

A successful deployment depends on infrastructure, access, containerization, image distribution, networking, and application configuration working together.

---

# 2. EC2 Provides the Compute Layer

Amazon EC2 provides the virtual compute environment where the application runs.

The application itself does not run "on AWS" automatically after an EC2 instance is created.

The EC2 host still needs to be:

1. Provisioned.
2. Accessed.
3. Configured.
4. Prepared with Docker.
5. Supplied with the application image.
6. Configured for network access.
7. Verified.

This helped establish a clearer understanding of the relationship between cloud infrastructure and application deployment.

---

# 3. Containerization Separates Application and Host Configuration

Docker allows the application and its runtime dependencies to be packaged into an image.

Instead of manually installing every application dependency directly on the EC2 host, the host provides the container runtime while the container provides the application environment.

The resulting model is:

```text
EC2
 │
 └── Docker
      │
      └── Application Container
```

This creates a cleaner separation between the infrastructure layer and application runtime.

---

# 4. The Container Image Becomes the Deployment Artifact

One of the important concepts demonstrated by the project is the role of the Docker image as a deployable artifact.

The application moves through the following lifecycle:

```text
Source Code
    │
    ▼
Docker Build
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

This is an important DevOps pattern because the same packaged artifact can be distributed from the registry to the deployment environment.

---

# 5. Image Storage and Application Hosting Are Different Concerns

The private Docker registry and EC2 serve different purposes.

The registry stores and distributes application images.

EC2 provides the compute environment where the application actually runs.

```text
Private Registry
       │
       │ Image
       ▼
      EC2
       │
       ▼
   Container
```

Understanding this separation makes later CI/CD and container orchestration workflows easier to reason about.

---

# 6. SSH Is an Administrative Access Mechanism

SSH provides direct administrative access to the EC2 instance.

This means that the deployment process can be performed manually from the host.

The workflow becomes:

```text
Administrator
     │
     │ SSH
     ▼
EC2 Instance
     │
     ├── Install Docker
     ├── Pull Image
     └── Run Container
```

However, manual SSH-based deployment also introduces operational overhead.

As deployment frequency increases, manually connecting to servers and executing commands becomes less efficient and more difficult to standardize.

This creates a natural motivation for automation.

---

# 7. Security Groups Are Part of Application Availability

A running container does not automatically mean the application is accessible from the internet.

There are multiple layers involved:

```text
Application
     ↓
Container
     ↓
Docker Port Mapping
     ↓
EC2
     ↓
Security Group
     ↓
Internet
     ↓
Browser
```

If any layer is incorrectly configured, the application may appear to be running while remaining inaccessible externally.

The AWS checklist explicitly includes Security Group configuration as part of the EC2 deployment process.

This reinforces the importance of understanding networking rather than treating application availability as purely an application concern.

---

# 8. Port Mapping Has Two Sides

When a container is started with a port mapping such as:

```bash
docker run -d -p <host-port>:<container-port> <image>
```

there are two relevant ports:

```text
EC2 Host Port
      │
      ▼
Container Port
      │
      ▼
Application
```

The application must be listening on the expected container port, while the EC2 host must expose the corresponding host port.

The Security Group must then permit the required traffic to the host port.

This creates three configuration points that must agree:

1. Application port
2. Docker port mapping
3. Security Group inbound rule

---

# 9. Troubleshooting Should Be Layered

One of the most useful operational lessons is to avoid changing multiple components simultaneously.

A better approach is to troubleshoot layer by layer:

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

For example, if the browser cannot reach the application, the first assumption should not automatically be that the application is broken.

The problem could exist at the EC2, Docker, networking, or Security Group layer.

---

# 10. Logs Are Essential for Container Troubleshooting

When a container fails, `docker ps` alone may not explain why.

The container logs provide additional information:

```bash
docker logs <container-id>
```

This makes logs one of the first places to investigate when an application container starts incorrectly or exits unexpectedly.

The general troubleshooting pattern is:

```text
Observe
   ↓
Collect Evidence
   ↓
Identify Failure Layer
   ↓
Make One Change
   ↓
Test
   ↓
Verify
```

---

# 11. Credentials and Private Keys Require Protection

The deployment also reinforces the importance of protecting credentials.

Examples include:

* EC2 private keys
* AWS credentials
* Registry credentials
* Passwords
* Tokens
* Application secrets

These should not be committed to Git.

A repository should contain the configuration required to understand and reproduce the deployment without exposing the credentials used to perform it.

---

# 12. Manual Deployment Reveals Opportunities for Automation

The manual deployment process makes every step visible:

```text
Build Image
     ↓
Push Image
     ↓
SSH to EC2
     ↓
Authenticate
     ↓
Pull Image
     ↓
Run Container
     ↓
Verify Application
```

While this is useful for learning and understanding the mechanics, manually repeating these steps for every application change introduces unnecessary operational work.

This is where CI/CD automation becomes valuable.

---

# 13. CI/CD Builds on the Manual Workflow

The next stage of the AWS Services work introduces Jenkins.

The manual process can be transformed into an automated pipeline:

```text
Developer
    │
    ▼
Source Code
    │
    ▼
Jenkins
    │
    ├── Build
    ├── Test
    ├── Build Docker Image
    ├── Push Image
    └── Deploy
           │
           ▼
        AWS EC2
           │
           ▼
      Application
```

The manual EC2 project therefore provides the foundation for understanding what Jenkins will automate.

---

# 14. Infrastructure and Application Deployment Are Connected

The project demonstrates that application deployment involves more than the application itself.

The deployment depends on:

```text
Infrastructure
      +
Networking
      +
Security
      +
Container Runtime
      +
Application
```

A DevOps engineer needs to understand how these layers interact.

An application can be correctly built and containerized but still fail because the infrastructure or networking configuration is incorrect.

---

# 15. Verification Is Part of Deployment

Deployment should not end when the Docker command succeeds.

The application should be verified from the perspective of the end user.

The verification path is:

```text
Docker Container
      ↓
Application
      ↓
EC2 Host Port
      ↓
Security Group
      ↓
Public Network
      ↓
Browser
```

Successful browser access provides end-to-end evidence that the deployment is functioning.

---

# 16. Reproducibility Matters

A good deployment process should be repeatable.

Someone reviewing the project should be able to understand:

* What infrastructure was required.
* How the application was packaged.
* Where the image was stored.
* How the image reached EC2.
* How the container was started.
* Which network configuration was required.
* How the application was verified.

Documenting these steps transforms the project from a one-time experiment into a reproducible engineering workflow.

---

# 17. Key Engineering Takeaways

### Compute

EC2 provides the infrastructure required to host the application.

### Containerization

Docker packages the application into a portable deployment artifact.

### Registry

A private registry provides centralized image storage and distribution.

### Access

SSH provides administrative access to the EC2 host.

### Networking

Security Groups control inbound traffic to the instance.

### Operations

Docker commands and container logs provide visibility into the running application.

### Security

Private keys and credentials must remain outside source control.

### Automation

Manual deployment exposes the steps that can later be automated through CI/CD.

---

# 18. Project-to-Next-Stage Connection

This project establishes the manual deployment foundation for the next AWS repository:

```text
AWS EC2 Web Deployment
          │
          │ Manual Process
          ▼
Understand Deployment Steps
          │
          ▼
Identify Repetitive Tasks
          │
          ▼
Automate with Jenkins
          │
          ▼
AWS Jenkins CI/CD Pipeline
```

The Jenkins project can therefore be understood as an evolution of this deployment rather than an unrelated project.

The infrastructure and application deployment concepts remain the same; the primary change is that Jenkins takes responsibility for automating the workflow.

---

# 19. Final Reflection

The most valuable outcome of this project is not simply getting a web application to load from an EC2 public address.

The project demonstrates the complete chain required to move a containerized application from a development environment into a cloud-hosted runtime:

```text
Application
     ↓
Docker Image
     ↓
Private Registry
     ↓
Amazon EC2
     ↓
Docker Container
     ↓
Security Group
     ↓
Internet
     ↓
User
```

Understanding this chain provides the foundation for more advanced deployment patterns involving CI/CD, infrastructure as code, container registries, and container orchestration.

The manual deployment experience also provides a concrete baseline against which automated deployment can be evaluated.

**Project Status:** Completed
