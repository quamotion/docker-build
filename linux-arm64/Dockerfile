FROM ubuntu:focal

ARG node_version=v12.18.3
ARG yq_version=3.3.2
ARG dotnet_version=5.0.100-preview.8.20417.9
ARG dotnet_slug=a1e93182-8026-4330-b78a-ee7d721107a2/003c59fc228a220df40d90d5ac434873
ARG dotnet_3_version=3.1.300	
ARG dotnet_3_slug=e5e70860-a6d4-48cf-b0d1-eeba32657d80/2da3c605aaa65c7e4ac2ad0507a2e429	

ENV DEBIAN_FRONTEND=noninteractive

WORKDIR /

RUN apt-get update \
&& apt-get upgrade -y \
&& apt-get install -y software-properties-common apt-transport-https curl \
##
## Install sudo and git
##
&& apt-get install -y sudo git \
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
&& apt-get install -y libgdiplus \
&& wget -O dotnet-sdk-${dotnet_3_version}.tar.gz https://download.visualstudio.microsoft.com/download/pr/${dotnet_3_slug}/dotnet-sdk-${dotnet_3_version}-linux-arm64.tar.gz \
&& mkdir -p /usr/local/dotnet \	
&& tar zxf dotnet-sdk-${dotnet_3_version}.tar.gz -C /usr/local/dotnet \	
&& ln -s /usr/local/dotnet/dotnet /usr/local/bin/dotnet \	
&& rm dotnet-sdk-${dotnet_3_version}.tar.gz \	
##
## .NET Core 5 preview
##
&& wget -O dotnet-sdk-${dotnet_version}.tar.gz https://download.visualstudio.microsoft.com/download/pr/${dotnet_slug}/dotnet-sdk-${dotnet_version}-linux-arm64.tar.gz \
&& tar zxf dotnet-sdk-${dotnet_version}.tar.gz -C /usr/local/dotnet \
&& rm dotnet-sdk-${dotnet_version}.tar.gz \
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
&& wget -O powershell.tar.gz https://github.com/PowerShell/PowerShell/releases/download/v7.1.0-preview.6/powershell-7.1.0-preview.6-linux-arm64.tar.gz \
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
