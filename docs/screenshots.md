# AWS EC2 Web Deployment — Evidence & Screenshots Guide

## 1. Purpose

This document defines the recommended evidence to capture for the AWS EC2 web application deployment.

Screenshots should demonstrate the actual implementation and provide visual evidence that the infrastructure, container deployment, networking configuration, and application verification were completed.

The AWS project checklist identifies the key implementation stages as:

* EC2 instance creation
* Private key management
* Docker image preparation
* Docker deployment
* Security Group configuration
* Web application access through a browser

The screenshots in this repository should therefore focus on those implementation milestones.

---

# 2. Evidence Principles

Screenshots should be:

* Relevant to the project
* Clear and readable
* Captured from the actual implementation
* Free from unnecessary personal information
* Free from exposed credentials or secrets
* Named consistently
* Organized according to the deployment workflow

Do not capture or publish:

* Private SSH keys
* AWS secret access keys
* Passwords
* Registry passwords
* Tokens
* Session credentials
* Other sensitive information

---

# 3. Recommended Evidence Structure

Store screenshots in the repository using a structure similar to:

```text
screenshots/
├── 01-ec2-instance.png
├── 02-ec2-instance-details.png
├── 03-security-group.png
├── 04-ssh-access.png
├── 05-docker-image.png
├── 06-registry-image.png
├── 07-docker-container.png
├── 08-application-browser.png
└── 09-deployment-verification.png
```

The exact screenshots included should reflect the evidence actually available from the completed project.

---

# 4. EC2 Instance

## Screenshot: EC2 Instance

**Suggested filename:**

```text
01-ec2-instance.png
```

### Capture

Show the EC2 console displaying the created instance.

The screenshot should ideally show:

* Instance identifier
* Instance state
* Instance type
* Public IPv4 address or hostname
* Security Group association

### Purpose

This provides evidence that the EC2 compute environment was successfully provisioned.

---

# 5. EC2 Instance Details

## Screenshot: Instance Configuration

**Suggested filename:**

```text
02-ec2-instance-details.png
```

### Capture

Show the relevant instance configuration details.

Where practical, demonstrate:

* AMI
* Instance type
* Network information
* Key pair association
* Security Group association

### Purpose

This provides additional evidence of how the EC2 host was configured.

---

# 6. Security Group

## Screenshot: Security Group Rules

**Suggested filename:**

```text
03-security-group.png
```

### Capture

Show the inbound rules associated with the EC2 instance.

The screenshot should demonstrate that the port required by the web application has been configured.

The AWS checklist specifically identifies Security Group configuration as part of making the deployed application accessible through a browser.

### Important

Before publishing the screenshot, verify that no unnecessary or sensitive information is visible.

The screenshot should demonstrate the required network configuration without exposing unrelated infrastructure.

---

# 7. SSH Access

## Screenshot: EC2 SSH Session

**Suggested filename:**

```text
04-ssh-access.png
```

### Capture

Show the successful SSH connection to the EC2 instance.

A useful terminal screenshot could show:

```bash
whoami
```

and:

```bash
hostname
```

followed by a Docker command such as:

```bash
docker --version
```

### Purpose

This demonstrates that administrative access to the EC2 host was successfully established.

### Security Warning

Never display the contents of the private key.

Do not include the `.pem` file itself in the repository.

---

# 8. Docker Image

## Screenshot: Docker Image

**Suggested filename:**

```text
05-docker-image.png
```

### Capture

Show the locally built Docker image.

Example:

```bash
docker images
```

The screenshot should demonstrate:

* Repository name
* Image tag
* Image availability

### Purpose

This provides evidence that the application was packaged into a Docker image before deployment.

---

# 9. Private Registry

## Screenshot: Registry Image

**Suggested filename:**

```text
06-registry-image.png
```

### Capture

Show the application image stored in the private Docker registry.

Where appropriate, demonstrate:

* Repository
* Image/tag
* Successful image publication

The project checklist explicitly includes building and pushing the Docker image to a private Docker repository as part of the deployment workflow.

### Security Warning

Do not expose:

* Registry passwords
* Access tokens
* Authentication credentials
* Private credentials

---

# 10. Running Docker Container

## Screenshot: Running Container

**Suggested filename:**

```text
07-docker-container.png
```

### Capture

On the EC2 instance, run:

```bash
docker ps
```

The screenshot should show the application container running.

Where possible, the output should demonstrate:

* Container ID
* Image
* Status
* Port mapping

### Purpose

This proves that the application image was successfully deployed and executed on EC2.

---

# 11. Application in Browser

## Screenshot: Deployed Application

**Suggested filename:**

```text
08-application-browser.png
```

### Capture

Open the deployed application in a browser using the EC2 public address and application port.

Example:

```text
http://<ec2-public-ip>:<application-port>
```

### Purpose

