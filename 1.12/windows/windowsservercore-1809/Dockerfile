FROM mcr.microsoft.com/windows/servercore:1809

# $ProgressPreference: https://github.com/PowerShell/PowerShell/issues/2138#issuecomment-251261324
SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

# install MinGit (especially for "go get")
# https://blogs.msdn.microsoft.com/visualstudioalm/2016/09/03/whats-new-in-git-for-windows-2-10/
# "Essentially, it is a Git for Windows that was stripped down as much as possible without sacrificing the functionality in which 3rd-party software may be interested."
# "It currently requires only ~45MB on disk."
ENV GIT_VERSION 2.23.0
ENV GIT_TAG v${GIT_VERSION}.windows.1
ENV GIT_DOWNLOAD_URL https://github.com/git-for-windows/git/releases/download/${GIT_TAG}/MinGit-${GIT_VERSION}-64-bit.zip
ENV GIT_DOWNLOAD_SHA256 8f65208f92c0b4c3ae4c0cf02d4b5f6791d539cd1a07b2df62b7116467724735
# steps inspired by "chcolateyInstall.ps1" from "git.install" (https://chocolatey.org/packages/git.install)
RUN Write-Host ('Downloading {0} ...' -f $env:GIT_DOWNLOAD_URL); \
	[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; \
	Invoke-WebRequest -Uri $env:GIT_DOWNLOAD_URL -OutFile 'git.zip'; \
	\
	Write-Host ('Verifying sha256 ({0}) ...' -f $env:GIT_DOWNLOAD_SHA256); \
	if ((Get-FileHash git.zip -Algorithm sha256).Hash -ne $env:GIT_DOWNLOAD_SHA256) { \
		Write-Host 'FAILED!'; \
		exit 1; \
	}; \
	\
	Write-Host 'Expanding ...'; \
	Expand-Archive -Path git.zip -DestinationPath C:\git\.; \
	\
	Write-Host 'Removing ...'; \
	Remove-Item git.zip -Force; \
	\
	Write-Host 'Updating PATH ...'; \
	$env:PATH = 'C:\git\cmd;C:\git\mingw64\bin;C:\git\usr\bin;' + $env:PATH; \
	[Environment]::SetEnvironmentVariable('PATH', $env:PATH, [EnvironmentVariableTarget]::Machine); \
	\
	Write-Host 'Verifying install ("git version") ...'; \
	git version; \
	\
	Write-Host 'Complete.';

# ideally, this would be C:\go to match Linux a bit closer, but C:\go is the recommended install path for Go itself on Windows
ENV GOPATH C:\\gopath

# PATH isn't actually set in the Docker image, so we have to set it from within the container
RUN $newPath = ('{0}\bin;C:\go\bin;{1}' -f $env:GOPATH, $env:PATH); \
	Write-Host ('Updating PATH: {0}' -f $newPath); \
	[Environment]::SetEnvironmentVariable('PATH', $newPath, [EnvironmentVariableTarget]::Machine);
# doing this first to share cache across versions more aggressively

ENV GOLANG_VERSION 1.12.12

RUN $url = ('https://golang.org/dl/go{0}.windows-amd64.zip' -f $env:GOLANG_VERSION); \
	Write-Host ('Downloading {0} ...' -f $url); \
	Invoke-WebRequest -Uri $url -OutFile 'go.zip'; \
	\
	$sha256 = '2d030b382514a3c4b76dd0786f810ce50409cebaf311a6b79addc353627263a8'; \
	Write-Host ('Verifying sha256 ({0}) ...' -f $sha256); \
	if ((Get-FileHash go.zip -Algorithm sha256).Hash -ne $sha256) { \
		Write-Host 'FAILED!'; \
		exit 1; \
	}; \
	\
	Write-Host 'Expanding ...'; \
	Expand-Archive go.zip -DestinationPath C:\; \
	\
	Write-Host 'Removing ...'; \
	Remove-Item go.zip -Force; \
	\
	Write-Host 'Verifying install ("go version") ...'; \
	go version; \
	\
	Write-Host 'Complete.';

WORKDIR $GOPATH
