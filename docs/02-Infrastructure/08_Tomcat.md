# Apache Tomcat

## Overview

Apache Tomcat is the application runtime platform within the Enterprise DevSecOps Infrastructure Platform.

It hosts Java web applications packaged as WAR (Web Application Archive) files that are produced by the CI/CD pipeline.

Within this platform, Tomcat integrates with:

- Jenkins
- Nexus Repository Manager
- Docker Compose
- Docker
- Maven
- SonarQube

The deployment workflow ensures that only validated, versioned artifacts are deployed to the application server.

Rather than compiling applications on the server, Tomcat executes immutable WAR artifacts produced earlier in the CI/CD pipeline.

This separation between build and runtime environments improves consistency, traceability, and deployment reliability.

---

# Apache Tomcat Architecture

The following diagram illustrates Tomcat's role within the Enterprise DevSecOps Infrastructure Platform.

![Architecture Overview](../../images/architecture-overview.png)

Deployment workflow:

```
Developer
      │
      ▼
GitHub
      │
      ▼
Jenkins
      │
Build
      │
Unit Tests
      │
SonarQube
      │
Quality Gate
      │
Package WAR
      │
Publish Artifact
      ▼
Nexus Repository
      │
Retrieve Version
      ▼
Tomcat
      │
Serve Application
```

Tomcat serves only approved application artifacts that have successfully passed the build and quality validation stages.

---

# Purpose

Apache Tomcat is responsible for executing Java web applications.

Its responsibilities include:

- WAR deployment
- Servlet execution
- JSP compilation
- HTTP request processing
- Session management
- Application lifecycle management
- Logging
- Static content delivery

Tomcat is intentionally isolated from the build process, allowing deployments to use immutable artifacts retrieved from Nexus Repository Manager.

---

# Docker Compose Deployment

Tomcat is deployed as a Docker container using Docker Compose.

Example service configuration:

```yaml
tomcat:
  image: tomcat:10-jdk17

  container_name: tomcat

  restart: unless-stopped

  ports:
    - "9090:8080"

  volumes:
    - tomcat_webapps:/usr/local/tomcat/webapps

  networks:
    - cicd-network
```

The deployment provides a consistent runtime environment across development and testing environments.

---

# Container Configuration

| Property | Value |
|----------|-------|
| Container Name | tomcat |
| Image | tomcat:10-jdk17 |
| Runtime | Apache Tomcat 10 |
| Java Version | JDK 17 |
| Restart Policy | unless-stopped |
| Webapps Directory | /usr/local/tomcat/webapps |
| Network | cicd-network |

---

# Port Configuration

Tomcat exposes its internal HTTP port (8080) through Docker.

| Host Port | Container Port | Purpose |
|-----------|----------------|---------|
| 9090 | 8080 | HTTP Application Access |

Application URL:

```
http://localhost:9090
```

Using a non-default host port avoids conflicts with other services running on the development workstation.

---

# Persistent Storage

Application deployments are stored using the persistent Docker volume:

| Volume | Purpose |
|---------|---------|
| tomcat_webapps | Stores deployed WAR files and expanded web applications |

Using a persistent volume ensures that deployed applications remain available after container recreation.

---

# Application Directory Structure

Within the container, Tomcat follows the standard directory layout.

```
/usr/local/tomcat
│
├── bin
├── conf
├── lib
├── logs
├── temp
├── webapps
├── work
└── webapps.dist
```

The `webapps` directory contains deployed WAR files and their expanded application directories.

---

# Network Configuration

Tomcat communicates with other platform services over the shared Docker network.

```
cicd-network
```

Primary integrations include:

| Component | Purpose |
|-----------|---------|
| Jenkins | Deployment automation |
| Nexus | Artifact retrieval |
| SonarQube | Pipeline quality validation (indirect) |

Internal service communication uses Docker DNS.

Example internal endpoint:

```
http://tomcat:8080
```

---

# Deployment Workflow

Within the Enterprise DevSecOps Infrastructure Platform, application deployment follows a controlled workflow.

```
Developer Commit
        │
        ▼
GitHub
        │
        ▼
Jenkins Pipeline
        │
Compile
        │
Unit Tests
        │
SonarQube Analysis
        │
Quality Gate
        │
Package WAR
        │
Publish to Nexus
        │
Retrieve Approved Version
        ▼
Tomcat Deployment
        │
Application Startup
        ▼
Ready to Serve Requests
```

