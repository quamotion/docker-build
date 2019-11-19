FROM ubuntu:bionic

ARG docker_version=18.06
ARG node_version=v10.16.0
ARG helm_version=v2.14.1
ARG yq_version=2.4.0
ARG dotnet_3_version=3.0.100
ARG dotnet_3_slug=cbc83a0e-895c-4959-99d9-21cd11596e64/b0e59c2ba2bd3ef0f592acbeae7ab27d

ENV DEBIAN_FRONTEND=noninteractive

WORKDIR /

RUN apt-get update \
&& apt-get upgrade -y \
&& apt-get install -y software-properties-common apt-transport-https curl \

## Install sudo
&& apt-get install -y sudo \

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

## wget
&& apt-get install -y wget \

## .NET Core 3.0
&& wget -O dotnet-sdk-${dotnet_3_version}.tar.gz https://download.visualstudio.microsoft.com/download/pr/${dotnet_3_slug}/dotnet-sdk-${dotnet_3_version}-linux-arm64.tar.gz \
&& mkdir -p /usr/local/dotnet \
&& tar zxf dotnet-sdk-${dotnet_3_version}.tar.gz -C /usr/local/dotnet \
&& ln -s /usr/local/dotnet/dotnet /usr/local/bin/dotnet \
&& rm dotnet-sdk-${dotnet_3_version}.tar.gz \

## node.js
&& wget https://nodejs.org/dist/${node_version}/node-${node_version}-linux-arm64.tar.gz \
&& tar --strip-components 1 -xzvf node-${node_version}-linux-arm64.tar.gz -C /usr/local \
&& rm node-${node_version}-linux-arm64.tar.gz \

## Bower & Gulp
&& npm install -g bower \
&& npm install -g gulp \

## Enable running bower with admin privileges
&& echo '{ "allow_root": true }' > /root/.bowerrc \

## eslint
&& npm install -g eslint eslint-plugin-react babel-eslint eslint-config-defaults \

## mocha (used to test the appium-xcuitest-driver)
&& npm install -g mocha mocha-junit-reporter \

# Install Powershell Core
&& wget -O powershell.tar.gz https://github.com/PowerShell/PowerShell/releases/download/v6.2.3/powershell-6.2.3-linux-arm64.tar.gz \
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
