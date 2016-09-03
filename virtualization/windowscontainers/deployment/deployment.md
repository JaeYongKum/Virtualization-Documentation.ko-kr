---
title: "Windows Server에 Windows 컨테이너 배포"
description: "Windows Server에 Windows 컨테이너 배포"
keywords: "Docker, 컨테이너"
author: neilpeterson
manager: timlt
ms.date: 08/22/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
translationtype: Human Translation
ms.sourcegitcommit: 2319649d1dd39677e59a9431fbefaf82982492c6
ms.openlocfilehash: b60329a09ea0f119446fa2aa20de68e3edc2b245

---

# 컨테이너 호스트 배포 - Windows Server

**이 예비 콘텐츠는 변경될 수 있습니다.**

Windows 컨테이너 호스트 배포 단계는 운영 체제와 호스트 시스템 유형(물리적 또는 가상)에 따라 다릅니다. 이 문서에서는 물리적 시스템이나 가상 시스템에서 Windows Server 2016 또는 Windows Server Core 2016에 Windows 컨테이너 호스트를 배포하는 방법에 대해 자세히 설명합니다.

## Azure 이미지 

완전히 구성된 Windows Server 이미지를 Azure에서 사용할 수 있습니다. 이 이미지를 사용하려면 아래 단추를 클릭하여 가상 컴퓨터를 배포하세요. 이 템플릿을 사용하여 Windows 컨테이너 시스템을 Azure에 배포하는 경우 이 문서의 나머지 부분을 건너뛸 수 있습니다.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Fmaster%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

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

Docker 엔진과 클라이언트를 Zip 보관 파일로 다운로드합니다.

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-1.12.0.zip" -OutFile "$env:TEMP\docker-1.12.0.zip" -UseBasicParsing
```

Zip 보관 파일을 프로그램 파일로 확장, 보관 파일 콘텐츠는 이미 Docker 디렉터리에 있습니다.

```none
Expand-Archive -Path "$env:TEMP\docker-1.12.0.zip" -DestinationPath $env:ProgramFiles
```

다음 두 명령을 실행하여 시스템 경로에 Docker 디렉터리를 추가합니다.

```none
# for quick use, does not require shell to be restarted
$env:path += ";c:\program files\docker"

# for persistent use, will apply even after a reboot 
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

Windows 컨테이너를 사용하기 전에 먼저 기본 이미지를 설치해야 합니다. 기본 이미지는 기본 운영 체제로 Windows Server Core 또는 Nano Server에서 사용할 수 있습니다. Windows 컨테이너 이미지에 대한 자세한 내용은 [컨테이너 이미지 관리](../management/manage_images.md)를 참조하세요.

Windows Server Core 기본 이미지를 설치하려면 다음을 실행합니다.

```none
docker pull microsoft/windowsservercore
```

Nano Server 기본 이미지를 설치하려면 다음을 실행합니다.

```none
docker pull microsoft/nanoserver
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



<!--HONumber=Aug16_HO4-->


