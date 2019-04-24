FROM centos:7

ARG docker_version=18.06
ARG dotnet_version=2.2
ARG node_version=v10.15.3
ARG helm_version=v2.13.0
ARG yq_version=2.2.1
# Make sure to pick up a recent version from the OpenBuild repository
ARG libturbojpeg_version=2.0.2
ARG libgdiplus_version=5.23.32

## This image contains both VSTS and GitLab runners, and provides no default entrypoint.
## You need to start the VSTS or GitLab runner manually when staring a container based on this image.

## VSTS runner
## see https://github.com/Microsoft/vsts-agent-docker/blob/master/ubuntu/16.04/Dockerfile

# Accept the TEE EULA
RUN mkdir -p "/root/.microsoft/Team Foundation/4.0/Configuration/TEE-Mementos" \
 && cd "/root/.microsoft/Team Foundation/4.0/Configuration/TEE-Mementos" \
 && echo '<ProductIdData><eula-14.0 value="true"/></ProductIdData>' > "com.microsoft.tfs.client.productid.xml"

WORKDIR /vsts

COPY ./start-vsts.sh ./start.sh
COPY ./kubernetes.repo /etc/yum.repos.d/kubernetes.repo
COPY ./google-cloud-sdk.repo /etc/yum.repos.d/google-cloud-sdk.repo
COPY ./azure-cli.repo /etc/yum.repos.d/azure-cli.repo
RUN chmod +x start.sh

WORKDIR /

# Install jq, required by the vsts-agent
RUN yum install epel-release -y \
&& yum install -y jq \

## Install various .NET Core / WebDriver dependencies
## See https://github.com/dotnet/corefx/issues/25102 for the libc6-dev dependency
&& yum install -y yum-utils \
&& yum-config-manager -y --add-repo https://download.opensuse.org/repositories/home:/qmfrederik/CentOS_7/home:qmfrederik.repo \
&& yum install -y glibc-devel libturbojpeg-devel-${libturbojpeg_version} libgdiplus0-${libgdiplus_version} \

## wget
&& yum install -y wget \

# Install Docker
&& yum install -y device-mapper-persistent-data lvm2 \
&& yum-config-manager -y --add-repo https://download.docker.com/linux/centos/docker-ce.repo \
&& yum install -y docker-ce:${docker_version} docker-ce-cli:${docker_version} containerd.io \

## GitLab runner
## see https://docs.gitlab.com/runner/install/linux-repository.html
&& curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh | bash \
&& yum install -y gitlab-runner \

## .NET Core
&& rpm -Uvh https://packages.microsoft.com/config/rhel/7/packages-microsoft-prod.rpm \
&& yum install -y dotnet-sdk-${dotnet_version} --nogpgcheck \

## trx2junit to send trx files to GitLab
&& dotnet tool install -g trx2junit \

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