FROM ubuntu:bionic

ARG docker_version=18.06
ARG dotnet_version=2.1
ARG node_version=8.x
ARG helm_version=v2.11.0
ARG yq_version=1.15.0

ENV DEBIAN_FRONTEND=noninteractive

## This image contains both VSTS and GitLab runners, and provides no default entrypoint.
## You need to start the VSTS or GitLab runner manually when staring a container based on this image.

## VSTS runner
## see https://github.com/Microsoft/vsts-agent-docker/blob/master/ubuntu/16.04/Dockerfile

# Accept the TEE EULA
RUN mkdir -p "/root/.microsoft/Team Foundation/4.0/Configuration/TEE-Mementos" \
 && cd "/root/.microsoft/Team Foundation/4.0/Configuration/TEE-Mementos" \
 && echo '<ProductIdData><eula-14.0 value="true"/></ProductIdData>' > "com.microsoft.tfs.client.productid.xml"

WORKDIR /vsts

COPY ./start-vsts.sh ./start.sh
RUN chmod +x start.sh

WORKDIR /

RUN apt-get update \
&& apt-get upgrade -y \
&& apt-get install -y software-properties-common apt-transport-https curl \

# Install jq, required by the vsts-agent
&& apt-get install -y jq \

# Install Docker
&& curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add - \
&& add-apt-repository "deb https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") $(lsb_release -cs) stable" \
&& apt-get update \
&& apt-get install -y docker-ce=$(apt-cache madison docker-ce | grep $docker_version | head -1 | awk '{print $3}') \

## GitLab runner
## see https://docs.gitlab.com/runner/install/linux-repository.html
&& curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | bash \
&& apt-get install -y gitlab-runner \

## .NET Core
&& apt-get install -y curl apt-transport-https libc6-dev libgdiplus libpng-dev \
&& curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > /etc/apt/trusted.gpg.d/microsoft.gpg \
&& echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-ubuntu-bionic-prod bionic main" > /etc/apt/sources.list.d/dotnetdev.list \
&& apt-get update \
&& apt-get install -y dotnet-sdk-$dotnet_version \

## node.js
&& curl -sL https://deb.nodesource.com/setup_$node_version -o setup-nodejs \
&& /bin/bash setup-nodejs \
&& rm setup-nodejs \
&& apt-get install -y nodejs \

## Bower & Gulp
&& npm install -g bower \
&& npm install -g gulp \

## Enable running bower with admin privileges
&& echo '{ "allow_root": true }' > /root/.bowerrc \

# Install Powershell Core
&& curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - \
&& curl https://packages.microsoft.com/config/ubuntu/18.04/prod.list | tee /etc/apt/sources.list.d/microsoft.list \
&& apt-get update \
&& apt-get install -y powershell \

## Install PowerShell modules
&& pwsh -c "Set-PSRepository -Name PSGallery -InstallationPolicy Trusted" \
&& pwsh -c "Install-Module -Name powershell-yaml" \
&& pwsh -c "Install-Module -Name Pester" \

# Configure AzCLI 2.0  Instructions taken from https://docs.microsoft.com/en-us/cli/azure/install-azure-cli
&& echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ bionic main" | tee /etc/apt/sources.list.d/azure-cli.list \
&& apt-key adv --keyserver packages.microsoft.com --recv-keys 417A0893 \
&& apt-get update \
&& apt-get install -y --no-install-recommends azure-cli \

## Kubectl
&& curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - \
&& echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | tee /etc/apt/sources.list.d/kubernetes.list \
&& apt-get update \
&& apt-get install -y kubectl \

## Helm
&& curl -sL https://kubernetes-helm.storage.googleapis.com/helm-$helm_version-linux-amd64.tar.gz -o helm-$helm_version-linux-amd64.tar.gz \
&& mkdir ~/helm \
&& tar xf helm-$helm_version-linux-amd64.tar.gz -C ~/helm/ \
&& mv ~/helm/linux-amd64/helm /usr/local/bin/helm \
&& rm -rf ~/helm \
&& helm init --client-only \

## Ansible
&& apt-add-repository -y ppa:ansible/ansible \
&& apt-get update \
&& apt-get install -y ansible \

## yq
&& curl -sL https://github.com/mikefarah/yq/releases/download/$yq_version/yq_linux_amd64 -o yq \
&& chmod +x yq \
&& mv yq /usr/local/bin \

## Google Cloud SDK
&& curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - \
&& echo "deb http://packages.cloud.google.com/apt cloud-sdk-bionic main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list \
&& apt-get update \
&& apt-get install -y google-cloud-sdk \

## wget
&& apt-get install -y wget \

## nuget, tzdata
&& apt-get install -y nuget tzdata \

## Prime the .NET Core cache
&& mkdir ~/prime \
&& cd ~/prime \
&& dotnet new webapi \
&& dotnet restore \
&& rm -rf ~/prime \

## cleanup
&& rm -rf /var/lib/apt/lists/*
