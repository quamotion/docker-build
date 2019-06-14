FROM ubuntu:bionic

ARG docker_version=18.06
ARG node_version=10.x
ARG helm_version=v2.13.0
ARG yq_version=2.2.1

ENV DEBIAN_FRONTEND=noninteractive

WORKDIR /

RUN apt-get update \
&& apt-get upgrade -y \
&& apt-get install -y software-properties-common apt-transport-https curl \

## WebDriver dependencies for which we don't ship any native files as part
## of the NuGet packages
&& apt-get install -y libavcodec-dev libavformat-dev libavutil-dev libswscale-dev libturbojpeg0-dev liblz4-dev \

## Install jq, required by the vsts-agent
&& apt-get install -y jq \

## Install Docker
&& curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add - \
&& add-apt-repository "deb https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") $(lsb_release -cs) stable" \
&& apt-get update \
&& apt-get install -y docker-ce=$(apt-cache madison docker-ce | grep $docker_version | head -1 | awk '{print $3}') \

## GitLab runner
## Use the Ubuntu-provided version of the GitLab runner, as GitLab itself doesn't
## have any ARM64 runners (yet)
&& apt-get install -y gitlab-runner \

## wget
&& apt-get install -y wget \

## .NET Core (2.2)
&& apt-get install -y curl apt-transport-https libc6-dev libgdiplus libpng-dev libunwind8 \
&& mkdir -p /usr/local/dotnet \
&& wget -O dotnet-sdk-2.2.300.tar.gz https://download.visualstudio.microsoft.com/download/pr/9605c24e-9bb2-499f-a003-4fb2bddcf09c/a8683ff89405d370961beb1909ddc295/dotnet-sdk-2.2.300-linux-arm64.tar.gz \
&& tar zxf dotnet-sdk-2.2.300.tar.gz -C /usr/local/dotnet \
&& rm dotnet-sdk-2.2.300.tar.gz \

## .NET Core (3.0 preview)
&& wget -O dotnet-sdk-3.0.100-preview5.tar.gz https://download.visualstudio.microsoft.com/download/pr/fa62ce11-f5d3-4f71-a5dd-07edf988edae/91cfccdeb4a58583f49c4a12dbd6f329/dotnet-sdk-3.0.100-preview5-011568-linux-arm64.tar.gz \
&& tar zxf dotnet-sdk-3.0.100-preview5.tar.gz -C /usr/local/dotnet \
&& ln -s /usr/local/dotnet/dotnet /usr/local/bin/dotnet \
&& rm dotnet-sdk-3.0.100-preview5.tar.gz \

## trx2junit to send trx files to GitLab
&& dotnet tool install -g trx2junit \

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

## eslint
&& npm install -g eslint eslint-plugin-react babel-eslint eslint-config-defaults \

# Install Powershell Core
&& wget -O powershell.tar.gz https://github.com/PowerShell/PowerShell/releases/download/v6.2.0-rc.1/powershell-6.2.0-rc.1-linux-arm64.tar.gz \
&& mkdir -p /usr/local/powershell \
&& tar zxf powershell.tar.gz -C /usr/local/powershell \
&& ln -s /usr/local/powershell/pwsh /usr/local/bin/pwsh \
&& rm powershell.tar.gz \

## Install PowerShell modules
&& pwsh -c "Set-PSRepository -Name PSGallery -InstallationPolicy Trusted" \
&& pwsh -c "Install-Module -Name powershell-yaml" \
&& pwsh -c "Install-Module -Name Pester" \

## Configure AzCLI 2.0  Instructions taken from https://docs.microsoft.com/en-us/cli/azure/install-azure-cli
## There are no ARM64 builds yet

## Kubectl
&& curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - \
&& echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | tee /etc/apt/sources.list.d/kubernetes.list \
&& apt-get update \
&& apt-get install -y kubectl \

## Helm
&& curl -sL https://kubernetes-helm.storage.googleapis.com/helm-$helm_version-linux-arm64.tar.gz -o helm-$helm_version-linux-arm64.tar.gz \
&& mkdir ~/helm \
&& tar xf helm-$helm_version-linux-arm64.tar.gz -C ~/helm/ \
&& mv ~/helm/linux-arm64/helm /usr/local/bin/helm \
&& rm -rf ~/helm \
&& helm init --client-only \
&& helm plugin install https://github.com/nouney/helm-gcs \

## Ansible
&& apt-add-repository -y ppa:ansible/ansible \
&& apt-get update \
&& apt-get install -y ansible \

## yq
&& curl -sL https://github.com/mikefarah/yq/releases/download/$yq_version/yq_linux_arm64 -o yq \
&& chmod +x yq \
&& mv yq /usr/local/bin \

## Google Cloud SDK
&& curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - \
&& echo "deb http://packages.cloud.google.com/apt cloud-sdk-bionic main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list \
&& apt-get update \
&& apt-get install -y google-cloud-sdk \

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
