---
title: Windows Server에 Windows 컨테이너 배포
description: Windows Server에 Windows 컨테이너 배포
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
---

# 컨테이너 호스트 배포 - Windows Server

**이 예비 콘텐츠는 변경될 수 있습니다.**

Windows 컨테이너 호스트 배포 단계는 운영 체제와 호스트 시스템 유형(물리적 또는 가상)에 따라 다릅니다. 이 문서에서는 물리적 시스템이나 가상 시스템에서 Windows Server 2016 또는 Windows Server Core 2016에 Windows 컨테이너 호스트를 배포하는 방법에 대해 자세히 설명합니다.

## 컨테이너 기능 설치

Windows 컨테이너를 사용하려면 먼저 컨테이너 기능을 사용하도록 설정해야 합니다. 이렇게 하려면 관리자 권한 PowerShell 세션에서 다음 명령을 실행합니다.

```none
Install-WindowsFeature containers
```

기능 설치가 완료되면 컴퓨터를 다시 부팅합니다.

```none
Restart-Computer -Force
```

## Docker 설치

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

## 기본 컨테이너 이미지 설치

컨테이너를 배포하려면 먼저 기본 OS 이미지를 다운로드해야 합니다. 다음 예제에서는 Windows Server Core 기본 OS 이미지를 다운로드합니다. 동일한 이 절차를 완료하여 Nano Server 기본 이미지를 설치할 수 있습니다. Windows 컨테이너 이미지에 대한 자세한 내용은 [컨테이너 이미지 관리](../management/manage_images.md)를 참조하세요.

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

마지막으로 이미지에 '최신' 버전으로 태그를 지정해야 합니다. 이렇게 하려면 다음 명령을 실행합니다.

```none
docker tag windowsservercore:10.0.14300.1000 windowsservercore:latest
```

## Hyper-V 컨테이너 호스트

Hyper-V 컨테이너를 실행하려면 Hyper-V 역할이 필요합니다. Windows 컨테이너 호스트 자체가 Hyper-V 가상 컴퓨터인 경우 Hyper-V 역할을 설치하기 전에 중첩된 가상화를 사용하도록 설정해야 합니다. 중첩된 가상화에 대한 자세한 내용은 [중첩된 가상화]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)를 참조하세요.

### 중첩된 가상화

다음 스크립트는 컨테이너 호스트에 대한 중첩된 가상화를 구성합니다. 이 스크립트는 부모 Hyper-V 컴퓨터에서 실행됩니다. 이 스크립트를 실행할 때 컨테이너 호스트 가상 컴퓨터가 꺼져 있는지 확인하세요.

```none
#replace with the virtual machine name
$vm = "<virtual-machine>"

#configure virtual processor
Set-VMProcessor -VMName $vm -ExposeVirtualizationExtensions $true -Count 2

#disable dynamic memory
Set-VMMemory $vm -DynamicMemoryEnabled $false

#enable mac spoofing
Get-VMNetworkAdapter -VMName $vm | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### Hyper-V 역할 사용

PowerShell을 사용하여 Hyper-V 기능을 사용하도록 설정하려면 관리자 권한 PowerShell 세션에서 다음 명령을 실행합니다.

```none
Install-WindowsFeature hyper-v
```


<!--HONumber=Jun16_HO2-->


