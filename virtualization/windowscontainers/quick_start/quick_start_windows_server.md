---
title: "Windows Server의 Windows 컨테이너"
description: "컨테이너 배포 빠른 시작"
keywords: "Docker, 컨테이너"
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
translationtype: Human Translation
ms.sourcegitcommit: d4b509054aebf510b650ec21ede4df2515a87827
ms.openlocfilehash: 6e2232c8a043d482c0d6b734a66762c2d22003fe

---

# Windows Server의 Windows 컨테이너

**이 예비 콘텐츠는 변경될 수 있습니다.**

이 연습에서는 Windows Server에서 Windows 컨테이너 기능의 기본 배포 및 사용에 대해 안내합니다. 완료 후에는 컨테이너 역할이 설치되고 간단한 Windows Server 컨테이너가 배포됩니다. 이 빠른 시작을 시작하기 전에 기본 컨테이너 개념과 용어를 잘 이해해야 합니다. 이 정보는 [빠른 시작 소개](./quick_start.md)에서 확인할 수 있습니다.

이 빠른 시작은 Windows Server 2016의 Windows Server 컨테이너와 관련이 있습니다. 추가 빠른 시작 설명서는 이 페이지 왼쪽에 있는 목차에서 확인할 수 있습니다.

**필수 구성 요소:**

[Windows Server 2016 Technical Preview 5](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-technical-preview)가 실행되는 컴퓨터 시스템(물리적 또는 가상) 1대.

완전히 구성된 Windows Server 이미지를 Azure에서 사용할 수 있습니다. 이 이미지를 사용하려면 아래 단추를 클릭하여 가상 컴퓨터를 배포하세요. 배포에는 약 10분 정도 걸립니다. 완료되면 Azure 가상 컴퓨터에 로그인하고 이 자습서의 4단계로 건너뜁니다. 

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Fmaster%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

## 1. 컨테이너 기능 설치

Windows 컨테이너를 사용하려면 먼저 컨테이너 기능을 사용하도록 설정해야 합니다. 이렇게 하려면 관리자 권한 PowerShell 세션에서 다음 명령을 실행합니다.

```none
Install-WindowsFeature containers
```

기능 설치가 완료되면 컴퓨터를 다시 부팅합니다.

```none
Restart-Computer -Force
```

## 2. Docker 설치

Windows 컨테이너를 사용하려면 Docker가 필요합니다. Docker는 Docker 엔진 및 Docker 클라이언트로 구성됩니다. 이 연습에서는 둘 다 설치됩니다.

Docker 엔진과 클라이언트를 Zip 보관 파일로 다운로드합니다.

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-1.12.1.zip" -OutFile "$env:TEMP\docker-1.12.1.zip" -UseBasicParsing
```

zip 보관 파일을 프로그램 파일로 확장합니다.

```none
Expand-Archive -Path "$env:TEMP\docker-1.12.1.zip" -DestinationPath $env:ProgramFiles
```

시스템 경로에 Docker 디렉터리를 추가합니다.

```none
# For quick use, does not require shell to be restarted.
$env:path += ";c:\program files\docker"

# For persistent use, will apply even after a reboot. 
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Docker를 Windows 서비스로 설치하려면 다음을 실행합니다.

```none
dockerd --register-service
```

설치되면 서비스를 시작할 수 있습니다.

```none
Start-Service docker
```

## 3. 기본 컨테이너 이미지 설치

Windows 컨테이너는 템플릿이나 이미지에서 배포됩니다. 컨테이너를 배포하려면 먼저 기본 OS 이미지를 다운로드해야 합니다. 다음 명령은 Windows Server Core 기본 이미지를 다운로드합니다.

```none
docker pull microsoft/windowsservercore
```

이 프로세스에는 다소 시간이 걸릴 수 있으므로 중단하고 끌어오기가 완료되면 백업을 선택합니다.

이미지를 끌어온 후 `docker images`를 실행하면 설치된 이미지(이 경우 Windows Server Core 이미지) 목록이 반환됩니다.

```none
docker images

REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
microsoft/windowsservercore   latest              02cb7f65d61b        8 weeks ago         7.764 GB
```

Windows 컨테이너 이미지에 대한 자세한 내용은 [컨테이너 이미지 관리](../management/manage_images.md)를 참조하세요.

## 4. 첫 번째 컨테이너 배포

