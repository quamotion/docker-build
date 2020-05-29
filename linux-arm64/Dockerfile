FROM ubuntu:bionic

ARG node_version=v12.17.0
ARG yq_version=3.3.0
ARG dotnet_3_version=3.1.300
ARG dotnet_3_slug=e5e70860-a6d4-48cf-b0d1-eeba32657d80/2da3c605aaa65c7e4ac2ad0507a2e429

ENV DEBIAN_FRONTEND=noninteractive

WORKDIR /

RUN apt-get update \
&& apt-get upgrade -y \
&& apt-get install -y software-properties-common apt-transport-https curl \
##
## Install sudo
##
&& apt-get install -y sudo \
##
## WebDriver dependencies for which we don't ship any native files as part
## of the NuGet packages
##
&& apt-get install -y libavcodec-dev libavformat-dev libavutil-dev libswscale-dev libturbojpeg liblz4-dev \
##
## Install jq, required by the vsts-agent
##
&& apt-get install -y jq \
##
## wget, libusb
##
&& apt-get install -y wget libusb-1.0-0 \
##
## .NET Core 3.1
##
&& wget -O dotnet-sdk-${dotnet_3_version}.tar.gz https://download.visualstudio.microsoft.com/download/pr/${dotnet_3_slug}/dotnet-sdk-${dotnet_3_version}-linux-arm64.tar.gz \
&& mkdir -p /usr/local/dotnet \
&& tar zxf dotnet-sdk-${dotnet_3_version}.tar.gz -C /usr/local/dotnet \
&& ln -s /usr/local/dotnet/dotnet /usr/local/bin/dotnet \
&& rm dotnet-sdk-${dotnet_3_version}.tar.gz \
##
## node.js
##
&& wget https://nodejs.org/dist/${node_version}/node-${node_version}-linux-arm64.tar.gz \
&& tar --strip-components 1 -xzvf node-${node_version}-linux-arm64.tar.gz -C /usr/local \
&& rm node-${node_version}-linux-arm64.tar.gz \
##
## Bower & Gulp
##
&& npm install -g bower \
&& npm install -g gulp \
##
## Enable running bower with admin privileges
##
&& echo '{ "allow_root": true }' > /root/.bowerrc \
##
## eslint
##
&& npm install -g eslint eslint-plugin-react babel-eslint eslint-config-defaults \
##
## mocha (used to test the appium-xcuitest-driver)
##
&& npm install -g mocha mocha-junit-reporter \
##
## Install Powershell Core
##
&& wget -O powershell.tar.gz https://github.com/PowerShell/PowerShell/releases/download/v6.2.3/powershell-6.2.3-linux-arm64.tar.gz \
&& mkdir -p /usr/local/powershell \
&& tar zxf powershell.tar.gz -C /usr/local/powershell \
&& ln -s /usr/local/powershell/pwsh /usr/local/bin/pwsh \
&& chmod +x /usr/local/bin/pwsh \
&& rm powershell.tar.gz \
##
## Install PowerShell modules
##
&& pwsh -c "Set-PSRepository -Name PSGallery -InstallationPolicy Trusted" \
&& pwsh -c "Install-Module -Name powershell-yaml" \
&& pwsh -c "Install-Module -Name Pester" \
##
## Configure AzCLI 2.0  Instructions taken from https://docs.microsoft.com/en-us/cli/azure/install-azure-cli
## There are no ARM64 builds yet
##
## Ansible
##
&& apt-add-repository -y ppa:ansible/ansible \
&& apt-get update \
&& apt-get install -y ansible \
##
## yq
##
&& curl -sL https://github.com/mikefarah/yq/releases/download/$yq_version/yq_linux_arm64 -o yq \
&& chmod +x yq \
&& mv yq /usr/local/bin \
##
## tzdata
##
&& apt-get install -y tzdata \
##
## Prime the .NET Core cache
##
&& mkdir ~/prime \
&& cd ~/prime \
&& dotnet new webapi \
&& dotnet restore \
&& rm -rf ~/prime \
##
## cleanup
##
&& rm -rf /var/lib/apt/lists/*
