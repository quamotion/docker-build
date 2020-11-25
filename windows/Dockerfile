FROM mcr.microsoft.com/windows/servercore:ltsc2019

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
ENV DOTNET_3_1_SDK_DOWNLOAD_URL https://download.visualstudio.microsoft.com/download/pr/355e1344-7f0e-45c7-aace-ac51a3d9183f/a356fda8c22ea1e0e3ba1c6f0b2e4815/dotnet-sdk-3.1.404-win-x64.zip
ENV DOTNET_5_0_SDK_DOWNLOAD_URL https://download.visualstudio.microsoft.com/download/pr/7b78bdaa-d0ac-41c4-9fdc-5820d7dc79b6/cea499dd314ba6394ccea51a2a2dcda9/dotnet-sdk-5.0.100-win-x64.zip

RUN Invoke-WebRequest $Env:DOTNET_3_1_SDK_DOWNLOAD_URL -OutFile dotnet-sdk-3.1.zip; \
    Expand-Archive dotnet-sdk-3.1.zip -DestinationPath $Env:ProgramFiles\dotnet -Force; \
    Remove-Item -Force dotnet-sdk-3.1.zip

RUN Invoke-WebRequest $Env:DOTNET_5_0_SDK_DOWNLOAD_URL -OutFile dotnet-sdk-5.0.zip; \
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
RUN dotnet tool install --global nbgv --version 3.0.28
