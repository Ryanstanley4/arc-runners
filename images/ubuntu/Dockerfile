# Base image: Ubuntu 24.04
FROM ubuntu:24.04

# Metadata
LABEL maintainer="ryanstanley4"
LABEL org.opencontainers.image.source = "https://github.com/Ryanstanley4/arc-runners"

# Set environment variables for runner
ENV RUNNER_VERSION=2.316.1
ENV RUNNER_USER=runner

# Install base tools
RUN apt-get update && apt-get install -y \
    curl \
    wget \
    git \
    unzip \
    ca-certificates \
    jq \
    tar \
    gzip \
    lsb-release \
    software-properties-common \
    apt-transport-https \
    gnupg \
    build-essential \
    libssl-dev

# Install kubectl
RUN curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | gpg --dearmor -o /etc/apt/trusted.gpg.d/kubernetes.gpg && \
    echo "deb https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | tee /etc/apt/sources.list.d/kubernetes.list && \
    apt-get update && \
    apt-get install -y kubectl

# Install helm
RUN curl https://baltocdn.com/helm/signing.asc | gpg --dearmor -o /usr/share/keyrings/helm.gpg && \
    echo "deb [signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" \
    | tee /etc/apt/sources.list.d/helm-stable-debian.list && \
    apt-get update && \
    apt-get install -y helm

# Install podman
RUN . /etc/os-release && \
    apt-get install -y podman

# Install golang (1.22.x)
ENV GO_VERSION=1.22.4
RUN curl -LO https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz && \
    tar -C /usr/local -xzf go${GO_VERSION}.linux-amd64.tar.gz && \
    rm go${GO_VERSION}.linux-amd64.tar.gz
ENV PATH="/usr/local/go/bin:${PATH}"

# Create runner user
RUN useradd -m ${RUNNER_USER}

# Download and install GitHub Actions Runner
WORKDIR /home/${RUNNER_USER}
RUN curl -L https://github.com/actions/runner/releases/download/v${RUNNER_VERSION}/actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz -o actions-runner.tar.gz && \
    tar xzf actions-runner.tar.gz && \
    rm actions-runner.tar.gz && \
    chown -R ${RUNNER_USER}:${RUNNER_USER} /home/${RUNNER_USER}

# Setup permissions
USER ${RUNNER_USER}

# Entrypoint
ENTRYPOINT ["/home/runner/run.sh"]
