FROM centos:7

ARG docker_version=18.06
ARG dotnet_version=2.2
ARG dotnet_package_version=2.2.402-1
ARG node_version=v10.16.0
ARG helm_version=v2.14.1
ARG yq_version=2.4.0
# Make sure to pick up a recent version from the OpenBuild repository
ARG libturbojpeg_version=2.0.2
ARG libgdiplus_version=5.23.32

COPY ./kubernetes.repo /etc/yum.repos.d/kubernetes.repo
COPY ./google-cloud-sdk.repo /etc/yum.repos.d/google-cloud-sdk.repo
COPY ./azure-cli.repo /etc/yum.repos.d/azure-cli.repo

WORKDIR /

# Update yum timeout, build.opensuse.org seems to have a very high time to first byte
# for some packages in the home repository
RUN cp /etc/yum.conf /etc/yum.conf.bak \
&& echo "minrate=1" >> /etc/yum.conf \
&& echo "timeout=300" >> /etc/yum.conf \
&& cat /etc/yum.conf

# Install jq, required by the vsts-agent
RUN yum install epel-release -y \
&& yum install -y jq \

## Install various .NET Core / WebDriver dependencies
## See https://github.com/dotnet/corefx/issues/25102 for the libc6-dev dependency
&& yum install -y yum-utils \
&& yum-config-manager -y --add-repo https://download.opensuse.org/repositories/home:/qmfrederik/CentOS_7/home:qmfrederik.repo \
&& yum install -y glibc-devel libturbojpeg-devel-${libturbojpeg_version} libgdiplus0-${libgdiplus_version} lz4-devel \

## wget and git
&& yum install -y wget git \

# Install Docker
&& yum install -y device-mapper-persistent-data lvm2 \
&& yum-config-manager -y --add-repo https://download.docker.com/linux/centos/docker-ce.repo \
&& yum install -y docker-ce:${docker_version} docker-ce-cli:${docker_version} containerd.io \

## .NET Core
&& rpm -Uvh https://packages.microsoft.com/config/rhel/7/packages-microsoft-prod.rpm \
&& yum install -y dotnet-sdk-${dotnet_version}-${dotnet_package_version} --nogpgcheck \

## Install the nbgv tool
&& dotnet tool install --tool-path /usr/local/share/dotnet-tools nbgv \
&& ln -s /usr/local/share/dotnet-tools/nbgv /usr/local/bin/nbgv \

## node.js
&& wget https://nodejs.org/dist/${node_version}/node-${node_version}-linux-x64.tar.gz \
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
&& curl https://packages.microsoft.com/config/rhel/7/prod.repo | tee /etc/yum.repos.d/microsoft.repo \
&& yum install -y powershell \

## Install PowerShell modules
&& pwsh -c "Set-PSRepository -Name PSGallery -InstallationPolicy Trusted" \
&& pwsh -c "Install-Module -Name powershell-yaml" \
&& pwsh -c "Install-Module -Name Pester" \

# Configure AzCLI 2.0  Instructions taken from https://docs.microsoft.com/en-us/cli/azure/install-azure-cli
&& yum install -y azure-cli \

## Kubectl
&& yum install -y kubectl \

## Helm
&& curl -sL https://kubernetes-helm.storage.googleapis.com/helm-${helm_version}-linux-amd64.tar.gz -o helm-${helm_version}-linux-amd64.tar.gz \
&& mkdir ~/helm \
&& tar xf helm-$helm_version-linux-amd64.tar.gz -C ~/helm/ \
&& mv ~/helm/linux-amd64/helm /usr/local/bin/helm \
&& rm -rf ~/helm \
&& helm init --client-only \
&& helm plugin install https://github.com/nouney/helm-gcs \

## Ansible
&& yum install -y ansible \

## yq
&& curl -sL https://github.com/mikefarah/yq/releases/download/${yq_version}/yq_linux_amd64 -o yq \
&& chmod +x yq \
&& mv yq /usr/local/bin \

## Google Cloud SDK
&& yum install -y google-cloud-sdk \

## nuget, tzdata
&& yum install -y nuget tzdata \

## Prime the .NET Core cache
&& mkdir ~/prime \
&& cd ~/prime \
&& dotnet new webapi \
&& dotnet restore \
&& rm -rf ~/prime \

## cleanup
&& rm -rf /var/lib/apt/lists/*
