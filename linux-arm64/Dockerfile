FROM ubuntu:bionic

ARG docker_version=18.06
ARG node_version=v10.16.0
ARG helm_version=v2.14.1
ARG yq_version=2.4.0
ARG dotnet_2_version=2.2.301
ARG dotnet_2_slug=0af74ee1-47bb-43bd-b55f-1657f079c309/6649fd1bc91b14aee4a6b4ed44a2f45d
ARG dotnet_3_version=3.0.100-preview6-012264
ARG dotnet_3_slug=8997987c-1fcc-4b83-ab49-08117ac40f86/13f3cc0b0dfcf37398d11caff3926bb9

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
&& wget -O dotnet-sdk-${dotnet_2_version}.tar.gz https://download.visualstudio.microsoft.com/download/pr/9605c24e-9bb2-499f-a003-4fb2bddcf09c/a8683ff89405d370961beb1909ddc295/dotnet-sdk-${dotnet_2_version}-linux-arm64.tar.gz \
&& tar zxf dotnet-sdk-${dotnet_2_version}.tar.gz -C /usr/local/dotnet \
&& rm dotnet-sdk-${dotnet_2_version}.tar.gz \

## .NET Core (3.0 preview)
&& wget -O dotnet-sdk-${dotnet_3_version}.tar.gz https://download.visualstudio.microsoft.com/download/pr/${dotnet_3_slug}/dotnet-sdk-${dotnet_3_version}-linux-arm64.tar.gz \
&& tar zxf dotnet-sdk-${dotnet_3_version}.tar.gz -C /usr/local/dotnet \
&& ln -s /usr/local/dotnet/dotnet /usr/local/bin/dotnet \
&& rm dotnet-sdk-${dotnet_3_version}.tar.gz \

## trx2junit to send trx files to GitLab
&& dotnet tool install -g trx2junit \

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
