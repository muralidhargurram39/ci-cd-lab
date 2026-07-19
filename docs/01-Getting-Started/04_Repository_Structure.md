# Repository Structure

## Overview

The **Enterprise DevSecOps Infrastructure Platform** is organized using a modular directory structure that separates infrastructure components, documentation, configuration, and supporting assets.

This organization makes the repository easier to understand, maintain, and extend while following Infrastructure as Code (IaC) and repository management best practices.

The repository is designed so that each major DevSecOps component is isolated in its own directory, allowing independent configuration and future enhancements.

---

# Repository Layout

```text
ci-cd-lab/
│
├── docker-compose.yml
├── README.md
├── LICENSE
├── CHANGELOG.md
├── VERSION.md
├── SECURITY.md
├── CONTRIBUTING.md
│
├── docs/
│   ├── README.md
│   ├── 01-Getting-Started/
│   ├── 02-Infrastructure/
│   ├── 03-Operations/
│   └── 04-Best-Practices/
│
├── images/
│
├── jenkins/
│   ├── Dockerfile
│   ├── plugins.txt
│   └── kube/
│
├── sonarqube/
│
├── nexus/
│
└── tomcat/
```

---

# Repository Organization

The repository is divided into five major areas.

| Area | Purpose |
|------|---------|
| Root | Project configuration and metadata |
| Documentation | User guides and technical documentation |
| Infrastructure | Individual service configurations |
| Images | Architecture diagrams and screenshots |
| Docker Compose | Platform orchestration |

---

# Root Directory

The root directory contains the primary project files.

| File | Purpose |
|------|---------|
| docker-compose.yml | Defines all platform services, networks, and volumes |
| README.md | Project landing page |
| LICENSE | Open-source license |
| CHANGELOG.md | Version history |
| VERSION.md | Current release information |
| SECURITY.md | Security policy |
| CONTRIBUTING.md | Contribution guidelines |

These files provide project metadata and operational guidance.

---

# Docker Compose

The `docker-compose.yml` file is the heart of the platform.

It defines:

- Infrastructure services
- Docker images
- Container names
- Network configuration
- Persistent volumes
- Restart policies
- Port mappings
- Environment variables
- Service dependencies

Current services include:

| Service | Purpose |
|----------|---------|
| Jenkins | CI/CD Automation |
| SonarQube | Code Quality |
| PostgreSQL | SonarQube Database |
| Nexus Repository | Artifact Repository |
| Apache Tomcat | Application Server |

---

# Documentation Directory

```text
docs/
```

The documentation directory contains all technical documentation for the project.

It is organized into four sections.

## 01 – Getting Started

Contains introductory documentation.

- Project Overview
- Prerequisites
- Installation Guide
- Repository Structure

---

## 02 – Infrastructure

Documents each infrastructure component.

- Jenkins
- SonarQube
- Nexus Repository
- Apache Tomcat
- Docker Compose

---

## 03 – Operations

Operational procedures.

- Network Architecture
- Persistent Storage
- Troubleshooting
- FAQ

---

## 04 – Best Practices

Operational recommendations.

- Security
- Performance
- Maintenance
- Release Process

---

# Images Directory

```text
images/
```

Stores visual assets used throughout the documentation.

Recommended structure:

```text
images/
│
├── architecture/
├── screenshots/
├── logos/
└── diagrams/
```

Example contents:

- Architecture diagrams
- Docker topology
- Jenkins dashboard
- SonarQube dashboard
- Nexus dashboard
- Tomcat dashboard

Keeping images separate from documentation improves maintainability.

---

# Jenkins Directory

```text
jenkins/
```

Contains everything required to build the custom Jenkins image.

Current contents:

```text
jenkins/
├── Dockerfile
├── plugins.txt
└── kube/
```

## Dockerfile

Builds the custom Jenkins image.

Installs:

- Java 17
- Maven
- Docker CLI
- Docker Compose
- Docker Buildx
- kubectl
- Kind
- Helm
- AWS CLI
- Node.js
- Trivy
- Python
- jq
- yq

---

## plugins.txt

Defines Jenkins plugins installed during image build.

Categories include:

- Git
- GitHub
- Pipeline
- Credentials
- Docker Workflow
- SonarQube Scanner
- Nexus Integration
- Blue Ocean
- Configuration as Code

Managing plugins through a file provides consistent Jenkins deployments.

---

## kube/

Contains Kubernetes configuration files mounted into the Jenkins container.

This allows Jenkins pipelines to communicate with the local Kind Kubernetes cluster.

---

# SonarQube Directory

```text
sonarqube/
```

Reserved for SonarQube-specific configuration.

Typical contents may include:

- Configuration files
- Custom plugins
- Initialization scripts

Keeping SonarQube resources isolated simplifies future customization.

---

# Nexus Directory

```text
nexus/
```

Reserved for Nexus Repository configuration.

Possible future additions:

- Repository definitions
- Backup scripts
- Security configuration
- Custom initialization scripts

---

# Tomcat Directory

```text
tomcat/
```

Contains Tomcat-specific resources.

Potential contents include:

- Context configuration
- Server configuration
- Sample applications
- Deployment scripts

---

# Docker Networks

The platform uses two Docker networks.

| Network | Purpose |
|----------|---------|
| cicd-network | Internal communication between platform services |
| kind | External network connecting Jenkins to the local Kind Kubernetes cluster |

This design separates internal service communication from Kubernetes integration.

---

# Persistent Volumes

The platform stores stateful data using Docker volumes.

| Volume | Purpose |
|---------|---------|
| jenkins_home | Jenkins configuration and jobs |
| sonar_postgres_data | PostgreSQL database |
| sonarqube_data | SonarQube data |
| sonarqube_logs | SonarQube logs |
| sonarqube_extensions | SonarQube plugins |
| nexus_data | Nexus repositories |
| tomcat_webapps | Deployed applications |

Using persistent volumes ensures data survives container recreation.

---

# Design Principles

The repository follows several design principles.

## Infrastructure as Code

All infrastructure configuration is maintained in version control.

---

## Separation of Concerns

Each DevSecOps component has its own directory and responsibility.

---

## Modular Design

Components can be modified independently without affecting unrelated services.

---

## Reproducibility

A new environment can be created using only the repository contents.

---

## Scalability

The directory structure supports future expansion with additional tools such as:

- Prometheus
- Grafana
- Loki
- Harbor
- Argo CD

---

# Best Practices

- Keep configuration files under version control.
- Avoid storing secrets in the repository.
- Document all structural changes.
- Keep service configurations isolated.
- Maintain consistent naming conventions.
- Store documentation alongside implementation.

---

# Summary

The repository structure is designed to provide a clear separation between infrastructure, documentation, and supporting assets while following Infrastructure as Code principles.

Each directory has a well-defined purpose, making the platform easier to understand, maintain, and extend. As additional DevSecOps capabilities are introduced, the modular structure allows the repository to grow without sacrificing clarity or maintainability.

The next document, **05_Jenkins.md**, provides a detailed explanation of the custom Jenkins implementation, including the Docker image, installed DevSecOps toolchain, plugins, configuration, and its role within the overall platform.