This workflow separates application compilation from runtime execution and ensures that only validated artifacts reach the application server.

---

# Startup Sequence

Tomcat initializes in the following order.

```
Docker Compose
        │
        ▼
Container Created
        │
        ▼
Load Configuration
        │
        ▼
Initialize JVM
        │
        ▼
Start Tomcat
        │
        ▼
Deploy WAR Files
        │
        ▼
Initialize Servlets
        │
        ▼
Accept HTTP Requests
```

During startup, Tomcat automatically scans the `webapps` directory and deploys all available applications.

---

# Design Principles

The Tomcat deployment follows the architectural principles used throughout the Enterprise DevSecOps Infrastructure Platform.

## Immutable Deployments

Applications are deployed as versioned WAR files that are not modified after publication.

---

## Separation of Build and Runtime

Compilation occurs in Jenkins.

Execution occurs in Tomcat.

This separation improves deployment consistency and simplifies troubleshooting.

---

## Infrastructure as Code

The application server is provisioned entirely through Docker Compose, enabling reproducible deployments.

---

## Persistent Application Storage

The `tomcat_webapps` volume preserves deployed applications across container restarts and upgrades.

---

## Standardized Runtime

All Java web applications execute within the same standardized runtime environment, reducing environment-specific issues.

---

# Summary

Apache Tomcat serves as the Java application runtime platform within the Enterprise DevSecOps Infrastructure Platform.

Deployed through Docker Compose and integrated with Jenkins and Nexus Repository Manager, Tomcat provides a consistent environment for hosting validated WAR artifacts while supporting immutable deployments, persistent storage, and standardized application execution.

The next section covers WAR deployment through Jenkins, Nexus integration, application lifecycle management, deployment verification, health endpoints, log management, and future Kubernetes migration considerations.

---

# Enterprise Deployment Pipeline

Within the Enterprise DevSecOps Infrastructure Platform, Apache Tomcat is the final runtime destination for Java web applications.

Unlike traditional deployments where applications are copied directly to the server, this platform deploys **versioned, immutable WAR artifacts** that have successfully completed the CI/CD pipeline.

Deployment workflow:

```
Developer
      │
      ▼
GitHub Repository
      │
      ▼
Jenkins Pipeline
      │
Compile
      │
Unit Tests
      │
SonarQube Analysis
      │
Quality Gate
      │
Package WAR
      │
Publish Artifact
      ▼
Nexus Repository
      │
Retrieve Approved Version
      │
Deploy WAR
      ▼
Apache Tomcat
      │
Health Verification
      ▼
Application Available
```

This workflow ensures that only validated application packages are deployed into the runtime environment.

---

# WAR Deployment Through Jenkins

Jenkins automates the deployment of Java web applications to Tomcat after successful completion of the CI pipeline.

Typical deployment stages include:

1. Checkout source code
2. Compile application
3. Execute unit tests
4. Perform SonarQube analysis
5. Validate Quality Gate
6. Package WAR
7. Publish WAR to Nexus
8. Retrieve approved version
9. Deploy WAR to Tomcat
10. Verify deployment

A simplified Jenkins Pipeline stage:

```groovy
stage('Deploy WAR') {
    steps {
        sh '''
        cp target/application.war /deployments/
        '''
    }
}
```

In production environments, deployment is typically performed through artifact retrieval from Nexus rather than copying directly from the build workspace.

---

# Nexus Integration

Nexus Repository Manager serves as the trusted artifact source for Tomcat deployments.

Deployment flow:

```
Jenkins Build
        │
        ▼
Package WAR
        │
        ▼
Publish to Nexus
        │
        ▼
Versioned Artifact
        │
        ▼
Tomcat Deployment
```

Benefits:

- Immutable deployments
- Version tracking
- Rollback capability
- Release promotion
- Centralized artifact storage
- Reproducible deployments

Instead of rebuilding the application, Tomcat receives the exact artifact that passed validation.

---

# WAR Artifact Lifecycle

Each application package follows a controlled lifecycle.

