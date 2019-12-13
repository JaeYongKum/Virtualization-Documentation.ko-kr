---
title: Windows 참가자 프로그램에서 컨테이너 사용
description: Windows 참가자 프로그램으로 windows 컨테이너를 사용 하 여 시작 하는 방법을 알아봅니다.
keywords: docker, 컨테이너, 참가자, windows
author: cwilhit
ms.openlocfilehash: 92fb359df1c207b848fb985caf7f46852f6b4f90
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909893"
---
# <a name="use-containers-with-the-windows-insider-program"></a>Windows Insider program에서 컨테이너 사용

이 연습에서는 Windows Insider Preview 프로그램의 최신 Windows Server 참가자 빌드에 Windows 컨테이너 기능을 배포하고 사용하는 방법을 안내합니다. 이 연습을 진행하는 과정에서 컨테이너 역할을 설치하고 기본 OS 이미지의 미리 보기 버전을 배포할 것입니다. 컨테이너에 대해 좀 더 숙지해야 하는 경우 [컨테이너 정보](../about/index.md)에서 이 정보를 찾을 수 있습니다.

## <a name="join-the-windows-insider-program"></a>Windows 참가자 프로그램에 참여하여

Windows 컨테이너의 참가자 버전을 실행 하려면 windows 참가자 프로그램에서 windows Server의 최신 빌드를 실행 하는 호스트와 Windows 참가자 프로그램에서 windows 10의 최신 빌드를 실행 해야 합니다. [Windows 참가자 프로그램](https://insider.windows.com/GettingStarted) 에 참여 하 고 사용 약관을 검토 합니다.

> [!IMPORTANT]
> 아래에 설명 된 기본 이미지를 사용 하려면 windows Server Insider preview 프로그램 또는 windows Insider Preview 프로그램의 Windows 10 빌드에서 Windows Server 빌드를 사용 해야 합니다. 두 빌드 중 하나를 사용하지 않을 경우 이러한 기본 이미지를 사용하면 컨테이너가 시작되지 않는 오류가 발생합니다.

## <a name="install-docker"></a>Docker 설치

Docker가 이미 설치 되어 있지 않은 경우 [시작](../quick-start/set-up-environment.md) 가이드에 따라 docker를 설치 합니다.

## <a name="pull-an-insider-container-image"></a>Insider container 이미지 끌어오기

Windows 참가자 프로그램의 일부로 기본 이미지에 대 한 최신 빌드를 사용할 수 있습니다.

Nano 서버 참가자 기본 이미지를 가져오려면 다음을 실행합니다.

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

Windows Server Core 참가자 기본 이미지를 가져오려면 다음을 실행합니다.

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

"Windows" 및 "IoTCore" 기본 이미지에는 끌어오기에 사용할 수 있는 참가자 버전도 있습니다. [컨테이너 기본 이미지](../manage-containers/container-base-images.md) 문서에서 사용 가능한 insider base 이미지에 대해 자세히 알아볼 수 있습니다.

> [!IMPORTANT]
> Windows 컨테이너 OS 이미지 [EULA](../images-eula.md ) 및 windows 참가자 프로그램 [사용 약관](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)을 읽어 보세요.
