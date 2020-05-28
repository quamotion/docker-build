FROM mcr.microsoft.com/windows/servercore:ltsc2019

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
ENV DOTNET_3_1_SDK_DOWNLOAD_URL https://download.visualstudio.microsoft.com/download/pr/b8fee6d8-0d26-4b0f-9082-52a46550e702/46c99a915b1200f0795ce05caa946e9a/dotnet-sdk-3.1.300-win-x64.zip

RUN Invoke-WebRequest $Env:DOTNET_3_1_SDK_DOWNLOAD_URL -OutFile dotnet-sdk-3.1.zip; \
    Expand-Archive dotnet-sdk-3.1.zip -DestinationPath $Env:ProgramFiles\dotnet -Force; \
    Remove-Item -Force dotnet-sdk-3.1.zip

# Upgrade Pester to the latest version
RUN Install-PackageProvider NuGet -force; \
    Install-Module -Name Pester -Force -SkipPublisherCheck

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
