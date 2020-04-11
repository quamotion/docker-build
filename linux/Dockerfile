FROM ubuntu:bionic

ARG dotnet_version=3.1
ARG dotnet_package_version=3.1.100-1
ARG node_version=v10.16.0
ARG yq_version=2.4.0

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
## Install jq, required by the vsts-agent
##
&& apt-get install -y jq \
##
## .NET Core
##
&& apt-get install -y curl apt-transport-https libc6-dev libgdiplus libturbojpeg \
&& curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > /etc/apt/trusted.gpg.d/microsoft.gpg \
&& echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-ubuntu-bionic-prod bionic main" > /etc/apt/sources.list.d/dotnetdev.list \
&& apt-get update \
&& apt-get install -y dotnet-sdk-$dotnet_version=${dotnet_package_version} \
##
## Install the nbgv tool
##
&& dotnet tool install --tool-path /usr/local/share/dotnet-tools nbgv \
&& ln -s /usr/local/share/dotnet-tools/nbgv /usr/local/bin/nbgv \
##
## node.js
##
&& curl -sL https://nodejs.org/dist/${node_version}/node-${node_version}-linux-x64.tar.gz  -o node-${node_version}-linux-x64.tar.gz \
&& tar --strip-components 1 -xzvf node-${node_version}-linux-x64.tar.gz -C /usr/local \
&& rm node-${node_version}-linux-x64.tar.gz \
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
&& curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - \
&& curl https://packages.microsoft.com/config/ubuntu/18.04/prod.list | tee /etc/apt/sources.list.d/microsoft.list \
&& apt-get update \
&& apt-get install -y powershell \
##
## Install PowerShell modules
##
&& pwsh -c "Set-PSRepository -Name PSGallery -InstallationPolicy Trusted" \
&& pwsh -c "Install-Module -Name powershell-yaml" \
&& pwsh -c "Install-Module -Name Pester" \
##
## Configure AzCLI 2.0  Instructions taken from https://docs.microsoft.com/en-us/cli/azure/install-azure-cli
##
&& echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ bionic main" | tee /etc/apt/sources.list.d/azure-cli.list \
&& curl -sL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/microsoft.asc.gpg > /dev/null \
&& apt-get update \
&& apt-get install -y --no-install-recommends azure-cli \
##
## Ansible
##
&& apt-add-repository -y ppa:ansible/ansible \
&& apt-get update \
&& apt-get install -y ansible \
##
## yq
##
&& curl -sL https://github.com/mikefarah/yq/releases/download/$yq_version/yq_linux_amd64 -o yq \
&& chmod +x yq \
&& mv yq /usr/local/bin \
##
## wget, libusb
##
&& apt-get install -y wget libusb-1.0-0 \
##
## nuget, tzdata
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
