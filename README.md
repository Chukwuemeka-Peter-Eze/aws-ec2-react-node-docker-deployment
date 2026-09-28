# AWS EC2 React + Node.js Docker Deployment

Deploy and host a containerized React/Node.js web application on an Amazon EC2 instance using Docker, SSH, and AWS Security Groups.

![AWS](https://img.shields.io/badge/AWS-EC2-orange)
![Docker](https://img.shields.io/badge/Docker-Container-blue)
![Node.js](https://img.shields.io/badge/Node.js-Express-green)
![React](https://img.shields.io/badge/React-Frontend-61DAFB)

## Overview

The app is a single Express server (`api/server.js`) that serves a React frontend (`my-app/`) as static files and exposes a small JSON API under `/api`. It's packaged as one Docker image and deployed to an EC2 instance, which provides the compute environment while Docker runs the container itself.

Getting it online also means handling basic AWS networking: an EC2 Security Group has to allow the right inbound traffic before the app is reachable from a browser at all.

The goal here is understanding the manual workflow end to end (build, push, pull, run, expose) before automating any of it with CI/CD.

This repo is the containerized counterpart to [`aws-ec2-spring-boot-deployment`](https://github.com/Chukwuemeka-Peter-Eze/aws-ec2-spring-boot-deployment), which deploys a different (Java/Spring Boot) app manually via SCP.

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

## Technology Stack

| Layer            | Technology                          |
| ------------------ | ------------------------------------ |
| Cloud Provider     | AWS                                   |
| Compute            | Amazon EC2                            |
| Container Runtime  | Docker Engine                          |
| Image Registry     | Docker Hub (`pierrechukason/demo-app`) |
| Backend            | Node.js / Express                     |
| Frontend           | React (`create-react-app`)            |
| Build Tools        | webpack, gulp (used for the frontend build only, not inside the Docker image) |
| Base Image (build) | `node:20-alpine`                       |
| Base Image (runtime)| `node:20-alpine`                      |
| Application Port   | `3080`                                 |

## Project Objectives

* Create and configure an Amazon EC2 instance.
* Securely manage the private SSH key required for instance access.
* Build and push a Docker image to Docker Hub.
* Connect to the EC2 instance using SSH.
* Install Docker on the EC2 instance.
* Retrieve and run the application Docker image.
* Configure the EC2 Security Group to allow access to the web application.
* Verify the deployed application through a web browser.

## Deployment Workflow

### 1. Create the EC2 Instance

Configuration includes the AMI, instance type, key pair, Security Group, and network settings. This instance is what will run the container.

### 2. Configure SSH Access

```bash
chmod 400 <private-key>.pem
```

This keeps the private key from being unnecessarily accessible to other users on the system.

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

Add an inbound rule allowing TCP traffic on port `3080`, plus SSH on `22` restricted to your own IP. Open only what the application actually needs.

### 9. Verify the Deployment

```text
http://<ec2-public-ip>:3080
```

If the page loads, that confirms the instance is running, the container is running, the app is listening on 3080, and the Security Group is letting the traffic through.

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

Open `http://<ec2-public-ip>:3080` in a browser.

## Security Considerations

### Protect private keys

Never commit `.pem` files or credentials to Git:

```gitignore
*.pem
.env
```

### Security Groups

Expose only the traffic the application needs. Avoid opening administrative ports to the entire internet.

### Credentials

AWS credentials, Docker Hub credentials, and other secrets should never be hard-coded into source or committed to version control.

## Troubleshooting

### Application cannot be accessed

Check these in order:

1. The EC2 instance is running.
2. The container is running (`docker ps`).
3. The app is listening on 3080 inside the container.
4. The port mapping is `3080:3080`.
5. The Security Group allows inbound traffic on 3080.
6. You're using the correct public IP.

### Container is not running

```bash
docker ps -a
docker logs demo-app
```

### Cannot connect through SSH

Verify the instance is running, the public IP is correct, the private key is correct with proper permissions, and the Security Group allows SSH from your IP.

### Docker image cannot be pulled

Check that the Docker Hub repository exists, the image name and tag are correct, and (if the repo is private) that `docker login` has been run on the EC2 instance.

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

## Key Takeaways

* EC2 provides the compute infrastructure required to host the application.
* Docker provides a consistent packaging and runtime environment for the Express + React app.
* Docker Hub provides a location for storing and retrieving the application image.
* Security Groups control network access to the instance, and matter just as much as the container config.
* This manual workflow is the foundation for the automated CI/CD deployment covered in later projects.

## Related Projects

1. [`aws-ec2-spring-boot-deployment`](https://github.com/Chukwuemeka-Peter-Eze/aws-ec2-spring-boot-deployment): manual SCP deployment of a Java/Spring Boot app to EC2.
2. **This repo**: Docker-based deployment of a React/Node.js app to EC2.

## Project Status

**Status:** Completed

Covers EC2 provisioning, SSH access, Docker image build/push to Docker Hub, pulling/running the container on EC2, Security Group configuration, and browser-based verification.