---
title: Windows Server에 Windows 컨테이너 배포
description: Windows Server에 Windows 컨테이너 배포
keywords: Docker, 컨테이너
author: taylorb-microsoft
ms.date: 09/09/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
ms.openlocfilehash: 9899a2d76bfa1fe312e3bd983f60d09d77c272e9
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853915"
---
# <a name="container-host-deployment-windows-server"></a>컨테이너 호스트 배포: Windows Server

Windows 컨테이너 호스트 배포 단계는 운영 체제와 호스트 시스템 유형(물리적 또는 가상)에 따라 다릅니다. 이 문서에서는 물리적 시스템이나 가상 시스템에서 Windows Server 2016 또는 Windows Server Core 2016에 Windows 컨테이너 호스트를 배포하는 방법에 대해 자세히 설명합니다.

## <a name="install-docker"></a>Docker 설치

Windows 컨테이너를 사용하려면 Docker가 필요합니다. Docker는 docker 엔진과 Docker 클라이언트로 구성 됩니다.

Docker를 설치 하기 위해 [Oneget 공급자 PowerShell 모듈](https://github.com/OneGet/MicrosoftDockerProvider)을 사용 합니다. 공급자는 컴퓨터에서 컨테이너 기능을 사용 하도록 설정 하 고 Docker를 설치 합니다 .이 기능을 다시 부팅 해야 합니다.

관리자 권한 PowerShell 세션을 열고 다음 cmdlet을 실행 합니다.

OneGet PowerShell 모듈을 설치합니다.

```PowerShell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

OneGet을 사용하여 최신 버전의 Docker를 설치합니다.

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider
```

설치가 완료되면 컴퓨터를 다시 부팅합니다.

```PowerShell
Restart-Computer -Force
```

## <a name="install-a-specific-version-of-docker"></a>특정 버전의 Docker 설치

현재 Windows Server 용 Docker EE에 사용할 수 있는 두 개의 채널이 있습니다.

* `17.06`-Docker Enterprise Edition (Docker 엔진, UCP, DTR)을 사용 하는 경우이 버전을 사용 합니다. 기본값은 `17.06`입니다.
* `18.03`-Docker EE 엔진만 실행 하는 경우이 버전을 사용 합니다.

특정 버전을 설치 하려면 `RequiredVersion` 플래그를 사용 합니다.

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Force -RequiredVersion 18.03
```

특정 Docker EE 버전을 설치 하려면 이전에 설치한 DockerMsftProvider 모듈에 대 한 업데이트가 필요할 수 있습니다. 업데이트 하려면:

```PowerShell
Update-Module DockerMsftProvider
```

## <a name="update-docker"></a>Docker 업데이트

이전 채널에서 이후 채널로 Docker EE 엔진을 업데이트 해야 하는 경우 `-Update` 및 `-RequiredVersion` 플래그를 모두 사용 합니다.

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Update -Force -RequiredVersion 18.03
```

## <a name="install-base-container-images"></a>기본 컨테이너 이미지 설치

Windows 컨테이너를 사용하려면 기본 이미지를 설치해야 합니다. 기본 이미지는 컨테이너 운영 체제로 Windows Server Core 또는 Nano Server에서 사용할 수 있습니다. Docker 컨테이너 이미지에 대한 자세한 내용은 [Build your own images on docker.com](https://docs.docker.com/engine/tutorials/dockerimages/)(docker.com에서 고유한 이미지 만들기)을 참조하세요.

> [!TIP]
> 2018 년 5 월부터 일관 되 고 신뢰할 수 있는 구매 환경을 제공 하 여 대부분의 Microsoft 원본 컨테이너 이미지는 Microsoft Container Registry _mcr.microsoft.com_에서 제공 되지만 [_Docker 허브_](https://hub.docker.com/publishers/microsoftowner)를 통해 현재 검색 프로세스를 유지 관리 합니다.

### <a name="windows-server-2019-and-newer"></a>Windows Server 2019 이상

' Windows Server Core ' 기본 이미지를 설치 하려면 다음을 실행 합니다.

```PowerShell
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

' Nano Server ' 기본 이미지를 설치 하려면 다음을 실행 합니다.

```PowerShell
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

### <a name="windows-server-2016-versions-1607-1803"></a>Windows Server 2016 (버전 1607-1803)

Windows Server Core 기본 이미지를 설치하려면 다음을 실행합니다.

```PowerShell
docker pull mcr.microsoft.com/windows/servercore:1607
```

Nano Server 기본 이미지를 설치하려면 다음을 실행합니다.

```PowerShell
docker pull mcr.microsoft.com/windows/nanoserver:1803
```

> Windows 컨테이너 OS 이미지 EULA를 읽어 보세요. 여기 – [eula](../images-eula.md)에서 찾을 수 있습니다.

## <a name="hyper-v-isolation-host"></a>Hyper-v 격리 호스트

Hyper-v 격리를 실행 하려면 Hyper-v 역할이 있어야 합니다. Windows 컨테이너 호스트 자체가 Hyper-V 가상 컴퓨터인 경우 Hyper-V 역할을 설치하기 전에 중첩된 가상화를 사용하도록 설정해야 합니다. 중첩된 가상화에 대한 자세한 내용은 [중첩된 가상화](https://docs.microsoft.com/virtualization/hyper-v-on-windows/user-guide/nested-virtualization)를 참조하세요.

### <a name="nested-virtualization"></a>중첩 된 가상화

다음 스크립트는 컨테이너 호스트에 대한 중첩된 가상화를 구성합니다. 이 스크립트는 부모 Hyper-V 컴퓨터에서 실행됩니다. 이 스크립트를 실행할 때 컨테이너 호스트 가상 컴퓨터가 꺼져 있는지 확인하세요.

```PowerShell
#replace with the virtual machine name
$vm = "<virtual-machine>"

#configure virtual processor
Set-VMProcessor -VMName $vm -ExposeVirtualizationExtensions $true -Count 2

#disable dynamic memory
Set-VMMemory -VMName $vm -DynamicMemoryEnabled $false

#enable mac spoofing
Get-VMNetworkAdapter -VMName $vm | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### <a name="enable-the-hyper-v-role"></a>Hyper-V 역할 사용

PowerShell을 사용 하 여 Hyper-v 기능을 사용 하도록 설정 하려면 관리자 권한 PowerShell 세션에서 다음 cmdlet을 실행 합니다.

```PowerShell
Install-WindowsFeature hyper-v
```
