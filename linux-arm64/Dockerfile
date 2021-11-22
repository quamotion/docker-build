# Use Ubuntu Impish (21.10), which should ship with glibc 2.34,
# which should contain a fix for the libgomp issue:
# https://sourceware.org/bugzilla/show_bug.cgi?id=25051
FROM public.ecr.aws/ubuntu/ubuntu:impish

ARG dotnet_3_version=3.1.413
ARG dotnet_3_slug=dfd0ad22-3e47-432f-9aa1-f65b11a2ced2/d096c5d1561732c1658543fa8fb7a31f
ARG dotnet_version=6.0.100
ARG dotnet_slug=adcd9310-5072-4179-9b8b-16563b897995/15a7595966f488c74909e4a9273c0e24
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
## libcap2-bin, contains setcap
##
&& apt-get install -y libcap2-bin \
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
