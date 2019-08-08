---
title: Windows Server에 Windows 컨테이너 배포
description: Windows Server에 Windows 컨테이너 배포
keywords: Docker, 컨테이너
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
ms.openlocfilehash: e045539b189eb8cd1594da0784ab0c88e848c948
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998780"
---
# <a name="container-host-deployment-windows-server"></a>컨테이너 호스트 배포: Windows Server

Windows 컨테이너 호스트 배포 단계는 운영 체제와 호스트 시스템 유형(물리적 또는 가상)에 따라 다릅니다. 이 문서에서는 물리적 시스템이나 가상 시스템에서 Windows Server 2016 또는 Windows Server Core 2016에 Windows 컨테이너 호스트를 배포하는 방법에 대해 자세히 설명합니다.

## <a name="install-docker"></a>Docker 설치

Windows 컨테이너를 사용하려면 Docker가 필요합니다. Docker는 Docker 엔진과 Docker 클라이언트로 구성 됩니다.

Docker를 설치 하려면 [Oneget Provider PowerShell 모듈](https://github.com/OneGet/MicrosoftDockerProvider)을 사용 합니다. 공급자는 컴퓨터에서 컨테이너 기능을 사용 하도록 설정 하 고 다시 부팅 해야 하는 Docker를 설치 합니다.

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

현재 Windows Server 용 Docker EE에 사용할 수 있는 채널은 두 가지가 있습니다.

* `17.06` -Docker 엔터프라이즈 버전 (Docker 엔진, UCP, DTR)을 사용 중인 경우이 버전을 사용 합니다. `17.06` 이 기본값입니다.
* `18.03` -Docker EE 엔진을 단독으로 실행 하는 경우이 버전을 사용 합니다.

특정 버전을 설치 하려면 다음 플래그를 `RequiredVersion` 사용 합니다.

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Force -RequiredVersion 18.03
```

특정 Docker EE 버전을 설치 하려면 이전에 설치 된 DockerMsftProvider 모듈을 업데이트 해야 할 수 있습니다. 업데이트 대상:

```PowerShell
Update-Module DockerMsftProvider
```

## <a name="update-docker"></a>Docker 업데이트

이전 채널에서 이후 채널로 Docker EE 엔진을 업데이트 해야 하는 경우 및 `-Update` `-RequiredVersion` 플래그를 모두 사용 합니다.

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Update -Force -RequiredVersion 18.03
```

## <a name="install-base-container-images"></a>기본 컨테이너 이미지 설치

Windows 컨테이너를 사용하려면 기본 이미지를 설치해야 합니다. 기본 이미지는 컨테이너 운영 체제로 Windows Server Core 또는 Nano Server에서 사용할 수 있습니다. Docker 컨테이너 이미지에 대한 자세한 내용은 [Build your own images on docker.com](https://docs.docker.com/engine/tutorials/dockerimages/)(docker.com에서 고유한 이미지 만들기)을 참조하세요.

Windows Server 2019 릴리스를 사용 하는 경우 Microsoft의 컨테이너 이미지는 Microsoft 컨테이너 레지스트리 라는 새 레지스트리로 이동 합니다. Microsoft에서 게시 한 컨테이너 이미지는 Docker 허브를 통해 계속 검색 되어야 합니다. Windows Server 2019 및 그 이후를 사용 하 여 게시 된 새 컨테이너 이미지의 경우 MCR에서 잡아 당기는 것을 찾아야 합니다. Windows Server 2019 이전에 게시 된 이전 컨테이너 이미지의 경우에는 Docker의 레지스트리로부터 계속 해 서 가져옵니다.

### <a name="windows-server-2019-and-newer"></a>Windows Server 2019 이상

' Windows Server Core ' 기본 이미지를 설치 하려면 다음을 실행 합니다.

```PowerShell
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

' Nano 서버 ' 기본 이미지를 설치 하려면 다음을 실행 합니다.

```PowerShell
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

### <a name="windows-server-2016-versions-1607-1803"></a>Windows Server 2016 (버전 1607-1803)

Windows Server Core 기본 이미지를 설치하려면 다음을 실행합니다.

```PowerShell
docker pull microsoft/windowsservercore
```

Nano Server 기본 이미지를 설치하려면 다음을 실행합니다.

```PowerShell
docker pull microsoft/nanoserver
```

> 여기에 나와 있는 Windows 컨테이너 OS 이미지 EULA ( [eula](../images-eula.md))를 참조 하세요.

## <a name="hyper-v-isolation-host"></a>Hyper-v 격리 호스트

Hyper-v 격리를 실행 하려면 Hyper-v 역할이 있어야 합니다. Windows 컨테이너 호스트 자체가 Hyper-V 가상 컴퓨터인 경우 Hyper-V 역할을 설치하기 전에 중첩된 가상화를 사용하도록 설정해야 합니다. 중첩된 가상화에 대한 자세한 내용은 [중첩된 가상화](https://docs.microsoft.com/virtualization/hyper-v-on-windows/user-guide/nested-virtualization)를 참조하세요.

### <a name="nested-virtualization"></a>중첩 가상화

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