이 연습에서는 미리 만든 IIS 이미지를 Docker 허브 레지스트리에서 다운로드하고 IIS를 실행하는 간단한 컨테이너를 배포합니다.  

Docker 허브에서 Windows 컨테이너 이미지를 검색하려면 `docker search Microsoft`를 실행합니다.  

```none
docker search microsoft

NAME                                         DESCRIPTION
microsoft/aspnet                             ASP.NET is an open source server-side Web ...
microsoft/dotnet                             Official images for working with .NET Core...
mono                                         Mono is an open source implementation of M...
microsoft/azure-cli                          Docker image for Microsoft Azure Command L...
microsoft/iis                                Internet Information Services (IIS) instal...
microsoft/mssql-server-2014-express-windows  Microsoft SQL Server 2014 Express installe...
microsoft/nanoserver                         Nano Server base OS image for Windows cont...
microsoft/windowsservercore                  Windows Server Core base OS image for Wind...
microsoft/oms                                Monitor your containers using the Operatio...
microsoft/dotnet-preview                     Preview bits for microsoft/dotnet image
microsoft/dotnet35
microsoft/applicationinsights                Application Insights for Docker helps you ...
microsoft/sample-redis                       Redis installed in Windows Server Core and...
microsoft/sample-node                        Node installed in a Nano Server based cont...
microsoft/sample-nginx                       Nginx installed in Windows Server Core and...
microsoft/sample-httpd                       Apache httpd installed in Windows Server C...
microsoft/sample-dotnet                      .NET Core running in a Nano Server container
microsoft/sqlite                             SQLite installed in a Windows Server Core ...
...
```

`docker pull`을 사용하여 IIS 이미지를 다운로드합니다.  

```none
docker pull microsoft/iis
```

이미지 다운로드는 `docker images` 명령으로 확인할 수 있습니다. 여기서는 기본 이미지(windowsservercore)와 IIS 이미지가 모두 표시됩니다.

```none
docker images

REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
microsoft/iis                 latest              accd044753c1        11 days ago         7.907 GB
microsoft/windowsservercore   latest              02cb7f65d61b        8 weeks ago         7.764 GB
```

`docker run`을 사용하여 IIS 컨테이너를 배포합니다.

```none
docker run -d -p 80:80 microsoft/iis ping -t localhost
```

이 명령은 IIS 이미지를 백그라운드 서비스(-d)로 실행하고 컨테이너 호스트의 포트 80이 컨테이너의 포트 80에 매핑되도록 네트워킹을 구성합니다.
Docker Run 명령에 대한 자세한 내용은 [Docker.com의 Docker Run Reference(Docker Run 참조)]( https://docs.docker.com/engine/reference/run/)를 참조하세요.


실행 중인 컨테이너는 `docker ps` 명령으로 확인할 수 있습니다. 컨테이너 이름을 기록해 두세요. 이 이름은 이후 단계에서 사용됩니다.

```none
docker ps

CONTAINER ID  IMAGE          COMMAND              CREATED             STATUS             PORTS               NAME
09c9cc6e4f83  microsoft/iis  "ping -t localhost"  About a minute ago  Up About a minute  0.0.0.0:80->80/tcp  big_jang
```

다른 컴퓨터에서 웹 브라우저를 열고 컨테이너 호스트의 IP 주소를 입력합니다. 모든 항목이 올바르게 구성된 경우 IIS 시작 화면이 표시됩니다. 이 화면은 Windows 컨테이너에서 호스트되는 IIS 인스턴스에서 제공됩니다.

**참고:** Azure에서 작업하는 경우 가상 컴퓨터의 외부 IP 주소 및 구성된 네트워크 보안이 필요합니다. 자세한 내용은 [네트워크 보안 그룹에서 규칙 만들기]( https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-create-nsg-arm-pportal/#create-rules-in-an-existing-nsg)를 참조하세요.

![](media/iis1.png)

다시 컨테이너 호스트에서 `docker rm` 명령을 사용하여 컨테이너를 제거합니다. 참고 - 이 예제에 있는 컨테이너의 이름을 실제 컨테이너 이름으로 바꿉니다.

```none
docker rm -f big_jang
```
## 다음 단계

[Windows Server의 컨테이너 이미지](./quick_start_images.md)

[Windows 10의 Windows 컨테이너](./quick_start_windows_10.md)



<!--HONumber=Sep16_HO3-->


