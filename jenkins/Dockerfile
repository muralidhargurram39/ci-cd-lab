FROM jenkins/jenkins:lts-jdk17

USER root

# -----------------------------------------------------------------------------
# Environment
# -----------------------------------------------------------------------------

ENV DEBIAN_FRONTEND=noninteractive
ENV DOCKER_BUILDKIT=1

# -----------------------------------------------------------------------------
# Tool Versions
# -----------------------------------------------------------------------------

ARG NODE_VERSION=22
ARG KUBECTL_VERSION=v1.34.1
ARG KIND_VERSION=v0.32.0
ARG HELM_VERSION=v3.19.0
ARG DOCKER_COMPOSE_VERSION=v2.39.1
ARG DOCKER_BUILDX_VERSION=v0.28.0
ARG DOCKER_CLI_VERSION=28.3.2
ARG AWSCLI_VERSION=2.34.7
ARG YQ_VERSION=v4.47.2
ARG DOCKER_GID=1001

# -----------------------------------------------------------------------------
# Base Packages
# -----------------------------------------------------------------------------

RUN apt-get update && apt-get install -y \
    ca-certificates \
    curl \
    wget \
    git \
    unzip \
    zip \
    jq \
    file \
    gnupg \
    python3 \
    python3-pip \
    maven \
    && rm -rf /var/lib/apt/lists/*

# -----------------------------------------------------------------------------
# NodeJS
# -----------------------------------------------------------------------------

RUN curl -fsSL https://deb.nodesource.com/setup_${NODE_VERSION}.x | bash - && \
    apt-get update && \
    apt-get install -y nodejs && \
    rm -rf /var/lib/apt/lists/*

# -----------------------------------------------------------------------------
# Docker CLI
# -----------------------------------------------------------------------------

RUN curl -fsSL \
    https://download.docker.com/linux/static/stable/x86_64/docker-${DOCKER_CLI_VERSION}.tgz \
    -o docker.tgz && \
    tar -xzf docker.tgz && \
    mv docker/docker /usr/local/bin/ && \
    rm -rf docker docker.tgz

# -----------------------------------------------------------------------------
# Docker Compose
# -----------------------------------------------------------------------------

RUN mkdir -p /usr/local/lib/docker/cli-plugins && \
    curl -fsSL \
    https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-linux-x86_64 \
    -o /usr/local/lib/docker/cli-plugins/docker-compose && \
    chmod +x /usr/local/lib/docker/cli-plugins/docker-compose

# -----------------------------------------------------------------------------
# Docker Buildx
# -----------------------------------------------------------------------------

RUN curl -fsSL \
    https://github.com/docker/buildx/releases/download/${DOCKER_BUILDX_VERSION}/buildx-${DOCKER_BUILDX_VERSION}.linux-amd64 \
    -o /usr/local/lib/docker/cli-plugins/docker-buildx && \
    chmod +x /usr/local/lib/docker/cli-plugins/docker-buildx

# -----------------------------------------------------------------------------
# kubectl
# -----------------------------------------------------------------------------

RUN curl -fsSL \
    https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl \
    -o /usr/local/bin/kubectl && \
    chmod +x /usr/local/bin/kubectl

# -----------------------------------------------------------------------------
# Kind
# -----------------------------------------------------------------------------

RUN curl -fsSL \
    https://kind.sigs.k8s.io/dl/${KIND_VERSION}/kind-linux-amd64 \
    -o /usr/local/bin/kind && \
    chmod +x /usr/local/bin/kind && \
    kind version

# -----------------------------------------------------------------------------
# Helm
# -----------------------------------------------------------------------------

RUN curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# -----------------------------------------------------------------------------
# AWS CLI
# -----------------------------------------------------------------------------

RUN curl -fsSL \
    https://awscli.amazonaws.com/awscli-exe-linux-x86_64-${AWSCLI_VERSION}.zip \
    -o awscliv2.zip && \
    unzip awscliv2.zip && \
    ./aws/install && \
    rm -rf aws awscliv2.zip

# -----------------------------------------------------------------------------
# Trivy
# -----------------------------------------------------------------------------

RUN wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | \
    gpg --dearmor -o /usr/share/keyrings/trivy.gpg && \
    echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb generic main" \
    > /etc/apt/sources.list.d/trivy.list && \
    apt-get update && \
    apt-get install -y trivy && \
    rm -rf /var/lib/apt/lists/*

# -----------------------------------------------------------------------------
# yq
# -----------------------------------------------------------------------------

RUN wget -qO /usr/local/bin/yq \
    https://github.com/mikefarah/yq/releases/download/${YQ_VERSION}/yq_linux_amd64 && \
    chmod +x /usr/local/bin/yq

# -----------------------------------------------------------------------------
# Docker Group
# -----------------------------------------------------------------------------

RUN groupadd -g ${DOCKER_GID} docker || true && \
    usermod -aG docker jenkins

# -----------------------------------------------------------------------------
# Tool Versions
# -----------------------------------------------------------------------------

RUN echo && \
    echo "================ Installed Tools ================" && \
    java -version && \
    mvn -version && \
    docker --version && \
    docker compose version && \
    docker buildx version && \
    kubectl version --client && \
    kind version && \
    helm version && \
    aws --version && \
    trivy --version && \
    yq --version && \
    node --version && \
    npm --version

USER jenkins
