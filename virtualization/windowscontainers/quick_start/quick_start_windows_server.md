---
title: Windows Server의 Windows 컨테이너
description: 컨테이너 배포 빠른 시작
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
---

# Windows Server의 Windows 컨테이너

**이 예비 콘텐츠는 변경될 수 있습니다.**

이 연습에서는 Windows Server에서 Windows 컨테이너 기능의 기본 배포 및 사용에 대해 안내합니다. 완료 후에는 컨테이너 역할이 설치되고 간단한 Windows Server 컨테이너가 배포됩니다. 이 빠른 시작을 시작하기 전에 기본 컨테이너 개념과 용어를 잘 이해해야 합니다. 이 정보는 [빠른 시작 소개](./quick_start.md)에서 확인할 수 있습니다.

이 빠른 시작은 Windows Server 2016의 Windows Server 컨테이너와 관련이 있습니다. 추가 빠른 시작 설명서는 이 페이지 왼쪽에 있는 목차에서 확인할 수 있습니다.

**필수 구성 요소:**

- [Windows Server 2016 Technical Preview 5](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-technical-preview)가 실행되는 컴퓨터 시스템(물리적 또는 가상) 1대.

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

Docker 실행 파일을 저장할 폴더를 만듭니다.

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

Docker 디먼을 다운로드합니다.

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile $env:ProgramFiles\docker\dockerd.exe
```

Docker 클라이언트를 다운로드합니다.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile $env:ProgramFiles\docker\docker.exe
```

시스템 경로에 Docker 디렉터리를 추가합니다. 완료되면 수정된 경로를 인식할 수 있도록 PowerShell 세션을 다시 시작합니다.

```none
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

먼저 컨테이너 이미지 패키지 공급자를 설치합니다.

```none
Install-PackageProvider ContainerImage -Force
```

그런 다음 Windows Server Core 이미지를 설치합니다. 이 프로세스에는 다소 시간이 걸릴 수 있으므로 중단하고 다운로드가 완료되면 백업을 선택합니다.

```none
Install-ContainerImage -Name WindowsServerCore    
```

기본 이미지를 설치한 후에는 Docker 서비스를 다시 시작해야 합니다.

```none
Restart-Service docker
```

이 단계에서 `docker images`를 실행하면 설치된 이미지(이 경우 Windows Server Core 이미지) 목록이 반환됩니다.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
windowsservercore   10.0.14300.1000     dbfee88ee9fd        7 weeks ago         9.344 GB
```

계속하려면 먼저 이 이미지에 ‘최신’ 버전으로 태그를 지정해야 합니다. 이렇게 하려면 다음 명령을 실행합니다.

```none
docker tag windowsservercore:10.0.14300.1000 windowsservercore:latest
```

Windows 컨테이너 이미지에 대한 자세한 내용은 [컨테이너 이미지 관리](../management/manage_images.md)를 참조하세요.

## 4. 첫 번째 컨테이너 배포

이 연습에서는 미리 만든 IIS 이미지를 Docker 허브 레지스트리에서 다운로드하고 IIS를 실행하는 간단한 컨테이너를 배포합니다.  

Docker 허브에서 Windows 컨테이너 이미지를 검색하려면 `docker search Microsoft`를 실행합니다.  

```none
docker search microsoft

NAME                                         DESCRIPTION                                     
microsoft/sample-django:windowsservercore    Django installed in a Windows Server Core ...   
microsoft/dotnet35:windowsservercore         .NET 3.5 Runtime installed in a Windows Se...   
microsoft/sample-golang:windowsservercore    Go Programming Language installed in a Win...   
microsoft/sample-httpd:windowsservercore     Apache httpd installed in a Windows Server...   
microsoft/iis:windowsservercore              Internet Information Services (IIS) instal...   
microsoft/sample-mongodb:windowsservercore   MongoDB installed in a Windows Server Core...   
microsoft/sample-mysql:windowsservercore     MySQL installed in a Windows Server Core b...   
microsoft/sample-nginx:windowsservercore     Nginx installed in a Windows Server Core b...  
microsoft/sample-python:windowsservercore    Python installed in a Windows Server Core ...   
microsoft/sample-rails:windowsservercore     Ruby on Rails installed in a Windows Serve...  
microsoft/sample-redis:windowsservercore     Redis installed in a Windows Server Core b...   
microsoft/sample-ruby:windowsservercore      Ruby installed in a Windows Server Core ba...   
microsoft/sample-sqlite:windowsservercore    SQLite installed in a Windows Server Core ...  
```

`docker pull`을 사용하여 IIS 이미지를 다운로드합니다.  

```none
docker pull microsoft/iis:windowsservercore
```

이미지 다운로드는 `docker images` 명령으로 확인할 수 있습니다. 여기서는 기본 이미지(windowsservercore)와 IIS 이미지가 모두 표시됩니다.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago         9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago         9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago         9.344 GB
```

`docker run`을 사용하여 IIS 컨테이너를 배포합니다.

```none
docker run -d -p 80:80 microsoft/iis:windowsservercore ping -t localhost
```

이 명령은 IIS 이미지를 백그라운드 서비스(-d)로 실행하고 컨테이너 호스트의 포트 80이 컨테이너의 포트 80에 매핑되도록 네트워킹을 구성합니다.
Docker Run 명령에 대한 자세한 내용은 [Docker.com의 Docker Run Reference(Docker Run 참조)]( https://docs.docker.com/engine/reference/run/)를 참조하세요.


실행 중인 컨테이너는 `docker ps` 명령으로 확인할 수 있습니다. 컨테이너 이름을 기록해 두세요. 이 이름은 이후 단계에서 사용됩니다.

```none
docker ps

CONTAINER ID    IMAGE                             COMMAND               CREATED              STATUS   PORTS                NAMES
9cad3ea5b7bc    microsoft/iis:windowsservercore   "ping -t localhost"   About a minute ago   Up       0.0.0.0:80->80/tcp   grave_jang
```

다른 컴퓨터에서 웹 브라우저를 열고 컨테이너 호스트의 IP 주소를 입력합니다. 모든 항목이 올바르게 구성된 경우 IIS 시작 화면이 표시됩니다. 이 화면은 Windows 컨테이너에서 호스트되는 IIS 인스턴스에서 제공됩니다.

**참고:** Azure에서 작업하는 경우 포트 80을 통한 트래픽을 허용하는 네트워크 보안 그룹 규칙이 있어야 합니다. 자세한 내용은 [네트워크 보안 그룹에서 규칙 만들기]( https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-create-nsg-arm-pportal/#create-rules-in-an-existing-nsg)를 참조하세요.

![](media/iis1.png)

다시 컨테이너 호스트에서 `docker rm` 명령을 사용하여 컨테이너를 제거합니다. 참고 - 이 예제에 있는 컨테이너의 이름을 실제 컨테이너 이름으로 바꿉니다.

```none
docker rm -f grave_jang
```
## 다음 단계

[Windows Server의 컨테이너 이미지](./quick_start_images.md)

[Windows 10의 Windows 컨테이너](./quick_start_windows_10.md)


<!--HONumber=Jun16_HO3-->


