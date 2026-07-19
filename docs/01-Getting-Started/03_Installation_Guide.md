# Installation Guide

## Overview

This guide explains how to deploy the **Enterprise DevSecOps Infrastructure Platform** using Docker Compose.

The platform provisions a complete DevSecOps environment consisting of:

- Jenkins LTS (Custom Image)
- SonarQube Community Edition
- PostgreSQL Database
- Nexus Repository OSS
- Apache Tomcat 10

Unlike a standard Jenkins deployment, this platform includes a **custom Jenkins image** preconfigured with enterprise DevSecOps tools, enabling CI/CD pipelines to build, scan, package, containerize, and deploy applications without additional tool installation.

---

# Installation Workflow

```
Clone Repository
        │
        ▼
Review Configuration
        │
        ▼
Build Custom Jenkins Image
        │
        ▼
Start Docker Compose Platform
        │
        ▼
Verify Containers
        │
        ▼
Verify Networks & Volumes
        │
        ▼
Initialize Jenkins
        │
        ▼
Verify SonarQube
        │
        ▼
Verify Nexus Repository
        │
        ▼
Verify Apache Tomcat
        │
        ▼
Platform Ready
```

---

# Platform Architecture

The platform deploys five core services.

| Service | Purpose |
|----------|---------|
| Jenkins | Continuous Integration & DevSecOps Automation |
| SonarQube | Static Code Analysis |
| PostgreSQL | SonarQube Database |
| Nexus Repository OSS | Artifact & Docker Registry |
| Apache Tomcat | Application Deployment Server |

All services communicate through a dedicated Docker bridge network.

---

# Step 1 – Clone the Repository

Clone the repository.

```bash
git clone git@github.com:muralidhargurram39/ci-cd-lab.git

cd ci-cd-lab
```

Verify the repository structure.

```text
ci-cd-lab/
├── docker-compose.yml
├── README.md
├── docs/
├── images/
├── jenkins/
├── nexus/
├── sonarqube/
└── tomcat/
```

---

# Step 2 – Review Docker Compose Configuration

Review the Docker Compose file.

```bash
cat docker-compose.yml
```

The platform deploys the following services.

| Service | Container |
|----------|-----------|
| Jenkins | jenkins |
| SonarQube | sonarqube |
| PostgreSQL | sonar-postgres |
| Nexus Repository | nexus |
| Tomcat | tomcat |

Review:

- Port mappings
- Docker networks
- Persistent volumes
- Restart policies
- Environment variables

---

# Step 3 – Build the Custom Jenkins Image

Build the Jenkins image.

```bash
docker compose build jenkins
```

The custom image is built from:

```text
jenkins/
├── Dockerfile
├── plugins.txt
└── kube/
```

---

# DevSecOps Toolchain

During the image build, Jenkins is configured with the following tools.

| Category | Installed Tool |
|----------|----------------|
| Java | JDK 17 |
| Build | Maven |
| Source Control | Git |
| JavaScript | NodeJS 22 + npm |
| Containers | Docker CLI |
| Container Orchestration | Docker Compose |
| Docker Build | Docker Buildx |
| Kubernetes | kubectl |
| Local Kubernetes | Kind |
| Kubernetes Package Manager | Helm |
| Cloud | AWS CLI v2 |
| Security | Trivy |
| YAML Processing | yq |
| Utilities | jq |
| Python | Python 3 + pip |

This transforms Jenkins into a complete enterprise DevSecOps build agent.

---

# Step 4 – Verify Jenkins Image

Verify that the image was successfully built.

```bash
docker images
```

Expected output:

```text
custom-jenkins:lts
```

---

# Step 5 – Start the Platform

Deploy all services.

```bash
docker compose up -d
```

Docker Compose performs the following tasks:

- Creates Docker networks
- Creates persistent volumes
- Starts PostgreSQL
- Starts SonarQube
- Starts Nexus Repository
- Starts Jenkins
- Starts Apache Tomcat

The initial startup may take several minutes.

---

# Step 6 – Verify Running Containers

Check running containers.

```bash
docker ps
```

Expected containers:

```text
jenkins
sonarqube
sonar-postgres
nexus
tomcat
```

---

# Step 7 – Verify Container Status

```bash
docker compose ps
```

Every container should report:

```text
Up
```

If a container exits unexpectedly:

```bash
docker logs <container-name>
```

Example:

```bash
docker logs sonarqube

docker logs nexus

docker logs jenkins
```

---

# Step 8 – Verify Docker Networks

List networks.

```bash
docker network ls
```

Expected networks include:

- cicd-network
- kind

Inspect the internal platform network.

```bash
docker network inspect cicd-network
```

The `kind` network provides connectivity between Jenkins and your local Kubernetes cluster.

---

# Step 9 – Verify Persistent Volumes

List Docker volumes.

```bash
docker volume ls
```

Expected volumes:

| Volume | Purpose |
|---------|---------|
| jenkins_home | Jenkins configuration & jobs |
| sonar_postgres_data | PostgreSQL Database |
| sonarqube_data | SonarQube Data |
| sonarqube_logs | SonarQube Logs |
| sonarqube_extensions | SonarQube Plugins |
| nexus_data | Nexus Repository Data |
| tomcat_webapps | Deployed Applications |

