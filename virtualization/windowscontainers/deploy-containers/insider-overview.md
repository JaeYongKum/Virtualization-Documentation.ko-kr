---
title: Windows 참가자 프로그램에서 컨테이너 사용
description: Windows 참가자 프로그램으로 windows 컨테이너를 사용 하 여 시작 하는 방법 알아보기
keywords: docker, 컨테이너, 참가자, windows
author: cwilhit
ms.openlocfilehash: 137209a66c3d0b907003498fe78a04a57a140130
ms.sourcegitcommit: d0411b05d1ef7328a770766b84fd0743f9d9c237
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/23/2019
ms.locfileid: "10254411"
---
# <a name="use-containers-with-the-windows-insider-program"></a>Windows 참가자 프로그램에서 컨테이너 사용

이 연습에서는 Windows Insider Preview 프로그램의 최신 Windows Server 참가자 빌드에 Windows 컨테이너 기능을 배포하고 사용하는 방법을 안내합니다. 이 연습을 진행하는 과정에서 컨테이너 역할을 설치하고 기본 OS 이미지의 미리 보기 버전을 배포할 것입니다. 컨테이너에 대해 좀 더 숙지해야 하는 경우 [컨테이너 정보](../about/index.md)에서 이 정보를 찾을 수 있습니다.

> [!NOTE]
> 이 콘텐츠는 Windows Server 참가자 미리 보기 프로그램의 Windows Server 컨테이너에만 해당 합니다. Windows 컨테이너 사용에 대 한 비 참가자 지침을 찾고 있다면 [시작](../quick-start/set-up-environment.md) 가이드를 참조 하세요.

## <a name="join-the-windows-insider-program"></a>Windows 참가자 프로그램에 참여하기

참가자 버전의 Windows 컨테이너를 실행 하려면 windows 참가자 프로그램 및/또는 windows 참가자 프로그램에서 windows 10의 최신 빌드를 실행 하는 windows Server의 최신 빌드를 사용 하는 호스트가 있어야 합니다. [Windows 참가자 프로그램](https://insider.windows.com/GettingStarted) 에 참여 하 고 사용 약관을 검토 합니다.

> [!IMPORTANT]
> Windows Insider preview 프로그램에서 windows server 참가자 미리 보기 프로그램 또는 windows 10의 빌드를 사용 하 여 아래 설명 된 기본 이미지를 사용 해야 합니다. 두 빌드 중 하나를 사용하지 않을 경우 이러한 기본 이미지를 사용하면 컨테이너가 시작되지 않는 오류가 발생합니다.

## <a name="install-docker"></a>Docker 설치

<!-- start tab view -->
# [<a name="windows-server-insider"></a>Windows Server 참가자](#tab/Windows-Server-Insider)

Docker EE를 설치하기 위해 OneGet provider PowerShell module(OneGet 공급자 PowerShell 모듈)을 사용합니다. 공급자는 컴퓨터에서 컨테이너 기능을 사용하도록 설정하고 Docker EE를 설치합니다. 그러면 컴퓨터를 다시 부팅해야 합니다. 관리자 권한으로 PowerShell 세션을 열고 다음 명령을 실행합니다.

> [!NOTE]
> Windows Server 참가자 빌드에 Docker EE를 설치 하려면 비 참가자 빌드에 사용 된 것과 다른 OneGet 공급자가 필요 합니다. Docker EE와 DockerMsftProvider OneGet 공급자가 이미 설치된 경우 계속하기 전에 이를 제거합니다.

```powershell
Stop-Service docker
Uninstall-Package docker
Uninstall-Module DockerMsftProvider
```

Windows 참가자 빌드와 함께 사용할 OneGet PowerShell 모듈을 설치합니다.

```powershell
Install-Module -Name DockerProvider -Repository PSGallery -Force
```

OneGet을 사용하여 최신 버전의 Docker EE 미리 보기를 설치합니다.

```powershell
Install-Package -Name docker -ProviderName DockerProvider -RequiredVersion Preview
```

설치가 완료되면 컴퓨터를 다시 부팅합니다.

```powershell
Restart-Computer -Force
```

# [<a name="windows-10-insider"></a>Windows 10 참가자](#tab/Windows-10-Insider)

Windows 10 참가자는 docker 데스크톱과 안정적으로 동일한 설치 관리자를 통해 Docker 가장자리만 설치 됩니다. [Docker 데스크톱](https://store.docker.com/editions/community/docker-ce-desktop-windows) 을 다운로드 하 고 설치 관리자를 실행 합니다. 로그인 하는 것이 필요 합니다. 아직 계정이 없는 경우 계정을 만듭니다. 자세한 설치 지침은 [Docker 설명서](https://docs.docker.com/docker-for-windows/install)에서 확인할 수 있습니다.

설치 후 Docker 설정을 열고 "Edge" 채널로 전환 합니다.

![](./media/docker-edge-instruction.png)

---
<!-- stop tab view -->

## <a name="pull-an-insider-container-image"></a>참가자 컨테이너 이미지 가져오기

Windows 컨테이너를 사용하려면 기본 이미지를 설치해야 합니다. Windows 참가자 프로그램의 일부인 경우 기본 이미지에 대 한 최신 빌드를 사용할 수 있습니다. [컨테이너 기본 이미지](../manage-containers/container-base-images.md) 문서에서 사용 가능한 기본 이미지에 대 한 자세한 내용을 확인할 수 있습니다.

Nano 서버 참가자 기본 이미지를 가져오려면 다음을 실행합니다.

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

Windows Server Core 참가자 기본 이미지를 가져오려면 다음을 실행합니다.

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

> [!IMPORTANT]
> Windows 컨테이너 OS 이미지 [EULA](../images-eula.md ) 및 windows 참가자 프로그램 [사용 약관](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)을 참조 하세요.
