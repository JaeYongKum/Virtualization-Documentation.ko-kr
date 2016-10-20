---
title: "Windows Server에 Windows 컨테이너 배포"
description: "Windows Server에 Windows 컨테이너 배포"
keywords: "Docker, 컨테이너"
author: neilpeterson
manager: timlt
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
translationtype: Human Translation
ms.sourcegitcommit: 41561cacc8c2531f1351154d85861f1712182c9a
ms.openlocfilehash: 091007ea301226ca98c93855d8c36b09f3e4d0be

---

# 컨테이너 호스트 배포 - Windows Server

Windows 컨테이너 호스트 배포 단계는 운영 체제와 호스트 시스템 유형(물리적 또는 가상)에 따라 다릅니다. 이 문서에서는 물리적 시스템이나 가상 시스템에서 Windows Server 2016 또는 Windows Server Core 2016에 Windows 컨테이너 호스트를 배포하는 방법에 대해 자세히 설명합니다.

## Docker 설치

Windows 컨테이너를 사용하려면 Docker가 필요합니다. Docker는 Docker 엔진 및 Docker 클라이언트로 구성됩니다. 

Docker를 설치하기 위해 [OneGet provider PowerShell module](https://github.com/oneget/oneget)(OneGet 공급자 PowerShell 모듈)을 사용합니다. 공급자는 컴퓨터에서 컨테이너 기능을 사용하도록 설정하고 Docker를 설치합니다. 그러면 컴퓨터를 다시 부팅해야 합니다. 

관리자 권한 PowerShell 세션을 열고 다음 명령을 실행합니다.

먼저 OneGet PowerShell 모듈을 설치합니다.

```none
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

그런 다음 OneGet을 사용하여 최신 버전의 Docker를 설치합니다.

```none
Install-Package -Name docker -ProviderName DockerMsftProvider
```

설치가 완료되면 컴퓨터를 다시 부팅합니다.

```none
Restart-Computer -Force
```

## 기본 컨테이너 이미지 설치

Windows 컨테이너를 사용하기 전에 먼저 기본 이미지를 설치해야 합니다. 기본 이미지는 기본 운영 체제로 Windows Server Core 또는 Nano Server에서 사용할 수 있습니다. Docker 컨테이너 이미지에 대한 자세한 내용은 [Build your own images on docker.com](https://docs.docker.com/engine/tutorials/dockerimages/)(docker.com에서 고유한 이미지 만들기)을 참조하세요.

Windows Server Core 기본 이미지를 설치하려면 다음을 실행합니다.

```none
docker pull microsoft/windowsservercore
```

Nano Server 기본 이미지를 설치하려면 다음을 실행합니다.

```none
docker pull microsoft/nanoserver
```

> [EULA](../Images_EULA.md)에서 Windows 컨테이너 OS 이미지 EULA를 읽어보세요.

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



<!--HONumber=Oct16_HO2-->


