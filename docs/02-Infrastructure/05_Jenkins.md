# Jenkins

## Overview

Jenkins is the primary Continuous Integration and Continuous Delivery (CI/CD) automation server within the **Enterprise DevSecOps Infrastructure Platform**.

Unlike a standard Jenkins deployment, this platform uses a **custom-built Jenkins Docker image** that includes a comprehensive DevSecOps toolchain. This enables Jenkins pipelines to compile applications, perform code quality analysis, execute security scans, build container images, and deploy workloads to Kubernetes without requiring additional software installation.

The custom image is maintained as Infrastructure as Code (IaC), ensuring consistent, repeatable, and portable CI/CD environments.

---

# Purpose

Jenkins is responsible for automating the software delivery lifecycle.

Primary responsibilities include:

- Source code checkout from Git repositories
- Build automation using Maven
- Continuous Integration (CI)
- Pipeline as Code
- Static code analysis with SonarQube
- Security scanning with Trivy
- Docker image creation
- Artifact publishing to Nexus Repository
- Kubernetes deployments using kubectl
- Helm-based application deployments
- AWS CLI integration for cloud deployments

---

# Architecture

![Pipeline](images/jenkins-pipeline-flow.png)

---

# Jenkins Deployment

Jenkins is deployed as a Docker container using Docker Compose.

## Service Definition

```yaml
jenkins:
  build:
    context: ./jenkins
    dockerfile: Dockerfile

  image: custom-jenkins:lts
  container_name: jenkins

  restart: unless-stopped
```

The platform builds the Jenkins image locally instead of pulling a preconfigured image.

---

# Container Configuration

| Property | Value |
|----------|-------|
| Container Name | jenkins |
| Image | custom-jenkins:lts |
| Base Image | jenkins/jenkins:lts-jdk17 |
| Restart Policy | unless-stopped |
| Jenkins Home | /var/jenkins_home |

---

# Port Configuration

| Port | Purpose |
|------|---------|
| 8080 | Jenkins Web Interface |
| 50000 | Jenkins Agent Communication |

Access Jenkins:

```
http://localhost:8080
```

---

# Volume Configuration

| Volume | Purpose |
|---------|---------|
| jenkins_home | Jenkins configuration, jobs, plugins, credentials |
| /var/run/docker.sock | Docker daemon access |
| ./jenkins/kube | Kubernetes configuration (read-only) |

Docker Compose configuration:

```yaml
volumes:
  - jenkins_home:/var/jenkins_home
  - /var/run/docker.sock:/var/run/docker.sock
  - ./jenkins/kube:/var/jenkins_home/.kube:ro
```

---

# Network Configuration

Jenkins is connected to two Docker networks.

| Network | Purpose |
|----------|---------|
| cicd-network | Internal platform communication |
| kind | External communication with the local Kind Kubernetes cluster |

This enables Jenkins to communicate with infrastructure services and deploy workloads directly into Kubernetes.

---

# Custom Jenkins Image

The custom image is built from:

```
jenkins/
├── Dockerfile
├── plugins.txt
└── kube/
```

The Dockerfile provisions a complete DevSecOps toolchain during image creation.

---

# Installed DevSecOps Toolchain

The Jenkins image includes the following tools.

| Category | Tool |
|----------|------|
| Java | OpenJDK 17 |
| Build | Maven |
| Source Control | Git |
| JavaScript | Node.js 22 |
| Package Manager | npm |
| Containers | Docker CLI |
| Container Orchestration | Docker Compose |
| Image Builder | Docker Buildx |
| Kubernetes | kubectl |
| Local Kubernetes | Kind |
| Kubernetes Package Manager | Helm |
| Cloud | AWS CLI v2 |
| Security | Trivy |
| YAML Processing | yq |
| JSON Processing | jq |
| Python | Python 3 + pip |

These tools are installed during the Docker image build, ensuring every Jenkins instance has an identical execution environment.

---

# Jenkins Plugins

Plugins are managed using `plugins.txt`, allowing automated and version-controlled plugin installation.

## Source Control

