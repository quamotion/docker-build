FROM public.ecr.aws/lts/ubuntu:focal

ARG dotnet_3_version=3.1.413
ARG dotnet_3_slug=dfd0ad22-3e47-432f-9aa1-f65b11a2ced2/d096c5d1561732c1658543fa8fb7a31f
ARG dotnet_version=5.0.401
ARG dotnet_slug=3795feca-e61d-4339-86a7-5e95aca2ad87/74300efb89e6664122f1f48c2beff734
ARG nbgv_version=3.4.231
ARG node_version=v14.17.6
ARG yq_version=4.12.2
ARG pester_version=4.10.1

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
&& apt-get install -y libavcodec-dev libavformat-dev libavutil-dev libswscale-dev libgdiplus libturbojpeg liblz4-1 libvncserver1 \
##
## Install jq, required by the vsts-agent
##
&& apt-get install -y jq \
##
## wget, libusb
##
&& apt-get install -y wget libusb-1.0-0 \
##
## .NET Core 5
##
&& mkdir -p /usr/local/dotnet \
&& wget -O dotnet-sdk-${dotnet_3_version}.tar.gz https://download.visualstudio.microsoft.com/download/pr/${dotnet_3_slug}/dotnet-sdk-${dotnet_3_version}-linux-arm64.tar.gz \
&& tar zxf dotnet-sdk-${dotnet_3_version}.tar.gz -C /usr/local/dotnet \
&& rm dotnet-sdk-${dotnet_3_version}.tar.gz \
&& wget -O dotnet-sdk-${dotnet_version}.tar.gz https://download.visualstudio.microsoft.com/download/pr/${dotnet_slug}/dotnet-sdk-${dotnet_version}-linux-arm64.tar.gz \
&& tar zxf dotnet-sdk-${dotnet_version}.tar.gz -C /usr/local/dotnet \
&& rm dotnet-sdk-${dotnet_version}.tar.gz \
&& ln -s /usr/local/dotnet/dotnet /usr/local/bin/dotnet \
##
## Install the nbgv tool
##
&& dotnet tool install --tool-path /usr/local/share/dotnet-tools nbgv --version ${nbgv_version} \
&& ln -s /usr/local/share/dotnet-tools/nbgv /usr/local/bin/nbgv \
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
&& pwsh -c "Install-Module -Name Pester -RequiredVersion $pester_version" \
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
