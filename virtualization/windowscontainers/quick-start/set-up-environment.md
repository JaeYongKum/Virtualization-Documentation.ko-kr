---
title: Windows 10의 Windows 및 Linux 컨테이너
description: 컨테이너에 대해 Windows 10 또는 Windows Server를 설정한 다음, 첫 번째 컨테이너 이미지를 실행합니다.
keywords: Docker, 컨테이너, LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 13d8f1ead90b2c2c86afe9f596717c1c09905895
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/24/2020
ms.locfileid: "80929987"
---
# <a name="get-started-prep-windows-for-containers"></a>시작: 컨테이너에 맞게 Windows 준비

이 자습서에서는 다음 작업 방법을 설명합니다.

- 컨테이너에 대해 Windows 10 또는 Windows Server 설정
- 첫 번째 컨테이너 이미지 실행
- 간단한 .NET Core 애플리케이션 컨테이너화

## <a name="prerequisites"></a>필수 구성 요소

<!-- start tab view -->
# <a name="windows-server"></a>[Windows Server](#tab/Windows-Server)

Windows Server에서 컨테이너를 실행하려면 Windows Server(반기 채널), Windows Server 2019 또는 Windows Server 2016을 실행하는 물리적 서버 또는 가상 머신이 필요합니다.

테스트를 위해 [Window Server 2019 Evaluation](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 ) 또는 [Windows Server Insider Preview](https://insider.windows.com/for-business-getting-started-server/)의 복사본을 다운로드할 수 있습니다.

# <a name="windows-10"></a>[Windows 10](#tab/Windows-10-Client)

Windows 10에서 컨테이너를 실행하려면 다음 항목이 필요합니다.

- 1주년 업데이트(버전 1607) 이상이 적용된 Windows 10 Professional 또는 Enterprise를 실행하는 물리적 컴퓨터 시스템 1대.
- [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements)를 사용하도록 설정해야 합니다.

> [!NOTE]
>  Windows 10 2018년 10월 업데이트부터 사용자가 개발/테스트 용도로 Windows 10 Enterprise 또는 Professional에서 Windows 컨테이너를 프로세스 격리 모드로 실행하는 것을 더 이상 허용하지 않습니다. 자세한 내용은 [FAQ](../about/faq.md)를 참조하세요. 
> 
> 프로덕션에 사용되는 동일한 커널 버전 및 구성을 개발자에게 제공하기 위해 Windows Server 컨테이너는 기본적으로 Windows 10에서 Hyper-V 격리를 사용합니다. 설명서의 [개념](../manage-containers/hyperv-container.md) 영역에서 Hyper-V 격리에 대해 자세히 알아보세요.

---
<!-- stop tab view -->

## <a name="install-docker"></a>Docker 설치

첫 번째 단계는 Windows 컨테이너를 사용하는 데 필요한 Docker를 설치하는 것입니다. Docker는 공용 API 및 CLI(명령줄 인터페이스)를 통해 컨테이너용 표준 런타임 환경을 제공합니다.

구성에 대한 자세한 내용은 [Windows의 Docker 엔진](../manage-docker/configure-docker-daemon.md)을 참조하세요.

<!-- start tab view -->
# <a name="windows-server"></a>[Windows Server](#tab/Windows-Server)

Windows Server에 Docker를 설치하려면 Microsoft에서 게시한 [DockerMicrosoftProvider](https://github.com/OneGet/MicrosoftDockerProvider)라고 하는 [OneGet 공급자 PowerShell 모듈](https://github.com/oneget/oneget)을 사용하면 됩니다. 이 공급자는 Windows에서 컨테이너 기능을 사용하도록 설정하고 Docker 엔진과 클라이언트를 설치합니다. 방법은 다음과 같습니다.

1. [PowerShell 갤러리](https://www.powershellgallery.com/packages/DockerMsftProvider)에서 관리자 권한으로 PowerShell 세션을 열고 Docker-Microsoft PackageManagement Provider를 설치합니다.

   ```powershell
   Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
   ```

   NuGet 공급자를 설치하라는 메시지가 표시되 면 `Y`를 입력하여 설치합니다.

2. PackageManagement PowerShell 모듈을 사용하여 최신 버전의 Docker를 설치합니다.

   ```powershell
   Install-Package -Name docker -ProviderName DockerMsftProvider
   ```

   PowerShell에서 'DockerDefault' 패키지 소스를 신뢰할 수 있는지 물으면 `A`를 입력하여 설치를 계속합니다.
3. 설치가 완료되면 컴퓨터를 다시 시작합니다.

   ```powershell
   Restart-Computer -Force
   ```

Docker를 나중에 업데이트하려면 다음을 수행합니다.

- `Get-Package -Name Docker -ProviderName DockerMsftProvider`를 사용하여 설치된 버전을 확인
- `Find-Package -Name Docker -ProviderName DockerMsftProvider`를 사용하여 현재 버전을 확인
- 준비를 마쳤으면 `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force`, `Start-Service Docker`를 차례로 사용하여 업그레이드

# <a name="windows-10"></a>[Windows 10](#tab/Windows-10-Client)

다음 단계에 따라 Windows 10 Professional 및 Enterprise 버전에 Docker를 설치할 수 있습니다. 

1. [Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows)을 다운로드하여 설치하고, 아직 Docker 체험 계정이 없는 경우 하나 만듭니다. 자세한 내용은 [Docker 설명서](https://docs.docker.com/docker-for-windows/install)를 참조하세요.

2. 설치하는 동안 기본 컨테이너 유형을 Windows 컨테이너로 설정합니다. 설치가 완료된 후 유형을 전환하려면 Windows 시스템 트레이의 Docker 항목(아래 참조)을 사용하거나 PowerShell 프롬프트에서 다음 명령을 사용하면 됩니다.

   ```console
   & $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
   ```

!["Windows 컨테이너로 전환" 명령을 보여주는 Docker 시스템 트레이 메뉴](./media/docker-for-win-switch.png)

---
<!-- stop tab view -->

## <a name="next-steps"></a>다음 단계

이제 환경을 올바르게 구성했으므로 링크를 따라 컨테이너를 실행하는 방법을 알아보세요.

> [!div class="nextstepaction"]
> [첫 번째 컨테이너 실행](./run-your-first-container.md)
