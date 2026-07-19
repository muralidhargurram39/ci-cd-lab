# SonarQube

## Overview

SonarQube is the centralized **Continuous Code Quality and Security Analysis Platform** within the Enterprise DevSecOps Infrastructure Platform.

It performs automated static code analysis during every CI/CD pipeline execution to identify bugs, vulnerabilities, code smells, duplicated code, maintainability issues, and technical debt before software reaches production.

Unlike manual code reviews, SonarQube continuously evaluates every commit against predefined quality standards, enabling development teams to maintain high-quality, secure, and maintainable code.

Within this platform, SonarQube is fully integrated with:

- Jenkins
- Maven
- GitHub
- PostgreSQL
- Docker Compose

The analysis results become part of the CI/CD pipeline, allowing Jenkins to automatically enforce Quality Gates before deployments proceed.

---

# SonarQube Architecture

The following diagram illustrates the overall Enterprise DevSecOps platform architecture in which SonarQube participates.

![Architecture Overview](../../images/architecture-overview.png)

SonarQube receives source code from Jenkins scanners, stores analysis results inside PostgreSQL, evaluates Quality Gates, and exposes dashboards through its web interface.

The architecture consists of four primary layers:

- Jenkins CI/CD
- Sonar Scanner
- SonarQube Server
- PostgreSQL Database

---

# Purpose

SonarQube exists to improve software quality by continuously analyzing source code.

Primary responsibilities include:

- Static code analysis
- Bug detection
- Security vulnerability detection
- Security Hotspot identification
- Code Smell detection
- Duplicate code analysis
- Maintainability analysis
- Technical debt estimation
- Quality Gate enforcement
- Code coverage reporting

Rather than finding issues after deployment, SonarQube shifts quality validation earlier into the development lifecycle.

---

# Key Features

The platform provides the following capabilities.

### Static Code Analysis

Automatically scans source code during every build.

---

### Quality Gates

Blocks deployments if predefined quality standards are not met.

---

### Security Analysis

Identifies:

- Vulnerabilities
- Security Hotspots
- Insecure coding practices

---

### Technical Debt

Calculates estimated remediation effort.

---

### Code Coverage

Displays unit test coverage metrics.

---

### Duplicate Code Detection

Identifies repeated code blocks.

---

### Historical Trends

Tracks quality improvements over time.

---

### Multi-language Support

Supports analysis for:

- Java
- JavaScript
- TypeScript
- Python
- C#
- Go
- Kotlin
- PHP
- Ruby
- Scala

---

# SonarQube Deployment

SonarQube is deployed as a Docker container using Docker Compose.

Example service definition:

```yaml
sonarqube:
  image: sonarqube:lts-community

  container_name: sonarqube

  restart: unless-stopped

  depends_on:
    sonar-postgres:
      condition: service_healthy

  environment:
    SONAR_JDBC_URL: jdbc:postgresql://sonar-postgres:5432/sonarqube
    SONAR_JDBC_USERNAME: sonar
    SONAR_JDBC_PASSWORD: sonar

  ports:
    - "9000:9000"

  volumes:
    - sonarqube_data:/opt/sonarqube/data
    - sonarqube_logs:/opt/sonarqube/logs
    - sonarqube_extensions:/opt/sonarqube/extensions

  networks:
    - cicd-network
```

The container starts only after PostgreSQL becomes healthy.

---

# Container Configuration

| Property | Value |
|----------|-------|
| Container Name | sonarqube |
| Image | sonarqube:lts-community |
| Restart Policy | unless-stopped |
| Database | PostgreSQL |
| Port | 9000 |
| Network | cicd-network |

---

# Port Configuration

| Port | Purpose |
|------|----------|
| 9000 | SonarQube Web Interface |

Access the dashboard:

```
http://localhost:9000
```

---

# PostgreSQL Integration

Unlike the embedded database used for demonstrations, this platform uses PostgreSQL as the production-ready backend.

Database container:

```
sonar-postgres
```

Database:

```
sonarqube
```

Database user:

```
sonar
```

Connection URL:

```text
jdbc:postgresql://sonar-postgres:5432/sonarqube
```

Using PostgreSQL provides:

- Better performance
- Persistent storage
- Scalability
- Reliability
- Backup support

---

# Persistent Storage

SonarQube uses dedicated Docker volumes.

| Volume | Purpose |
|---------|----------|
| sonarqube_data | Analysis indexes |
| sonarqube_logs | Server logs |
| sonarqube_extensions | Plugins |

These volumes ensure that configuration and historical analysis results remain available after container recreation.

---

# Network Configuration

The SonarQube container communicates with:

