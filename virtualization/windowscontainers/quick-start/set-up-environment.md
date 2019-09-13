---
title: Windows 10의 windows 및 Linux 컨테이너
description: 컨테이너 배포 빠른 시작
keywords: docker, 컨테이너, LCOW
author: cwilhit
ms.date: 09/11/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 5f0922a1ee2588b6e5a06091fe34e07ceadf89cb
ms.sourcegitcommit: 868a64eb97c6ff06bada8403c6179185bf96675f
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/13/2019
ms.locfileid: "10129374"
---
# <a name="get-started-configure-your-environment-for-containers"></a>시작: 컨테이너에 대 한 환경 구성

이 퀵 스타트는 다음을 수행 하는 방법을 보여 줍니다.

> [!div class="checklist"]
> * 컨테이너에 대 한 환경 설정
> * 첫 번째 컨테이너 이미지를 실행 합니다.
> * 간단한 .NET 핵심 응용 프로그램 Containerize

## <a name="prerequisites"></a>필수 구성 요소

<!-- start tab view -->
# [<a name="windows-server"></a>Windows Server](#tab/Windows-Server)

다음 요구 사항을 충족 하는지 확인 하세요.

- Windows Server 2016 이상을 실행 하는 컴퓨터 시스템 (물리적 또는 가상) 1 개.

> [!NOTE]
> Windows Server 2019 Insider Preview를 사용 하는 경우 [Window server 2019 평가](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 )로 업데이트 하세요.

# [<a name="windows-10-professional-and-enterprise"></a>Windows 10 Professional 및 Enterprise](#tab/Windows-10-Client)

다음 요구 사항을 충족 하는지 확인 하세요.

- Windows 10 Professional 또는 Enterprise for 기념일 업데이트 (버전 1607) 이상을 실행 하는 물리적 컴퓨터 시스템 한 대
- [Hyper-v](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) 를 사용 하도록 설정 해야 합니다.

> [!NOTE]
>  Windows 10 10 월 업데이트 2018부터 개발자/테스트 목적으로 더 이상 windows 10 Enterprise 또는 Professional의 프로세스 격리 모드에서 Windows 컨테이너를 실행 하는 것을 허용 하지 않습니다. 자세히 알아보려면 [FAQ](../about/faq.md) 를 참조 하세요. 
> 
> Windows Server 컨테이너는 프로덕션에 사용 되는 것과 동일한 커널 버전과 구성을 제공 하기 위해 Windows 10에서 기본적으로 Hyper-v 격리를 사용 합니다. 이 문서의 [개념](../manage-containers/hyperv-container.md) 영역에서 hyper-v 격리에 대해 자세히 알아보세요.

---
<!-- stop tab view -->

## <a name="install-docker"></a>Docker 설치

Docker는 Windows 컨테이너 작업을 위한 결정적인 도구 체인. Docker는 사용자가 지정 된 호스트, 빌드 컨테이너, 컨테이너 제거 등의 컨테이너를 관리 하는 CLI를 제공 합니다. 이 문서의 [개념](../manage-containers/configure-docker-daemon.md) 영역에서 Docker에 대해 자세히 알아보세요.

<!-- start tab view -->
# [<a name="windows-server"></a>Windows Server](#tab/Windows-Server)

Windows Server에서 Docker는 Microsoft에서 [DockerMicrosoftProvider](https://github.com/OneGet/MicrosoftDockerProvider)이라고 하는 [Oneget 공급자 PowerShell 모듈](https://github.com/oneget/oneget) 을 통해 설치 됩니다. 이 공급자:

- 컴퓨터에서 컨테이너 기능을 사용 하도록 설정 합니다.
- 컴퓨터에 Docker 엔진과 클라이언트를 설치 합니다.

Docker를 설치 하려면 관리자 권한 PowerShell 세션을 열고 [PowerShell 갤러리](https://www.powershellgallery.com/packages/DockerMsftProvider)에서 Docker-Microsoft PackageManagement 공급자를 설치 합니다.

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

다음으로 PackageManagement PowerShell 모듈을 사용 하 여 최신 버전의 Docker를 설치 합니다.

```powershell
Install-Package -Name docker -ProviderName DockerMsftProvider
```

PowerShell에서 'DockerDefault' 패키지 소스를 신뢰할 수 있는지 물으면 `A`를 입력하여 설치를 계속합니다. 설치가 완료 되 면 컴퓨터를 다시 부팅 해야 합니다.

```powershell
Restart-Computer -Force
```

> [!TIP]
> 나중에 Docker를 업데이트 하려면 다음을 수행 합니다.
>  - 다음을 사용하여 설치된 버전을 확인 `Get-Package -Name Docker -ProviderName DockerMsftProvider`
>  - 다음을 사용하여 현재 버전을 검색 `Find-Package -Name Docker -ProviderName DockerMsftProvider`
>  - 준비를 마쳤으면 `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force` 명령을 사용하여 업그레이드, 그리고 `Start-Service Docker`

# [<a name="windows-10-professional-and-enterprise"></a>Windows 10 Professional 및 Enterprise](#tab/Windows-10-Client)

Windows 10 Professional 및 Enterprise에서는 클래식 설치 관리자를 통해 Docker가 설치 됩니다. [Docker 데스크톱](https://store.docker.com/editions/community/docker-ce-desktop-windows) 을 다운로드 하 고 설치 관리자를 실행 합니다. 로그인 하는 것이 필요 합니다. 아직 계정이 없는 경우 계정을 만듭니다. 자세한 설치 지침은 [Docker 설명서](https://docs.docker.com/docker-for-windows/install)에서 확인할 수 있습니다.

설치 후에 Docker 데스크톱은 기본적으로 Linux 컨테이너를 실행 합니다. Docker 트레이 메뉴 중 하나를 사용 하거나 PowerShell 프롬프트에서 다음 명령을 실행 하 여 Windows 컨테이너로 전환 합니다.

```console
& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
```

![](./media/docker-for-win-switch.png)

---
<!-- stop tab view -->

## <a name="next-steps"></a>다음 단계

이제 환경이 올바르게 구성 되었으므로 링크를 따라 컨테이너를 가져오고 실행 하는 방법을 알아보세요.

> [!div class="nextstepaction"]
> [첫 번째 컨테이너 실행](./run-your-first-container.md)