```
Developer Commit
        │
        ▼
Compile
        │
        ▼
Unit Tests
        │
        ▼
Static Analysis
        │
        ▼
Quality Gate
        │
        ▼
Package WAR
        │
        ▼
Publish to Nexus
        │
        ▼
Retrieve Version
        │
        ▼
Deploy to Tomcat
        │
        ▼
Application Running
```

This process ensures deployment consistency across environments.

---

# Application Deployment Lifecycle

When Tomcat receives a WAR file, it performs several initialization steps before serving requests.

```
WAR Uploaded
      │
      ▼
Expand Archive
      │
      ▼
Create Context
      │
      ▼
Initialize Servlets
      │
      ▼
Load Filters
      │
      ▼
Initialize Listeners
      │
      ▼
Ready
```

Tomcat automatically expands WAR files into application directories unless configured otherwise.

---

# Deployment Verification

After deployment, verify that the application has started successfully.

Access the application:

```bash
curl http://localhost:9090
```

Expected response:

- HTTP 200
- Application homepage
- Health endpoint response (if implemented)

---

# Application Context Verification

Verify that the deployed application context exists.

Example:

```
http://localhost:9090/application
```

Successful deployment confirms:

- WAR expansion completed
- Context initialized
- Servlets loaded
- Application available

---

# Health Endpoints

Modern Java applications often expose health endpoints for monitoring.

Examples include:

```
/
```

```
/health
```

```
/actuator/health
```

If Spring Boot Actuator is enabled, the health endpoint provides application status and readiness information.

Example response:

```json
{
  "status": "UP"
}
```

These endpoints are commonly used by monitoring tools and orchestration platforms.

---

# Jenkins Deployment Verification

After deployment, Jenkins should validate that the application is reachable.

Typical verification commands:

```bash
curl http://localhost:9090
```

or

```bash
curl http://localhost:9090/actuator/health
```

If the verification fails, Jenkins should stop the pipeline and report the deployment failure.

---

# Log Management

Tomcat generates several log files that are useful for troubleshooting and operational monitoring.

| Log | Purpose |
|------|----------|
| catalina.out | Server startup and runtime logs |
| localhost.log | Application deployment events |
| access logs | HTTP request logging |
| manager logs | Deployment operations |

For containerized deployments, logs are typically viewed through Docker.

View container logs:

```bash
docker logs tomcat
```

List log files inside the container:

```bash
docker exec tomcat ls /usr/local/tomcat/logs
```

Monitoring these logs helps identify deployment failures, startup issues, and runtime exceptions.

---

# Deployment Rollback Strategy

Versioned artifacts stored in Nexus simplify rollback procedures.

Rollback workflow:

```
Current Release
        │
Deployment Failure
        │
        ▼
Select Previous Version
        │
        ▼
Retrieve WAR from Nexus
        │
        ▼
Redeploy
        │
        ▼
Application Restored
```

This approach minimizes downtime and avoids rebuilding older application versions.

---

# Future Kubernetes Migration

Although the current platform uses Docker Compose, the deployment model is designed for future migration to Kubernetes.

Current architecture:

```
Docker Compose
      │
      ▼
Tomcat Container
      │
      ▼
Persistent Volume
```

Future architecture:

```
Kubernetes Deployment
        │
        ▼
ReplicaSet
        │
        ▼
Service
        │
        ▼
Ingress
        │
        ▼
PersistentVolumeClaim
```

Resource mapping:

| Current Platform | Kubernetes Equivalent |
|------------------|-----------------------|
| Docker Container | Deployment |
| Docker Network | Cluster Network |
| Port Mapping | Service |
| tomcat_webapps Volume | PersistentVolumeClaim |
| Restart Policy | ReplicaSet |
| Health Verification | Liveness & Readiness Probes |

By following immutable deployment practices today, the transition to Kubernetes becomes significantly simpler.

---

# Summary

The deployment architecture combines Jenkins, Nexus Repository Manager, and Apache Tomcat into a controlled application delivery pipeline.

Validated WAR artifacts are versioned, stored centrally, and deployed to Tomcat only after successfully passing build, test, and quality validation stages. Health verification, centralized logging, rollback support, and Kubernetes-ready deployment practices provide a strong operational foundation for modern Enterprise DevSecOps environments.

The next section covers operational verification, health checks, backup and restore procedures, troubleshooting, security best practices, operational maintenance, and integration with the companion automation deployment project.