- Jenkins
- PostgreSQL
- Nexus
- Tomcat

using:

```
cicd-network
```

Internal service communication uses Docker DNS.

Example:

```
http://sonarqube:9000
```

instead of localhost.

---

# SonarQube Edition

This project uses:

```
SonarQube Community LTS
```

Community Edition provides:

- Static analysis
- Quality Gates
- Security Hotspots
- Code Smells
- Bugs
- Vulnerabilities
- Technical Debt

For enterprise environments, additional editions introduce:

- Branch Analysis
- Pull Request Decoration
- Portfolio Management
- Governance
- Enterprise Reporting

The current platform is intentionally based on the Community LTS edition to provide an open-source, production-inspired environment.

---

# Startup Sequence

Docker Compose starts SonarQube in the following order.

```
Docker Compose
        │
        ▼
PostgreSQL
        │
Healthy
        │
        ▼
SonarQube
        │
Load Plugins
        │
Initialize Database
        │
Start Web Server
        │
        ▼
Ready
```

The `depends_on` configuration ensures SonarQube waits for PostgreSQL before initialization.

---

# Service Dependencies

SonarQube depends on the following platform components.

| Component | Purpose |
|------------|----------|
| PostgreSQL | Stores analysis results |
| Jenkins | Executes code analysis |
| Maven | Runs Sonar Scanner |
| Docker Compose | Service orchestration |

---

# Design Principles

The SonarQube deployment follows the same design principles as the rest of the platform.

## Infrastructure as Code

Deployment is completely defined in Docker Compose.

---

## Persistent Storage

Historical analysis is preserved using Docker volumes.

---

## Service Isolation

SonarQube runs independently from Jenkins while remaining connected through Docker networking.

---

## Reproducibility

Every deployment creates an identical SonarQube environment.

---

## Scalability

The architecture can easily evolve to support:

- Enterprise Edition
- Data Center Edition
- External authentication
- Reverse proxy
- SSL termination
- Monitoring integrations

without redesigning the deployment.

---

# Summary

SonarQube provides the centralized code quality and security analysis capability within the Enterprise DevSecOps Infrastructure Platform.

Backed by PostgreSQL and integrated with Jenkins, Maven, and Docker Compose, it enables continuous inspection of source code while enforcing quality standards before deployment.

The next section explains the internal analysis engine, Quality Gates, Quality Profiles, scanner architecture, Jenkins integration, Maven integration, security hotspots, and reporting dashboards.

---

# Code Analysis Engine

The SonarQube Analysis Engine is responsible for inspecting source code, computing software quality metrics, and identifying issues that impact maintainability, reliability, and security.

Unlike traditional compilers, the analysis engine examines the source code without executing the application. This approach, known as **Static Application Security Testing (SAST)**, enables developers to detect problems early in the software development lifecycle.

The analysis engine performs:

- Static Code Analysis
- Bug Detection
- Vulnerability Detection
- Security Hotspot Analysis
- Code Smell Detection
- Duplicate Code Detection
- Complexity Analysis
- Technical Debt Estimation
- Test Coverage Processing
- Maintainability Rating

The computed metrics are stored in PostgreSQL and displayed through the SonarQube dashboard.

---

# Analysis Workflow

During every Jenkins pipeline execution, the following workflow occurs.

```
Developer Commit
        │
        ▼
GitHub Repository
        │
        ▼
Jenkins Pipeline
        │
        ▼
Maven Build
        │
        ▼
Sonar Scanner
        │
        ▼
SonarQube Server
        │
        ▼
Rules Engine
        │
        ▼
Quality Gate
        │
        ▼
Dashboard & Reports
```

Each analysis is associated with a specific project, allowing historical tracking of quality metrics across builds.

---

# Code Metrics

SonarQube continuously calculates a comprehensive set of software quality metrics.

| Metric | Description |
|---------|-------------|
| Lines of Code | Total executable code |
| Bugs | Reliability issues |
| Vulnerabilities | Security weaknesses |
| Security Hotspots | Potential security risks requiring review |
| Code Smells | Maintainability issues |
| Technical Debt | Estimated remediation effort |
| Duplicated Code | Repeated code blocks |
| Coverage | Unit test coverage |
| Complexity | Cyclomatic complexity |
| Maintainability Rating | Overall maintainability score |
| Reliability Rating | Overall reliability score |
| Security Rating | Overall security score |

These metrics provide objective indicators of software quality and help teams prioritize remediation efforts.

---

# Quality Gates

Quality Gates define the minimum acceptable quality standards for an application.

A Quality Gate is evaluated after every analysis.

If any condition fails, Jenkins can automatically stop the deployment pipeline.