This is the most important end-to-end evidence.

It demonstrates that the application can be reached externally after:

```text
EC2
  ↓
Security Group
  ↓
Docker
  ↓
Container
  ↓
Application
  ↓
Browser
```

---

# 12. Deployment Verification

## Screenshot: Final Verification

**Suggested filename:**

```text
09-deployment-verification.png
```

### Capture

Create a final terminal screenshot showing the deployed state.

For example:

```bash
docker ps
```

and, if useful:

```bash
docker images
```

The objective is to show the application image and running container together.

### Purpose

This provides concise operational evidence that the deployment completed successfully.

---

# 13. Evidence Mapping

The screenshots can be mapped to the deployment workflow as follows:

| Evidence                         | Deployment Stage         |
| -------------------------------- | ------------------------ |
| `01-ec2-instance.png`            | EC2 provisioning         |
| `02-ec2-instance-details.png`    | EC2 configuration        |
| `03-security-group.png`          | Network access           |
| `04-ssh-access.png`              | Remote administration    |
| `05-docker-image.png`            | Image creation           |
| `06-registry-image.png`          | Image distribution       |
| `07-docker-container.png`        | Application deployment   |
| `08-application-browser.png`     | Application verification |
| `09-deployment-verification.png` | Final operational state  |

---

# 14. README Integration

The most useful screenshots should also be referenced from the main README.

Avoid placing every screenshot directly into the README.

Instead, select the strongest evidence.

For example:

```markdown
## Deployment Evidence

### EC2 Instance

![EC2 Instance](screenshots/01-ec2-instance.png)

### Security Group

![Security Group](screenshots/03-security-group.png)

### Running Container

![Docker Container](screenshots/07-docker-container.png)

### Deployed Application

![Application](screenshots/08-application-browser.png)
```

The remaining evidence can remain in the `screenshots/` directory and be referenced from the detailed documentation.

---

# 15. Screenshot Naming Convention

Use lowercase filenames with numbered prefixes.

Recommended pattern:

```text
<number>-<short-description>.png
```

Examples:

```text
01-ec2-instance.png
02-ec2-instance-details.png
03-security-group.png
04-ssh-access.png
05-docker-image.png
06-registry-image.png
07-docker-container.png
08-application-browser.png
09-deployment-verification.png
```

The numerical order should follow the actual deployment sequence.

---

# 16. Screenshot Quality Checklist

Before committing screenshots to GitHub, verify:

* [ ] Screenshot is readable
* [ ] Relevant information is visible
* [ ] Sensitive credentials are hidden
* [ ] Private keys are not visible
* [ ] Passwords are not visible
* [ ] Access tokens are not visible
* [ ] Unnecessary personal information is removed
* [ ] Filename follows repository convention
* [ ] Screenshot corresponds to an actual project milestone
* [ ] Screenshot does not contain unrelated applications or information

---

# 17. Evidence Integrity

Screenshots should represent the actual implementation.

Do not create screenshots solely to make the repository appear complete.

If a particular screenshot was not captured during the original implementation, it is better to omit it than to manufacture evidence.

The repository should distinguish between:

```text
Implemented and verified
```

and:

```text
Documented conceptually
```

This keeps the project portfolio technically credible.

---

# 18. Recommended Evidence Set

A compact but strong evidence set consists of at least:

1. EC2 instance
2. Security Group
3. SSH session
4. Docker image
5. Private registry image
6. Running container
7. Browser-accessible application

This provides evidence across the complete deployment chain:

```text
Infrastructure
      ↓
Security
      ↓
Remote Access
      ↓
Container Image
      ↓
Image Registry
      ↓
Running Container
      ↓
Application
```

---

# 19. Final Evidence Checklist

Before publishing the repository:

```text
AWS EC2
├── [ ] Instance created
├── [ ] Instance running
├── [ ] Instance configuration captured
│
├── Security
│   ├── [ ] Security Group captured
│   └── [ ] No sensitive information exposed
│
├── SSH
│   ├── [ ] SSH access verified
│   └── [ ] Private key excluded
│
├── Docker
│   ├── [ ] Image built
│   └── [ ] Container running
│
├── Registry
│   └── [ ] Image available in private registry
│
└── Application
    ├── [ ] Application accessible
    └── [ ] Browser verification captured
```

---

## 20. Evidence Philosophy

The purpose of project screenshots is not to create a collection of console images.

Each screenshot should answer a specific engineering question:

> **What does this prove?**

For example:

* EC2 screenshot → Was the compute environment provisioned?
* Security Group screenshot → Was network access configured?
* SSH screenshot → Can the host be administered?
* Registry screenshot → Is the deployment artifact available?
* Docker screenshot → Is the application running?
* Browser screenshot → Does the complete deployment work?

A strong project repository uses evidence to tell the story of the deployment rather than simply displaying screenshots.