---

# Step 10 – Access the Platform

Open the following URLs.

| Service | URL |
|----------|-----|
| Jenkins | http://localhost:8080 |
| SonarQube | http://localhost:9000 |
| Nexus Repository | http://localhost:8081 |
| Nexus Docker Registry | http://localhost:8082 |
| Nexus Additional Repository | http://localhost:8083 |
| Apache Tomcat | http://localhost:9090 |

---

# Step 11 – Initialize Jenkins

Retrieve the initial administrator password.

```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Open:

```
http://localhost:8080
```

Complete the setup wizard:

- Unlock Jenkins
- Install plugins (if required)
- Create administrator account
- Configure Jenkins URL

---

# Step 12 – Verify Installed DevSecOps Tools

Verify the tools installed inside Jenkins.

```bash
docker exec jenkins java -version

docker exec jenkins mvn -version

docker exec jenkins docker --version

docker exec jenkins docker compose version

docker exec jenkins docker buildx version

docker exec jenkins kubectl version --client

docker exec jenkins kind version

docker exec jenkins helm version

docker exec jenkins aws --version

docker exec jenkins trivy --version

docker exec jenkins yq --version

docker exec jenkins node --version

docker exec jenkins npm --version
```

All commands should return version information without errors.

---

# Step 13 – Verify SonarQube

Open:

```
http://localhost:9000
```

Default credentials:

```
Username : admin
Password : admin
```

The platform stores SonarQube data in PostgreSQL.

| Property | Value |
|----------|-------|
| Database | sonar |
| Username | sonar |
| Container | sonar-postgres |

After the first login:

- Change the administrator password.
- Verify the dashboard loads successfully.
- Confirm there are no startup errors.

---

# Step 14 – Verify Nexus Repository

Open:

```
http://localhost:8081
```

Retrieve the administrator password.

```bash
docker exec nexus cat /nexus-data/admin.password
```

After logging in:

- Change the administrator password.
- Verify repository manager status.
- Confirm repositories are available.

---

# Step 15 – Verify Apache Tomcat

Open:

```
http://localhost:9090
```

Verify:

- Default Tomcat page loads.
- Web application directory is accessible.
- Deployment directory is writable.

---

# Jenkins Plugins

The custom Jenkins image includes the following plugins.

| Category | Plugins |
|----------|----------|
| SCM | Git, Git Client, GitHub |
| Pipeline | Workflow Aggregator, Pipeline Graph View |
| Credentials | Credentials, SSH Agent |
| Security | Matrix Authorization, Role Strategy |
| Docker | Docker Workflow |
| Quality | SonarQube Scanner |
| Artifact Repository | Nexus Artifact Uploader |
| Configuration | Configuration as Code, Job DSL |
| UI | Blue Ocean |

---

# Installation Verification Checklist

Confirm the following before continuing.

| Verification | Status |
|--------------|:------:|
| Repository cloned | ☐ |
| Jenkins image built | ☐ |
| Docker Compose started | ☐ |
| Jenkins running | ☐ |
| SonarQube running | ☐ |
| PostgreSQL running | ☐ |
| Nexus running | ☐ |
| Tomcat running | ☐ |
| Docker networks created | ☐ |
| Docker volumes created | ☐ |
| Jenkins accessible | ☐ |
| SonarQube accessible | ☐ |
| Nexus accessible | ☐ |
| Tomcat accessible | ☐ |
| DevSecOps tools verified | ☐ |

---

# Stopping the Platform

Stop all services.

```bash
docker compose down
```

---

# Remove the Platform

Remove containers, networks and persistent volumes.

```bash
docker compose down -v
```

> **Warning:** This permanently removes all platform data stored in Docker volumes.

---

# Common Issues

## Jenkins does not start

Check:

```bash
docker logs jenkins
```

---

## SonarQube initialization is slow

The first startup performs database initialization. Wait until the logs indicate that SonarQube is fully operational.

---

## Nexus takes several minutes to start

Nexus performs repository initialization during the first launch. This behavior is expected.

---

## Docker socket permission issues

Verify that the Docker socket is mounted correctly.

```bash
ls -l /var/run/docker.sock
```

Ensure the Jenkins container has access to the Docker group.

---

## Kind network not found

If Docker Compose reports:

```
Network kind declared as external but could not be found
```

Create the Kind cluster (or its Docker network) before starting the platform.

---

# Best Practices

- Build the custom Jenkins image before starting the platform.
- Keep the Docker Compose file under version control.
- Avoid modifying container data directly.
- Back up Docker volumes regularly.
- Update the custom Jenkins image periodically to keep tools current.
- Monitor container logs after upgrades.
- Verify tool versions after rebuilding the Jenkins image.

---

# Summary

You have successfully deployed the **Enterprise DevSecOps Infrastructure Platform** and verified that all core services are operational.

The platform now provides:

- A fully equipped Jenkins DevSecOps build agent
- SonarQube with PostgreSQL backend
- Nexus Repository OSS
- Apache Tomcat deployment server
- Shared Docker networking
- Persistent storage for all stateful services

Continue with **04_Repository_Structure.md** to understand the purpose of each directory, configuration file, and component within the repository before configuring pipelines and integrations.
