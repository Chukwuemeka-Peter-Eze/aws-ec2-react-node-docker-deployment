# AWS EC2 React + Node.js Docker Deployment

Deploy and host a containerized React/Node.js web application on an Amazon EC2 instance using Docker, SSH, and AWS Security Groups.

![AWS](https://img.shields.io/badge/AWS-EC2-orange)
![Docker](https://img.shields.io/badge/Docker-Container-blue)
![Node.js](https://img.shields.io/badge/Node.js-Express-green)
![React](https://img.shields.io/badge/React-Frontend-61DAFB)

## Overview

This project demonstrates the deployment of a **React + Node.js/Express web application** to **Amazon Elastic Compute Cloud (EC2)**.

The app is a single Express server (`api/server.js`) that serves a React frontend (`my-app/`) as static files and exposes a small JSON API under `/api`. The whole thing is packaged as one Docker image and deployed manually to an EC2 instance: the EC2 instance provides the compute environment, and Docker runs the application container.

The deployment also demonstrates basic AWS networking and security configuration through an EC2 Security Group, allowing the deployed application to be accessed from a web browser.

This project focuses on understanding the fundamental workflow involved in deploying a containerized application to AWS EC2 before introducing automated CI/CD deployment.

This is the containerized counterpart to [`aws-ec2-spring-boot-deployment`](https://github.com/Chukwuemeka-Peter-Eze/aws-ec2-spring-boot-deployment), which deploys a different (Java/Spring Boot) app manually via SCP.

---

## Architecture

```text
Developer
   │
   │ docker build
   ▼
Docker Image (pierrechukason/demo-app)
   │
   │ docker push
   ▼
Docker Hub
   │
   │ docker pull
   ▼
Amazon EC2 Instance
   │
   ├── Docker
   │     │
   │     └── Container: demo-app
   │           ├── Express API (api/server.js)
   │           └── React static build (my-app/build)
   │
   └── Security Group
            │
            │ Allow port 3080
            ▼
        Web Browser
```

### Components

| Component      | Purpose                                                    |
| --------------- | ----------------------------------------------------------- |
| Amazon EC2      | Provides the compute environment for the application        |
| Docker          | Packages and runs the application                            |
| Docker Image    | Contains the Express server + built React app                |
| Docker Hub      | Stores the Docker image (`pierrechukason/demo-app`) before deployment |
| EC2 Security Group | Controls inbound network traffic to the instance          |
| SSH             | Provides remote administrative access to the EC2 instance     |
| Web Browser     | Used to verify that the deployed application is accessible    |

---

## Technology Stack

| Layer            | Technology                          |
| ------------------ | ------------------------------------ |
| Cloud Provider     | AWS                                   |
| Compute            | Amazon EC2                            |
| Container Runtime  | Docker Engine                          |
| Image Registry     | Docker Hub (`pierrechukason/demo-app`) |
| Backend            | Node.js / Express                     |
| Frontend           | React (`create-react-app`)            |
| Build Tools        | webpack, gulp (frontend build only — not used inside the Docker image) |
| Base Image (build) | `node:20-alpine`                       |
| Base Image (runtime)| `node:20-alpine`                      |
| Application Port   | `3080`                                 |

---

## Project Objectives

* Create and configure an Amazon EC2 instance.
* Securely manage the private SSH key required for instance access.
* Build and push a Docker image to Docker Hub.
* Connect to the EC2 instance using SSH.
* Install Docker on the EC2 instance.
* Retrieve and run the application Docker image.
* Configure the EC2 Security Group to allow access to the web application.
* Verify the deployed application through a web browser.
* Understand the basic workflow for manually deploying a containerized application to AWS.

---

## Deployment Workflow

### 1. Create the EC2 Instance

An EC2 instance is created to provide the compute environment where the application container will run. Configuration includes the AMI, instance type, key pair, Security Group, and network settings.

### 2. Configure SSH Access

```bash
chmod 400 <private-key>.pem
```

This ensures the private key isn't unnecessarily accessible to other users on the system.

### 3. Build the Docker Image (locally)

```bash
docker build -t pierrechukason/demo-app:latest .
```

### 4. Push the Image to Docker Hub

```bash
docker login
docker push pierrechukason/demo-app:latest
```

### 5. Connect to the EC2 Instance

```bash
ssh -i <private-key>.pem ubuntu@<ec2-public-ip>
```

### 6. Install Docker on EC2

```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable --now docker
sudo usermod -aG docker ubuntu
```
> Log out and back in (or run `newgrp docker`) for the group change to take effect.

### 7. Run the Application Container

```bash
docker pull pierrechukason/demo-app:latest
docker run -d --name demo-app -p 3080:3080 --restart unless-stopped pierrechukason/demo-app:latest
```

Or with the `docker-compose.yml` in this repo:

```bash
docker compose up -d
```

### 8. Configure the Security Group

Add an inbound rule allowing TCP traffic on port `3080` (and SSH on `22`, restricted to your IP). Open only what the application actually needs.

### 9. Verify the Deployment

```text
http://<ec2-public-ip>:3080
```

Successful browser access confirms the EC2 instance is running, the container is running, the app is listening on port 3080, and the Security Group permits the traffic.

---

## Verification

**EC2**
```bash
aws ec2 describe-instances
```

**Docker**
```bash
docker ps
docker logs demo-app
```

**Application**
```text
http://<ec2-public-ip>:3080
```

---

## Security Considerations

**Protect private keys** — never commit `.pem` files or credentials to Git:
```gitignore
*.pem
.env
```

**Security Groups** — expose only the traffic the application needs; avoid opening administrative ports to the entire internet.

**Credentials** — AWS credentials, Docker Hub credentials, and other secrets should never be hard-coded into source or committed to version control.

---

## Troubleshooting

**Application cannot be accessed** — check, in order: EC2 instance is running → container is running (`docker ps`) → app is listening on 3080 inside the container → port mapping is `3080:3080` → Security Group allows inbound 3080 → you're using the correct public IP.

**Container is not running**
```bash
docker ps -a
docker logs demo-app
```

**Cannot connect through SSH** — verify the instance is running, the public IP is correct, the private key is correct with proper permissions, and the Security Group allows SSH from your IP.

**Docker image cannot be pulled** — check the Docker Hub repository exists, the image name/tag are correct, and (if private) that `docker login` has been run on the EC2 instance.

---

## Repository Structure

```text
aws-ec2-react-node-docker-deployment/
│
├── README.md
├── Dockerfile
├── docker-compose.yml
├── .dockerignore
├── .gitignore
│
├── api/
│   ├── server.js
│   ├── package.json
│   ├── gulpfile.js
│   └── webpack.config.js
│
├── my-app/
│   ├── public/
│   ├── src/
│   └── package.json
│
├── docs/
│   ├── architecture.md
│   ├── deployment.md
│   └── troubleshooting.md
│
└── screenshots/
```

---

## Key Takeaways

* EC2 provides the compute infrastructure required to host the application.
* Docker provides a consistent packaging and runtime environment for the Express + React app.
* Docker Hub provides a location for storing and retrieving the application image.
* SSH provides administrative access to the EC2 instance.
* Security Groups control network access to the instance.
* Manual deployment provides the foundation for understanding automated CI/CD deployment introduced in later projects.

---

## Related Projects

1. [`aws-ec2-spring-boot-deployment`](https://github.com/Chukwuemeka-Peter-Eze/aws-ec2-spring-boot-deployment) — manual SCP deployment of a Java/Spring Boot app to EC2.
2. **This repo,** Docker-based deployment of a React/Node.js app to EC2.

---

## Project Status

**Status:** Completed

Covers EC2 provisioning, SSH access, Docker image build/push to Docker Hub, pulling/running the container on EC2, Security Group configuration, and browser-based verification.