---

# Verification

After deploying Apache Tomcat, verify that the application server is fully operational before routing traffic or triggering downstream deployment stages.

---

## Verify Container Status

Ensure the Tomcat container is running.

```bash
docker ps
```

Expected output:

```
tomcat    Up
```

---

## Verify Docker Compose Services

```bash
docker compose ps
```

Expected status:

```
tomcat    Up
```

---

## Verify Container Logs

Review Tomcat startup logs.

```bash
docker logs tomcat
```

Successful startup typically includes messages indicating:

- Tomcat server initialized
- Catalina started
- Web application deployed
- Server ready to accept requests

---

## Verify HTTP Access

Open the application:

```
http://localhost:9090
```

or verify using curl:

```bash
curl http://localhost:9090
```

Expected result:

- HTTP 200 OK
- Application homepage or landing page

---

## Verify Application Context

If the application is deployed under a context path:

```bash
curl http://localhost:9090/application
```

Successful deployment confirms:

- WAR extracted
- Context loaded
- Servlets initialized
- Application responding

---

## Verify WAR Deployment

Inspect the webapps directory.

```bash
docker exec tomcat ls /usr/local/tomcat/webapps
```

Expected contents:

```
ROOT
application
application.war
```

---

## Verify Application Logs

Inspect log files.

```bash
docker exec tomcat ls /usr/local/tomcat/logs
```

Common logs include:

- catalina.out
- localhost.log
- localhost_access_log.*

---

## Verify Jenkins Deployment

Confirm Jenkins successfully completed:

- Build
- Unit Tests
- SonarQube Analysis
- Quality Gate
- WAR Packaging
- Nexus Publication
- Deployment

Deployment should only occur after successful validation.

---

# Health Checks

Routine operational checks should verify:

- Tomcat container running
- Application context loaded
- HTTP endpoint responding
- Health endpoint available
- Logs free of critical errors
- WAR deployment completed
- Persistent volume mounted
- Jenkins deployment successful

---

# Operational Maintenance

Routine maintenance improves long-term stability.

Recommended tasks include:

- Upgrade Tomcat releases
- Upgrade Java runtime
- Review deployed applications
- Remove obsolete WAR files
- Rotate logs
- Monitor JVM memory
- Monitor thread pools
- Monitor disk usage
- Review deployment history
- Validate backup completion

---

# Backup Strategy

Application persistence is provided by:

| Volume | Purpose |
|--------|----------|
| tomcat_webapps | WAR files and expanded applications |

Recommended backup schedule:

| Environment | Frequency |
|------------|-----------|
| Development | Weekly |
| Production | Daily |

Backup should include:

- WAR artifacts
- Expanded applications (if retained)
- Configuration files
- SSL certificates (if applicable)
- Custom libraries

---

# Restore Strategy

Restore workflow:

Stop services:

```bash
docker compose down
```

Restore the persistent volume.

Restart:

```bash
docker compose up -d
```

Verify:

- Applications deployed
- HTTP access
- Logs
- Context loading
- Health endpoints

---

# Performance Optimization

For production environments, optimize the following areas.

## JVM

Monitor:

- Heap usage
- Garbage collection
- CPU utilization

Increase JVM heap as required.

---

## Connector Configuration

Tune:

- Maximum threads
- Connection timeout
- Keep-alive settings
- Compression

to support anticipated workloads.

---

## Session Management

Use external session storage where high availability is required.

---

## Static Resources

Enable caching for:

- CSS
- JavaScript
- Images
- Fonts

to improve response times.

---

# Troubleshooting

## Container Does Not Start

Inspect logs.

```bash
docker logs tomcat
```

Common causes:

- Invalid configuration
- Port conflict
- Insufficient memory
- Corrupted deployment

---

## HTTP 404

Possible causes:

- WAR not deployed
- Incorrect context path
- Deployment failure

Verify:

```bash
docker exec tomcat ls /usr/local/tomcat/webapps
```

---

## HTTP 500

Review application logs.

Typical causes:

- Database connectivity
- Missing configuration
- Runtime exceptions

---

## Port Already in Use

Verify:

```bash
netstat -tulpn | grep 9090
```

or

```bash
ss -tulpn | grep 9090
```

