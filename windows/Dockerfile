FROM mcr.microsoft.com/windows/servercore:ltsc2019
# ENV WINDOWS_IMAGE_VERSION=10.0.14393

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
    docfx \
    netfx-4.5.1-devpack \
    nuget.commandline \
    --confirm \
    --limit-output \
    --timeout 216000 \
    && rmdir /S /Q C:\chococache

# common node tools
RUN npm install gulp -g && npm install bower -g
    
SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

# Install .NET Core SDK
ENV DOTNET_2_2_SDK_DOWNLOAD_URL https://download.visualstudio.microsoft.com/download/pr/3c7fcb0b-52ee-40b2-853d-710c58883371/78bbdf5fcd85697e8e306c355d02d0b0/dotnet-sdk-2.2.301-win-x64.zip

RUN Invoke-WebRequest $Env:DOTNET_2_2_SDK_DOWNLOAD_URL -OutFile dotnet-sdk-2.2.zip; \
    Expand-Archive dotnet-sdk-2.2.zip -DestinationPath $Env:ProgramFiles\dotnet -Force; \
    Remove-Item -Force dotnet-sdk-2.2.zip

ENV DOTNET_3_0_SDK_DOWNLOAD_URL https://download.visualstudio.microsoft.com/download/pr/4e73514f-0646-4600-bdc2-38c90fc207b6/22ad70cbe1a254177e6d7b7f7e7425f8/dotnet-sdk-3.0.100-preview6-012264-win-x64.zip

RUN Invoke-WebRequest $Env:DOTNET_3_0_SDK_DOWNLOAD_URL -OutFile dotnet-sdk-3.0.zip; \
    Expand-Archive dotnet-sdk-3.0.zip -DestinationPath $Env:ProgramFiles\dotnet -Force; \
    Remove-Item -Force dotnet-sdk-3.0.zip

# Install GitLab, VSTS runners
ENV GITLAB_RUNNER_DOWNLOAD_URL https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-windows-amd64.exe
RUN New-Item -ItemType directory -Path C:\GitLab; \
    Invoke-WebRequest $Env:GITLAB_RUNNER_DOWNLOAD_URL -OutFile C:\GitLab\gitlab-runner.exe

ENV VSTS_AGENT_DOWNLOAD_URL https://vstsagentpackage.azureedge.net/agent/2.144.2/vsts-agent-win-x86-2.144.2.zip
RUN Invoke-WebRequest $Env:VSTS_AGENT_DOWNLOAD_URL -OutFile vsts.zip; \
    Expand-Archive vsts.zip -DestinationPath C:\vsts-agent -Force; \
    Remove-Item -Force vsts.zip

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
