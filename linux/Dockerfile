FROM ubuntu:bionic

ARG docker_version=18.06
ARG dotnet_version=2.2
ARG dotnet_package_version=2.2.402-1 
ARG node_version=v10.16.0
ARG helm_version=v2.14.1
ARG yq_version=2.4.0

ENV DEBIAN_FRONTEND=noninteractive

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

## .NET Core
&& apt-get install -y curl apt-transport-https libc6-dev libgdiplus libpng-dev \
&& curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > /etc/apt/trusted.gpg.d/microsoft.gpg \
&& echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-ubuntu-bionic-prod bionic main" > /etc/apt/sources.list.d/dotnetdev.list \
&& apt-get update \
&& apt-get install -y dotnet-sdk-$dotnet_version=${dotnet_package_version} \

## Install the nbgv tool
&& dotnet tool install --tool-path /usr/local/share/dotnet-tools nbgv \
&& ln -s /usr/local/share/dotnet-tools/nbgv /usr/local/bin/nbgv \

## node.js
&& curl -sL https://nodejs.org/dist/${node_version}/node-${node_version}-linux-x64.tar.gz  -o node-${node_version}-linux-x64.tar.gz \
&& tar --strip-components 1 -xzvf node-${node_version}-linux-x64.tar.gz -C /usr/local \
&& rm node-${node_version}-linux-x64.tar.gz \

## Bower & Gulp
&& npm install -g bower \
&& npm install -g gulp \

## Enable running bower with admin privileges
&& echo '{ "allow_root": true }' > /root/.bowerrc \

## eslint
&& npm install -g eslint eslint-plugin-react babel-eslint eslint-config-defaults \

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
&& helm plugin install https://github.com/nouney/helm-gcs \

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