Typical Quality Gate conditions include:

- No new bugs
- No new vulnerabilities
- Coverage greater than 80%
- Duplicated code below 3%
- Maintainability rating of A
- Reliability rating of A
- Security rating of A

This ensures that only high-quality code progresses through the CI/CD pipeline.

---

# Quality Gate Workflow

```
Source Code
      │
      ▼
Analysis
      │
      ▼
Quality Gate
      │
 ┌────┴────┐
 │         │
Pass      Fail
 │         │
 ▼         ▼
Deploy   Stop Pipeline
```

By integrating Quality Gates with Jenkins, deployment decisions become automated and objective.

---

# Quality Profiles

Quality Profiles define the coding standards and rules applied during analysis.

Each programming language has an associated Quality Profile.

Examples include:

- Java
- JavaScript
- TypeScript
- Python
- C#
- Go

Organizations can customize profiles to match internal coding standards and compliance requirements.

---

# Rules Engine

The Rules Engine evaluates source code against thousands of predefined rules.

Rules are categorized into:

### Bugs

Logic or programming errors that may produce incorrect results.

Examples:

- Null pointer dereference
- Resource leaks
- Infinite recursion

---

### Vulnerabilities

Security flaws that attackers may exploit.

Examples:

- SQL Injection
- Cross-Site Scripting (XSS)
- Command Injection
- Hardcoded Credentials

---

### Code Smells

Maintainability issues that increase technical debt.

Examples:

- Long methods
- Duplicate logic
- Unused variables
- Excessive complexity

---

### Security Hotspots

Security-sensitive code requiring manual review.

Examples:

- Cryptography
- Authentication
- Authorization
- File uploads
- HTTP headers

Security Hotspots are not necessarily vulnerabilities but indicate code that deserves additional inspection.

---

# Sonar Scanner Architecture

The Sonar Scanner acts as the bridge between Jenkins and the SonarQube Server.

Workflow:

```
Jenkins
     │
     ▼
Sonar Scanner
     │
     ▼
HTTP API
     │
     ▼
SonarQube Server
     │
     ▼
PostgreSQL
```

The scanner collects project metadata, uploads source code metrics, and waits for analysis results.

---

# Scanner Configuration

Typical Maven command:

```bash
mvn clean verify sonar:sonar
```

Typical parameters include:

```text
sonar.projectKey
sonar.projectName
sonar.host.url
sonar.login
```

These values are usually injected by Jenkins credentials or pipeline variables.

---

# Jenkins Integration

Jenkins integrates with SonarQube using the SonarQube Scanner plugin.

Typical pipeline stages include:

1. Checkout source code
2. Compile application
3. Execute unit tests
4. Run Sonar Scanner
5. Wait for Quality Gate
6. Continue deployment only if Quality Gate passes

Example Jenkins Pipeline:

```groovy
stage('SonarQube Analysis') {
    steps {
        withSonarQubeEnv('SonarQube') {
            sh 'mvn clean verify sonar:sonar'
        }
    }
}

stage('Quality Gate') {
    steps {
        timeout(time: 10, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
        }
    }
}
```

This integration ensures code quality is automatically enforced.

---

# Maven Integration

Maven projects use the Sonar Maven Plugin.

Typical execution:

```bash
mvn clean verify sonar:sonar
```

The plugin automatically:

- Detects Java source files
- Uploads compiled bytecode
- Calculates code coverage
- Sends metrics to SonarQube

No manual file uploads are required.

---

# Branch Analysis

Enterprise editions support independent analysis for each Git branch.

Benefits include:

- Compare feature branches
- Validate release branches
- Analyze long-lived branches
- Prevent regressions

Although Community Edition focuses on the main branch, the platform architecture is compatible with future upgrades.

---

# Pull Request Analysis

Enterprise editions also provide Pull Request decoration.

Capabilities include:

- Inline code review
- Changed file analysis
- New code metrics
- Pull Request Quality Gates

This helps reviewers identify quality issues before merging code.

---

# Security Hotspots

Security Hotspots highlight code requiring manual security review.

Examples include:

- Password management
- Encryption
- Authentication
- Session handling
- HTTP security
- File handling
- Input validation

Unlike vulnerabilities, Security Hotspots require developer review to determine whether a genuine risk exists.

---

# Technical Debt

Technical Debt estimates the effort required to remediate all identified issues.

The estimate is calculated automatically based on:

- Number of issues
- Severity
- Estimated remediation time

This metric helps teams prioritize improvement efforts.

---

# Dashboard and Reporting

The SonarQube dashboard provides a centralized view of project quality.

Key dashboard components include:

- Overall Project Status
- Quality Gate Result
- Code Coverage
- Bugs
- Vulnerabilities
- Security Hotspots
- Technical Debt
- Duplicated Code
- Maintainability Rating
- Reliability Rating
- Security Rating
- Historical Trends

These dashboards enable engineering teams to monitor quality improvements over time.

---

# Continuous Inspection

SonarQube supports Continuous Inspection by analyzing every code change.

Benefits include:

- Immediate developer feedback
- Reduced production defects
- Lower technical debt
- Improved maintainability
- Better security posture
- Automated quality enforcement

Continuous Inspection complements Continuous Integration and Continuous Delivery, completing the DevSecOps feedback loop.

---

# Summary

The SonarQube Analysis Engine forms the quality and security foundation of the Enterprise DevSecOps Infrastructure Platform.

Through Quality Gates, Quality Profiles, the Rules Engine, Sonar Scanner, Jenkins integration, and Maven automation, SonarQube provides continuous inspection of source code while ensuring that only high-quality, secure software progresses through the deployment pipeline.

The next section covers operational verification, health checks, backup and restore procedures, troubleshooting, security best practices, operational guidance, and the relationship between SonarQube and the companion automation deployment project.

---

# Verification

After the platform has been deployed, verify that SonarQube is functioning correctly before integrating it into Jenkins pipelines.

## Verify Container Status

Ensure the SonarQube and PostgreSQL containers are running.

```bash
docker ps
```

Expected containers:

- sonarqube
- sonar-postgres

---

## Verify Docker Compose Services

```bash
docker compose ps
```

Expected status:

```
sonarqube         Up
sonar-postgres    Up (healthy)
```

---

## Verify SonarQube Logs

Review the startup logs.

```bash
docker logs sonarqube
```

A successful startup ends with messages similar to:

```
SonarQube is operational
Web Server is started
```

---

## Verify PostgreSQL Logs

```bash
docker logs sonar-postgres
```

Confirm that:

- Database initialization completed
- No authentication failures are present
- No connection errors exist

---

## Verify Web Interface

Open:

```
http://localhost:9000
```

Expected result:

- SonarQube login page loads successfully.

---

## Verify Database Connectivity

Enter the SonarQube container.

```bash
docker exec -it sonarqube bash
```

Confirm the configured database endpoint.

```bash
echo $SONAR_JDBC_URL
```

Expected output:

```
jdbc:postgresql://sonar-postgres:5432/sonarqube
```

---

## Verify Jenkins Connectivity

From the Jenkins container:

```bash
docker exec jenkins curl http://sonarqube:9000
```

A successful HTTP response confirms internal Docker network connectivity.

---

## Verify Scanner Integration

Execute a Maven analysis.

```bash
mvn clean verify sonar:sonar
```

Expected outcome:

- Analysis completes successfully.
- Results appear in the SonarQube dashboard.

---

# Health Checks

Routine health verification is recommended before pipeline execution.

Checklist:

- SonarQube container running
- PostgreSQL healthy
- Docker network available
- Volumes mounted
- Dashboard accessible
- Jenkins connectivity verified
- Scanner authentication valid
- Quality Gate operational

---

# Operational Maintenance

Regular maintenance ensures consistent performance and reliability.

Recommended tasks:

- Upgrade SonarQube LTS
- Upgrade PostgreSQL
- Update plugins
- Review Quality Profiles
- Clean obsolete projects
- Monitor disk utilization
- Archive historical analyses
- Monitor JVM memory usage
- Monitor database growth
- Review authentication settings

---

# Backup Strategy

Persistent data is stored within Docker volumes.

| Volume | Purpose |
|---------|----------|
| sonarqube_data | Elasticsearch indexes and application data |
| sonarqube_logs | Application logs |
| sonarqube_extensions | Installed plugins |
| sonar_postgres_data | PostgreSQL database |

These volumes should be included in routine backups.

Recommended backup schedule:

- Development: Weekly
- Production: Daily

---

# Restore Strategy

To restore SonarQube:

1. Stop Docker Compose.

```bash
docker compose down
```

2. Restore the Docker volumes.

3. Start the platform.

```bash
docker compose up -d
```

4. Verify:

- Dashboard accessibility
- Database connectivity
- Historical projects
- Quality Profiles
- Quality Gates

---

# Performance Optimization

For larger projects, consider tuning:

## JVM Heap

Increase Java heap memory for the SonarQube server.

Example:

```text
SONAR_WEB_JAVAOPTS
SONAR_CE_JAVAOPTS
```

---

## Database

Optimize PostgreSQL by:

- Routine vacuum operations
- Index maintenance
- Disk monitoring

---

## Analysis Cache

Maintain adequate disk space for analysis indexes.

---

## Docker Resources

Allocate sufficient:

- CPU
- Memory
- Storage

to avoid slow analyses.

---

# Troubleshooting

## SonarQube Does Not Start

Check logs:

```bash
docker logs sonarqube
```

Possible causes:

- Database unavailable
- Port conflict
- Insufficient memory
- Corrupted volume

---

## PostgreSQL Connection Failure

Verify:

```bash
docker logs sonar-postgres
```

Confirm:

- Database credentials
- JDBC URL
- Docker network
- Container health

---

## Dashboard Not Accessible

Verify:

```bash
docker ps
```

Ensure:

- Port 9000 exposed
- Firewall permits access
- Container is running

---

## Jenkins Cannot Reach SonarQube

Check Docker networking.

```bash
docker network inspect cicd-network
```

Verify:

- Jenkins and SonarQube belong to the same network.

---

## Scanner Authentication Failure

Verify:

- Sonar token
- Jenkins credentials
- Project key
- Scanner configuration

---

## Quality Gate Timeout

Possible causes:

- Large analysis
- Busy Compute Engine
- Database delays

Recommended actions:

- Increase Jenkins timeout.
- Monitor Compute Engine tasks.

---

## Plugin Compatibility Issues

Use plugins compatible with the installed LTS release.

After plugin updates:

```bash
docker restart sonarqube
```

---

## Elasticsearch Startup Failure

Common causes:

- Low memory
- Corrupted indexes
- Disk full

Review:

```bash
docker logs sonarqube
```

---

# Security Best Practices

To improve platform security:

- Use generated authentication tokens instead of passwords.
- Rotate tokens periodically.
- Restrict administrative access.
- Enable HTTPS behind a reverse proxy.
- Keep SonarQube LTS updated.
- Keep PostgreSQL updated.
- Remove unused projects.
- Audit user permissions regularly.
- Back up analysis data.
- Restrict Docker host access.

---

# Operational Best Practices

Recommended operational guidelines:

- Analyze every commit.
- Enforce Quality Gates in Jenkins.
- Maintain language-specific Quality Profiles.
- Monitor Technical Debt trends.
- Review Security Hotspots promptly.
- Archive inactive projects.
- Keep plugins up to date.
- Version-control scanner configuration.
- Document Quality Gate changes.
- Periodically review project permissions.

---

# Integration with Jenkins

Within this platform, Jenkins uses SonarQube during every CI/CD execution.

Typical pipeline sequence:

```
GitHub
    │
    ▼
Jenkins Checkout
    │
    ▼
Maven Build
    │
    ▼
Sonar Scanner
    │
    ▼
SonarQube Analysis
    │
    ▼
Quality Gate
    │
 ┌──────┴──────┐
 │             │
Pass         Fail
 │             │
 ▼             ▼
Deploy      Stop Pipeline
```

This integration ensures that quality standards are automatically enforced before application deployment.

---

# Related Project

This infrastructure repository provides the SonarQube platform used by the companion repository:

**automation-deployment-project**

The companion repository demonstrates:

- Maven builds
- Sonar Scanner execution
- Jenkins pipeline integration
- Quality Gate validation
- Trivy security scanning
- Docker image creation
- Nexus artifact publishing
- Kubernetes deployment

Together, the repositories provide a complete Enterprise DevSecOps implementation.

---

# Summary

SonarQube is the centralized software quality and static analysis platform within the Enterprise DevSecOps Infrastructure Platform.

Integrated with Jenkins, Maven, PostgreSQL, and Docker Compose, it enables continuous inspection of source code while enforcing quality standards through automated Quality Gates.

Key capabilities include:

- Static Application Security Testing (SAST)
- Code quality analysis
- Security vulnerability detection
- Security Hotspot identification
- Technical Debt tracking
- Code coverage reporting
- Quality Gate enforcement
- PostgreSQL-backed persistence
- Jenkins CI/CD integration
- Maven scanner automation
- Docker Compose deployment
- Infrastructure as Code

By embedding SonarQube directly into the CI/CD pipeline, the platform promotes secure, maintainable, and production-ready software delivery while demonstrating modern Enterprise DevSecOps practices.

---

## Next Document

Continue with:

**07_Nexus.md**

The next guide will cover:

- Nexus Repository Architecture
- Repository Types
- Docker Registry
- Maven Hosted Repository
- Blob Storage
- Docker Compose Deployment
- Jenkins Integration
- Artifact Lifecycle
- Backup & Restore
- Security Best Practices
- Enterprise Operations
