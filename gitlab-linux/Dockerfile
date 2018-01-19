FROM ubuntu:xenial

RUN apt-get update \

## GitLab runner
## see https://docs.gitlab.com/runner/install/linux-repository.html
&& curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | bash \
&& apt-get install gitlab-runner \

## .NET Core
&& apt-get install -y curl apt-transport-https libc6-dev libgdiplus \
&& curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > /etc/apt/trusted.gpg.d/microsoft.gpg \
&& echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-ubuntu-xenial-prod xenial main" > /etc/apt/sources.list.d/dotnetdev.list \
&& apt-get update \
&& apt-get install -y dotnet-sdk-2.1.4 \

## node.js 8 (LTS)
&& curl -sL https://deb.nodesource.com/setup_8.x -o setup-nodejs \
&& /bin/bash setup-nodejs \
&& rm setup-nodejs \
&& apt-get install -y nodejs \

## Bower & Gulp
&& npm install -g bower \
&& npm install -g gulp \

## Enable running bower with admini privileges
&& echo '{ "allow_root": true }' > /root/.bowerrc \

# Install Powershell Core
&& curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - \
&& curl https://packages.microsoft.com/config/ubuntu/16.04/prod.list | tee /etc/apt/sources.list.d/microsoft.list \
&& apt-get update \
&& apt-get install -y powershell \

# Configure AzCLI 2.0  Instructions taken from https://docs.microsoft.com/en-us/cli/azure/install-azure-cli
&& echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ wheezy main" | tee /etc/apt/sources.list.d/azure-cli.list \
&& apt-key adv --keyserver packages.microsoft.com --recv-keys 417A0893 \
&& apt-get update \
&& apt-get install -y --no-install-recommends azure-cli \

## cleanup
&& rm -rf /var/lib/apt/lists/*
