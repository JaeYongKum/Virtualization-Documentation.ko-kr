---
title: "Windows Server에 Windows 컨테이너 배포"
description: "Windows Server에 Windows 컨테이너 배포"
keywords: "Docker, 컨테이너"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
ms.openlocfilehash: 1db66d5dfe0f0060c56625e396ecff3bb72e3189
ms.sourcegitcommit: 456485f36ed2d412cd708aed671d5a917b934bbe
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/08/2017
---
# <a name="container-host-deployment---windows-server"></a>컨테이너 호스트 배포 - Windows Server

Windows 컨테이너 호스트 배포 단계는 운영 체제와 호스트 시스템 유형(물리적 또는 가상)에 따라 다릅니다. 이 문서에서는 물리적 시스템이나 가상 시스템에서 Windows Server 2016 또는 Windows Server Core 2016에 Windows 컨테이너 호스트를 배포하는 방법에 대해 자세히 설명합니다.

## <a name="install-docker"></a>Docker 설치

Windows 컨테이너를 사용하려면 Docker가 필요합니다. Docker는 Docker 엔진 및 Docker 클라이언트로 구성됩니다. 

Docker를 설치하기 위해 [OneGet provider PowerShell module](https://github.com/OneGet/MicrosoftDockerProvider)(OneGet 공급자 PowerShell 모듈)을 사용합니다. 공급자는 컴퓨터에서 컨테이너 기능을 사용하도록 설정하고 Docker를 설치합니다. 그러면 컴퓨터를 다시 부팅해야 합니다. 

관리자 권한으로 PowerShell 세션을 열고 다음 명령을 실행합니다.

OneGet PowerShell 모듈을 설치합니다.

```
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

OneGet을 사용하여 최신 버전의 Docker를 설치합니다.

```
Install-Package -Name docker -ProviderName DockerMsftProvider
```

설치가 완료되면 컴퓨터를 다시 부팅합니다.

```
Restart-Computer -Force
```

## <a name="install-base-container-images"></a>기본 컨테이너 이미지 설치

Windows 컨테이너를 사용하기 전에 먼저 기본 이미지를 설치해야 합니다. 기본 이미지는 컨테이너 운영 체제로 Windows Server Core 또는 Nano Server에서 사용할 수 있습니다. Docker 컨테이너 이미지에 대한 자세한 내용은 [Build your own images on docker.com](https://docs.docker.com/engine/tutorials/dockerimages/)(docker.com에서 고유한 이미지 만들기)을 참조하세요.

Windows Server Core 기본 이미지를 설치하려면 다음을 실행합니다.

```
docker pull microsoft/windowsservercore
```

Nano Server 기본 이미지를 설치하려면 다음을 실행합니다.

```
docker pull microsoft/nanoserver
```

> [EULA](../images-eula.md)에서 Windows 컨테이너 OS 이미지 EULA를 읽어보세요.

## <a name="hyper-v-container-host"></a>Hyper-V 컨테이너 호스트

Hyper-V 컨테이너를 실행하려면 Hyper-V 역할이 필요합니다. Windows 컨테이너 호스트 자체가 Hyper-V 가상 컴퓨터인 경우 Hyper-V 역할을 설치하기 전에 중첩된 가상화를 사용하도록 설정해야 합니다. 중첩된 가상화에 대한 자세한 내용은 [중첩된 가상화]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)를 참조하세요.

### <a name="nested-virtualization"></a>중첩된 가상화

다음 스크립트는 컨테이너 호스트에 대한 중첩된 가상화를 구성합니다. 이 스크립트는 부모 Hyper-V 컴퓨터에서 실행됩니다. 이 스크립트를 실행할 때 컨테이너 호스트 가상 컴퓨터가 꺼져 있는지 확인하세요.

```
#replace with the virtual machine name
$vm = "<virtual-machine>"

#configure virtual processor
Set-VMProcessor -VMName $vm -ExposeVirtualizationExtensions $true -Count 2

#disable dynamic memory
Set-VMMemory $vm -DynamicMemoryEnabled $false

#enable mac spoofing
Get-VMNetworkAdapter -VMName $vm | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### <a name="enable-the-hyper-v-role"></a>Hyper-V 역할 사용

PowerShell을 사용하여 Hyper-V 기능을 사용하도록 설정하려면 관리자 권한 PowerShell 세션에서 다음 명령을 실행합니다.

```
Install-WindowsFeature hyper-v
```
