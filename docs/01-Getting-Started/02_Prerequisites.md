# Prerequisites

## Overview

This document outlines the hardware, software, and system requirements necessary to deploy and run the **Enterprise DevSecOps Infrastructure Platform**.

Ensuring these prerequisites are met before starting the installation process will help avoid common deployment issues and provide a consistent environment across different systems.

---

# Supported Platforms

The platform has been validated on the following environments.

| Platform | Supported | Notes |
|----------|:---------:|------|
| Ubuntu 22.04 LTS | ✅ | Recommended |
| Ubuntu 24.04 LTS | ✅ | Recommended |
| Windows 11 + WSL2 | ✅ | Fully Supported |
| Windows Server 2022 | ⚠️ | Docker Desktop Required |
| macOS (Apple Silicon / Intel) | ⚠️ | Should work with Docker Desktop but not officially validated |

---

# Minimum Hardware Requirements

| Resource | Minimum | Recommended |
|-----------|----------|--------------|
| CPU | 4 Cores | 8 Cores |
| RAM | 8 GB | 16 GB |
| Disk Space | 20 GB | 50 GB SSD |
| Internet | Required | Broadband |

> **Note:** SonarQube and Nexus consume a significant amount of memory. For optimal performance, allocate at least **16 GB RAM** to your system.

---

# Required Software

Install the following software before proceeding.

| Software | Version | Required |
|----------|----------|:--------:|
| Git | Latest Stable | ✅ |
| Docker Engine / Docker Desktop | 24.x or later | ✅ |
| Docker Compose | v2.x | ✅ |
| Java (Optional for host) | JDK 17+ | ⚠️ |
| Web Browser | Chrome / Edge / Firefox | ✅ |

> **Note:** Java is included within the relevant containers. A local Java installation is only required if you plan to build Java applications outside Docker.

---

# Docker Requirements

Docker must be installed and running.

Verify using:

```bash
docker --version
```

Example output:

```text
Docker version 29.x.x
```

Verify Docker Compose:

```bash
docker compose version
```

Example output:

```text
Docker Compose version v2.x.x
```

Verify Docker is operational:

```bash
docker ps
```

The command should execute without permission or daemon errors.

---

# Git Requirements

Verify Git installation:

```bash
git --version
```

Example output:

```text
git version 2.x.x
```

---

# WSL Requirements (Windows Users)

If using Windows, WSL2 is recommended.

Verify WSL version:

```bash
wsl --status
```

Expected output should indicate:

- WSL Version 2
- Ubuntu Distribution Installed

---

# Docker Desktop Configuration

Ensure Docker Desktop is configured with sufficient resources.

Recommended settings:

| Setting | Recommended |
|----------|-------------|
| CPUs | 4 or more |
| Memory | 8–12 GB (16 GB preferred if available) |
| Swap | 2 GB or more |
| Disk Image | 50 GB or more |

---

# Network Requirements

The following ports must be available.

| Service | Port |
|----------|------|
| Jenkins | 8080 |
| SonarQube | 9000 |
| Nexus Repository | 8081 |
| Apache Tomcat | 8082 |

Check if a port is already in use:

```bash
sudo lsof -i :8080
```

Repeat for any required port.

---

# Docker Resources

Verify Docker has enough free resources:

```bash
docker system df
```

If disk usage is high, clean unused resources:

```bash
docker system prune -a
```

> **Warning:** This command removes unused images, stopped containers, networks, and build cache.

---

# Internet Connectivity

An active internet connection is required for:

- Pulling Docker images
- Downloading Jenkins plugins
- Maven dependency resolution
- Git repository access
- Package updates

Verify connectivity:

```bash
ping github.com
```

---

# Repository Access

Clone the repository:

```bash
git clone git@github.com:muralidhargurram39/ci-cd-lab.git
```

Navigate into the project:

```bash
cd ci-cd-lab
```

---

# Directory Verification

Verify the expected repository structure:

```text
ci-cd-lab/
├── docker-compose.yml
├── README.md
├── docs/
├── images/
├── jenkins/
├── nexus/
├── sonarqube/
├── tomcat/
├── LICENSE
├── CHANGELOG.md
├── VERSION.md
├── SECURITY.md
└── CONTRIBUTING.md
```

---

# Environment Verification Checklist

Before continuing, verify the following:

| Check | Status |
|--------|:------:|
| Docker Installed | ☐ |
| Docker Running | ☐ |
| Docker Compose Installed | ☐ |
| Git Installed | ☐ |
| Repository Cloned | ☐ |
| Required Ports Available | ☐ |
| Internet Connectivity | ☐ |
| Minimum Hardware Available | ☐ |

All items should be complete before proceeding.

---

# Common Issues

## Docker daemon not running

Symptoms:

```text
Cannot connect to the Docker daemon
```

Solution:

Start Docker Desktop or the Docker service:

```bash
sudo systemctl start docker
```

---

## Permission denied while running Docker

Symptoms:

```text
permission denied while trying to connect to the Docker daemon socket
```

Solution:

Add your user to the Docker group:

```bash
sudo usermod -aG docker $USER
```

Log out and log back in for the change to take effect.

---

## Port already in use

Symptoms:

```text
Bind for 0.0.0.0:8080 failed
```

Solution:

Identify the process using the port:

```bash
sudo lsof -i :8080
```

Stop the conflicting service or modify the port mapping in `docker-compose.yml`.

---

## Docker Desktop memory issues

Symptoms:

- Slow startup
- Container crashes
- SonarQube fails to initialize

Solution:

Increase the memory allocation in Docker Desktop and restart the platform.

---

# Best Practices

- Keep Docker Desktop updated.
- Allocate sufficient CPU and memory resources.
- Use SSD storage for improved container performance.
- Avoid running other resource-intensive applications during initial setup.
- Verify Docker and Git versions before reporting issues.
- Pull the latest repository changes before starting a new installation.

---

# Summary

You have now verified that your system meets the requirements to run the Enterprise DevSecOps Infrastructure Platform.

In the next document, **03_Installation_Guide.md**, you will deploy the complete infrastructure, verify all services, and access the Jenkins, SonarQube, Nexus Repository, and Apache Tomcat dashboards.
