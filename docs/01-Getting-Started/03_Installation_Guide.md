# Installation Guide

## Overview

This guide walks you through the installation and initial configuration of the **Enterprise DevSecOps Infrastructure Platform**.

By the end of this guide, you will have a fully functional local DevSecOps environment consisting of:

- Jenkins LTS
- SonarQube Community Edition
- Nexus Repository OSS
- Apache Tomcat
- Docker Compose Networking

The platform is deployed using Docker Compose, ensuring a consistent and repeatable setup across supported environments.

---

# Installation Workflow

The installation process follows the workflow below:

```

Clone Repository
↓
Review Configuration
↓
Build Custom Jenkins Image
↓
Start Docker Compose Services
↓
Verify Running Containers
↓
Access Service Dashboards
↓
Initialize Jenkins
↓
Platform Ready

```

---

# Step 1 – Clone the Repository

Clone the repository using Git.

```bash
git clone git@github.com:muralidhargurram39/ci-cd-lab.git

cd ci-cd-lab
```

Expected directory structure:

```text
ci-cd-lab/
├── docker-compose.yml
├── jenkins/
├── nexus/
├── sonarqube/
├── tomcat/
├── docs/
└── images/
```

---

# Step 2 – Review the Docker Compose Configuration

Open the Docker Compose file.

```bash
cat docker-compose.yml
```

Verify that the services include:

- Jenkins
- SonarQube
- Nexus Repository
- Apache Tomcat

Review:

- Port mappings
- Volume mappings
- Docker network
- Environment variables
- Restart policies

If you need to customize ports or paths, update `docker-compose.yml` before proceeding.

---

# Step 3 – Build the Custom Jenkins Image

The platform uses a custom Jenkins image that includes the tools required for CI/CD pipelines.

Build the image:

```bash
docker compose build jenkins
```

During the build, Docker installs the required components defined in:

```text
jenkins/
├── Dockerfile
├── plugins.txt
└── kube/
```

Verify the image:

```bash
docker images
```

Expected output should include your custom Jenkins image.

---

# Step 4 – Start the Platform

Start all infrastructure services:

```bash
docker compose up -d
```

Docker Compose will:

- Create the shared Docker network.
- Create persistent volumes.
- Start all configured containers.
- Apply restart policies.

Initial startup may take several minutes, especially for SonarQube and Nexus.

---

# Step 5 – Verify Running Containers

List the running containers:

```bash
docker ps
```

Expected output should include containers similar to:

```text
jenkins
sonarqube
nexus
tomcat
```

If a container exits unexpectedly, inspect its logs:

```bash
docker logs <container-name>
```

---

# Step 6 – Verify Container Health

Check the status of each service:

```bash
docker compose ps
```

All containers should report a running state.

If any service is unhealthy, review the logs before continuing.

---

# Step 7 – Access the Platform

Open the following URLs in your browser:

| Service | URL |
|----------|-----|
| Jenkins | http://localhost:8080 |
| SonarQube | http://localhost:9000 |
| Nexus Repository | http://localhost:8081 |
| Apache Tomcat | http://localhost:8082 |

The initial startup of SonarQube and Nexus may take a few minutes.

---

# Step 8 – Initialize Jenkins

When accessing Jenkins for the first time, an administrator password is required.

Retrieve it using:

```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

> **Note:** Replace `jenkins` with the actual container name if it differs in your `docker-compose.yml`.

Copy the password and complete the Jenkins setup wizard.

Recommended actions:

- Install suggested plugins (or verify that your custom image already includes the required plugins).
- Create the initial administrator account.
- Confirm the Jenkins URL.

---

# Step 9 – Verify SonarQube

Open:

```text
http://localhost:9000
```

Default credentials:

```text
Username: admin
Password: admin
```

After the first login:

- Change the default password.
- Verify that the dashboard loads correctly.
- Confirm that no startup errors are displayed.

---

# Step 10 – Verify Nexus Repository

Open:

```text
http://localhost:8081
```

The first startup may take several minutes.

Retrieve the initial administrator password (the exact path depends on your Nexus image and version). Refer to the official Nexus documentation if needed.

Example:

```text
docker exec nexus cat /nexus-data/admin.password
```

After logging in:

- Change the default password.
- Configure repositories as required.
- Verify that the repository manager is operational.

---

# Step 11 – Verify Apache Tomcat

Open:

```text
http://localhost:8082
```

Verify:

- Tomcat home page loads successfully.
- Manager application is accessible (if enabled).
- Configured users and roles work as expected.

---

# Step 12 – Verify Docker Network

List Docker networks:

```bash
docker network ls
```

Inspect the network created by Docker Compose:

```bash
docker network inspect <network-name>
```

Confirm that all services are attached to the same network.

---

# Step 13 – Verify Persistent Volumes

List Docker volumes:

```bash
docker volume ls
```

Verify that persistent volumes exist for services that require data retention.

---

# Installation Verification Checklist

Before proceeding to pipeline configuration, confirm:

| Verification | Status |
|--------------|:------:|
| Repository cloned | ☐ |
| Jenkins image built | ☐ |
| Docker Compose started | ☐ |
| All containers running | ☐ |
| Jenkins accessible | ☐ |
| SonarQube accessible | ☐ |
| Nexus accessible | ☐ |
| Tomcat accessible | ☐ |
| Docker network created | ☐ |
| Persistent volumes created | ☐ |

---

# Stopping the Platform

To stop all services:

```bash
docker compose down
```

To stop services while preserving persistent data:

```bash
docker compose down
```

Docker volumes are retained unless explicitly removed.

---

# Removing the Platform

To remove containers, networks, and associated volumes:

```bash
docker compose down -v
```

> **Warning:** This permanently deletes data stored in Docker volumes.

---

# Common Issues

## Jenkins takes a long time to start

This is normal during the first startup while Jenkins completes initialization.

---

## SonarQube fails to start

Possible causes:

- Insufficient memory
- Existing port conflicts
- Docker resource limits

Increase the Docker memory allocation and restart the service.

---

## Nexus startup is slow

Nexus performs repository initialization during its first launch.

Wait until the logs indicate that the application has started successfully.

---

## Port conflict

Check which process is using the port:

```bash
sudo lsof -i :8080
```

Update the port mapping in `docker-compose.yml` if necessary.

---

## Docker build fails

Rebuild the images:

```bash
docker compose build --no-cache
```

---

# Best Practices

- Build the custom Jenkins image before starting the platform.
- Verify all services are healthy before configuring integrations.
- Avoid modifying container data directly.
- Keep Docker Compose configuration under version control.
- Regularly pull the latest repository changes before upgrades.

---

# Summary

You have successfully deployed the Enterprise DevSecOps Infrastructure Platform and verified that all core services are operational.

The platform is now ready for configuration and integration.

Continue with:

**04_Repository_Structure.md**

to understand the purpose and organization of every directory and configuration file in the repository before configuring Jenkins, SonarQube, Nexus Repository, and Apache Tomcat.
