---
title: Windows 10의 windows 및 Linux 컨테이너
description: 컨테이너에 대해 Windows 10 또는 Windows Server를 설정 하 고 첫 번째 컨테이너 이미지 실행으로 이동 합니다.
keywords: docker, 컨테이너, LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 2c52dd96b3bf2402d41ec5b178af36521d00a649
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909563"
---
# <a name="get-started-prep-windows-for-containers"></a>시작 하기: 컨테이너에 대 한 Windows 준비

이 자습서에서는 다음을 수행 하는 방법을 설명 합니다.

- 컨테이너에 대해 Windows 10 또는 Windows Server 설정
- 첫 번째 컨테이너 이미지 실행
- 간단한 .NET core 응용 프로그램 컨테이너 화

## <a name="prerequisites"></a>필수 구성 요소

<!-- start tab view -->
# <a name="windows-servertabwindows-server"></a>[Windows Server](#tab/Windows-Server)

Windows Server에서 컨테이너를 실행 하려면 Windows Server (반기 채널), Windows Server 2019 또는 Windows Server 2016를 실행 하는 물리적 서버 또는 가상 머신이 필요 합니다.

테스트를 위해 [Window server 2019 Evaluation](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 ) 또는 [Windows server Insider preview](https://insider.windows.com/for-business-getting-started-server/)의 복사본을 다운로드할 수 있습니다.

# <a name="windows-10tabwindows-10-client"></a>[Windows 10](#tab/Windows-10-Client)

Windows 10에서 컨테이너를 실행 하려면 다음이 필요 합니다.

- 기념일 업데이트 (버전 1607) 이상에서 Windows 10 Professional 또는 Enterprise를 실행 하는 물리적 컴퓨터 시스템이 하나 이상 있습니다.
- [Hyper-v](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) 를 사용 하도록 설정 해야 합니다.

> [!NOTE]
>  Windows 10 10 월 업데이트 2018 부터는 개발/테스트용으로 windows 10 Enterprise 또는 Professional에서 Windows 컨테이너를 실행 하는 것을 더 이상 허용 하지 않습니다. 자세히 알아보려면 [FAQ](../about/faq.md) 를 참조 하세요. 
> 
> Windows Server 컨테이너는 프로덕션에서 사용 되는 것과 동일한 커널 버전 및 구성을 개발자에 게 제공 하기 위해 기본적으로 Windows 10에서 Hyper-v 격리를 사용 합니다. 문서의 [개념](../manage-containers/hyperv-container.md) 영역에서 hyper-v 격리에 대해 자세히 알아보세요.

---
<!-- stop tab view -->

## <a name="install-docker"></a>Docker 설치

첫 번째 단계는 Windows 컨테이너를 사용 하는 데 필요한 Docker를 설치 하는 것입니다. Docker는 공용 API 및 CLI (명령줄 인터페이스)를 사용 하 여 컨테이너에 대 한 표준 런타임 환경을 제공 합니다.

자세한 구성 정보는 [Windows의 Docker 엔진](../manage-docker/configure-docker-daemon.md)을 참조 하세요.

<!-- start tab view -->
# <a name="windows-servertabwindows-server"></a>[Windows Server](#tab/Windows-Server)

Windows Server에 Docker를 설치 하려면 Microsoft에서 게시 한 [Oneget Provider PowerShell 모듈](https://github.com/oneget/oneget) ( [DockerMicrosoftProvider](https://github.com/OneGet/MicrosoftDockerProvider)이라고 함)을 사용할 수 있습니다. 이 공급자는 Windows의 컨테이너 기능을 사용 하도록 설정 하 고 Docker 엔진과 클라이언트를 설치 합니다. 방법은 다음과 같습니다.

1. 관리자 권한 PowerShell 세션을 열고 [PowerShell 갤러리](https://www.powershellgallery.com/packages/DockerMsftProvider)에서 Docker-Microsoft PackageManagement 공급자를 설치 합니다.

   ```powershell
   Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
   ```

   NuGet 공급자를 설치할지 묻는 메시지가 표시 되 면 `Y`를 입력 하 여 설치 합니다.

2. PackageManagement PowerShell 모듈을 사용 하 여 최신 버전의 Docker를 설치 합니다.

   ```powershell
   Install-Package -Name docker -ProviderName DockerMsftProvider
   ```

   PowerShell에서 'DockerDefault' 패키지 소스를 신뢰할 수 있는지 물으면 `A`를 입력하여 설치를 계속합니다.
3. 설치가 완료 되 면 컴퓨터를 다시 시작 합니다.

   ```powershell
   Restart-Computer -Force
   ```

Docker를 나중에 업데이트 하려면 다음을 수행 합니다.

- `Get-Package -Name Docker -ProviderName DockerMsftProvider`를 사용 하 여 설치 된 버전 확인
- `Find-Package -Name Docker -ProviderName DockerMsftProvider`를 사용 하 여 현재 버전 찾기
- 준비가 되 면 `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force`로 업그레이드 한 다음 `Start-Service Docker`

# <a name="windows-10tabwindows-10-client"></a>[Windows 10](#tab/Windows-10-Client)

다음 단계를 사용 하 여 Windows 10 Professional 및 Enterprise edition에 Docker를 설치할 수 있습니다. 

1. [Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows)을 다운로드 하 여 설치 합니다. 아직 없는 경우 무료 docker 계정을 만듭니다. 자세한 내용은 [Docker 설명서](https://docs.docker.com/docker-for-windows/install)를 참조 하세요.

2. 설치 하는 동안 기본 컨테이너 유형을 Windows 컨테이너로 설정 합니다. 설치가 완료 된 후 전환 하려면 Windows 시스템 트레이의 Docker 항목 (아래 참조)을 사용 하거나 PowerShell 프롬프트에서 다음 명령을 사용 하면 됩니다.

   ```console
   & $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
   ```

!["Windows 컨테이너로 전환" 명령을 보여 주는 Docker 시스템 트레이 메뉴입니다.](./media/docker-for-win-switch.png)

---
<!-- stop tab view -->

## <a name="next-steps"></a>다음 단계

이제 환경을 올바르게 구성 했으므로 링크를 따라 컨테이너를 실행 하는 방법을 알아보세요.

> [!div class="nextstepaction"]
> [첫 번째 컨테이너 실행](./run-your-first-container.md)
