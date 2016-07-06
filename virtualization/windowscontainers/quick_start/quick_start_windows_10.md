---
title: "Windows 10의 Windows 컨테이너"
description: "컨테이너 배포 빠른 시작"
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 06/28/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
translationtype: Human Translation
ms.sourcegitcommit: 3fc388632dee4d714ab5a7869fa852c079c11910
ms.openlocfilehash: e2d86c6f1aba07c7c40d1f932b3884c99bfd8f0a

---

# Windows 10의 Windows 컨테이너

**이 예비 콘텐츠는 변경될 수 있습니다.** 

이 연습에서는 Windows 10(참가자 빌드 14352 이상)에서 Windows 컨테이너 기능의 기본 배포 및 사용에 대해 안내합니다. 완료 후에는 컨테이너 역할이 설치되고 간단한 Hyper-V 컨테이너가 배포됩니다. 이 빠른 시작을 시작하기 전에 기본 컨테이너 개념과 용어를 잘 이해해야 합니다. 이 정보는 [빠른 시작 소개](./quick_start.md)에서 확인할 수 있습니다. 

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

## 2. Docker 설치

Windows 컨테이너를 사용하려면 Docker가 필요합니다. Docker는 Docker 엔진 및 Docker 클라이언트로 구성됩니다. 이 연습에서는 둘 다 설치됩니다. 이렇게 하려면 다음 명령을 실행합니다. 

Docker 실행 파일에 대한 폴더를 만듭니다.

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

시스템 경로에 Docker 디렉터리를 추가합니다.

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
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
    
현재 PowerShell 프로세스에 대한 PowerShell 실행 정책을 설정합니다. 이 정책은 현재 PowerShell 세션에서 실행되는 스크립트에만 영향을 주지만 실행 정책을 변경할 때는 주의해야 합니다.

```none
Set-ExecutionPolicy Bypass -scope Process
```

컨테이너 이미지 패키지 공급자를 설치합니다.

```none  
Install-PackageProvider ContainerImage -Force
```

그런 다음 Nano Server 이미지를 설치합니다.

```none
Install-ContainerImage -Name NanoServer
```

기본 이미지를 설치한 후에는 Docker 서비스를 다시 시작해야 합니다.

```none
Restart-Service docker
```

이 단계에서 `docker images`를 실행하면 설치된 이미지(이 경우 Nano Server 이미지) 목록이 반환됩니다.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14300.1016     3f5112ddd185        3 weeks ago         810.2 MB
```

계속하려면 먼저 이 이미지에 ‘최신’ 버전으로 태그를 지정해야 합니다. 이렇게 하려면 다음 명령을 실행합니다.

```none
docker tag nanoserver:10.0.14300.1016 nanoserver:latest
```

Windows 컨테이너 이미지에 대한 자세한 내용은 [컨테이너 이미지 관리](../management/manage_images.md)를 참조하세요.

## 4. 첫 번째 컨테이너 배포

이 간단한 예제에서는 .NET Core 이미지를 미리 만들었습니다. `docker pull` 명령을 사용하여 이 이미지를 다운로드합니다.

실행하면 컨테이너가 시작되고 간단한 .NET Core 응용 프로그램이 실행된 다음 컨테이너는 종료됩니다. 

```none
docker pull microsoft/sample-dotnet
```

이는 `docker images` 명령으로 확인할 수 있습니다.

```none
docker images

REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
microsoft/sample-dotnet  latest              28da49c3bff4        41 hours ago        918.3 MB
nanoserver               10.0.14300.1016     3f5112ddd185        3 weeks ago         810.2 MB
nanoserver               latest              3f5112ddd185        3 weeks ago         810.2 MB
```

`docker run` 명령으로 컨테이너를 실행합니다. 다음 예제에서는 `--rm` 매개 변수를 지정합니다. 이 매개 변수는 Docker 엔진이 더 이상 실행되는 않는 컨테이너를 삭제하도록 지시합니다. 

Docker Run 명령에 대한 자세한 내용은 [Docker.com의 Docker Run Reference(Docker Run 참조)]( https://docs.docker.com/engine/reference/run/)를 참조하세요.

```none
docker run --isolation=hyperv --rm microsoft/sample-dotnet
```

**참고** - 시간 제한 이벤트가 나타나면서 오류가 발생하는 경우 다음 PowerShell 스크립트를 실행하고 작업을 다시 시도하세요.

```none
Set-ItemProperty -Path 'HKLM:SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers' -Name VSmbDisableOplocks -Type DWord -Value 1 -Force
```

`docker run` 명령의 결과로 sample-dotnet 이미지에서 Hyper-V 컨테이너를 만들고 샘플 응용 프로그램을 실행(출력은 셸에 에코됨)한 다음 컨테이너를 중지하고 제거했습니다. 이후 Windows 10 및 컨테이너 빠른 시작에서는 Windows 10의 컨테이너에서 응용 프로그램을 만들고 배포하는 과정을 자세히 살펴봅니다.

## 다음 단계

[Windows Server의 Windows 컨테이너](./quick_start_windows_server.md)





<!--HONumber=Jun16_HO4-->