Resolve any conflicting services before restarting Tomcat.

---

## Deployment Failure

Review:

```bash
docker logs tomcat
```

Common causes:

- Corrupted WAR
- Missing libraries
- Invalid deployment descriptor
- Servlet initialization failure

---

## Out of Memory

Symptoms include:

- Slow responses
- Frequent garbage collection
- JVM crashes

Increase allocated heap memory and monitor usage.

---

## Jenkins Deployment Failure

Verify:

- Build completed
- Nexus artifact available
- Correct WAR version retrieved
- Deployment permissions
- Network connectivity

---

# Security Best Practices

Improve Tomcat security using the following practices.

- Keep Tomcat updated.
- Keep JDK updated.
- Remove default sample applications.
- Remove unused management applications.
- Restrict Manager application access.
- Use HTTPS behind a reverse proxy.
- Disable directory listings.
- Limit exposed ports.
- Deploy only signed and approved artifacts.
- Review deployment permissions regularly.

---

# Operational Best Practices

Recommended enterprise practices:

- Deploy immutable WAR files.
- Store releases in Nexus.
- Never deploy directly from developer workstations.
- Automate deployments through Jenkins.
- Validate deployments using health checks.
- Monitor application logs.
- Monitor JVM performance.
- Version application releases.
- Back up deployment volumes.
- Document deployment procedures.

---

# Integration with Enterprise DevSecOps Platform

Apache Tomcat serves as the runtime layer for validated application artifacts.

Overall workflow:

```
Developer
      │
      ▼
GitHub
      │
      ▼
Jenkins
      │
Compile
      │
Unit Tests
      │
SonarQube
      │
Quality Gate
      │
Package WAR
      │
Publish to Nexus
      │
Retrieve Approved Version
      │
Deploy to Tomcat
      │
Health Verification
      ▼
Application Available
```

This workflow separates:

- Source Control
- Build Automation
- Code Quality
- Artifact Management
- Runtime Execution

creating a robust and repeatable deployment process.

---

# Future Kubernetes Migration

The current Docker Compose implementation has been designed with Kubernetes migration in mind.

| Current Platform | Kubernetes Equivalent |
|------------------|-----------------------|
| Docker Compose | Helm / Kustomize |
| Tomcat Container | Deployment |
| tomcat_webapps | PersistentVolumeClaim |
| Port Mapping | Service |
| Restart Policy | ReplicaSet |
| Health Verification | Liveness & Readiness Probes |
| Reverse Proxy | Ingress Controller |

Migration benefits include:

- Rolling updates
- Horizontal scaling
- Self-healing
- Automated failover
- Native service discovery
- Resource quotas

---

# Related Project

This infrastructure repository provides the Apache Tomcat runtime used by the companion repository:

**automation-deployment-project**

The companion repository demonstrates:

- Maven build automation
- WAR packaging
- SonarQube Quality Gate validation
- Nexus artifact publication
- Jenkins deployment pipeline
- Docker containerization
- Kubernetes-ready deployment strategy

Together, the repositories demonstrate a complete Enterprise DevSecOps software delivery pipeline from source code commit through production-ready application deployment.

---

# Summary

Apache Tomcat serves as the Java application runtime within the Enterprise DevSecOps Infrastructure Platform.

Integrated with Jenkins, Nexus Repository Manager, Maven, SonarQube, Docker, and Docker Compose, it provides a standardized environment for hosting immutable, version-controlled Java web applications.

Key capabilities include:

- Docker Compose deployment
- Persistent application storage
- WAR deployment automation
- Jenkins integration
- Nexus artifact retrieval
- Health verification
- Centralized logging
- Version-controlled releases
- Rollback support
- Kubernetes-ready architecture
- Infrastructure as Code

By separating application build, validation, artifact management, and runtime execution, the platform enables secure, repeatable, and enterprise-grade Java application deployments while providing a clear migration path toward Kubernetes-based application hosting.

---

## Next Document

Continue with:

**09_Docker_Compose.md**

The final infrastructure guide will cover:

- Overall platform architecture
- Complete `docker-compose.yml` walkthrough
- Service dependencies
- Networks and volumes
- Startup and shutdown workflow
- Configuration management
- Scaling considerations
- Operations and troubleshooting
- Security best practices
- Future Kubernetes migration