- Git
- Git Client
- GitHub
- GitHub Branch Source

## Pipeline

- Workflow Aggregator
- Pipeline Stage View
- Pipeline Graph View
- Pipeline GitHub Library

## Credentials

- Credentials
- Credentials Binding
- Plain Credentials
- SSH Credentials
- SSH Agent

## Security

- Matrix Authorization Strategy
- Role Strategy

## Configuration

- Configuration as Code
- Job DSL

## Build

- Maven Integration

## Docker

- Docker Workflow

## Code Quality

- SonarQube Scanner

## Artifact Repository

- Nexus Artifact Uploader

## User Interface

- Blue Ocean

## Utilities

- Email Extension
- Workspace Cleanup
- Build Timeout
- Timestamper
- Mailer

Managing plugins through source control ensures reproducible Jenkins deployments.

---

# Docker Integration

The Docker socket is mounted into the Jenkins container.

```yaml
- /var/run/docker.sock:/var/run/docker.sock
```

This enables Jenkins pipelines to:

- Build Docker images
- Run Docker containers
- Push images to registries
- Execute Docker Compose commands

No Docker daemon runs inside the container; Jenkins communicates with the host Docker engine.

---

# Kubernetes Integration

The Kubernetes configuration is mounted as read-only.

```yaml
./jenkins/kube:/var/jenkins_home/.kube:ro
```

This allows Jenkins to:

- Connect to the Kind cluster
- Deploy Kubernetes manifests
- Execute kubectl commands
- Perform rolling updates
- Verify deployments

---

# AWS Integration

AWS CLI v2 is pre-installed, enabling pipelines to interact with AWS services.

Typical use cases include:

- Amazon ECR
- Amazon ECS
- Amazon EKS
- Amazon S3
- CloudFormation

AWS credentials should be managed using Jenkins Credentials, not hardcoded in pipelines.

---

# Security Scanning

The Jenkins image includes Trivy for vulnerability scanning.

Example:

```bash
trivy image my-image:latest
```

This allows pipelines to detect vulnerabilities before deployment.

---

# Verification

Verify Jenkins is running:

```bash
docker ps
```

Check logs:

```bash
docker logs jenkins
```

Retrieve the administrator password:

```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Verify installed tools:

```bash
docker exec jenkins java -version
docker exec jenkins mvn -version
docker exec jenkins docker --version
docker exec jenkins docker compose version
docker exec jenkins kubectl version --client
docker exec jenkins helm version
docker exec jenkins aws --version
docker exec jenkins trivy --version
docker exec jenkins node --version
docker exec jenkins npm --version
```

---

# Best Practices

- Store pipelines in source control using Jenkinsfile.
- Manage credentials through Jenkins Credentials.
- Keep plugins under version control using `plugins.txt`.
- Rebuild the custom image after updating tool versions.
- Use least-privilege credentials for external integrations.
- Keep Docker and Kubernetes tooling up to date.
- Back up the `jenkins_home` volume regularly.

---

# Troubleshooting

## Jenkins does not start

```bash
docker logs jenkins
```

Review startup logs for plugin or configuration errors.

---

## Docker commands fail

Verify the Docker socket is mounted:

```bash
ls -l /var/run/docker.sock
```

Ensure the Jenkins user belongs to the Docker group.

---

## Kubernetes commands fail

Verify the kubeconfig mount:

```bash
docker exec jenkins kubectl config view
```

Confirm the Kind cluster is running.

---

## Plugin installation issues

Rebuild the Jenkins image:

```bash
docker compose build --no-cache jenkins
docker compose up -d
```

---

# Summary

Jenkins serves as the automation engine of the Enterprise DevSecOps Infrastructure Platform.

By combining a custom Docker image, a preinstalled DevSecOps toolchain, version-controlled plugins, Docker integration, Kubernetes connectivity, and AWS tooling, the platform provides a powerful and reproducible CI/CD environment suitable for modern software delivery workflows.

The next document, **06_SonarQube.md**, explains how SonarQube integrates with PostgreSQL and Jenkins to provide continuous code quality analysis and quality gate enforcement.
