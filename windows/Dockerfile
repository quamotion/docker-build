FROM mcr.microsoft.com/windows/servercore:ltsc2019

ARG dotnet_3_version=3.1.413
ARG dotnet_3_slug=587d67c3-393e-4915-b9c4-5dda816d4945/068c1fe53054c217290a8980f8c49b91
ARG dotnet_version=5.0.401
ARG dotnet_slug=7bb2b576-f96c-465d-80bf-0a3956e75f4c/4ba1f2657ac6eb4b725284028c4f8707
ARG nbgv_version=3.4.231
ARG pester_version=4.10.1

RUN mkdir C:\BuildAgent
WORKDIR C:/BuildAgent
COPY ./Start.* ./

ENV chocolateyUseWindowsCompression=false
RUN @powershell -NoProfile -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"

RUN choco config set cachelocation C:\chococache

RUN choco install \
    git  \
    nodejs \
    curl \
    docker \
    netfx-4.5.1-devpack \
    nuget.commandline \
    --confirm \
    --limit-output \
    --timeout 216000 \
    && rmdir /S /Q C:\chococache

RUN choco install \
    docfx --version 2.42.4 \
    --confirm \
    --limit-output \
    --timeout 216000 \
    && rmdir /S /Q C:\chococache

# common node tools
RUN npm install gulp -g && npm install bower -g
    
SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

# Install .NET Core SDK
RUN Write-Host "Downloading https://download.visualstudio.microsoft.com/download/pr/${env:dotnet_3_slug}/dotnet-sdk-${env:dotnet_3_version}-win-x64.zip"; \
    Invoke-WebRequest https://download.visualstudio.microsoft.com/download/pr/${env:dotnet_3_slug}/dotnet-sdk-${env:dotnet_3_version}-win-x64.zip -OutFile dotnet-sdk-3.1.zip; \
    Expand-Archive dotnet-sdk-3.1.zip -DestinationPath $Env:ProgramFiles\dotnet -Force; \
    Remove-Item -Force dotnet-sdk-3.1.zip

RUN Write-Host "Downloading https://download.visualstudio.microsoft.com/download/pr/${env:dotnet_slug}/dotnet-sdk-${env:dotnet_version}-win-x64.zip"; \
    Invoke-WebRequest https://download.visualstudio.microsoft.com/download/pr/${env:dotnet_slug}/dotnet-sdk-${env:dotnet_version}-win-x64.zip -OutFile dotnet-sdk-5.0.zip; \
    Expand-Archive dotnet-sdk-5.0.zip -DestinationPath $Env:ProgramFiles\dotnet -Force; \
    Remove-Item -Force dotnet-sdk-5.0.zip

# Upgrade Pester to the latest version
RUN Install-PackageProvider NuGet -force; \
    Install-Module -Name Pester -RequiredVersion $env:pester_version -Force -SkipPublisherCheck

# Install AzCopy, used by the release process
ENV AZ_COPY_URL https://azcopy.azureedge.net/azcopy-8-1-0/MicrosoftAzureStorageAzCopy_netcore_x64.msi
RUN Invoke-WebRequest $Env:AZ_COPY_URL -OutFile MicrosoftAzureStorageTools.msi; \
    Start-Process msiexec.exe -Wait -ArgumentList '/i MicrosoftAzureStorageTools.msi /qn'; \
    Remove-Item -Force MicrosoftAzureStorageTools.msi

SHELL ["cmd", "/S", "/C"]

# Add .NET Core, AzCopy to path, warm up .NET
RUN setx /M PATH "%PATH%;%ProgramFiles%\dotnet;%ProgramFiles(x86)%\Microsoft SDKs\Azure\AzCopy"

RUN mkdir C:\warmup \
    && cd C:\warmup \
    && dotnet new \
    && cd .. \
    && rmdir /S /Q C:\warmup 

## Install nbgv tool
RUN dotnet tool install --global nbgv --version %NBGV_VERSION% --add-source https://api.nuget.org/v3/index.json
