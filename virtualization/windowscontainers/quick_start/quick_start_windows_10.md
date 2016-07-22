---
title: "Windows 10의 Windows 컨테이너"
description: "컨테이너 배포 빠른 시작"
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 07/13/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
translationtype: Human Translation
ms.sourcegitcommit: edf2c2597e57909a553eb5e6fcc75cdb820fce68
ms.openlocfilehash: b37d402f2e6c950db061f5de0c86f0e9aace62b4

---

# Windows 10의 Windows 컨테이너

**이 예비 콘텐츠는 변경될 수 있습니다.** 

이 연습에서는 Windows 10(참가자 빌드 14372 이상)에서 Windows 컨테이너 기능의 기본 배포 및 사용에 대해 안내합니다. 완료 후에는 컨테이너 역할이 설치되고 간단한 Hyper-V 컨테이너가 배포됩니다. 이 빠른 시작을 시작하기 전에 기본 컨테이너 개념과 용어를 잘 이해해야 합니다. 이 정보는 [빠른 시작 소개](./quick_start.md)에서 확인할 수 있습니다. 

이 빠른 시작은 Windows 10의 Hyper-V 컨테이너와 관련이 있습니다. 추가 빠른 시작 설명서는 이 페이지 왼쪽에 있는 목차에서 확인할 수 있습니다.

**필수 구성 요소:**

- [Windows 10 참가자 릴리스](https://insider.windows.com/)가 실행되는 물리적 컴퓨터 시스템 1대.   
- 이 빠른 시작은 Windows 10 가상 컴퓨터에서 실행할 수 있지만 중첩된 가상화를 사용하도록 설정해야 합니다. 자세한 정보는 [중첩된 가상화 가이드](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)에서 확인할 수 있습니다.

## 1. 컨테이너 기능 설치

Windows 컨테이너를 사용하려면 먼저 컨테이너 기능을 사용하도록 설정해야 합니다. 이렇게 하려면 관리자 권한 PowerShell 세션에서 다음 명령을 실행합니다. 

```none
Enable-WindowsOptionalFeature -Online -FeatureName containers -All
```

Windows 10은 Hyper-V 컨테이너만 지원하므로 Hyper-V 기능도 사용하도록 설정해야 합니다. PowerShell을 사용하여 Hyper-V 기능을 사용하도록 설정하려면 관리자 권한 PowerShell 세션에서 다음 명령을 실행합니다.

```none
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

설치가 완료되면 컴퓨터를 다시 부팅합니다.

```none
Restart-Computer -Force
```

다시 부팅되면 다음 명령을 실행하여 Windows 컨테이너 Technical Preview의 알려진 문제를 수정합니다.  

 ```none
Set-ItemProperty -Path 'HKLM:SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers' -Name VSmbDisableOplocks -Type DWord -Value 1 -Force
```

## 2. Docker 설치

Windows 컨테이너를 사용하려면 Docker가 필요합니다. Docker는 Docker 엔진 및 Docker 클라이언트로 구성됩니다. 이 연습에서는 둘 다 설치됩니다. 이렇게 하려면 다음 명령을 실행합니다. 

Docker 실행 파일에 대한 폴더를 만듭니다.

```none
New-Item -Type Directory -Path $env:ProgramFiles\docker\
```

Docker 디먼을 다운로드합니다.

```none
Invoke-WebRequest https://master.dockerproject.org/windows/amd64/dockerd.exe -OutFile $env:ProgramFiles\docker\dockerd.exe
```

Docker 클라이언트를 다운로드합니다.

```none
Invoke-WebRequest https://master.dockerproject.org/windows/amd64/docker.exe -OutFile $env:ProgramFiles\docker\docker.exe
```

시스템 경로에 Docker 디렉터리를 추가합니다.

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";$env:ProgramFiles\docker\", [EnvironmentVariableTarget]::Machine)
```

수정된 경로를 인식할 수 있도록 PowerShell 세션을 다시 시작합니다.

Docker를 Windows 서비스로 설치하려면 다음을 실행합니다.

```none
dockerd --register-service
```

설치되면 서비스를 시작할 수 있습니다.

```none
Start-Service Docker
```

## 3. 기본 컨테이너 이미지 설치

Windows 컨테이너는 템플릿이나 이미지에서 배포됩니다. 컨테이너를 배포하려면 먼저 기본 OS 이미지를 다운로드해야 합니다. 다음 명령은 Nano Server 기본 이미지를 다운로드합니다.
    
> 이 절차는 Windows 참가자 빌드 14372 이상에 적용되며 'docker pull'이 작동할 때까지 임시로 사용됩니다.

Nano Server 서버 기본 이미지를 다운로드합니다. 

```none
Start-BitsTransfer https://aka.ms/tp5/6b/docker/nanoserver -Destination nanoserver.tar.gz
```

기본 이미지를 설치합니다.

```none  
docker load -i nanoserver.tar.gz
```

이 단계에서 `docker images`를 실행하면 설치된 이미지(이 경우 Nano Server 이미지) 목록이 반환됩니다.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14300.1030     3f5112ddd185        3 weeks ago         810.2 MB
```

계속하려면 먼저 이 이미지에 ‘최신’ 버전으로 태그를 지정해야 합니다. 이렇게 하려면 다음 명령을 실행합니다.

```none
docker tag microsoft/nanoserver:10.0.14300.1030 nanoserver:latest
```

Windows 컨테이너 이미지에 대한 자세한 내용은 [컨테이너 이미지 관리](../management/manage_images.md)를 참조하세요.

## 4. 첫 번째 컨테이너 배포

이 간단한 예제에서는 'Hello World' 컨테이너 이미지를 만들고 배포합니다. 최상의 환경을 유지하려면 관리자 권한 Windows CMD 셸에서 이 명령을 실행합니다.

먼저 `nanoserver` 이미지에서 대화형 세션을 사용하여 컨테이너를 시작합니다. 컨테이너가 시작되면 컨테이너 내에서 명령 셸이 표시됩니다.  

```none
docker run -it nanoserver cmd
```

컨테이너 내에서 간단한 'Hello World' 스크립트를 만듭니다.

```none
powershell.exe Add-Content C:\helloworld.ps1 'Write-Host "Hello World"'
```   

만들었으면 컨테이너를 종료합니다.

```none
exit
```

이제 수정된 컨테이너에서 새 컨테이너 이미지를 만듭니다. 컨테이너 목록을 보려면 다음을 실행하고 컨테이너 ID를 적어둡니다.

```none
docker ps -a
```

다음 명령을 실행하여 새 ‘HelloWorld’ 이미지를 만듭니다. <containerid>를 컨테이너 ID로 바꿉니다.

```none
docker commit <containerid> helloworld
```

완료되면 hello world 스크립트가 포함된 사용자 지정 이미지가 생깁니다. 이 이미지를 확인하려면 다음 명령을 사용합니다.

```none
docker images
```

마지막으로 컨테이너를 실행하려면 `docker run` 명령을 사용입니다.

```none
docker run --rm helloworld powershell c:\helloworld.ps1
```

`docker run` 명령의 결과로 'HelloWorld' 이미지에서 Hyper-V 컨테이너를 만들고 샘플 'Hello World' 스크립트를 실행(출력은 셸에 에코됨)한 다음 컨테이너를 중지하고 제거했습니다. 이후 Windows 10 및 컨테이너 빠른 시작에서는 Windows 10의 컨테이너에서 응용 프로그램을 만들고 배포하는 과정을 자세히 살펴봅니다.

## 다음 단계

[Windows Server의 Windows 컨테이너](./quick_start_windows_server.md)





<!--HONumber=Jul16_HO2-->


