# Base image: Ubuntu 22.04
FROM ubuntu:24.04

# Metadata
LABEL maintainer="ryanstanley4"
LABEL org.opencontainers.image.source="https://github.com/Ryanstanley4/arc-runners"

# Set environment variables for runner
ENV RUNNER_VERSION=2.331.0
ENV RUNNER_USER=runner

# Upgrade all packages for security and latest updates
RUN apt-get upgrade -y && apt-get dist-upgrade -y && apt-get autoremove -y && apt-get clean

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
    libssl-dev \
    gpg

# Install kubectl
ENV KUBECTL_VERSION=v1.35
RUN curl -fsSL https://pkgs.k8s.io/core:/stable:/${KUBECTL_VERSION}/deb/Release.key | gpg --dearmor -o /etc/apt/trusted.gpg.d/kubernetes.gpg && \
    echo "deb https://pkgs.k8s.io/core:/stable:/${KUBECTL_VERSION}/deb/ /" | tee /etc/apt/sources.list.d/kubernetes.list && \
    apt-get update && \
    apt-get install -y kubectl

# Install helm
RUN curl -fsSL https://packages.buildkite.com/helm-linux/helm-debian/gpgkey | gpg --dearmor | tee /usr/share/keyrings/helm.gpg > /dev/null && \
    echo "deb [signed-by=/usr/share/keyrings/helm.gpg] https://packages.buildkite.com/helm-linux/helm-debian/any/ any main" \
    | tee /etc/apt/sources.list.d/helm-stable-debian.list && \
    apt-get update && \
    apt-get install -y helm

# Install podman
RUN . /etc/os-release && \
    apt-get install -y podman

# Install golang (1.26.x)
ENV GO_VERSION=1.26.0
RUN curl -LO https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz && \
    tar -C /usr/local -xzf go${GO_VERSION}.linux-amd64.tar.gz && \
    rm go${GO_VERSION}.linux-amd64.tar.gz
ENV PATH="/usr/local/go/bin:${PATH}"

# Install GitHub CLI
RUN type -p curl >/dev/null || apt-get install -y curl && \
    curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg && \
    chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | tee /etc/apt/sources.list.d/github-cli.list > /dev/null && \
    apt-get update && \
    apt-get install -y gh

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