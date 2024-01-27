
#-------------------------------------------------------------------------------------------------------------
# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License. See https://go.microsoft.com/fwlink/?linkid=2090316 for license information.
#-------------------------------------------------------------------------------------------------------------

# Note: You can use any Debian/Ubuntu based image you want. Using the microsoft base Ubuntu image.
FROM mcr.microsoft.com/vscode/devcontainers/base:focal

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# Docker Compose version
ARG COMPOSE_VERSION=1.24.0

# Helm Version
ARG HELM_VERSION=3.6.3

# Configure apt and install packages
RUN apt-get update \
    && apt-get -y install --no-install-recommends apt-utils dialog 2>&1 \
    && apt-get -y install git iproute2 procps bash-completion

# Install Docker CE CLI
RUN apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common lsb-release \
    && curl -fsSL https://download.docker.com/linux/$(lsb_release -is | tr '[:upper:]' '[:lower:]')/gpg | (OUT=$(apt-key add - 2>&1) || echo $OUT) \
    && add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/$(lsb_release -is | tr '[:upper:]' '[:lower:]') $(lsb_release -cs) stable" \
    && apt-get update \
    && apt-get install -y docker-ce-cli

# Install Docker Compose
RUN curl -sSL "https://github.com/docker/compose/releases/download/${COMPOSE_VERSION}/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose \
    && chmod +x /usr/local/bin/docker-compose

# Install the Azure CLI && aks-preview extension
RUN apt-get install -y apt-transport-https gnupg2 lsb-release \
    && echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $(lsb_release -cs) main" > /etc/apt/sources.list.d/azure-cli.list \
    && curl -sL https://packages.microsoft.com/keys/microsoft.asc | apt-key add - 2>/dev/null \
    && apt-get update \
    && apt-get install -y azure-cli \
    && az extension add -n aks-preview

# Install Kubectl
RUN echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list \
    && curl -sL https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - 2>/dev/null \
    && apt-get update \
    && apt-get install -y kubectl

# Install Helm (currently v3.0.2)
RUN mkdir -p /tmp/downloads/helm \
    && curl -sL -o /tmp/downloads/helm.tar.gz https://get.helm.sh/helm-v${HELM_VERSION}-linux-amd64.tar.gz \
    && tar -C /tmp/downloads/helm -zxvf /tmp/downloads/helm.tar.gz \
    && mv /tmp/downloads/helm/linux-amd64/helm /usr/local/bin

# Make kubectl completions work with 'k' alias
RUN echo 'alias k=kubectl' >> "/root/.zshrc" \
    && echo 'complete -F __start_kubectl k' >> "/root/.zshrc" \
    && echo "[[ $commands[kubectl] ]] && source <(kubectl completion zsh)" >> "/root/.zshrc"

# Install Sqlcmd
RUN echo "Installing mssql-tools" \
    && curl -sSL https://packages.microsoft.com/keys/microsoft.asc | (OUT=$(apt-key add - 2>&1) || echo $OUT) \
    && DISTRO=$(lsb_release -is | tr '[:upper:]' '[:lower:]') \
    && CODENAME=$(lsb_release -cs) \
    && echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-${DISTRO}-${CODENAME}-prod ${CODENAME} main" > /etc/apt/sources.list.d/microsoft.list \
    && apt-get update \
    && ACCEPT_EULA=Y apt-get -y install unixodbc-dev msodbcsql17 libunwind8 mssql-tools

RUN echo "Installing sqlpackage" \
    && curl -sSL -o sqlpackage.zip "https://aka.ms/sqlpackage-linux" \
    && mkdir /opt/sqlpackage \
    && unzip sqlpackage.zip -d /opt/sqlpackage \
    && rm sqlpackage.zip \
    && chmod a+x /opt/sqlpackage/sqlpackage \
    && ln -sfn /opt/mssql-tools/bin/sqlcmd /usr/bin/sqlcmd

# Clean up
RUN apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/downloads

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=dialog