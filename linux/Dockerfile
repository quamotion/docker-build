FROM microsoft/vsts-agent:ubuntu-16.04-docker-17.06.0-ce

RUN apt-get update \
&& apt-get upgrade -y \

## .NET Core
&& apt-get install -y curl apt-transport-https libc6-dev libgdiplus \
&& curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > /etc/apt/trusted.gpg.d/microsoft.gpg \
&& echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-ubuntu-xenial-prod xenial main" > /etc/apt/sources.list.d/dotnetdev.list \
&& apt-get update \
&& apt-get install -y dotnet-sdk-2.1.4 \

## node.js 8 (LTS)
&& curl -sL https://deb.nodesource.com/setup_8.x -o setup-nodejs \
&& /bin/bash setup-nodejs \
&& rm setup-nodejs \
&& apt-get install -y nodejs \

## Bower & Gulp
&& npm install -g bower \
&& npm install -g gulp \

## Enable running bower with admini privileges
&& echo '{ "allow_root": true }' > /root/.bowerrc \

# Install Powershell Core
&& curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - \
&& curl https://packages.microsoft.com/config/ubuntu/16.04/prod.list | tee /etc/apt/sources.list.d/microsoft.list \
&& apt-get update \
&& apt-get install -y powershell \

## Install PowerShell modules
&& pwsh -c "Set-PSRepository -Name PSGallery -InstallationPolicy Trusted" \
&& pwsh -c "Install-Module -Name powershell-yaml" \

# Configure AzCLI 2.0  Instructions taken from https://docs.microsoft.com/en-us/cli/azure/install-azure-cli
&& echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ wheezy main" | tee /etc/apt/sources.list.d/azure-cli.list \
&& apt-key adv --keyserver packages.microsoft.com --recv-keys 417A0893 \
&& apt-get update \
&& apt-get install -y --no-install-recommends azure-cli \

## Latest .NET Core 2.1 builds
&& curl -sL https://download.microsoft.com/download/D/7/8/D788D3CD-44C4-487D-829B-413E914FB1C3/dotnet-sdk-2.1.300-preview1-008174-linux-x64.tar.gz -o dotnet-sdk-latest-linux-x64.tar.gz \
&& mkdir ~/dotnet-latest/ \
&& tar xf dotnet-sdk-latest-linux-x64.tar.gz -C ~/dotnet-latest/ \
&& rm dotnet-sdk-latest-linux-x64.tar.gz \

## Kubectl
&& curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - \
&& echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | tee /etc/apt/sources.list.d/kubernetes.list \
&& apt-get update \
&& apt-get install -y kubectl \

## Helm
&& curl -sL https://kubernetes-helm.storage.googleapis.com/helm-v2.7.2-linux-amd64.tar.gz -o helm-v2.7.2-linux-amd64.tar.gz \
&& mkdir ~/helm \
&& tar xf helm-v2.7.2-linux-amd64.tar.gz -C ~/helm/ \
&& mv ~/helm/linux-amd64/helm /usr/local/bin/helm \
&& rm -rf ~/helm \
&& helm init --client-only \

## Ansible
&& apt-add-repository -y ppa:ansible/ansible \
&& apt-get update \
&& apt-get install -y ansible \

## yq
&& curl -sL https://github.com/mikefarah/yq/releases/download/1.14.0/yq_linux_amd64 -o yq \
&& chmod +x yq \
&& mv yq /usr/local/bin \

## wget
&& apt-get install -y wget \

## Prime the .NET Core cache
&& mkdir ~/prime \
&& cd ~/prime \
&& dotnet new webapi \
&& dotnet restore \
&& rm -rf ~/prime \

&& mkdir ~/prime \
&& cd ~/prime \
&& ~/dotnet-latest/dotnet new webapi \
&& ~/dotnet-latest/dotnet restore \
&& rm -rf ~/prime \

## cleanup
&& rm -rf /var/lib/apt/lists/*

ENV agent.disableupdate=true
