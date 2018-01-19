FROM microsoft/vsts-agent:ubuntu-16.04-docker-17.06.0-ce

RUN apt-get update \

## .NET Core
&& apt-get install -y curl apt-transport-https libc6-dev libgdiplus \
&& curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > /etc/apt/trusted.gpg.d/microsoft.gpg \
&& echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-ubuntu-xenial-prod xenial main" > /etc/apt/sources.list.d/dotnetdev.list \
&& apt-get update \
&& apt-get install -y dotnet-sdk-2.1.4 \

## node.js 8 (LTS)
&& curl -sL https://deb.nodesource.com/setup_8.x | -E bash - \
&& apt-get install -y nodejs \

## Bower & Gulp
&& npm install -g bower \
&& npm install -g gulp \

## cleanup
&& rm -rf /var/lib/apt/lists/*
